# Multi-Tier Enterprise Campus Network Design

## Project Overview

A practical Enterprise Campus Network Lab designed and implemented in
Cisco Packet Tracer. The project demonstrates VLAN segmentation, VTP,
DTP, 802.1Q trunking, LACP EtherChannel, STP/Rapid-PVST, SVI, Inter-VLAN
Routing, Layer 3 switching, redundancy, and network verification.

The topology contains **2 Cisco 3560 Multilayer Switches, 5 Cisco 2960
Access Switches, 5 VLANs, and 20 PCs**.

> **Design note:** `ROOT` and `SEC` are organizational labels for the
> two Multilayer Switches. The Cisco 2960 devices are Layer 2 access
> switches. Layer 3 gateway and Inter-VLAN Routing functions are
> provided by the Multilayer Switch.

------------------------------------------------------------------------

## Objectives

-   Design a structured enterprise campus topology.
-   Segment departments with VLANs.
-   Propagate VLAN information with VTP.
-   Configure switch uplinks as trunks.
-   Demonstrate DTP trunk negotiation.
-   Bundle redundant links with LACP EtherChannel.
-   Implement STP / Rapid-PVST for Layer 2 loop prevention.
-   Configure SVIs and Inter-VLAN Routing on a Multilayer Switch.
-   Verify the design with Cisco IOS commands and end-to-end ping tests.

------------------------------------------------------------------------

# Topology

``` text
                              +----------------------+
                              |        ROOT          |
                              |   Cisco 3560-24PS    |
                              |   Multilayer Switch  |
                              +----------+-----------+
                                         ||
                                  LACP EtherChannel
                                  Fa0/4 + Fa0/5
                                         ||
                                  Fa0/1 + Fa0/2
                              +----------+-----------+
                              |         SEC          |
                              |   Cisco 3560-24PS   |
                              |   Multilayer Switch  |
                              +-----+-----------+----+
                                    |           |
                                  Trunk       Trunk
                                    |           |
                                   SW4         SW5
                                 VLAN 40     VLAN 50

             ROOT
            /  |  \
         Trunk Trunk Trunk
          /      |      \
        SW1     SW2     SW3
       VLAN10  VLAN20  VLAN30
        |         |        |
       PCs       PCs      PCs
```

## Device Roles

  Device   Model             Role
  -------- ----------------- ----------------------------------------
  ROOT     Cisco 3560-24PS   Multilayer switch / Primary STP Root
  SEC      Cisco 3560-24PS   Multilayer switch / Secondary STP Root
  SW1      Cisco 2960        Access switch / VLAN 10
  SW2      Cisco 2960        Access switch / VLAN 20
  SW3      Cisco 2960        Access switch / VLAN 30
  SW4      Cisco 2960        Access switch / VLAN 40
  SW5      Cisco 2960        Access switch / VLAN 50

------------------------------------------------------------------------

# VLAN & IP Addressing Plan

    VLAN Department / Segment     Network           Default Gateway
  ------ ------------------------ ----------------- -----------------
      10 Sales                    192.168.10.0/24   192.168.10.1
      20 HR                       192.168.20.0/24   192.168.20.1
      30 IT                       192.168.30.0/24   192.168.30.1
      40 Management               192.168.40.0/24   192.168.40.1
      50 Administration / Users   192.168.50.0/24   192.168.50.1

## End Devices

### VLAN 10 - Sales

  Packet Tracer Device   Hostname     IP              Mask            Gateway
  ---------------------- ------------ --------------- --------------- --------------
  PC0                    SALES-PC01   192.168.10.10   255.255.255.0   192.168.10.1
  PC1                    SALES-PC02   192.168.10.11   255.255.255.0   192.168.10.1
  PC2                    SALES-PC03   192.168.10.12   255.255.255.0   192.168.10.1
  PC3                    SALES-PC04   192.168.10.13   255.255.255.0   192.168.10.1

