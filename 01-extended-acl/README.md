<div align="center">

# 🔒 Extended ACL Configuration

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Network%20Security-red?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Configure Extended Access Control Lists to filter traffic by source IP, destination IP, protocol, and port number.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Topology](#-network-topology)
- [Technologies Used](#-technologies-used)
- [Lab Steps](#-lab-steps)
- [Verification](#-verification)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This lab demonstrates how to configure and apply **Extended ACLs** on Cisco routers to filter network traffic with granular control over source IP, destination IP, protocol type, and port number — unlike standard ACLs which only filter on source IP.

---

## 🗺️ Network Topology

```
[PC-A] ──── [SW1] ──── [R1] ──────── [R2] ──── [SW2] ──── [Server]
              │                        │
           VLAN 10                  VLAN 20
        192.168.1.0/24           192.168.2.0/24
```

> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|:---|:---|
| Cisco IOS CLI | Device configuration |
| Extended Named ACLs | Traffic filtering rules |
| IPv4 Routing | Static or dynamic routing between subnets |
| TCP / UDP / ICMP | Protocols being filtered |
| `ip access-group` | Applying ACL to an interface |

---

## 📝 Lab Steps

### Step 1 — Configure IP Addressing

Assign IP addresses to all router interfaces and end devices.

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip address 192.168.1.1 255.255.255.0
Router(config-if)# no shutdown
```

### Step 2 — Verify Connectivity Before ACL

Confirm full connectivity before applying any restrictions.

```cisco
PC> ping 192.168.2.10
```

### Step 3 — Create the Extended ACL

```cisco
Router(config)# ip access-list extended RESTRICT_TRAFFIC
Router(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 any eq 23
Router(config-ext-nacl)# deny tcp 192.168.1.0 0.0.0.255 any eq 21
Router(config-ext-nacl)# permit ip any any
```

> ⚠️ **Important:** Every ACL ends with an implicit `deny any any`. Always add `permit ip any any` at the end if you only want to block specific traffic.

### Step 4 — Apply ACL to Interface

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ip access-group RESTRICT_TRAFFIC in
```

> 💡 **Placement Rule:** Extended ACLs should be placed **closest to the source** to avoid unnecessary traffic traversing the network.

### Step 5 — Verify the ACL

```cisco
Router# show access-lists
Router# show ip interface GigabitEthernet0/0
```

### Step 6 — Test Traffic Filtering

| Test | Expected Result |
|:---|:---:|
| Telnet to restricted host | ❌ Denied |
| FTP to restricted host | ❌ Denied |
| HTTP traffic | ✅ Permitted |
| ICMP Ping | ✅ Permitted |

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show access-lists` | View ACL rules and match counters |
| `show ip interface Gi0/0` | Confirm ACL is bound to interface |
| `show running-config` | View full ACL in config context |
| `debug ip packet` | Live packet filtering trace *(use carefully)* |

---

## 💡 Key Concepts

<details>
<summary><b>Extended vs Standard ACL</b></summary>

| Feature | Standard ACL | Extended ACL |
|:---|:---:|:---:|
| Filters by Source IP | ✅ | ✅ |
| Filters by Destination IP | ❌ | ✅ |
| Filters by Protocol | ❌ | ✅ |
| Filters by Port Number | ❌ | ✅ |
| Placement | Near destination | Near source |
| ACL Number Range | 1–99 | 100–199 |

</details>

<details>
<summary><b>ACL Direction — Inbound vs Outbound</b></summary>

- **`in`** → Filters packets **entering** the interface (before routing)
- **`out`** → Filters packets **leaving** the interface (after routing)

Most commonly, Extended ACLs are applied **inbound** on the interface closest to the traffic source.

</details>

<details>
<summary><b>Wildcard Masks Quick Reference</b></summary>

| Subnet Mask | Wildcard Mask |
|:---|:---|
| /32 (255.255.255.255) | 0.0.0.0 |
| /24 (255.255.255.0) | 0.0.0.255 |
| /25 (255.255.255.128) | 0.0.0.127 |
| /30 (255.255.255.252) | 0.0.0.3 |
| Any host | `host x.x.x.x` |
| All traffic | `any` |

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | Full network topology |
| [`acl-config.png`](screenshots/acl-config.png) | ACL configuration output |
| [`denied-traffic.png`](screenshots/denied-traffic.png) | Telnet blocked by ACL |
| [`permitted-traffic.png`](screenshots/permitted-traffic.png) | HTTP/Ping allowed through |

---

## 🏆 Skills Demonstrated

- ✅ Configuring extended named ACLs on Cisco IOS
- ✅ Filtering traffic by protocol (TCP, UDP, ICMP) and port number
- ✅ Applying and verifying ACLs on router interfaces
- ✅ Testing permit/deny behavior using Packet Tracer simulation mode
- ✅ Understanding ACL placement best practices

---

<div align="center">

**[⬆ Back to Top](#-extended-acl-configuration)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
