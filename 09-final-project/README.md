<div align="center">

# 🏢 Final Project — Enterprise Network Design

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Enterprise%20Network-gold?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA%20Capstone-green?style=for-the-badge)

> A comprehensive capstone project integrating VLANs, OSPF, HSRP, EtherChannel, ACLs, DHCP, Wireless (WLC), and IPv6 into a complete enterprise network.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Overview](#-network-overview)
- [Technologies Integrated](#-technologies-integrated)
- [Addressing Table](#-addressing-table)
- [Configuration Phases](#-configuration-phases)
- [Verification Checklist](#-verification-checklist)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This capstone project integrates **all networking concepts** from the course into a single, fully functional enterprise network. The design includes a headquarters site with hierarchical switching, a remote branch connected over WAN, dynamic routing, redundant gateways, wireless infrastructure, and access control policies.

---

## 🗺️ Network Overview

```
┌─────────────────────────────── HEADQUARTERS ────────────────────────────────┐
│                                                                              │
│   [WLC]──[LAP]                                                               │
│      │                                                                       │
│  [SW_Access] ──EtherChannel──  [SW_Distribution] ──── [R_HQ1 (Active) ]──┐  │
│      │                               │                [R_HQ2 (Standby)]──┤  │
│  [End Devices]               [Server VLAN]             HSRP VIP: .1      │  │
│                                                                            │  │
└────────────────────────────────────────────────────────────────────────────┼─┘
                                                                             │
                                                                    WAN Serial Link
                                                                    10.0.0.0/30
                                                                             │
┌────────────────────────── BRANCH SITE ──────────────────────────────────────┼─┐
│                                                                             │  │
│  [PC_Branch] ──── [SW_Branch] ──── [R_Branch] ──────────────────────────────┘  │
│                                  OSPF Area 0                                    │
│                              172.16.1.0/24                                      │
└─────────────────────────────────────────────────────────────────────────────────┘
```

> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Integrated

| Technology | Role in This Project |
|:---|:---|
| **VLANs + Trunking** | Segment HQ into Users, Servers, Wireless, Management |
| **Inter-VLAN Routing** | Router-on-a-Stick subinterfaces for VLAN communication |
| **OSPF** | Dynamic routing between HQ and Branch |
| **HSRP** | Redundant default gateway — automatic failover |
| **DHCP** | Centralized address assignment per VLAN |
| **EtherChannel** | Aggregated uplinks between HQ switches |
| **Extended ACLs** | Enforce traffic policy between segments |
| **WLC + LAP** | Centralized wireless for HQ users |
| **SLAAC / DHCPv6** | IPv6 addressing on wireless segment |
| **Port Security** | Secure access layer switch ports |
| **DHCP Snooping** | Prevent rogue DHCP on access layer |

---

## 📊 Addressing Table

| Location | VLAN | Network | Gateway |
|:---|:---:|:---|:---|
| HQ — Users | 10 | 192.168.10.0/24 | 192.168.10.1 (HSRP) |
| HQ — Servers | 20 | 192.168.20.0/24 | 192.168.20.1 (HSRP) |
| HQ — Wireless | 30 | 192.168.30.0/24 | 192.168.30.1 |
| HQ — Management | 99 | 192.168.99.0/24 | 192.168.99.1 |
| WAN Link | — | 10.0.0.0/30 | — |
| Branch — Users | — | 172.16.1.0/24 | 172.16.1.1 |

---

## 🔧 Configuration Phases

<details open>
<summary><b>Phase 1 — Switching (VLANs, Trunks, EtherChannel)</b></summary>

**Create VLANs:**
```cisco
SW_Access(config)# vlan 10
SW_Access(config-vlan)# name HQ-Users
SW_Access(config)# vlan 20
SW_Access(config-vlan)# name Servers
SW_Access(config)# vlan 30
SW_Access(config-vlan)# name Wireless
SW_Access(config)# vlan 99
SW_Access(config-vlan)# name Management
```

**Configure EtherChannel uplink:**
```cisco
SW_Access(config)# interface range GigabitEthernet0/1 - 2
SW_Access(config-if-range)# channel-group 1 mode active
SW_Access(config)# interface port-channel 1
SW_Access(config-if)# switchport mode trunk
```

**Configure access ports:**
```cisco
SW_Access(config)# interface FastEthernet0/1
SW_Access(config-if)# switchport mode access
SW_Access(config-if)# switchport access vlan 10
SW_Access(config-if)# spanning-tree portfast
SW_Access(config-if)# spanning-tree bpduguard enable
```

</details>

<details>
<summary><b>Phase 2 — Routing (Inter-VLAN + OSPF)</b></summary>

**Inter-VLAN Routing (Router-on-a-Stick):**
```cisco
R_HQ1(config)# interface GigabitEthernet0/0.10
R_HQ1(config-subif)# encapsulation dot1Q 10
R_HQ1(config-subif)# ip address 192.168.10.2 255.255.255.0

R_HQ1(config)# interface GigabitEthernet0/0.20
R_HQ1(config-subif)# encapsulation dot1Q 20
R_HQ1(config-subif)# ip address 192.168.20.2 255.255.255.0
```

**WAN Link:**
```cisco
R_HQ1(config)# interface Serial0/0/0
R_HQ1(config-if)# ip address 10.0.0.1 255.255.255.252
R_HQ1(config-if)# clock rate 128000
R_HQ1(config-if)# no shutdown
```

**OSPF Configuration:**
```cisco
R_HQ1(config)# router ospf 1
R_HQ1(config-router)# router-id 1.1.1.1
R_HQ1(config-router)# network 192.168.10.0 0.0.0.255 area 0
R_HQ1(config-router)# network 192.168.20.0 0.0.0.255 area 0
R_HQ1(config-router)# network 10.0.0.0 0.0.0.3 area 0
R_HQ1(config-router)# passive-interface GigabitEthernet0/0.10
R_HQ1(config-router)# passive-interface GigabitEthernet0/0.20
```

</details>

<details>
<summary><b>Phase 3 — HSRP (Gateway Redundancy)</b></summary>

**R_HQ1 — Active (Priority 110):**
```cisco
R_HQ1(config)# interface GigabitEthernet0/0.10
R_HQ1(config-subif)# standby version 2
R_HQ1(config-subif)# standby 10 ip 192.168.10.1
R_HQ1(config-subif)# standby 10 priority 110
R_HQ1(config-subif)# standby 10 preempt
R_HQ1(config-subif)# standby 10 track Serial0/0/0 30
```

**R_HQ2 — Standby (Priority 100):**
```cisco
R_HQ2(config)# interface GigabitEthernet0/0.10
R_HQ2(config-subif)# standby version 2
R_HQ2(config-subif)# standby 10 ip 192.168.10.1
R_HQ2(config-subif)# standby 10 priority 100
R_HQ2(config-subif)# standby 10 preempt
```

</details>

<details>
<summary><b>Phase 4 — DHCP (Per-VLAN Pools)</b></summary>

```cisco
R_HQ1(config)# ip dhcp excluded-address 192.168.10.1 192.168.10.20
R_HQ1(config)# ip dhcp pool VLAN10-USERS
R_HQ1(config-dhcp)# network 192.168.10.0 255.255.255.0
R_HQ1(config-dhcp)# default-router 192.168.10.1
R_HQ1(config-dhcp)# dns-server 8.8.8.8

R_HQ1(config)# ip dhcp pool VLAN20-SERVERS
R_HQ1(config-dhcp)# network 192.168.20.0 255.255.255.0
R_HQ1(config-dhcp)# default-router 192.168.20.1
```

</details>

<details>
<summary><b>Phase 5 — Security (ACLs + Port Security + DHCP Snooping)</b></summary>

**Block Branch users from accessing HQ Servers:**
```cisco
R_HQ1(config)# ip access-list extended BRANCH-POLICY
R_HQ1(config-ext-nacl)# deny ip 172.16.1.0 0.0.0.255 192.168.20.0 0.0.0.255
R_HQ1(config-ext-nacl)# permit ip any any
R_HQ1(config)# interface Serial0/0/0
R_HQ1(config-if)# ip access-group BRANCH-POLICY in
```

**DHCP Snooping on Access Switch:**
```cisco
SW_Access(config)# ip dhcp snooping
SW_Access(config)# ip dhcp snooping vlan 10,20,30
SW_Access(config)# interface GigabitEthernet0/1
SW_Access(config-if)# ip dhcp snooping trust
```

</details>

<details>
<summary><b>Phase 6 — Wireless (WLC + SSID)</b></summary>

Configure the WLC with SSID `HQ-Corp-WiFi` mapped to VLAN 30.  
See the full [WLC Lab README](../08-wlc/README.md) for step-by-step WLC GUI instructions.

</details>

---

## ✅ Verification Checklist

| Test | Method | Expected Result |
|:---|:---|:---:|
| HQ Users → HQ Servers | Ping | ✅ Allowed |
| HQ Users → Branch | Ping | ✅ Allowed |
| Branch → HQ Servers | Ping | ❌ Blocked by ACL |
| Wireless → HQ LAN | Ping | ✅ Allowed |
| R_HQ1 fails → users reconnect | Shutdown Gi0/0 on R_HQ1 | ✅ HSRP Failover |
| All routers OSPF full | `show ip ospf neighbor` | ✅ FULL state |
| EtherChannel operational | `show etherchannel summary` | ✅ SU |
| DHCP working per VLAN | `show ip dhcp binding` | ✅ IPs assigned |
| Wireless client gets IP | Connect to SSID | ✅ DHCP from VLAN 30 |

---

## 💡 Key Concepts

<details>
<summary><b>Hierarchical Network Design</b></summary>

```
Access Layer     → Connect end devices, enforce security (Port Security, BPDU Guard)
Distribution Layer → Inter-VLAN routing, policy enforcement (ACLs), EtherChannel
Core Layer       → High-speed backbone, minimal config, maximum performance
```

</details>

<details>
<summary><b>Redundancy at Every Layer</b></summary>

| Layer | Redundancy Feature |
|:---|:---|
| Link level | EtherChannel (multiple physical → one logical) |
| Gateway level | HSRP (two routers → one virtual gateway) |
| Routing level | OSPF (automatic rerouting around failed links) |
| Wireless | Multiple APs managed by WLC (seamless roaming) |

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | Full HQ + Branch enterprise topology |


---

## 🏆 Skills Demonstrated

- ✅ End-to-end enterprise network design and implementation
- ✅ Integrating routing, switching, security, wireless, and IPv6
- ✅ Applying hierarchical design principles (Access/Distribution/Core)
- ✅ Implementing redundancy at link, gateway, and routing layers
- ✅ Verifying complex multi-protocol environments systematically
- ✅ Applying real-world network design best practices

---

<div align="center">

**[⬆ Back to Top](#-final-project--enterprise-network-design)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