### VLAN 20 - HR

  Packet Tracer Device   Hostname   IP              Mask            Gateway
  ---------------------- ---------- --------------- --------------- --------------
  PC4                    HR-PC01    192.168.20.10   255.255.255.0   192.168.20.1
  PC5                    HR-PC02    192.168.20.11   255.255.255.0   192.168.20.1
  PC6                    HR-PC03    192.168.20.12   255.255.255.0   192.168.20.1
  PC7                    HR-PC04    192.168.20.13   255.255.255.0   192.168.20.1

### VLAN 30 - IT

  Packet Tracer Device   Hostname   IP              Mask            Gateway
  ---------------------- ---------- --------------- --------------- --------------
  PC8                    IT-PC01    192.168.30.10   255.255.255.0   192.168.30.1
  PC9                    IT-PC02    192.168.30.11   255.255.255.0   192.168.30.1
  PC10                   IT-PC03    192.168.30.12   255.255.255.0   192.168.30.1
  PC11                   IT-PC04    192.168.30.13   255.255.255.0   192.168.30.1

### VLAN 40 - Management

  Packet Tracer Device   Hostname    IP              Mask            Gateway
  ---------------------- ----------- --------------- --------------- --------------
  PC12                   MGMT-PC01   192.168.40.10   255.255.255.0   192.168.40.1
  PC13                   MGMT-PC02   192.168.40.11   255.255.255.0   192.168.40.1
  PC14                   MGMT-PC03   192.168.40.12   255.255.255.0   192.168.40.1
  PC15                   MGMT-PC04   192.168.40.13   255.255.255.0   192.168.40.1

### VLAN 50 - Administration / Users

  Packet Tracer Device   Hostname     IP              Mask            Gateway
  ---------------------- ------------ --------------- --------------- --------------
  PC16                   ADMIN-PC01   192.168.50.10   255.255.255.0   192.168.50.1
  PC17                   ADMIN-PC02   192.168.50.11   255.255.255.0   192.168.50.1
  PC18                   ADMIN-PC03   192.168.50.12   255.255.255.0   192.168.50.1
  PC19                   ADMIN-PC04   192.168.50.13   255.255.255.0   192.168.50.1

------------------------------------------------------------------------

# Port & Link Plan

  Link   Device A   Port    Device B   Port    Purpose
  ------ ---------- ------- ---------- ------- -------------
  1      ROOT       Fa0/1   SW1        Fa0/1   Trunk
  2      ROOT       Fa0/2   SW2        Fa0/1   Trunk
  3      ROOT       Fa0/3   SW3        Fa0/1   Trunk
  4      ROOT       Fa0/4   SEC        Fa0/1   LACP member
  5      ROOT       Fa0/5   SEC        Fa0/2   LACP member
  6      SEC        Fa0/3   SW4        Fa0/1   Trunk
  7      SEC        Fa0/4   SW5        Fa0/1   Trunk

### Access Port Assignment

``` text
SW1: Fa0/2-Fa0/5 -> VLAN 10
SW2: Fa0/2-Fa0/5 -> VLAN 20
SW3: Fa0/2-Fa0/5 -> VLAN 30
SW4: Fa0/2-Fa0/5 -> VLAN 40
SW5: Fa0/2-Fa0/5 -> VLAN 50
```

> If the final `.pkt` file uses different physical ports, update this
> table to match the actual topology.

------------------------------------------------------------------------

# Configuration Steps

## 1. Basic Switch Configuration

Example for ROOT:

``` cisco
enable
configure terminal
hostname ROOT
no ip domain-lookup
end
write memory
```

Repeat with the correct hostname for `SEC`, `SW1`, `SW2`, `SW3`, `SW4`,
and `SW5`.

------------------------------------------------------------------------

## 2. VTP

### ROOT - VTP Server

``` cisco
configure terminal
vtp domain AMR
vtp mode server
vtp version 2

vlan 10
 name SALES
vlan 20
 name HR
vlan 30
 name IT
vlan 40
 name MANAGEMENT
vlan 50
 name ADMINISTRATION
end
```

