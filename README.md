Practical 1 (Part 1) - OSPF MD5 authentication
Practical 1 (Part 2) - NTP
Practical 1 (Part 3) - SYSLOG

=======Steps to Do Practical 1============

Step 1 — Open Packet Tracer

2 Router 1941
2 Switch 2960
2 PC
2 Server

===

Step 2 — Connect Devices

NTP Server → Switch0
SYSLOG Server → Switch0
Switch0 → Router0
Router0 → Router1
Router1 → Switch1
Switch1 → PC0
Switch1 → PC1

===

Step 3 — Configure Server IP

Click Server → Desktop → IP Configuration
NTP Server(server 1)
IP Address: 192.168.1.3
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1

SYSLOG Server(server 2)
IP Address: 192.168.1.2
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1

===

Step 4 — Configure PC IP
PC0
IP: 192.168.3.3
Mask: 255.255.255.0
Gateway: 192.168.3.1

PC1
IP: 192.168.3.2
Mask: 255.255.255.0
Gateway: 192.168.3.1

===

Step 5 — Configure Router Interfaces

Click Router0 → CLI

enable
configure terminal
interface g0/0
ip address 192.168.1.1 255.255.255.0
no shutdown
interface g0/1
ip address 192.168.2.1 255.255.255.0
no shutdown

Router1

enable
configure terminal
interface g0/0
ip address 192.168.2.2 255.255.255.0
no shutdown
interface g0/1
ip address 192.168.3.1 255.255.255.0
no shutdown

Now all devices are physically connected.

===

Step 1 — Configure OSPF on Router0
Click Router0 → CLI
Then type commands one by one.

enable
configure terminal
router ospf 1
network 192.168.1.0 0.0.0.255 area 1
network 192.168.2.0 0.0.0.255 area 1
exit

===

Step 2 — Configure OSPF on Router1
Click Router1 → CLI

enable
configure terminal
router ospf 1
network 192.168.2.0 0.0.0.255 area 1
network 192.168.3.0 0.0.0.255 area 1
exit

===

Step 4 — Configure MD5 on Router0

Go to interface connecting routers.(in cli)
interface g0/1
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 admin

===

Step 5 — Configure MD5 on Router1

Go to interface connected to Router0.(in cli)
interface g0/0
ip ospf authentication message-digest
ip ospf message-digest-key 1 md5 admin
exit


________________________________________________________________________________________________

TACACS+ Authentication using AAA on Router in Cisco Packet Tracer.

=======Steps to Do Practical 2============

