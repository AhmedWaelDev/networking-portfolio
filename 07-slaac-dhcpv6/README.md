<div align="center">

# 🌍 SLAAC & DHCPv6 — IPv6 Address Assignment Lab

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-IPv6%20Addressing-9b59b6?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Configure all three IPv6 address assignment methods — SLAAC, Stateless DHCPv6, and Stateful DHCPv6 — and understand how Router Advertisement flags control client behavior.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Topology](#-network-topology)
- [Technologies Used](#-technologies-used)
- [Assignment Methods Comparison](#-assignment-methods-comparison)
- [Lab Steps](#-lab-steps)
- [Verification](#-verification)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This lab demonstrates the three methods of **IPv6 address assignment**: SLAAC, Stateless DHCPv6, and Stateful DHCPv6. The router is configured to send Router Advertisements (RAs) with different flag combinations to control how end devices obtain their IPv6 configuration.

---

## 🗺️ Network Topology


> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|:---|:---|
| `ipv6 unicast-routing` | Enables IPv6 routing and RA transmission |
| ICMPv6 Router Advertisements | Routers announce prefix and flags to clients |
| SLAAC (RFC 4862) | Client self-generates address from RA prefix |
| EUI-64 | Algorithm that creates interface ID from MAC |
| Stateless DHCPv6 | SLAAC address + DNS/options from DHCPv6 |
| Stateful DHCPv6 | Full address + options assigned by DHCPv6 |
| Link-Local Address | Auto-configured `FE80::/10` address |

---

## 📊 Assignment Methods Comparison

| Method | Address From | DNS / Options | RA M Flag | RA O Flag |
|:---|:---|:---|:---:|:---:|
| **SLAAC** | RA Prefix + EUI-64 | None | 0 | 0 |
| **Stateless DHCPv6** | RA Prefix + EUI-64 | DHCPv6 Server | 0 | 1 |
| **Stateful DHCPv6** | DHCPv6 Server | DHCPv6 Server | 1 | 1 |

> - **M flag** (Managed) = 1 → Get address from DHCPv6
> - **O flag** (Other) = 1 → Get other config (DNS) from DHCPv6
> - **A flag** (Autonomous) = 0 → Suppress SLAAC self-configuration

---

## 📝 Lab Steps

### Step 0 — Enable IPv6 Routing (Required!)

```cisco
Router(config)# ipv6 unicast-routing
```

> ⚠️ **Critical:** Without this command, the router will NOT send RA messages — no IPv6 client can autoconfigure.

### Step 1 — Configure Router Interface

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ipv6 address 2001:DB8:ACAD:1::1/64
Router(config-if)# ipv6 address fe80::1 link-local
Router(config-if)# no shutdown
```

---

### Method A — SLAAC Only

No DHCPv6 server needed. The router's RA carries the prefix; the client generates its own interface ID using EUI-64.

> ✅ SLAAC is automatic after `ipv6 unicast-routing` is enabled. No extra configuration needed.

Verify on PC:
```
PC> ipconfig
```
The PC should show a Global Unicast address starting with `2001:DB8:ACAD:1::`

---

### Method B — Stateless DHCPv6

Clients use SLAAC for their address but get DNS and domain info from a DHCPv6 server.

**Configure the DHCPv6 pool (DNS info only):**
```cisco
Router(config)# ipv6 dhcp pool STATELESS-POOL
Router(config-dhcpv6)# dns-server 2001:DB8:ACAD::254
Router(config-dhcpv6)# domain-name example.com
Router(config-dhcpv6)# exit
```

**Apply to interface and set O flag:**
```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ipv6 dhcp server STATELESS-POOL
Router(config-if)# ipv6 nd other-config-flag
```

---

### Method C — Stateful DHCPv6

The DHCPv6 server assigns the complete IPv6 address, prefix length, and DNS.

**Configure the DHCPv6 pool with address range:**
```cisco
Router(config)# ipv6 dhcp pool STATEFUL-POOL
Router(config-dhcpv6)# address prefix 2001:DB8:ACAD:2::/64 lifetime infinite infinite
Router(config-dhcpv6)# dns-server 2001:DB8:ACAD::254
Router(config-dhcpv6)# domain-name example.com
Router(config-dhcpv6)# exit
```

**Apply to interface and set M flag + suppress SLAAC:**
```cisco
Router(config)# interface GigabitEthernet0/1
Router(config-if)# ipv6 dhcp server STATEFUL-POOL
Router(config-if)# ipv6 nd managed-config-flag
Router(config-if)# ipv6 nd prefix default no-autoconfig
```

---

### DHCPv6 Relay (Server on a different subnet)

```cisco
Router(config)# interface GigabitEthernet0/0
Router(config-if)# ipv6 dhcp relay destination 2001:DB8:ACAD::100
```

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show ipv6 interface Gi0/0` | IPv6 addresses and RA flag settings |
| `show ipv6 dhcp pool` | DHCPv6 pool details and lease count |
| `show ipv6 dhcp binding` | Active stateful DHCPv6 address leases |
| `show ipv6 routers` | RA contents from neighboring routers |
| `show ipv6 route` | Full IPv6 routing table |
| `ping 2001:DB8:ACAD:1::1` | Test IPv6 connectivity |

---

## 💡 Key Concepts

<details>
<summary><b>How EUI-64 Generates Interface IDs</b></summary>

SLAAC uses **EUI-64** to derive a 64-bit interface ID from the device's 48-bit MAC address:

```
Step 1: Take MAC          AA:BB:CC:DD:EE:FF
Step 2: Split in half      AA:BB:CC  |  DD:EE:FF
Step 3: Insert FF:FE       AA:BB:CC:FF:FE:DD:EE:FF
Step 4: Flip bit 7         A8:BB:CC:FF:FE:DD:EE:FF
                             ↑ (U/L bit flipped)

Result IPv6 suffix:        A8BB:CCFF:FEDD:EEFF
Full address:              2001:DB8:ACAD:1:A8BB:CCFF:FEDD:EEFF/64
```

</details>

<details>
<summary><b>IPv6 Address Types</b></summary>

| Type | Prefix | Scope |
|:---|:---|:---|
| Global Unicast | `2000::/3` | Internet-routable |
| Link-Local | `FE80::/10` | Single link only — never routed |
| Loopback | `::1/128` | Local device only |
| Multicast | `FF00::/8` | Group delivery |
| Unspecified | `::` | Not assigned yet |

</details>

<details>
<summary><b>Why IPv6 Doesn't Need ARP</b></summary>

IPv4 uses ARP (broadcast-based) to resolve MAC addresses.  
IPv6 replaces it with **Neighbor Discovery Protocol (NDP)** using ICMPv6 multicast:

- **NS (Neighbor Solicitation)** → Replaces ARP Request
- **NA (Neighbor Advertisement)** → Replaces ARP Reply
- **RS (Router Solicitation)** → Client asks for RA
- **RA (Router Advertisement)** → Router sends prefix + flags

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | IPv6 topology with multiple segments |

---

## 🏆 Skills Demonstrated

- ✅ Enabling IPv6 routing and configuring Global Unicast and Link-Local addresses
- ✅ Configuring SLAAC, Stateless DHCPv6, and Stateful DHCPv6
- ✅ Understanding and setting RA flags (M, O, A) to control client behavior
- ✅ Verifying IPv6 address assignment methods on Cisco IOS
- ✅ Understanding EUI-64 interface ID generation

---

<div align="center">

**[⬆ Back to Top](#-slaac--dhcpv6--ipv6-address-assignment-lab)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