### SW1-SW5 - VTP Clients

``` cisco
configure terminal
vtp domain AMR
vtp mode client
vtp version 2
end
```

Verify:

``` cisco
show vtp status
show vlan brief
```

------------------------------------------------------------------------

## 3. LACP EtherChannel: ROOT \<-\> SEC

### ROOT

``` cisco
configure terminal
interface range fa0/4 - 5
 channel-group 1 mode active
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50
exit
end
```

### SEC

``` cisco
configure terminal
interface range fa0/1 - 2
 channel-group 1 mode active
exit

interface port-channel 1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50
exit
end
```

Verify:

``` cisco
show etherchannel summary
```

Expected member status:

``` text
Po1(SU)
Fa0/4(P)
Fa0/5(P)
```

on ROOT, and:

``` text
Po1(SU)
Fa0/1(P)
Fa0/2(P)
```

on SEC.

------------------------------------------------------------------------

## 4. DTP / Trunking

The lab uses DTP to demonstrate dynamic trunk negotiation.

### ROOT -\> SW1

ROOT:

``` cisco
interface fa0/1
 switchport mode dynamic desirable
```

SW1:

``` cisco
interface fa0/1
 switchport mode dynamic auto
```

### ROOT -\> SW2

ROOT:

``` cisco
interface fa0/2
 switchport mode dynamic desirable
```

SW2:

``` cisco
interface fa0/1
 switchport mode dynamic auto
```

### ROOT -\> SW3

ROOT:

``` cisco
interface fa0/3
 switchport mode dynamic desirable
```

SW3:

``` cisco
interface fa0/1
 switchport mode dynamic auto
```

### SEC -\> SW4

SEC:

``` cisco
interface fa0/3
 switchport mode dynamic desirable
```

SW4:

``` cisco
interface fa0/1
 switchport mode dynamic auto
```

### SEC -\> SW5

SEC:

``` cisco
interface fa0/4
 switchport mode dynamic desirable
```

SW5:

``` cisco
interface fa0/1
 switchport mode dynamic auto
```

Verify:

``` cisco
show interfaces trunk
```

------------------------------------------------------------------------

## 5. Access VLANs

### SW1 - VLAN 10

``` cisco
configure terminal
interface range fa0/2 - 5
 switchport mode access
 switchport access vlan 10
 spanning-tree portfast
exit
end
```

### SW2 - VLAN 20

``` cisco
configure terminal
interface range fa0/2 - 5
 switchport mode access
 switchport access vlan 20
 spanning-tree portfast
exit
end
```

### SW3 - VLAN 30

``` cisco
configure terminal
interface range fa0/2 - 5
 switchport mode access
 switchport access vlan 30
 spanning-tree portfast
exit
end
```

### SW4 - VLAN 40

``` cisco
configure terminal
interface range fa0/2 - 5
 switchport mode access
 switchport access vlan 40
 spanning-tree portfast
exit
end
```

### SW5 - VLAN 50

``` cisco
configure terminal
interface range fa0/2 - 5
 switchport mode access
 switchport access vlan 50
 spanning-tree portfast
exit
end
```

------------------------------------------------------------------------

## 6. Rapid-PVST / STP

On all switches:

``` cisco
configure terminal
spanning-tree mode rapid-pvst
end
```

Make ROOT the primary root:

``` cisco
configure terminal
spanning-tree vlan 10,20,30,40,50 root primary
end
```

Make SEC the secondary root:

``` cisco
configure terminal
spanning-tree vlan 10,20,30,40,50 root secondary
end
```

Verify:

``` cisco
show spanning-tree
show spanning-tree vlan 10
```

------------------------------------------------------------------------

## 7. SVI and Inter-VLAN Routing

On ROOT:

