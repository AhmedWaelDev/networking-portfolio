<div align="center">

# 🔗 EtherChannel — Link Aggregation Lab

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Link%20Aggregation-purple?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Bundle multiple physical switch links into one logical high-bandwidth channel — eliminating STP blocked ports and increasing redundancy.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Topology](#-network-topology)
- [Technologies Used](#-technologies-used)
- [Negotiation Modes](#-negotiation-modes)
- [Lab Steps](#-lab-steps)
- [Verification](#-verification)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This lab demonstrates how to configure **EtherChannel** on Cisco switches to bundle multiple physical links into a single logical link. EtherChannel increases bandwidth, provides link redundancy, and removes STP blocked ports — all bundled interfaces carry traffic simultaneously.

---

## 🗺️ Network Topology


> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|:---|:---|
| EtherChannel | Logical link bundling |
| LACP (IEEE 802.3ad) | Open standard negotiation protocol |
| PAgP (Cisco proprietary) | Cisco-only negotiation protocol |
| Port-Channel Interface | Logical interface representing the bundle |
| STP (Spanning Tree) | EtherChannel removes blocked ports |
| Trunk Port | Carries multiple VLANs over Port-Channel |

---

## ⚙️ Negotiation Modes

| Mode | Protocol | Behavior |
|:---|:---:|:---|
| `active` | LACP | Actively initiates LACP negotiation |
| `passive` | LACP | Waits — responds to LACP only |
| `desirable` | PAgP | Actively initiates PAgP negotiation |
| `auto` | PAgP | Waits — responds to PAgP only |
| `on` | None | Forces channel with no negotiation |

> ✅ **Compatible combinations:** `active/active`, `active/passive`, `desirable/desirable`, `desirable/auto`

---

## 📝 Lab Steps

### Step 1 — Verify Physical Links

```cisco
SW1# show interfaces status
SW1# show cdp neighbors
```

### Step 2 — Ensure Consistent Interface Settings

> ⚠️ All interfaces in the bundle **must match** — same speed, duplex, VLAN config.

```cisco
SW1(config)# interface range FastEthernet0/1 - 2
SW1(config-if-range)# duplex full
SW1(config-if-range)# speed 100
```

### Step 3 — Configure EtherChannel with LACP

**On Switch 1 (Active):**
```cisco
SW1(config)# interface range FastEthernet0/1 - 2
SW1(config-if-range)# channel-group 1 mode active
```

**On Switch 2 (Passive):**
```cisco
SW2(config)# interface range FastEthernet0/1 - 2
SW2(config-if-range)# channel-group 1 mode passive
```

### Step 4 — Configure the Port-Channel as Trunk

```cisco
SW1(config)# interface port-channel 1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan all
```

*Repeat on SW2.*

### Step 5 — Verify EtherChannel

```cisco
SW1# show etherchannel summary
SW1# show interfaces port-channel 1
SW1# show spanning-tree
```

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show etherchannel summary` | Overall status — look for `SU` flags |
| `show etherchannel detail` | Full per-channel information |
| `show interfaces port-channel 1` | Bandwidth and logical interface status |
| `show spanning-tree` | Confirm STP treats bundle as one port |
| `show lacp neighbor` | LACP negotiation partner details |

### Understanding Output Flags

```
Flags:  S - Layer2    U - In use    P - Bundled in port-channel
        I - Stand-alone  D - Down   s - Suspended
```

| Flag | Meaning |
|:---|:---|
| `SU` | ✅ Layer 2, in use — healthy |
| `P` | ✅ Port is bundled |
| `D` | ❌ Down — check physical connection |
| `I` | ❌ Individual — not negotiating |

---

## 💡 Key Concepts

<details>
<summary><b>Why EtherChannel? — The STP Problem</b></summary>

Without EtherChannel, if you connect two switches with two parallel links:
- STP sees a loop and **blocks one link entirely**
- You lose 50% of your bandwidth
- The blocked port only activates if the active one fails (slow convergence)

With EtherChannel:
- STP sees **one logical link** — no loop detected
- **Both physical links carry traffic** simultaneously
- Built-in redundancy — if one link fails, the other carries all traffic instantly

</details>

<details>
<summary><b>LACP vs PAgP</b></summary>

| Feature | LACP | PAgP |
|:---|:---:|:---:|
| Standard | IEEE 802.3ad (Open) | Cisco Proprietary |
| Interoperability | Any vendor | Cisco only |
| Recommended | ✅ Yes | Only in all-Cisco environments |
| Max ports per bundle | 8 active + 8 standby | 8 |

</details>

<details>
<summary><b>EtherChannel Consistency Requirements</b></summary>

All member interfaces **must be identical**:
- Same speed and duplex
- Same VLAN assignments (access) or allowed VLANs (trunk)
- Same trunking mode (access/trunk)
- Same STP settings

If any mismatch exists, EtherChannel will fail to form (`I` — Individual state).

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | Multi-link switch topology |


---

## 🏆 Skills Demonstrated

- ✅ Configuring EtherChannel using LACP (active/passive)
- ✅ Configuring EtherChannel using PAgP (desirable/auto)
- ✅ Setting up Port-Channel trunk interfaces
- ✅ Verifying EtherChannel formation and STP interaction
- ✅ Troubleshooting EtherChannel with `show etherchannel summary`

---

<div align="center">

**[⬆ Back to Top](#-etherchannel--link-aggregation-lab)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
