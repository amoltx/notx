# GRUB

- Bios / UEFI starts the computer and hands of to boot loader
- GRUB2 is the bootloader for modern linux
- Grub menu may/maynot be displayed based on your OS configuration, this behavior can be changed from grub startup config
- Grub has a startup config at /etc/default/grub
- After the grub is loaded it will read another config file under /boot/grub/grub.cfg. We should never update this cfg file manually (kernel update will change/reppace it)
- After edifing the default grub config, run command `sudo update-grub` to update grub config


# Kernel

Duties of Kernel
- Process sceduling and resource allocation / IPS
- Memory management
- File System
- Networking

Kernel file: `/boot/vmliuz`

Kernel module: can load functionality on the fly, can extend kernel functionality by loading propritary drivers

sudo lsmod : list loaded kernel modules

Admin can lock kernel updates if needed to prevent updates using command `apt-mark hold <pkg>`

# systemd

sysmtemd process is pid 1 (/sbin/init is softlink to systemd)
Kernel starts systemd and then systemd starts all the other processes

systemd contains a list of tools such as: systemd-journald, systemd-timesyncd

## Criticism
- Very big program with wide functionality: violates unix philosophy : do one thing and do it well
- Overly complex
- Runs only on Linux and not on all unix flavours
- Doing many things in parallel can result in slower startup time because the HHD/DVD is spinning

## Advantages
- Parllellizing boot process provides significant perf boost on modern systems (even with slower SSD)

## Units
There are various types of units defined for example: Service, Target, Timer...

### Service Unit
- A service is defined by a unit config file

/lib/systemd/system : contains systemd units for system configuration, we should not modify these
/run/systemd/system : non-persistent configuration, wiped after reboot
/etc/systemd/system : admin/user specific units, these will take precedence over standard paths
To list all systemd paths, use command: systemd-analyze --system unit-paths

#### Viewing the unit files : systemctl cat

We can view a unit file using `cat` command. However it will not display the contents of the dependent units
To see the system unit with all dependecies we should use command `systemctl cat <unit-file>
Another advantage is that `systemctl cat` can be run from any directory

#### Unit commands

`systemctl list-units`: list all units
`systemctl status/start/stop/reload <unit-file>` : start/stop or view status of a service

## CGroups

cgroups are used to organize a process hierarchy and limit the resource consumption
The cgroup defined for a process is also inherited by the child processes started by specified service

`systemctl status`: lists all the existing CGroups
`systemd-cgtop`: view resource consumption by cgroups

Here's an example way to limit FireFox memory consumption to 100MB using CGroups

1. Create a file `~/.config/systemd//user/browser.slice`, inseide it add a section `Slince` and set `MemoryHigh=100M` under se
2. Now launch Firefox using command `susyemd-run --user --slice=browser.slice /usr/bin/firefox`

## journald / journalctl

systemd-journald is part of systemd suite and is responsible for managing system logs
It replaces traditional syslog
journald logs are binary

system/journal log is meant to be used by the applications for major events such as crash. It is not meant to be used for detailed application logging

journalctl is the frontend

`journalctl --list-boots` : list past boots and shows the id for each boot instance which can be used to display logs for that boot session
`journalctl -b`: display logs for boot session
`journalctl -b <id>`: display logs for a specific boot session
`journalctl -u <unit-file>`: display logs for a specific unit
`journalctl --since <date> --unti <date>`: Display logs for a date range
`journalctl -r`: show most recent entry at the top (reverse)
`journalctl -f`: follow logs
`echo hello world | systemd-cat`: Logs "hello world" to systemd logs
`journalctl -t <identifier>`: filters logs by an identifier, one example identifier is systemd or CRON

# Networking

## ip

- `ip` command is commonly used to inspect and modify ip addresses, routes and interfaces
- `ip` replaces `ifconfig`, `route` and `ifconfig` commands

`ip addr show`
`iproute1mac`: provides `ip` command emulation for Mac OS, install this using `brew install iproute2mac`

## OSI

Application Layer  (7): Enables application specific communication (HTTP/FTPSMTP)
Presentation Layer (6): Translate/encrypt/compress data (eg: TLS/SSL)
Session Layer      (5): establish, maintain and terminate sessions between applications
Transport Layer    (4): Ensure reliable data transfer between hosts (eg: tcp/udp) for example retransmit a lost packet or maybe not