``` cisco
configure terminal
ip routing

interface vlan 10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit

interface vlan 20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit

interface vlan 30
 ip address 192.168.30.1 255.255.255.0
 no shutdown
exit

interface vlan 40
 ip address 192.168.40.1 255.255.255.0
 no shutdown
exit

interface vlan 50
 ip address 192.168.50.1 255.255.255.0
 no shutdown
exit

end
write memory
```

Verify:

``` cisco
show ip interface brief
show ip route
```

Expected connected routes:

``` text
192.168.10.0/24
192.168.20.0/24
192.168.30.0/24
192.168.40.0/24
192.168.50.0/24
```

------------------------------------------------------------------------

# PC Configuration

Each PC is configured with its VLAN network, `/24` subnet mask, and the
corresponding SVI gateway.

Example:

``` text
SALES-PC01
IP Address:      192.168.10.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.10.1
```

``` text
HR-PC01
IP Address:      192.168.20.10
Subnet Mask:     255.255.255.0
Default Gateway: 192.168.20.1
```

The remaining PCs follow the IP Addressing Plan in this README.

------------------------------------------------------------------------

# Testing & Verification

## VLAN Verification

``` cisco
show vlan brief
```

Confirm VLANs 10, 20, 30, 40, and 50 are present.

## VTP Verification

``` cisco
show vtp status
```

Confirm:

``` text
Domain: AMR
ROOT: Server
SW1-SW5: Client
```

## Trunk Verification

``` cisco
show interfaces trunk
```

Confirm the expected uplinks are trunking and the required VLANs are
allowed.

## EtherChannel Verification

``` cisco
show etherchannel summary
```

Look for:

``` text
Po1(SU)
member ports with (P)
```

## STP Verification

``` cisco
show spanning-tree
show spanning-tree vlan 10
```

Confirm ROOT is the intended root bridge.

## SVI Verification

``` cisco
show ip interface brief
```

Required SVIs should be `up/up`.

## Routing Verification

``` cisco
show ip route
```

Confirm the five connected VLAN networks appear in the routing table.

## MAC / ARP Verification

``` cisco
show mac address-table
show ip arp
```

------------------------------------------------------------------------

# Connectivity Tests

## Same-VLAN Test

From SALES-PC01:

``` text
ping 192.168.10.11
```

## Default Gateway Test

``` text
ping 192.168.10.1
```

## Inter-VLAN Tests

From SALES-PC01:

``` text
ping 192.168.20.10
ping 192.168.30.10
ping 192.168.40.10
ping 192.168.50.10
```

Successful replies demonstrate end-to-end Inter-VLAN Routing.

------------------------------------------------------------------------

# Troubleshooting Checklist

### EtherChannel Down

``` cisco
show etherchannel summary
show interfaces fa0/4 etherchannel
show interfaces fa0/5 etherchannel
```

Check that both sides use compatible EtherChannel settings and LACP
`mode active`.

### Trunk Not Working

``` cisco
show interfaces trunk
show interfaces fa0/1 switchport
```

Check DTP state, trunk status, and allowed VLANs.

### SVI Down

``` cisco
show ip interface brief
show vlan brief
```

Make sure the VLAN exists, is active, and has an active Layer 2 path.

### Inter-VLAN Ping Fails

Check in this order:

``` text
PC IP configuration
        ↓
Default Gateway
        ↓
Access VLAN
        ↓
Trunk
        ↓
SVI
        ↓
ip routing
        ↓
Routing Table
```

------------------------------------------------------------------------

# Useful Cisco IOS Commands

``` cisco
show running-config
show startup-config
show vlan brief
show interfaces status
show interfaces trunk
show interfaces fa0/1 switchport
show etherchannel summary
show etherchannel port-channel
show spanning-tree
show spanning-tree vlan 10
show vtp status
show ip interface brief
show ip route
show ip arp
show mac address-table
```

------------------------------------------------------------------------

# Basic Switch Hardening (Optional)

For a basic lab hardening layer:

