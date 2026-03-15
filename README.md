# 🔐 Security in Computing — Practical Cheat Sheet

> **Course:** TYIT Security in Computing | **Tool:** Cisco Packet Tracer

---

## 📋 Table of Contents

- [Practical 1 — OSPF MD5 Authentication, NTP & SYSLOG](#practical-1-ospf-md5-authentication-ntp--syslog)
  - [Part 1 — OSPF MD5 Authentication](#part-1-ospf-md5-authentication)
  - [Part 2 — NTP](#part-2-ntp-network-time-protocol)
  - [Part 3 — SYSLOG](#part-3-syslog)
- [Practical 2 — Configure AAA Authentication](#practical-2-configure-aaa-authentication-on-cisco-routers)
- [Practical 3 — Configure Extended ACLs](#practical-3-configure-extended-acls)
- [Practical 4 — Configure IP ACLs to Mitigate Attacks](#practical-4-configure-ip-acls-to-mitigate-attacks)
- [Practical 5 — Configure IPv6 ACLs](#practical-5-configure-ipv6-acls)
- [Practical 8 — Layer 2 Security](#practical-8-layer-2-security)
- [Quick Command Reference](#quick-command-reference)

---

# Practical 1: OSPF MD5 Authentication, NTP & SYSLOG

> **Tool used:** Cisco Packet Tracer  
> **Devices:** 2× Router 1941, 2× Switch 2960-24TT, 2× PC, 1× NTP Server, 1× SYSLOG Server

---

## Network Topology

```
[NTP Server]       [SYSLOG Server]
192.168.1.3        192.168.1.2
      \                /
     [Switch0 — R0(G0/0) — 192.168.1.1]
                  |
              R0(G0/1) — 192.168.2.1
                  :  (serial link)
              R1(G0/0) — 192.168.2.2
                  |
              R1(G0/1) — 192.168.3.1
                  |
            [Switch1]
            /         \
         [PC0]        [PC1]
      192.168.3.3   192.168.3.2
```

### IP Address Table

| Name        | Device       | Port | Default Gateway | IP Address   | Subnet Mask   |
|-------------|--------------|------|-----------------|--------------|---------------|
| NTP         | Server       | -    | 192.168.1.1     | 192.168.1.3  | 255.255.255.0 |
| SYSLOG      | Server       | -    | 192.168.1.1     | 192.168.1.2  | 255.255.255.0 |
| PC0         | PC           | -    | 192.168.3.1     | 192.168.3.3  | 255.255.255.0 |
| PC1         | PC           | -    | 192.168.3.1     | 192.168.3.2  | 255.255.255.0 |
| R0          | Router 1941  | G0/0 | -               | 192.168.1.1  | 255.255.255.0 |
| R0          | Router 1941  | G0/1 | -               | 192.168.2.1  | 255.255.255.0 |
| R1          | Router 1941  | G0/0 | -               | 192.168.2.2  | 255.255.255.0 |
| R1          | Router 1941  | G0/1 | -               | 192.168.3.1  | 255.255.255.0 |

---

## Part 1: OSPF MD5 Authentication

### Objective
Configure OSPF routing on both routers and secure it using MD5 authentication.

### Prerequisites / Setup
- Cisco Packet Tracer with the topology above built and IPs assigned
- Ensure basic connectivity (ping) between routers before starting

### Background
- **OSPF** is a link-state routing protocol where routers exchange information about known routes and their costs
- **MD5 Authentication** computes a hash from the OSPF packet contents + a shared password (key). The hash, key ID, and a non-decreasing sequence number are transmitted in the packet
- The receiving router recalculates the hash using the same password — if hashes match, authentication passes
- The sequence number prevents **replay attacks**
- MD5 passwords must be **identical between neighbors** (but can differ across an area)

### Steps & Commands

**Step 1: Verify OSPF interface status (before config)**

Check whether OSPF is already running on both routers.

```bash
# Run on Router0
Router>exit
Router>enable
Router#show ip ospf interface
```

```bash
# Run on Router1
Router>exit
Router>enable
Router#show ip ospf interface
```

---

**Step 2: Configure OSPF on Router0**

Enable OSPF process 1 and advertise the two directly connected networks.

```bash
Router>en
Router#conf t
Router(config)#router ospf 1
Router(config-router)#network 192.168.1.0 0.255.255.255 area 1
Router(config-router)#network 192.168.2.0 0.255.255.255 area 1
Router(config-router)#exit
Router(config)#exit
Router#
```

---

**Step 3: Configure OSPF on Router1**

Same process — advertise Router1's two connected networks.

```bash
Router>en
Router#conf t
Router(config)#router ospf 1
Router(config-router)#network 192.168.3.0 0.255.255.255 area 1
Router(config-router)#network 192.168.2.0 0.255.255.255 area 1
Router(config-router)#exit
Router(config)#exit
Router#
```

---

**Step 4: Verify connectivity**

After OSPF is up, verify that packets flow between the two sides of the network (e.g., ping from PC0 to NTP/SYSLOG servers).

---

**Step 5: Configure MD5 Authentication on Router0**

Apply MD5 authentication on both interfaces of Router0 using key ID `1` and password `dalmia`.

```bash
Router>enable
Router# conf t
Router(config)#int g0/0
Router(config-if)#ip ospf authentication message-digest
Router(config-if)#ip ospf message-digest-key 1 md5 dalmia
Router(config-if)#exit
Router(config)#int g0/1
Router(config-if)#ip ospf authentication message-digest
Router(config-if)#ip ospf message-digest-key 1 md5 dalmia
Router(config)#exit
```

---

**Step 6: Configure MD5 Authentication on Router1**

Apply the **same** key ID and password on Router1 so neighbors can authenticate each other.

```bash
Router>enable
Router# conf t
Router(config)#int g0/0
Router(config-if)#ip ospf authentication message-digest
Router(config-if)#ip ospf message-digest-key 1 md5 dalmia
Router(config-if)#exit
Router(config)#int g0/1
Router(config-if)#ip ospf authentication message-digest
Router(config-if)#ip ospf message-digest-key 1 md5 dalmia
Router(config)#exit
```

---

**Step 7: Verify MD5 Authentication**

Re-run the OSPF interface check on both routers to confirm authentication is active.

```bash
Router>exit
Router>enable
Router#show ip ospf interface
```

Expected Output:
```
GigabitEthernet0/1 is up, line protocol is up
Internet address is 192.168.2.1/24, Area 1
Process ID 1, Router ID 192.168.2.1, Network Type BROADCAST, Cost: 1
```

### Notes / Important Points
- MD5 password (`dalmia`) **must be identical on both routers** for OSPF neighbors to form
- Key ID (`1`) must also match between neighbors
- After configuring MD5, OSPF may briefly drop adjacency then re-establish — this is expected
- `message-digest` = MD5 mode; `authentication` alone (without `message-digest`) = plain-text mode (less secure)
- The sequence number in MD5 packets prevents replay attacks

---

## Part 2: NTP (Network Time Protocol)

### Objective
Synchronize the clocks of both routers to the NTP server at `192.168.1.3`.

### Prerequisites / Setup
- Use the **same topology** as Part 1
- Ensure packet transfer (ping) is working before starting
- **Enable the NTP service** on the NTP Server (`192.168.1.3`) via its GUI: Services tab → NTP → On
- **Disable the NTP service** on all other servers (e.g., SYSLOG server) — otherwise output won't be obtained

### Background
- **NTP** (Network Time Protocol) is a TCP/IP protocol used to synchronize computer clocks across data networks
- Developed in the 1980s by D.L. Mills at the University of Delaware
- Uses a jitter buffer to handle variable latency across packet-switched networks

### Steps & Commands

**Step 1: Configure NTP Server in Packet Tracer GUI**

On the NTP Server device → Services tab → NTP → toggle **On**.

> ⚠️ Also go to the SYSLOG server → Services → NTP → toggle **Off**

---

**Step 2: Configure NTP on Both Routers (CLI)**

Point both Router0 and Router1 to the NTP server and enable time sync.

```bash
Router#config
Router#configure t
Router#configure terminal
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)#ntp server 192.168.1.3
Router(config)#ntp up
Router(config)#ntp update-calendar
Router(config)#exit
Router#
```

> Run the same commands on **both** Router0 and Router1.

---

**Step 3: Verify NTP Synchronization**

Check that the router clock has been updated by the NTP server.

```bash
Router#show clock
```

Expected Output:
```
*21:7:3.987 UTC Fri Nov 25 2022
```

### Notes / Important Points
- The `*` before the time means the clock is **not yet fully synced** (still authoritative from NTP but not stratum-confirmed) — this is normal in Packet Tracer
- `ntp update-calendar` writes the NTP time to the hardware calendar
- The NTP service **must be ON** on the NTP server and **OFF** on the SYSLOG server — having multiple NTP services active causes conflicts
- Both routers must point to `192.168.1.3` (the dedicated NTP server)

---

## Part 3: SYSLOG

### Objective
Configure both routers to send system log messages to the SYSLOG server at `192.168.1.2`.

### Prerequisites / Setup
- Use the **same topology** as Part 1
- Ensure packet transfer (ping) is working before starting
- **Enable the SYSLOG service** on the SYSLOG Server (`192.168.1.2`) via GUI: Services tab → SYSLOG → On
- **Disable the SYSLOG service** on all other servers (e.g., NTP server) to avoid conflicts

### Steps & Commands

**Step 1: Enable SYSLOG service on SYSLOG Server (GUI)**

On the SYSLOG Server → Services tab → SYSLOG → toggle **On**.

Also go to the NTP Server → Services tab → SYSLOG → toggle **Off**.

---

**Step 2: Configure Logging on Both Routers (CLI)**

Tell both routers to send logs to the SYSLOG server's IP address.

```bash
Router#
Router#configure terminal
Router(config)#logging 192.168.1.2
Router(config)#exit
Router#
```

> Run the same commands on **both** Router0 and Router1.

---

**Step 3: Verify SYSLOG Output**

On the SYSLOG Server GUI → Services → SYSLOG, you should see log entries appearing in the log table.

Expected Output (visible in SYSLOG Server GUI):

```
Time                HostName       Message
Jan 01 00:00:00.000 192.168.1.1    %SYS-5-CONFIG_I:...
Jan 01 00:00:00.000 192.168.1.1    %SYS-6-LOGGINGHOSTNAME:...
Jan 01 00:00:00.000 192.168.2.2    %SYS-5-CONFIG_I:...
Jan 01 00:00:00.000 192.168.2.2    %SYS-6-LOGGINGHOSTNAME:...
```

### Notes / Important Points
- `logging <ip>` sends all router syslog messages to the specified remote server
- SYSLOG service must be **ON** on `192.168.1.2` and **OFF** on other servers
- Log messages from **both routers** (192.168.1.1 and 192.168.2.2) should appear in the SYSLOG server
- `%SYS-5-CONFIG_I` = severity level 5 (notification), configuration change event
- `%SYS-6-LOGGINGHOSTNAME` = severity level 6 (informational), logging host registered
- Lower severity number = higher priority (0=Emergency, 7=Debug)

---

# Practical 2: Configure AAA Authentication on Cisco Routers

## Network Topology

```
[PC1]              [TACACS Server]
192.168.1.3        192.168.2.3
     \                  /
   [Switch0] — Router0 — [Switch1]
   192.168.1.x  G0/0=192.168.1.1
                G0/1=192.168.2.1
     /                  \
[PC0]              [RADIUS Server]
192.168.1.2        192.168.2.2
```

### IP Address Table

| Device        | IP Address   | Subnet Mask   | Default Gateway |
|---------------|--------------|---------------|-----------------|
| PC0           | 192.168.1.2  | 255.255.255.0 | 192.168.1.1     |
| PC1           | 192.168.1.3  | 255.255.255.0 | 192.168.1.1     |
| Router0 G0/0  | 192.168.1.1  | 255.255.255.0 | -               |
| Router0 G0/1  | 192.168.2.1  | 255.255.255.0 | -               |
| TACACS Server | 192.168.2.3  | 255.255.255.0 | 192.168.2.1     |
| RADIUS Server | 192.168.2.2  | 255.255.255.0 | 192.168.2.1     |

### Prerequisites / Setup

- Assign IPs to all devices via GUI (Config tab)
- **TACACS Server** → Services → AAA → Service: **On**, ServerType: **Tacacs**
  - Client Name: `dalmia` | Client IP: `192.168.2.1` | Secret: `cisco` → Add
  - User Setup: Username: `dalmia` | Password: `dalmia` → Add
- **RADIUS Server** → Services → AAA → Service: **On**, ServerType: **Radius**
  - Client Name: `dalmia` | Client IP: `192.168.2.1` | Secret: `cisco` → Add
  - User Setup: Username: `dalmia` | Password: `dalmia` → Add

### Steps & Commands

**Step 1: Configure AAA on Router0**

Enable AAA new-model, point to both servers, and apply authentication to VTY lines.

```bash
Router>en
Router#conf t
Router(config)#aaa new-model
Router(config)#tacacs-server host 192.168.2.3 key cisco
Router(config)#radius-server host 192.168.2.2 key cisco
Router(config)#aaa authentication login dalmia group tacacs+ group radius local
Router(config)#line vty 0 4
Router(config-line)#login authentication dalmia
Router(config-line)#exit
Router(config)#
```

**Step 2: Verify TACACS Authentication (from any PC)**

Telnet to the router IP — enter TACACS credentials when prompted.

```bash
PC>telnet 192.168.2.1
```

Expected Output:
```
Trying 192.168.2.1 ...Open
User Access Verification
Username: dalmia
Password:
Router>
```

**Step 3: Verify RADIUS Authentication**

Turn OFF the TACACS service on the TACACS Server (Services → AAA → Off), then telnet again.

```bash
PC>telnet 192.168.2.1
```

> On first attempt you may see `% Login invalid` — this is TACACS failing and falling through to RADIUS. Enter credentials again and it succeeds.

**Step 4: Verify Local Authentication**

Turn OFF both TACACS and RADIUS services. Telnet again — authentication falls back to local.

### Notes / Important Points
- AAA order: TACACS+ → RADIUS → local (as configured in the `aaa authentication` command)
- `key cisco` must match the Secret configured in the server's AAA settings
- Client IP in the server GUI must be the **Router's interface IP** (192.168.2.1), not the server's own IP

---

# Practical 3: Configure Extended ACLs

## Network Topology

```
[Server1]  [Server0]           [PC0]    [PC1]
192.168.1.2 192.168.1.3     192.168.3.2  192.168.3.3
      \         /                  \        /
     [Switch0]                    [Switch1]
         |                             |
    R0(G0/0)=192.168.1.1         R1(G0/1)=192.168.3.1
    R0(G0/1)=192.168.2.1 —— R1(G0/0)=192.168.2.2
```

### IP Address Table

| Device       | Port | IP Address   | Subnet Mask   | Gateway      |
|--------------|------|--------------|---------------|--------------|
| PC0          | -    | 192.168.3.2  | 255.255.255.0 | 192.168.3.1  |
| PC1          | -    | 192.168.3.3  | 255.255.255.0 | 192.168.3.1  |
| Server0      | -    | 192.168.1.3  | 255.255.255.0 | 192.168.1.1  |
| Server1      | -    | 192.168.1.2  | 255.255.255.0 | 192.168.1.1  |
| Router0 G0/0 | -    | 192.168.1.1  | 255.255.255.0 | -            |
| Router0 G0/1 | -    | 192.168.2.1  | 255.255.255.0 | -            |
| Router1 G0/0 | -    | 192.168.2.2  | 255.255.255.0 | -            |
| Router1 G0/1 | -    | 192.168.3.1  | 255.255.255.0 | -            |

### Prerequisites / Setup

- Assign all IPs via GUI
- Set RIP routing on both routers:
  - Router0 → Config → RIP → add `192.168.1.0` and `192.168.2.0`
  - Router1 → Config → RIP → add `192.168.2.0` and `192.168.3.0`
- Verify connectivity with ping before applying ACLs

---

## Part 1: Configure, Apply and Verify an Extended Numbered ACL

**Goal:** Allow FTP from PC0 (192.168.3.2) to Server1 (192.168.1.2) only. PC1 should be blocked.

**Step 1: Configure and apply ACL 100 on Router1**

```bash
Router>en
Router#conf t
Router(config)# access-list 100 permit tcp host 192.168.3.2 host 192.168.1.2 eq ftp
Router(config)# interface GigabitEthernet0/0
Router(config)# ip access-group 100 out
Router(config-line)#exit
Router(config)#
```

**Step 2: Verify FTP from both PCs**

```bash
# From PC0 (should SUCCEED)
PC>ftp 192.168.1.2
```

Expected Output (PC0):
```
Trying to connect...192.168.1.2
Connected to 192.168.1.2
220- Welcome to PT Ftp server
Username:cisco
331- Username ok, need password
Password:
230- Logged in
(passive mode On)
ftp>
```

```bash
# From PC1 (should FAIL)
PC>ftp 192.168.1.2
```

Expected Output (PC1):
```
%Error opening ftp://192.168.1.2/ (Timed out)
```

---

## Part 2: Configure, Apply and Verify an Extended Named ACL

**Goal:** Allow HTTP (www) from PC1 (192.168.3.3) to Server0 (192.168.1.3) only. PC0 should be blocked.

**Step 1: Configure and apply named ACL on Router1**

```bash
Router>en
Router#conf t
Router(config)# ip access-list extended DALMIA
Router(config-ext-nacl)# permit tcp host 192.168.3.3 host 192.168.1.3 eq www
Router(config-ext-nacl)#exit
Router(config)#
Router(config)#interface GigabitEthernet0/0
Router(config-if)#ip access-group DALMIA out
Router(config-if)#exit
Router(config)#
```

**Step 2: Verify HTTP from both PCs**

Open Web Browser on each PC and go to `http://192.168.1.3`

- **PC1** → Success (Cisco Packet Tracer page loads)
- **PC0** → Failure (`Request Timeout`)

### Notes / Important Points
- ACL number range: Standard = 1–99, Extended = 100–199 (also 2000–2699)
- `in` = filter traffic entering the interface; `out` = filter traffic leaving the interface
- There is an **implicit deny any** at the end of every ACL (not visible)
- Place ACLs **close to the destination** for extended ACLs
- `eq ftp` = port 21, `eq www` = port 80, `eq 443` = HTTPS

---

# Practical 4: Configure IP ACLs to Mitigate Attacks

## Network Topology

```
[Server0]      [Switch0] — Router0 — Router1 — Router2 — [Switch1] — [PC1]
192.168.1.2    192.168.1.x  S0/1/0=  S0/1/0=  S0/1/1=  192.168.4.x  192.168.4.2
                           192.168.2.1 192.168.2.2 192.168.3.2
                                        S0/1/1=  G0/0=
                                       192.168.3.1 192.168.4.1
```

### IP Address Table

| Device          | Interface   | IP Address   | Subnet Mask   |
|-----------------|-------------|--------------|---------------|
| Server0         | Fa0         | 192.168.1.2  | 255.255.255.0 |
| PC1             | -           | 192.168.4.2  | 255.255.255.0 |
| Router0 G0/0    | G0/0        | 192.168.1.1  | 255.255.255.0 |
| Router0 Serial  | S0/1/0      | 192.168.2.1  | 255.255.255.0 |
| Router1 Serial  | S0/1/0      | 192.168.2.2  | 255.255.255.0 |
| Router1 Serial  | S0/1/1      | 192.168.3.1  | 255.255.255.0 |
| Router2 Serial  | S0/1/1      | 192.168.3.2  | 255.255.255.0 |
| Router2 G0/0    | G0/0        | 192.168.4.1  | 255.255.255.0 |

### Prerequisites / Setup

- Add serial WIC-2T module to each router (Physical tab → power off → drag WIC-2T → power on)
- Assign IPs via GUI (Config tab for each interface)
- Set RIP routing on all three routers:
  - Router0: `192.168.1.0`, `192.168.2.0`
  - Router1: `192.168.2.0`, `192.168.3.0`
  - Router2: `192.168.3.0`, `192.168.4.0`
- Verify ping from PC1 to Server0 before applying ACLs

---

## Part 1: Verify Basic Connectivity

```bash
# From PC1
PC>ping 192.168.1.2

# From Server0
SERVER>ping 192.168.4.2
```

---

## Part 2: Secure Access to Routers (SSH + ACL)

### Part a: Set up SSH protocol on ALL Routers (R0, R1, R2)

```bash
Router>en
Router#conf t
Router(config)# ip domain-name dalmia.com
Router(config)# hostname R0
R0(config)# crypto key generate rsa
# When prompted for key size, enter: 512
R0(config)# line vty 0 4
R0(config-line)# transport input ssh
R0(config-line)# login local
R0(config-line)# exit
R0(config)# username SSHadmin privilege 15 password dalmia
R0(config)#exit
```

> Repeat for R1 and R2 (change hostname accordingly)

### Part b: Create ACL 10 to permit SSH only from PC1 — on ALL Routers

```bash
Router>en
Router#conf t
Router(config)# access-list 10 permit host 192.168.4.2
Router(config)# line vty 0 4
Router(config-line)# access-class 10 in
```

**Verify from PC1 (should succeed):**

```bash
PC>ssh -l SSHadmin 192.168.4.1
# Password: dalmia
```

**Verify from Server0 (should fail):**

```bash
SERVER>ssh -l SSHadmin 192.168.1.1
```

Expected Output:
```
% Connection refused by remote host
```

---

## Part 3: Create Numbered IP ACL 120 on Router1

**Goal:** Permit DNS, SMTP, FTP to Server. Deny HTTPS to Server.

**Step 1: Configure ACL 120 on Router1**

```bash
R1>enable
R1#
R1#configure terminal
R1(config)#access-list 120 permit udp any host 192.168.1.2 eq domain
R1(config)#access-list 120 permit tcp any host 192.168.1.2 eq smtp
R1(config)#access-list 120 permit tcp any host 192.168.1.2 eq ftp
R1(config)#access-list 120 deny tcp any host 192.168.1.2 eq 443
R1(config)#exit
R1#configure terminal
R1(config)#interface Serial0/1/1
R1(config-if)#ip access-group 120 in
```

**Step 2: Verify from PC1**

```bash
# FTP to server (should succeed)
PC>ftp 192.168.1.2
```

Expected Output:
```
Connected to 192.168.1.2
220- Welcome to PT Ftp server
Username:cisco
230- Logged in
ftp>
```

### Notes / Important Points
- `eq domain` = UDP port 53, `eq smtp` = TCP port 25, `eq ftp` = TCP port 21, `eq 443` = HTTPS
- `access-class` applies ACL to VTY lines; `ip access-group` applies ACL to interfaces
- ACL 10 (standard) only filters by source IP; ACL 120 (extended) filters by source + destination + protocol + port
- Apply extended ACLs **close to the source** to prevent wasted bandwidth

---

# Practical 5: Configure IPv6 ACLs

## Network Topology

```
[PC0]          [Switch0]
2002::2             |
             2002::1 (R0 G0/0)
[PC1]        2001::1 (R0 G0/1)    2003::1 (R0 Se0/1/0)
2001::2      [Switch1]                   |
                                   2003::2 (R1 Se0/1/0)
                                   2004::1 (R1 Se0/1/1)
                                         |
                                   2004::2 (R2 Se0/1/1)
                                   2005::1 (R2 G0/0)
                                         |
                                   [Switch2] — [Server0]
                                               2005::2
```

### IPv6 Address Table

| Device       | Interface | IPv6 Address | Gateway  |
|--------------|-----------|--------------|----------|
| PC0          | Fa0       | 2002::2 /64  | 2002::1  |
| PC1          | Fa0       | 2001::2 /64  | 2001::1  |
| Server0      | Fa0       | 2005::2 /64  | 2005::1  |
| Router0      | G0/0      | 2002::1 /64  | -        |
| Router0      | G0/1      | 2001::1 /64  | -        |
| Router0      | Se0/1/0   | 2003::1 /64  | -        |
| Router1      | Se0/1/0   | 2003::2 /64  | -        |
| Router1      | Se0/1/1   | 2004::1 /64  | -        |
| Router2      | Se0/1/1   | 2004::2 /64  | -        |
| Router2      | G0/0      | 2005::1 /64  | -        |

### Prerequisites / Setup

- Add WIC-2T serial module to each router (Physical tab)
- Set IPv6 addresses on PCs and Server via Config GUI
- Configure routers via CLI (IPv6 cannot be set via GUI)

---

### Steps & Commands

**Step 1: Configure Router0**

```bash
Router>
Router>en
Router#
Router#conf t
Router(config)#ipv6 unicast-routing
Router(config)#int G0/0
Router(config-if)#ipv6 address 2002::1/64
Router(config-if)#ipv6 rip a enable
Router(config-if)#no shut
Router(config-if)#exit
Router(config)#
Router(config)#int G0/1
Router(config-if)#ipv6 address 2001::1/64
Router(config-if)#ipv6 rip a enable
Router(config-if)#no shut
Router(config-if)#exit
Router(config)#
Router(config)#int Se0/1/0
Router(config-if)#ipv6 address 2003::1/64
Router(config-if)#ipv6 rip a enable
Router(config-if)#no shut
Router(config-if)#exit
Router(config)#
```

**Step 2: Configure Router1**

```bash
Router>en
Router#conf t
Router(config)#ipv6 unicast-routing
Router(config)#int Se0/1/0
Router(config-if)#ipv6 address 2003::2/64
Router(config-if)#ipv6 rip a enable
Router(config-if)#no shut
Router(config-if)#exit
Router(config)#
Router(config)#int Se0/1/1
Router(config-if)#ipv6 address 2004::1/64
Router(config-if)#ipv6 rip a enable
Router(config-if)#no shut
Router(config-if)#exit
Router(config)#
```

**Step 3: Configure Router2**

```bash
Router>en
Router#conf t
Router(config)#ipv6 unicast-routing
Router(config)#int Se0/1/1
Router(config-if)#ipv6 address 2004::2/64
Router(config-if)#ipv6 rip a enable
Router(config-if)#no shut
Router(config-if)#exit
Router(config)#int G0/0
Router(config-if)#ipv6 address 2005::1/64
Router(config-if)#ipv6 rip a enable
Router(config-if)#no shut
Router(config-if)#exit
Router(config)#
```

**Step 4: Verify IPv6 Connectivity**

```bash
# From PC0
C:\>ping 2005::2

# From PC1
C:\>ping 2005::2
```

Expected Output:
```
Pinging 2005::2 with 32 bytes of data:
Reply from 2005::2: bytes=32 time=2ms TTL=125
...
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

**Step 5: Configure IPv6 ACL on Router1**

ACL conditions:
1. Deny HTTP (www / port 80) to Server
2. Deny HTTPS (port 443) to Server
3. Permit all other IPv6 traffic

```bash
Router>
Router>enable
Router#conf t
Router(config)#ipv6 access-list dalmia
Router(config-ipv6-acl)#deny tcp any host 2005::2 eq www
Router(config-ipv6-acl)#deny tcp any host 2005::2 eq 443
Router(config-ipv6-acl)#permit ipv6 any any
Router(config-ipv6-acl)#
Router(config-ipv6-acl)#exit
Router(config)#
Router(config)#int Se0/1/0
Router(config-if)#ipv6 traffic-filter dalmia in
Router(config-if)#exit
Router(config)#
```

**Step 6: Verify ACL — Web access should FAIL**

Open Web Browser on PC0 or PC1 → navigate to `http://2005::2`

Expected Output:
```
Request Timeout
```

**Step 7: Verify IPv6 ping still works (should SUCCEED)**

```bash
C:\>ping 2005::2
```

Expected Output:
```
Reply from 2005::2: bytes=32 time=2ms TTL=125
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
```

### Notes / Important Points
- IPv6 ACLs are always **named** (no numbered ACLs in IPv6)
- Use `ipv6 traffic-filter` to apply (not `ip access-group`)
- Use `ipv6 access-class` for VTY line filtering
- `ipv6 unicast-routing` must be enabled on all routers
- `ipv6 rip a enable` — `a` is the RIP process name (arbitrary, must be consistent)

---

# Practical 8: Layer 2 Security

## Network Topology

```
                       [Cloud0]
                           |
                       Router0 (Se0/1/0)
                       G0/0
                           |
                    [Multilayer Switch0] (3560-24PS)
                    G0/1            G0/2
                   /                    \
          [Switch1]                   [Switch2]
          Fa0/23-24                   Fa0/23-24
          /       \                   /       \
    [SwitchA]  [Switch1]        [SwitchB]  [Switch2]
    Fa0/1-2   (uplinks)         Fa0/1-2   (uplinks)
    /     \                     /     \
 [PC0]  [PC1]               [PC2]  [PC3]
```

### Prerequisites / Setup

- Add WIC-2T serial module to Router0 (Physical tab)
- All switch connections use FastEthernet/GigabitEthernet ports as shown in the topology

---

## Part 1: Configure Root Bridge

**Step 1: Check current root bridge (on Multilayer Switch0)**

```bash
switch>en
switch# show spanning-tree
```

Expected Output shows current Root ID and Bridge ID with port roles (Desg/Altn/Root).

**Step 2: Assign Multilayer Switch0 as PRIMARY root bridge**

```bash
switch#conf t
switch(config)#spanning-tree vlan 1 root primary
switch(config)#do show span
```

Expected Output:
```
Root ID   Priority  24577
          Address   00D0.D31C.634C
          This bridge is the root
```

**Step 3: Assign Switch1 as SECONDARY root bridge**

```bash
switch#conf t
switch(config)#spanning-tree vlan 1 root secondary
```

**Step 4: Verify spanning-tree**

```bash
switch# show spanning-tree
```

---

## Part 2: Protect Against STP Attacks

**Step 1: Enable PortFast on access ports of SwitchA and SwitchB**

```bash
SwitchA>en
SwitchA#conf t
SwitchA(config)#int range f0/1-2
SwitchA(config-if-range)#spanning-tree portfast

SwitchB>en
SwitchB#conf t
SwitchB(config)#int range f0/1-2
SwitchB(config-if-range)#spanning-tree portfast
```

**Step 2: Enable BPDU Guard on access ports of SwitchA and SwitchB**

```bash
SwitchA(config)#int range f0/1-2
SwitchA(config-if-range)#spanning-tree bpduguard enable

SwitchB(config)#int range f0/1-2
SwitchB(config-if-range)#spanning-tree bpduguard enable
```

**Step 3: Enable Root Guard on Switch1 and Switch2 uplink ports**

```bash
Switch1>en
Switch1#conf t
Switch1(config)#int range f0/23-24
Switch1(config-if-range)#spanning-tree guard root

Switch2>en
Switch2#conf t
Switch2(config)#int range f0/23-24
Switch2(config-if-range)#spanning-tree guard root
```

---

## Part 3: Configure Port Security and Disable Unused Ports

**Step 1: Configure port security on SwitchA access ports**

Max 2 MAC addresses, dynamic (sticky) learning, shutdown on violation.

```bash
SwitchA>en
SwitchA#conf t
SwitchA(config)#int range f0/1-2
SwitchA(config-if-range)#switchport mode access
SwitchA(config-if-range)#switchport port-security
SwitchA(config-if-range)#switchport port-security maximum 2
SwitchA(config-if-range)#switchport port-security violation shutdown
SwitchA(config-if-range)#switchport port-security mac-address sticky
```

**Step 2: Configure port security on SwitchB access ports**

```bash
SwitchB>en
SwitchB#conf t
SwitchB(config)#int range f0/1-2
SwitchB(config-if-range)#switchport mode access
SwitchB(config-if-range)#switchport port-security
SwitchB(config-if-range)#switchport port-security maximum 2
SwitchB(config-if-range)#switchport port-security violation shutdown
SwitchB(config-if-range)#switchport port-security mac-address sticky
```

**Step 3: Verify port security on SwitchA**

```bash
SwitchA#show port-security int f0/1
```

Expected Output:
```
Port Security              : Enabled
Port Status                : Secure-up
Violation Mode             : Shutdown
Maximum MAC Addresses      : 2
Total MAC Addresses        : 0
Sticky MAC Addresses       : 0
Security Violation Count   : 0
```

**Step 4: Disable all unused ports on SwitchA and SwitchB**

```bash
SwitchA(config)#int range f0/3-22
SwitchA(config-if-range)#shutdown

SwitchB(config)#int range f0/3-22
SwitchB(config-if-range)#shutdown
```

### Notes / Important Points
- `spanning-tree vlan 1 root primary` sets priority to 24576 (lower = more likely to be root)
- `spanning-tree vlan 1 root secondary` sets priority to 28672
- **PortFast** — skips STP listening/learning states; use only on end-device ports, never on trunk/uplink ports
- **BPDU Guard** — shuts down a PortFast port if it receives a BPDU (prevents rogue switches)
- **Root Guard** — prevents a port from becoming the root port (protects against STP topology attacks)
- **Port Security** with `mac-address sticky` — dynamically learns MACs and saves them to config
- **Violation modes:** `shutdown` (default, disables port) | `restrict` (drops packets, logs) | `protect` (drops packets silently)
- CAM table overflow attack = flood switch with fake MACs to fill the table, forcing it to broadcast all frames

---

## Quick Command Reference

| Task | Command |
|------|---------|
| Enter privileged mode | `enable` |
| Enter global config | `conf t` / `configure terminal` |
| **— OSPF & MD5 (P1) —** | |
| Enable OSPF process | `router ospf 1` |
| Advertise network in OSPF | `network <net> <wildcard> area <id>` |
| Enable MD5 auth on interface | `ip ospf authentication message-digest` |
| Set MD5 key | `ip ospf message-digest-key 1 md5 <password>` |
| Verify OSPF interfaces | `show ip ospf interface` |
| **— NTP (P1) —** | |
| Set NTP server | `ntp server 192.168.1.3` |
| Sync hardware calendar | `ntp update-calendar` |
| Verify router clock | `show clock` |
| **— SYSLOG (P1) —** | |
| Send logs to SYSLOG server | `logging <syslog-ip>` |
| **— AAA (P2) —** | |
| Enable AAA | `aaa new-model` |
| Set TACACS server | `tacacs-server host <ip> key <secret>` |
| Set RADIUS server | `radius-server host <ip> key <secret>` |
| Create AAA login list | `aaa authentication login <name> group tacacs+ group radius local` |
| Apply AAA to VTY | `login authentication <n>` |
| **— Extended ACLs (P3) —** | |
| Create numbered extended ACL | `access-list <100-199> permit/deny <proto> <src> <dst> eq <port>` |
| Create named extended ACL | `ip access-list extended <n>` |
| Apply ACL to interface | `ip access-group <acl> in/out` |
| Apply ACL to VTY lines | `access-class <acl> in` |
| **— SSH & ACLs to Mitigate Attacks (P4) —** | |
| Set domain name | `ip domain-name <n>` |
| Set hostname | `hostname <n>` |
| Generate RSA key (SSH) | `crypto key generate rsa` |
| Restrict VTY to SSH only | `transport input ssh` |
| Require local login | `login local` |
| Create local admin user | `username <n> privilege 15 password <pass>` |
| SSH from PC | `ssh -l <user> <router-ip>` |
| **— IPv6 ACLs (P5) —** | |
| Enable IPv6 routing | `ipv6 unicast-routing` |
| Set IPv6 address | `ipv6 address <addr>/64` |
| Enable IPv6 RIP on interface | `ipv6 rip a enable` |
| Bring up interface | `no shut` |
| Create IPv6 ACL | `ipv6 access-list <n>` |
| Apply IPv6 ACL to interface | `ipv6 traffic-filter <n> in/out` |
| **— Layer 2 Security (P8) —** | |
| Check spanning tree | `show spanning-tree` |
| Set primary root bridge | `spanning-tree vlan 1 root primary` |
| Set secondary root bridge | `spanning-tree vlan 1 root secondary` |
| Enable PortFast on port | `spanning-tree portfast` |
| Enable BPDU Guard on port | `spanning-tree bpduguard enable` |
| Enable Root Guard on port | `spanning-tree guard root` |
| Set port as access port | `switchport mode access` |
| Enable port security | `switchport port-security` |
| Set max MACs | `switchport port-security maximum 2` |
| Set violation action | `switchport port-security violation shutdown` |
| Enable sticky MACs | `switchport port-security mac-address sticky` |
| Verify port security | `show port-security int f0/1` |
| Disable unused port range | `int range f0/3-22` → `shutdown` |
