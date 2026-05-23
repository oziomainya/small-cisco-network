# Small Cisco Network: Full Initial Configuration

> **Category:** Networking
> **Status:** Live
> **Course:** Cisco Networking Devices and Initial Configuration · Junior Cybersecurity Analyst Career Path
> **Author:** Ozioma Inya         <!-- REPLACE with your name -->
> **Date:** August, 2025              <!-- REPLACE with completion date e.g. "May, 2025" -->

---

## Overview

A from-scratch initial configuration of a Cisco router and switch built in Cisco
Packet Tracer. Covers device hardening, secure access configuration, IP addressing,
switch management interface setup, and end-to-end connectivity verification.

---

## Objectives

- [x] Build a small network topology with one router, one switch, and two end devices
- [x] Perform initial configuration of a Cisco router from factory defaults
- [x] Perform initial configuration of a Cisco switch from factory defaults
- [x] Secure both devices with encrypted passwords, console line security, and VTY line security
- [x] Configure the switch management IP and default gateway
- [x] Verify end-to-end connectivity between all devices using ping
- [x] Document all configurations, verification outputs, and lessons learned

---

## Tools & Technologies

| Tool | Purpose |
|---|---|
| Cisco Packet Tracer | Network simulation and device configuration |
| Cisco IOS CLI | Router and switch configuration |
| ICMP Ping | End-to-end connectivity verification |

---

## Skills Demonstrated

`Initial Device Configuration` `Cisco IOS CLI` `Device Hardening`
`Console Line Security` `VTY Line Security` `Enable Secret`
`IPv4 Addressing` `Switch Management IP` `Default Gateway`
`MOTD Banner` `Packet Tracer` `Network Troubleshooting`

---

## Network Topology

```
  Network: 192.168.1.0/24

  [PC1: 192.168.1.10] --+
                         +--> [SW1: VLAN1 192.168.1.2] --> [R1 Fa0/0: 192.168.1.1]
  [PC2: 192.168.1.20] --+
```

### Addressing Table

| Device | Interface | IP Address | Subnet Mask | Default Gateway |
|---|---|---|---|---|
| PC1 | NIC | 192.168.1.10 | 255.255.255.0 | 192.168.1.1 |
| PC2 | NIC | 192.168.1.20 | 255.255.255.0 | 192.168.1.1 |
| R1 | G0/0 | 192.168.1.1 | 255.255.255.0 | -- |
| SW1 | VLAN 1 | 192.168.1.2 | 255.255.255.0 | 192.168.1.1 |

---

## Methodology

### Step 1 -- Build the Topology
Placed one 1941 router (R1), one 2960 switch (SW1), and two PCs in Packet Tracer.
Connected PC1 and PC2 to SW1 using straight-through cables, and SW1 to R1 G0/0.

### Step 2 -- Configure Initial Router Settings
Set the hostname, configured a MOTD banner, assigned an IP to G0/0,
and brought the interface up.

```
Router> enable
Router# configure terminal

Router(config)# hostname R1
R1(config)# banner motd # Authorised Access Only. Disconnect immediately if not authorised. #

R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.1 255.255.255.0
R1(config-if)# description Link to SW1 - LAN Interface
R1(config-if)# no shutdown
R1(config-if)# exit
```

### Step 3 -- Secure the Router
Configured enable secret, console line password, VTY line password,
and enabled password encryption.

```
R1(config)# enable secret Cisco123!

R1(config)# line console 0
R1(config-line)# password Console123!
R1(config-line)# login
R1(config-line)# exit

R1(config)# line vty 0 15
R1(config-line)# password Vty123!
R1(config-line)# login
R1(config-line)# exit

R1(config)# service password-encryption
R1(config)# end
R1# write memory
```

### Step 4 -- Configure Initial Switch Settings
Applied the same baseline hardening to SW1 -- hostname, MOTD banner,
enable secret, console and VTY line security, and password encryption.

```
Switch> enable
Switch# configure terminal

Switch(config)# hostname SW1
SW1(config)# banner motd # Authorised Access Only. Disconnect immediately if not authorised. #

SW1(config)# enable secret Cisco123!

SW1(config)# line console 0
SW1(config-line)# password Console123!
SW1(config-line)# login
SW1(config-line)# exit

SW1(config)# line vty 0 15
SW1(config-line)# password Vty123!
SW1(config-line)# login
SW1(config-line)# exit

SW1(config)# service password-encryption
```