Network Layer      (3): Routing and msg forwarding between networks (eg: ip addr, routers etc)
Data Link Layer    (2): Provides error-free transfer between "ADJACENT" network nodes (not routed to a destination) eg: ethernet/mac addr
Physical Layer     (1): Physical cables, switches etc, data is transferred as raw bits


## Physical Layer 
Convert data to electrical signal. We may have to do several things such as only one device transmit at a time, encode data such that average voltage is zero otherwise we might develope a electrical potential between two computers, simple error detection such as parity bits

`lp link set dev <device-name> up/down`: turn a network device up or down

## Data Link Layer
Unit of data transfer: frame
Goal: Reliable data transfer between nodes on same network

This layer provides support to identify a recepient for a data frame: frame addressing
Limit the number of frames so as to not choke the recepient
Size: depends on protocol, ethernet has approx frame size of 1500 bytes

DLL is divided into two sublayers
- Logocal Link Control: this handles flow control and error detection
- Media Access Control: MAC - uniquely idetifies each hardware endpoint

Data Link Layer works by organizing data into frames and adding source/dst mac addr to frames
Common protocols at layer 2
- Ethernet (IEEE 802.3): Divides data into frames of 1.5kb and includes source/dst mac addr in frames, error detection codes are also added
- WiFi (IEEE 802.11): WiFi frames are very similar to ethernet frames with some additions. Hence wifi and ethernet protocols are compatible. A wireless access point can convert ethernet frames to/from wifi frames

MAC address: 6 byte (48-bit) - first 3 bytes identify manufacurer, next 3 bytes are device identifier

#### Wireshark to look at layer 2 frames
We can look at any msg in wireshark and expand it to reveal the ethernate frame.
Ethernet frame will reveal to contain: a src and dest mac address on the same network

#### Link Layer hardware
- Bridge
- Switch (some advanced switches can work at layer 3 and provide routing capabilities)
- Wireless Access Point

#### Switch
Nodes are not aware that they are connected to a switch. In that way, switches are completely transperant to node

Switch can store the MAC addresses of machines connected to it and then forward the frames only to the intended mac addrs. This way switch can reduce some traffic on the network and also allow multiple macs to send messages at the same time.

Note again that: nodes do not have a knowledge of switch and they communicate as if they are directly connected to the adjacent node. The only way for a node to find if it is connected to a switch is possibly by looking at all the packets it is receiving and if it is only receiving packets intended to it and not any other node, most likely this node is connected to a switch.

## Network Layer
Unit of data transfer: packet
Goal: Route packets between networks

Link Layer frames can only talk with nodes on a local network. In order to be able to talk to a node across a different network, we should be able to send a frame across networks. This functionality is provided by the network layer.

Packets are wrapped on a frame and sent to next destination on the network (say router) where the packet is taken out of the frame and wrapped in a new frame and sent on. For the frame: packet is data

packet contains src/dest IPs and data

`ip route show`: see route info

Also the packet can be inspected in wireshark

+++++++++++++++++++++++++++++++++++++++++++++++++++++++
+ Ethernet Frame                                      +
+ Source MAC: 00:1A:2B:3C:4D:5E                       +
+ Target MAC: 00:1A:2B:3C:4D:5F                       +
+ Data:                                               +
+                                                     +
+    ##############################################   +
+    #   IP Packet                                #   +
+    #   Source IP: 192.168.1.10                  #   +
+    #   Destination IP: 192.168.1.20             #   +
+    #   Data: Hello, world!                      #   +
+    ##############################################   +
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

`
###############################################################
# Ethernet Frame                                              #
# Source MAC: 00:1A:2B:3C:4D:5E                               #
# Target MAC: 00:1A:2B:3C:4D:5F                               #
# Data:  //////////////////////////////////////////////       #
#        /   IP Packet                                /       #
#        /   Source IP: 192.168.1.10                  /       #
#        /   Destination IP: 192.168.1.20             /       #
#        /   Data: Hello, world!                      /       #
#        //////////////////////////////////////////////       #
###############################################################
`
### Subnetting

