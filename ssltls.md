# SSL/TLS

SSL / TLS refer to the same thing and are used interchanble. SSL was the original protocol invented by Netscape and TLS is managed by IETF

### SSL / TLS provide following features

- Cofidentiality: encryption - no one should be able to read your data
- Integrity: prevent malicious modification to the data
- Authentication: ability to identify the sender and recepient 

These 3 features in turn also provide us two added benefits

- Non-replay : it should not be possible to send a captured message again to trigger the transaction
- Non-repudiation : sender should not be able to deny sending a message


###  Parties

Client: Initiates a TLS handshake. Client is rarely authenticated (using SSL cert, however client will authenticate using say a username/pass)
Server : Receives the TLS handshake. Server is almost always authenticated (such as bank)
CA : Issues certificates, is trusted by both client and server, provides trust anchor between client and server

### Hashing: 
Hashing algos must satusfy 4 requirements
- Should be infeasible to produce a given hash
- Should be impossible to extract original msg from hash
- Slight change in text should produce drastically different hash
- Hash should be fixed length

Hash collision occurs when two msgs produce same hash. Collisions are unavoidable (*collision is due to fixed width)

SHA2 alorithm family includes: SHA224, SHA256, 384, 512

### HMAC
- Only HASH does not provide msg interity as a hacker can modify the msg and trasmit it with a new hash
- HMAC is used to prevent this by using Hash based Msg Authntication Code. In this scheme, both parties agree upon a secret key. The sender then sends a msg with the key. The received computes the hash using the received msg and the key and verifies it against the hash of received msg. The sender and receiver must agree on the order in which the msg and key are used. This is defined by HMAC

### Encryption
- Simple Encryption : Keyless, such as rot13, not secure
- Key based Encryption : keys can be generated randomly making it more secure. There are two trypes of this
  - Symmetric Encryption : use same key to encrypt and decrypt. Used for large amount of/bulk data. Example: DES, AES
  - Asymmetrick Encryption : use DIFFERENT key to encrypt and decrypt. This is more secure because the key is never shared, it is computed independently by each party. The key sizes are much bigger and this scheme is also computationally more intensive. Examples: DSA, RSA, DH, ECDSA, ECDH

#### Hybrid Encryption:
Public/Private keypair is used to exchange a symmetric key. Then this symmetric key 

### Certificate
Certificate is signed by a CA and it contains a public key for a recognized entity. This is a way to say that: we certify that whoever owns a private key corresponding to this publix key is example.com

### PKI : Public Key Infrastructure
The trifacta of client, server and CA forms the PKI
Client needs to verify the identity, server needs to prove the identity and CA validates the identity and generates certificate

PKI can exist in scenarios other than web traffic. For example
- Installing a software downloaded from internet:
    OS is seeking (client) to verify the identity of the installer, package needs to prove its identity (server) and code signing CAs provide signature
- Corporate PKI can be setup using
    A coporate CA to issue certs, corprate apps such as HR portal and employee phone/laptop acting as client

### RSA
RSA is an asymmetric algo involving key exchange.

RSA algorithm works on the concept of semi-prime numbers (commutative keys) and its factors (semi-prime is a number which has two prime numbers as factors. eg: 33)

In 1991 RSA lab published a 1024 bit semi prime number and challenged the hackers to factor that number. It has not been factored until now.

Now a days it is a standard practice to use 2048 bit semi prime for keys

RSA can be used for key exchange, encryption and signatures

### Diffe Hellman
DH is an asymmetric algo that allows two parties to establish a shared secret over an UNSECURED medium (however shared secret is never transmitted)
The only way to break DH algorithm is by brut-force and hence very large numbers are used

DH is only used for key exchange, it cannot be used for encryption and signatures

### DSA : Digital Signature Algorithm
DSA is an asymmetric algo. As the name signifies, this algorithm can only be used for signing and signature verification

In DSA every msg must be signed with a different random number

DSA can only be used for signatures, it cannot be used for encryption and key exchange

### File formats

Keys, certs, requests, params all can be saved in files
OpenSSL uses a few file formats to store this information

- PEM: This is the most common file type. PEM files can be coverted directly (using `openssl` command) to DER or PFX.
- DER: Format is used to exchange security artifacts. Typically only certs and csrs are exchanged on the wire (keys are not). Hence its uncommon to see a key in DER format. DER files can be converted to PEM but not to PFX directly
- PFX / PKCS12: These are container files. A pfx file can contain multiple files such as: certificate chain or matching pari of key/cert. A PFX file cannot contain only private key (if must be paired with cert). PFX files can only be corverted to PEM format directly (and not DER). 

`openssl` command supports an option `-inform` to specify the format type of an file. Example:

    `openssl x509 -in my.cert -inform DER`

### Common OpenSSL commands

- Generate an unencrypted key:

    `openssl genrsa -out key.pem 2048`

- Generate an encrypted key (encrypted using aes128 algo):

    `openssl genrsa -out key.pem -aes128 4096`

- Inspect RSA key:

    `openssl rsa -in key.pem --noout --text`

- Add encryption to an unencrypted RSA key:

    `openssl rsa -in key.pem -aes128 -out ENCRYPTED-KEY.pem`

- Remove encryption from an encrypted RSA key:

    `openssl rsa -in ENCRYPTED-KEY.pem -out KEY.pem`

- Algorithm agnostic command for managing keys:
    `openssl pkey` command can be used to perform common key operations which will work irrespective of the type of the key rsa/dsa/ec etc
    The benefit of this is that you can use same command without having to figure out key type

- Find the default configuration file for openssl:
  Default config file is named `openssl.cnf` and is located in the directory specified by below command
  We can customize this file and pass the filename as parameter

    `openssl version -d`

- Extract specific fields from an x509 cert:
    For example below command will extract only issuer and CN info from the cert
    `openssl x509 -in my.cert -noout -issuer -subject`

Resources:
- Learn OpenSSL with a real world cheatsheet (Udemy)