### Step 5 -- Set Switch Management IP and Default Gateway
Assigned a static IP to VLAN 1 and configured the default gateway on SW1.

```
SW1(config)# interface vlan 1
SW1(config-if)# ip address 192.168.1.2 255.255.255.0
SW1(config-if)# no shutdown
SW1(config-if)# exit

SW1(config)# ip default-gateway 192.168.1.1
SW1(config)# end
SW1# write memory
```

### Step 6 -- Assign IP Addresses to PCs

```
PC1  IP: 192.168.1.10   Mask: 255.255.255.0   GW: 192.168.1.1
PC2  IP: 192.168.1.20   Mask: 255.255.255.0   GW: 192.168.1.1
```

### Step 7 -- Verify End-to-End Connectivity

```
C:\> ping 192.168.1.20   -- PC1 to PC2        SUCCESS
C:\> ping 192.168.1.1    -- PC1 to R1 G0/0   SUCCESS
C:\> ping 192.168.1.2    -- PC1 to SW1 VLAN1  SUCCESS

R1# show ip interface brief
GigabitEthernet0/0  192.168.1.1  YES  manual  up  up

R1# show running-config | include secret
enable secret 5 $1$mERr$9cTjUIlmO7RHSbPQZObB./
```

---

## Results

| # | Result | Status |
|---|---|---|
| 1 | All ping tests from PC1 to PC2, R1, and SW1 VLAN 1 succeeded | OK |
| 2 | Running config confirmed all passwords were encrypted | OK |
| 3 | MOTD banner displayed correctly on console login for both devices | OK |
| 4 | show ip interface brief confirmed R1 G0/0 and SW1 VLAN 1 both up/up | OK |
| 5 | SW1 VLAN 1 not reachable on first attempt -- no shutdown missing | WARNING |
| 6 | Fixed by adding no shutdown to VLAN 1 interface | FIXED |
| 7 | SW1 not pingable from different subnet -- ip default-gateway missing | WARNING |
| 8 | Fixed by adding ip default-gateway 192.168.1.1 under global config | FIXED |

---

## Lessons Learned

**1. Device hardening is the first task, not an afterthought**
On a real network, an unconfigured device is an open door. Setting the enable
secret, securing console and VTY lines, encrypting passwords, and adding a MOTD
banner are the baseline before anything else is configured.

**2. The switch needs a default gateway too**
Without a default gateway, the switch can only be managed from within its own
subnet. This became clear when trying to reach the switch from a different
network segment during extended testing.

**3. no shutdown is required for VLAN interfaces too**
Just like physical router interfaces, VLAN interfaces on a switch are
administratively down by default. The management IP is configured but
completely unreachable without it.

**4. show commands are part of the configuration process**
Using show running-config, show ip interface brief after each step made
it possible to catch mistakes immediately rather than only at the end.

---

## Repository Structure

```
small-cisco-network/
|-- index.html                         <- Full project write-up (live via GitHub Pages)
|-- README.md                          <- This file
|-- small-cisco-network.pkt            <- Cisco Packet Tracer project file
|-- topology.png                       <- Full topology screenshot
|-- R1-running-config.txt              <- R1 running configuration output
|-- SW1-running-config.txt             <- SW1 running configuration output
|-- pings.png                          <- Ping test results from PC1
+-- interface-brief.png                <- show ip interface brief output
```

<!-- REMOVE any files from the structure above that you have not added yet -->

---

## Links

- [Full Project Write-up](https://oziomainya.github.io/small-cisco-network)
- [Portfolio](https://oziomainya.github.io)

<!-- REPLACE [YOUR_GITHUB_USERNAME] and [YOUR_PORTFOLIO_URL] with your real details -->

---

*Ozioma Inya · [LinkedIn ↗](https://linkedin.com/in/ozioma-inya-a46327304) · [Github ↗](https://github.com/oziomainya)*
<!-- REPLACE the placeholders above with your real details -->