Subnets are used to divide a very large network into multiple networks
Subnet mask is used to determine if source and destination ips are on same network
If src/dst are on same network, the frames can be sent directly
If src/dst are NOT on same network, the request needs to be sent to gateway/router

*Calculating if an two IPs are on same network*

- Perform logical AND operation on SRC address and SUBNET MASK: this gives a network address
- Perform logical AND operation on DST address and SUBNET MASK: this gives a network address
- If both these network addresses are same, then both IPs are on the same network, else they are on diffrent networks

#### Network Address and IP Range

- Network address can be obtained by doing IP AND SUBNET MASK
- The first IP in the network is typically reserved for the gateway
- The last IP in the network is typically reserved for broadcast IP
- Rest of the IPs are available for nodes

eg: IP 192.168.1.5 and subnet 24 (255.255.255.0) gives us
Network address: 192.168.1.0
Gateware IP: 192.168.1.0
Broadcast IP: 192.168.1.255
IPs available for nodes: 192.168.1.1    to   192.168.1.254

- With a 24 bit subnet mask, we have last byte availabe for IPs thus giving us approx 250 available IPs
- With 23 bit subnet mask: approx 500 ips are available
- With 22 bit subnet mask: approx 1000 ips are availabe

### ARP : Address Resolution Protocol

ARP is used to map IP addr to MAC addr.
Devices on a network use IP addrs to identify each other, but data transmission on physical layer uses MAC addr.
ARP is used to do the mapping between IP/MAC addr

When a device wants to send data to another device on same node, it checks its ARP cache, if mac is not in cache, it broadcasts an ARP request asking "Who has the IP xyz", the node that has the requested IP responds back with its MAC address, which then is stored in the ARP cache

ARP packets can be seen in Wireshark.

### Setting IP address

`ip addr add <ip>/<subnet bits> dev <device name>` : set an ip
`ip addr del <ip>/<subnet bits> dev <device name>` : delete an ip

### Network Device

One network device can have multiple IPs

## Routing
Routing lecture (304) is very good in the Mastering Linux Udemy course
`ip route show` : shows the routing table
`ip route get <ip addr>` : shows which route the ip can be reached from
  if the output of `ip route get` shows connectivity via a different ip, this means the target ip is in a different network and  the packet needs to be fourwarded to that gateway. If the command output shows connection via the network device, then the ip is directly connected as it is on the same network
`ip route add <ip addr>/<subnet bits> via <target ip> dev <device name>` : add a route for a network via target ip
`ip route del <ip addr>/<subnet bits> via <target ip> dev <device name>` : delete a route

## DHCP
DHCP works using UDP protocol
To view DHCP packet exchange for a host, you can perform a `ip link set dev down` followed bu `ip link set dev up` and view the DHCP packets in Wireshark

### DHCP Server
- Stores ip addr pool
- Manages leases
- Assigns and reclaim addrs
- In home networks, router usually acts as dhcp server

### DHCP Relay Client
- If the DHCP network involves multiple networks, this relays the requests to other subnets

### DHCP Client

### Troubleshooting

If system (eg Ubuntu) is using systemd-networkd then DHCP logs can be seen using
`hournalctl -u systemd-networkd`

If system (eg CentOS) is using NetworkManager then DHCP logs can be seen using
`hournalctl -u NetworkManager`


### ICMP

`ping` : uses icmp, icmop can be disabled
`traceroute` : use to visualize the path of a ping request, the command show the time taken for each hop and hence can be used to identify bottlenecks, also the command

In Wireshark, we can apply filder for a destination ip using filter `ip.dst == <ip addr>`

Working of traceroute is interesting, it sends a packet to the destination with the count of 1, when router reaches the packet, it reduces the count by 1. At any point, if the count reaches 0, it is sent back to the sender. The sender will understand this as partial route and send a new packet with count incremented count (in this case 2), the packet will again travel the path and will be returned if count of 0 is reached before the final destination, in which case the client will again send the packet with higher initial count

## Transport Layer
Unit of data transfer: segment
Goal: End to end connection and reliability

Packet can be lost (unable to reach destination) or can be dropped (router can drop a received packet if it is overloaded)

