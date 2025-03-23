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


### Physical Layer 
Convert data to electrical signal. We may have to do several things such as only one device transmit at a time, encode data such that average voltage is zero otherwise we might develope a electrical potential between two computers, simple error detection such as parity bits

`lp link set dev <device-name> up/down`: turn a network device up or down

### Data Link Layer
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

### Network Layer
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