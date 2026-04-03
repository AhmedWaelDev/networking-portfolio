<div align="center">

# 🛡️ Network Attack Mitigation Lab

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Network%20Security-red?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Identify and mitigate common Layer 2 attacks including MAC flooding, DHCP spoofing, ARP poisoning, and STP manipulation.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Topology](#-network-topology)
- [Attacks & Mitigations Overview](#-attacks--mitigations-overview)
- [Lab Steps](#-lab-steps)
- [Verification](#-verification)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This lab demonstrates how to identify common **Layer 2 and Layer 3 network attacks** and apply Cisco IOS security features to mitigate them — covering Port Security, DHCP Snooping, Dynamic ARP Inspection, VLAN hardening, and STP protection.

---

## 🗺️ Network Topology


> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## ⚔️ Attacks & Mitigations Overview

| Attack | Layer | What It Does | Mitigation |
|:---|:---:|:---|:---|
| MAC Flooding | L2 | Overwhelms CAM table → switch broadcasts all traffic | Port Security |
| DHCP Starvation | L2/L3 | Exhausts DHCP pool with fake requests | DHCP Snooping |
| DHCP Spoofing | L2/L3 | Rogue DHCP server sends wrong gateway/DNS | DHCP Snooping (trust ports) |
| ARP Poisoning | L2 | Corrupts ARP cache → man-in-the-middle | Dynamic ARP Inspection |
| VLAN Hopping | L2 | Accesses other VLANs via DTP/double-tagging | Disable DTP, change native VLAN |
| STP Manipulation | L2 | Rogue switch becomes Root Bridge | BPDU Guard, Root Guard |

---

## 📝 Lab Steps

### 1️⃣ Port Security — Against MAC Flooding

```cisco
SW1(config)# interface FastEthernet0/1
SW1(config-if)# switchport mode access
SW1(config-if)# switchport port-security
SW1(config-if)# switchport port-security maximum 2
SW1(config-if)# switchport port-security mac-address sticky
SW1(config-if)# switchport port-security violation restrict
```

**Violation Modes:**

| Mode | On Violation |
|:---|:---|
| `protect` | Drops frames silently |
| `restrict` | Drops frames + increments counter |
| `shutdown` | ❌ Shuts port (err-disabled) — default |

Recover an err-disabled port:
```cisco
SW1(config-if)# shutdown
SW1(config-if)# no shutdown
```

---

### 2️⃣ DHCP Snooping — Against Rogue DHCP Servers

```cisco
SW1(config)# ip dhcp snooping
SW1(config)# ip dhcp snooping vlan 10
SW1(config)# no ip dhcp snooping information option

! Trust ONLY the port connected to the real DHCP server
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# ip dhcp snooping trust
```

> ⚠️ Any port **without** `ip dhcp snooping trust` is untrusted — DHCP offers from it are dropped.

---

### 3️⃣ Dynamic ARP Inspection — Against ARP Spoofing

> DAI validates ARP packets against the DHCP Snooping binding table.

```cisco
SW1(config)# ip arp inspection vlan 10

! Trust the uplink to the router/DHCP server
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# ip arp inspection trust
```

---

### 4️⃣ VLAN Hopping Prevention

```cisco
! Disable DTP on all access ports
SW1(config)# interface FastEthernet0/2
SW1(config-if)# switchport mode access
SW1(config-if)# switchport nonegotiate

! Change native VLAN on trunks away from VLAN 1
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport trunk native vlan 999
SW1(config-if)# switchport nonegotiate
```

---

### 5️⃣ BPDU Guard — Against STP Manipulation

```cisco
! Apply to individual access port
SW1(config)# interface FastEthernet0/1
SW1(config-if)# spanning-tree portfast
SW1(config-if)# spanning-tree bpduguard enable

! Or enable globally on ALL PortFast interfaces
SW1(config)# spanning-tree portfast bpduguard default
```

---

### 6️⃣ Storm Control

```cisco
SW1(config)# interface FastEthernet0/1
SW1(config-if)# storm-control broadcast level 20
SW1(config-if)# storm-control action shutdown
```

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show port-security interface Fa0/1` | Port security status and violation count |
| `show port-security address` | Learned/sticky MAC addresses |
| `show ip dhcp snooping` | Global DHCP snooping status |
| `show ip dhcp snooping binding` | Trusted MAC-to-IP-to-port bindings |
| `show ip arp inspection` | DAI statistics and drop counts |
| `show ip arp inspection vlan 10` | Per-VLAN DAI status |
| `show spanning-tree` | STP topology and BPDU guard status |

---

## 💡 Key Concepts

<details>
<summary><b>Defense in Depth — Why Use Multiple Features?</b></summary>

No single feature protects against everything:

```
Port Security    → Limits who connects physically
DHCP Snooping    → Ensures legitimate IP assignment
DAI              → Prevents ARP cache poisoning
BPDU Guard       → Protects STP from rogue switches
VLAN Hardening   → Prevents cross-VLAN attacks
Storm Control    → Protects against traffic floods
```

Together, these features create **layered defense** — an attacker must defeat all layers.

</details>

<details>
<summary><b>DHCP Snooping Trusted vs Untrusted Ports</b></summary>

```
Trusted Ports  → Router, DHCP server, uplinks
                 DHCP offers and replies ALLOWED

Untrusted Ports → End devices, access ports
                  DHCP offers and replies DROPPED
                  Only DHCP requests allowed
```

</details>

<details>
<summary><b>How ARP Poisoning Works (and how DAI stops it)</b></summary>

**Without DAI:**
1. Attacker sends fake ARP reply: "192.168.1.1 is at AA:BB:CC:DD:EE:FF (attacker's MAC)"
2. Victims update ARP cache with attacker's MAC
3. All traffic to gateway goes to attacker → **man-in-the-middle attack**

**With DAI:**
1. Switch checks incoming ARP against DHCP Snooping binding table
2. If MAC+IP combination doesn't match → ARP is **dropped**
3. Attacker cannot inject false ARP entries

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | Network security lab topology |


---

## 🏆 Skills Demonstrated

- ✅ Configuring Port Security with sticky MAC learning
- ✅ Implementing DHCP Snooping to prevent rogue DHCP servers
- ✅ Deploying Dynamic ARP Inspection for ARP spoofing prevention
- ✅ Hardening switch configurations against VLAN hopping
- ✅ Protecting STP topology with BPDU Guard and PortFast

---

<div align="center">

**[⬆ Back to Top](#%EF%B8%8F-network-attack-mitigation-lab)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