``` cisco
configure terminal
enable secret Cisco@123
service password-encryption

line console 0
 password Console@123
 login
exit

line vty 0 4
 password VTY@123
 login
 transport input telnet ssh
exit

end
write memory
```

> For production networks, use strong unique credentials and prefer SSH
> instead of Telnet.

------------------------------------------------------------------------

# Project Verification Matrix

  -----------------------------------------------------------------------------
  Feature                 Command                       Expected Result
  ----------------------- ----------------------------- -----------------------
  VLANs                   `show vlan brief`             VLAN 10-50 present

  VTP                     `show vtp status`             Correct domain and mode

  Trunking                `show interfaces trunk`       Uplinks trunking

  EtherChannel            `show etherchannel summary`   `Po1(SU)` and members
                                                        `(P)`

  STP                     `show spanning-tree`          ROOT is root bridge

  SVI                     `show ip interface brief`     SVIs up/up

  Routing                 `show ip route`               Five connected networks

  MAC                     `show mac address-table`      Learned MAC addresses

  ARP                     `show ip arp`                 IP/MAC mappings

  Connectivity            `ping`                        Successful replies
  -----------------------------------------------------------------------------

------------------------------------------------------------------------

# Recommended GitHub Repository Structure

``` text
Multi-Tier-Enterprise-Campus-Network/
│
├── Enterprise-Campus-Network.pkt
├── README.md
│
├── topology/
│   └── topology.png
│
├── documentation/
│   ├── vlan-ip-table.png
│   ├── port-map.png
│   └── network-design.png
│
└── verification/
    ├── show-vlan.png
    ├── show-trunk.png
    ├── show-etherchannel.png
    ├── show-stp.png
    ├── show-ip-route.png
    └── inter-vlan-ping.png
```

------------------------------------------------------------------------

# LinkedIn Presentation Plan

Recommended screenshots for the LinkedIn post:

1.  **Full topology** - show ROOT, SEC, SW1-SW5, VLAN labels, end
    devices, trunks, and EtherChannel.
2.  **VLAN & IP table** - show department segmentation and addressing.
3.  **EtherChannel** - `show etherchannel summary`.
4.  **STP** - `show spanning-tree`.
5.  **Routing** - `show ip route` and `show ip interface brief`.
6.  **Connectivity** - successful pings between different VLANs.

------------------------------------------------------------------------

# Skills Demonstrated

-   Cisco IOS
-   Cisco Packet Tracer
-   VLANs and VLAN Segmentation
-   VTP
-   DTP
-   802.1Q Trunking
-   LACP EtherChannel
-   STP / PVST / Rapid-PVST
-   SVI
-   Inter-VLAN Routing
-   Layer 2 Switching
-   Layer 3 Switching
-   IP Addressing
-   Network Redundancy
-   Network Troubleshooting

------------------------------------------------------------------------

# Project Outcome

The completed lab demonstrates an enterprise-style campus switching
environment. Multiple departments are isolated using VLANs, VLAN
information is propagated using VTP, switch uplinks use trunking,
redundant ROOT-to-SEC links are aggregated using LACP EtherChannel,
Rapid-PVST provides Layer 2 loop prevention, and the Multilayer Switch
provides SVI-based Inter-VLAN Routing.

The implementation is validated through Cisco IOS verification commands
and end-to-end ICMP connectivity tests.

------------------------------------------------------------------------

# Author

**Amr Abdel-Khaleq Ibrahim**\
Aspiring Network Engineer\
Cisco Networking \| Switching \| Routing \| Windows Server

## Project Type

Cisco Networking / Enterprise Campus Network Lab

## Tool

Cisco Packet Tracer

------------------------------------------------------------------------

## Hashtags

`#Cisco` `#Networking` `#NetworkEngineer` `#CCNA` `#CCNP`
`#CiscoPacketTracer` `#VLAN` `#STP` `#EtherChannel` `#VTP` `#DTP`
`#InterVLANRouting` `#NetworkEngineering`
