<div align="center">

# 🔄 HSRP — Hot Standby Router Protocol Lab

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Gateway%20Redundancy-blue?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Configure HSRP so that two routers share a virtual gateway IP — providing seamless failover if the primary router goes down.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Topology](#-network-topology)
- [Technologies Used](#-technologies-used)
- [HSRP Roles](#-hsrp-roles)
- [Lab Steps](#-lab-steps)
- [Verification](#-verification)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This lab demonstrates **HSRP (Hot Standby Router Protocol)** — a Cisco First Hop Redundancy Protocol (FHRP) that provides **automatic default gateway failover**. Two routers share a virtual IP address; if the active router fails, the standby takes over instantly with no reconfiguration needed on end devices.

---

## 🗺️ Network Topology



> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|:---|:---|
| HSRP v2 | Gateway redundancy protocol |
| Virtual IP Address | Shared gateway that never changes |
| Virtual MAC Address | Shared MAC for the virtual gateway |
| HSRP Priority | Determines which router becomes Active |
| HSRP Preemption | Allows recovery of Active role after a failure |
| Interface Tracking | Triggers failover if uplink fails |

---

## 👥 HSRP Roles

| Role | Description |
|:---|:---|
| 🟢 **Active Router** | Forwards all traffic for the virtual IP |
| 🟡 **Standby Router** | Monitors Active router — takes over if it fails |
| 🔵 **Virtual Router** | The logical gateway devices use (Virtual IP + Virtual MAC) |

---

## 📝 Lab Steps

### Step 1 — Assign Real IPs on Both Routers

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip address 192.168.1.2 255.255.255.0
R1(config-if)# no shutdown

R2(config)# interface GigabitEthernet0/0
R2(config-if)# ip address 192.168.1.3 255.255.255.0
R2(config-if)# no shutdown
```

### Step 2 — Configure HSRP on R1 (Active)

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# standby version 2
R1(config-if)# standby 1 ip 192.168.1.1
R1(config-if)# standby 1 priority 110
R1(config-if)# standby 1 preempt
```

### Step 3 — Configure HSRP on R2 (Standby)

```cisco
R2(config)# interface GigabitEthernet0/0
R2(config-if)# standby version 2
R2(config-if)# standby 1 ip 192.168.1.1
R2(config-if)# standby 1 priority 100
R2(config-if)# standby 1 preempt
```

### Step 4 — Set Default Gateway on End Devices

> ✅ Set all PCs to use **192.168.1.1** (Virtual IP) — **not** R1 or R2's real IP.

### Step 5 — Configure Interface Tracking

If R1's uplink fails, automatically lower its priority so R2 takes over:

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# standby 1 track GigabitEthernet0/1 30
```

> When `Gi0/1` on R1 goes down → R1 priority drops from 110 to **80** → R2 (priority 100) becomes Active.

### Step 6 — Test Failover

```cisco
! On R1 — simulate failure:
R1(config)# interface GigabitEthernet0/0
R1(config-if)# shutdown

! Watch: pings from PC to external IP pause briefly, then resume via R2

! Restore R1:
R1(config-if)# no shutdown

! R1 reclaims Active role (because preempt is set)
```

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show standby` | Full HSRP details — state, virtual IP, timers |
| `show standby brief` | One-line summary per HSRP group |
| `show ip arp` | Confirm virtual MAC in ARP table |
| `debug standby` | Live HSRP state transition messages |

### Sample `show standby brief` Output

```
                     P indicates configured to preempt.
                     |
Interface   Grp  Pri P State    Active          Standby         Virtual IP
Gi0/0       1    110 P Active   local           192.168.1.3     192.168.1.1
```

---

## 💡 Key Concepts

<details>
<summary><b>HSRP Timers</b></summary>

| Timer | Default | Purpose |
|:---|:---:|:---|
| Hello | 3 sec | How often the Active router announces itself |
| Hold | 10 sec | How long Standby waits before taking over |

Speed up failover with:
```cisco
R1(config-if)# standby 1 timers 1 3
```

</details>

<details>
<summary><b>HSRP vs VRRP vs GLBP</b></summary>

| Feature | HSRP | VRRP | GLBP |
|:---|:---:|:---:|:---:|
| Standard | Cisco only | Open (RFC 5798) | Cisco only |
| Load Balancing | ❌ | ❌ | ✅ |
| Multiple Active Gateways | ❌ | ❌ | ✅ |
| Preemption | Configurable | On by default | Configurable |

</details>

<details>
<summary><b>Why Interface Tracking Matters</b></summary>

Without tracking:
- R1's LAN interface is UP → HSRP stays Active
- But R1's uplink (to ISP) is DOWN → traffic is blackholed!

With tracking:
- If R1's uplink fails → priority drops below R2's → R2 becomes Active with a working uplink

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | HSRP dual-router topology |


---

## 🏆 Skills Demonstrated

- ✅ Configuring HSRP groups with virtual IP on Cisco routers
- ✅ Tuning HSRP priority and enabling preemption
- ✅ Testing and verifying gateway failover behavior
- ✅ Configuring HSRP interface tracking for uplink-aware failover
- ✅ Understanding HSRP timers and failover convergence

---

<div align="center">

**[⬆ Back to Top](#-hsrp--hot-standby-router-protocol-lab)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