So we need to have a mechanism to determine the reliability of the data. This is handled by transport layer protocols. There are 2 main protocols

#### UDP : User Datagram Protocol
The application is expected to handle out-of-order packets.
Eg: video call - probably it is ok if a few packets are dropped
Other examples: dns/dhcp/snmp/tftp/ntp/rtp (real time protocol for audio/video)

### TCP
Packets are ordered by the receiver and are re-transmitted if needed
The TCP stack implementation would take care of details such as: error correctio, retransmission. The user only needs to send data without having to worry about these things. TCP implements the feature to support flow control (how much data the receiver can handle) and congestion control (wha a connection can handle, say a router in between is overloaded)

TCP ports bring in the notion of application by supporting source and destination ports. The source ports are generally assigned randomly and destination ports are usually well known

TCP packets are wrapped inside IP packets

Wireshark: In Wireshark we can see that each TCP packet has a sequence number. TCP packets also show source and destination ports for a packet.

#### TCP Handshake

- SYN: Sender sends a SYN packet: indicating that it wants to initiates a connection
- SYN-ACK: Reiver sends SYN-ACK acknowledgong the connection
- ACK: The sender then responds with an ACK packet to indicate that it received the SYN-ACK

With each packet, a sequence number is sent and the ack packet sends back ack with the original sequence number to indicate which packet it is acknowleding. For each iteration, the sender increases the sequence number usually by the number of bytes in the packet

### NAT : Network Address Translation

When an internal IP wants to talk to another IP outside of the router, it will send a packet with its own (internal IP) and port to the router. The router then changes source IP and port from this packet to its own IP and the port it is using for this connection, because the outer entity would be able to only reach the router and not the sending device.

When a reply is received, router would agin look up the target port and identify the initial sender and send the packet to the sender.

This is mainly done to address the limited address space of IPv4 and hence is not needed in IPv6

NAT is useful only for outgoing connection. However if we want the outside entity to connect with internal IP, this needs to be handled by *port forwarding* at the router (this is not NAT)

## Session Layer

*NOTE: With some (especially modern) protocols the distinction between the higher level layers may not be clear*

Unit of data transfer: data
Goal: Aprovide additional application connectivity such as authentication and state management

Eg:
- NFS (Network File System)
- RPC (Remote Procedure Call)
- SCP (Session Control Protocol)

## Presentation Layer

Goal: Data representation: encryption, encoding, formatting etc


Eg
- Encryption/decryption : SSL/TLS
- Compression/decompression
- MIME

## Application Layer

Eg
- HTTP/HTTPs
- IMAP
- SSH (on top of SSL at presentation layer)
- DNS

### DNS

If the dns cache on the host does not have hostname/ip mapping for a hostname, the request is sent to a DNS server. Below is a brief description of how DNS request works

- DNS request is sent (at random) to one of the 13 ROOT NAME SERVER asing "I need IP for google.com"
- Root name server will look at the .COM and send the request to one of the TLD (top level domain servers) for .COM
- TLD will respond with name of the server who knows about resolving .COM domain
- TLD name server will now responds with the authoritative name server for google.com (eg: ns1.google.com)
- The autoritative name server will respond with the correct IP for google.com

#### Common DNS record types
- A : maps a domain name to an ip addr
- AAAA : maps domain name to an IPv6 addr
- CNAME : provides an alias for another domain name
- MX: Mail server for a domain
- NS : authoritative name server for a domain

`host -a google.com` : lists the received DNS entries for a domain name

## Firewall

*netfilter*: is a kernel subsystem for managing network filters
*iptables*: is user-space program that allows us to manage netfilters using command line, this is the process of being deprecated
*nftables*: is the newer alternative to iptables
firewalld: firewall daemon that uses nftables (or iptables) as backend
firewall-cmd: commandline
firewall-config: gui

Firewall allows us to define custom rules for network traffic. These rules mostly concern with incoming traffic
Eg: If we want to use a linux system as web server, we can block all the incoming traffic except HTTP and SSH (for management)

`ss -4nap` : list all incoming/outgoing open ports, incoming port is shown as LISTEN and outgoing as ESTAB
`firewall-cmd --state` : show current firewalld state
`firewall-cmd --list-all` : list acive rules in firewalld