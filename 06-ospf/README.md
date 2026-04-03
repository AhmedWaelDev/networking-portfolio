<div align="center">

# 🌐 OSPF — Open Shortest Path First Routing Lab

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Dynamic%20Routing-blue?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Configure OSPFv2 in a multi-router topology — covering adjacency formation, DR/BDR election, cost tuning, and default route redistribution.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Topology](#-network-topology)
- [Technologies Used](#-technologies-used)
- [Activity 1 — Basic OSPF](#activity-1--basic-ospf-configuration)
- [Activity 2 — Advanced OSPF](#activity-2--advanced-ospf-tuning)
- [Verification](#-verification)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This lab covers **OSPFv2 (Open Shortest Path First)** — a link-state dynamic routing protocol. Two activity files are included: Activity 1 covers basic OSPF setup and neighbor adjacency, while Activity 2 covers DR/BDR election, cost tuning, and default route redistribution.

---

## 🗺️ Network Topology



> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|:---|:---|
| OSPFv2 | Dynamic IPv4 link-state routing |
| OSPF Area 0 | Backbone area — all routers |
| Router ID | Unique identifier per OSPF router |
| DR / BDR Election | Reduces OSPF traffic on broadcast segments |
| OSPF Cost | Metric for path selection |
| Passive Interface | Stops OSPF hellos on LAN-facing ports |
| Default Route Redistribution | Advertise default route to all OSPF routers |

---

## Activity 1 — Basic OSPF Configuration

### Step 1 — Configure IP Addressing

Assign IPs to all router interfaces per the addressing table before enabling OSPF.

### Step 2 — Enable OSPF and Set Router ID

```cisco
R1(config)# router ospf 1
R1(config-router)# router-id 1.1.1.1
```

> 💡 Always set the Router ID explicitly. Without it, OSPF picks the highest IP — which can cause issues if interfaces change.

### Step 3 — Advertise Networks

```cisco
R1(config-router)# network 192.168.1.0 0.0.0.255 area 0
R1(config-router)# network 10.0.0.0 0.0.0.3 area 0
R1(config-router)# network 10.0.0.4 0.0.0.3 area 0
```

**Wildcard Mask Quick Reference:**

| Subnet | Wildcard |
|:---|:---|
| /24 | 0.0.0.255 |
| /30 | 0.0.0.3 |
| /25 | 0.0.0.127 |

### Step 4 — Configure Passive Interfaces

Prevent OSPF hellos from being sent to end devices:

```cisco
R1(config-router)# passive-interface GigabitEthernet0/0
```

### Step 5 — Verify Neighbor Adjacency

```cisco
R1# show ip ospf neighbor
```

> ✅ Look for state **`FULL`** — this means OSPF adjacency is fully established.

### Step 6 — Verify OSPF Routes

```cisco
R1# show ip route ospf
```

> OSPF routes appear with `O` prefix. Connected networks are `C`.

---

## Activity 2 — Advanced OSPF Tuning

### Step 1 — Inspect the OSPF Database

```cisco
R1# show ip ospf database
```

> All routers in the same area should have an **identical** LSDB (Link State Database).

### Step 2 — Check DR/BDR Election

```cisco
R1# show ip ospf interface GigabitEthernet0/0
```

### Step 3 — Influence DR/BDR Election

```cisco
R1(config)# interface GigabitEthernet0/0
R1(config-if)# ip ospf priority 200
```

> Priority 0 = never becomes DR or BDR. Highest priority wins.

### Step 4 — Tune OSPF Cost

```cisco
R1(config)# interface Serial0/0/0
R1(config-if)# ip ospf cost 50
```

Formula: **Cost = 100,000,000 / Bandwidth (bps)**

| Interface | Default Bandwidth | Default Cost |
|:---|:---:|:---:|
| Gigabit Ethernet | 1 Gbps | 1 |
| Fast Ethernet | 100 Mbps | 1 |
| Serial | 1.544 Mbps | 64 |

### Step 5 — Redistribute Default Route

Advertise a default route from the edge router to all OSPF neighbors:

```cisco
R_Edge(config)# ip route 0.0.0.0 0.0.0.0 [next-hop]
R_Edge(config)# router ospf 1
R_Edge(config-router)# default-information originate
```

> Other routers receive an `O*E2` route — the external default route.

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show ip ospf neighbor` | Neighbor list and adjacency state |
| `show ip ospf database` | Full LSDB — should be identical on all routers |
| `show ip route ospf` | OSPF-learned routes (`O` prefix) |
| `show ip ospf interface` | Cost, DR/BDR, hello/dead timers per interface |
| `show ip ospf` | OSPF process summary — Router ID, areas |
| `debug ip ospf adj` | Live adjacency formation troubleshooting |

---

## 💡 Key Concepts

<details>
<summary><b>OSPF Neighbor State Machine</b></summary>

```
Down → Init → 2-Way → ExStart → Exchange → Loading → Full
```

| State | Meaning |
|:---|:---|
| `2-Way` | Normal between non-DR/BDR routers on broadcast segments |
| `Full` | ✅ Full adjacency — LSDBs are synchronized |
| `Loading` | Still exchanging LSAs — transitional |
| `Down` | ❌ No hellos received |

</details>

<details>
<summary><b>DR/BDR Election Rules</b></summary>

On multi-access networks (Ethernet), OSPF elects:
- **DR (Designated Router)** — receives and floods all LSAs
- **BDR (Backup DR)** — monitors DR, takes over if DR fails

Election order:
1. Highest OSPF **Priority** wins (default = 1)
2. Tie → Highest **Router ID** wins
3. Priority **0** = never elected

> ⚠️ DR/BDR election is **non-preemptive** — changing priority has no effect until the election resets.

</details>

<details>
<summary><b>OSPF vs RIP vs EIGRP</b></summary>

| Feature | RIP | OSPF | EIGRP |
|:---|:---:|:---:|:---:|
| Type | Distance Vector | Link State | Advanced Distance Vector |
| Metric | Hop count | Cost (bandwidth) | Composite (BW + delay) |
| Convergence | Slow | Fast | Very Fast |
| Scalability | Small only | Large enterprise | Large enterprise |
| Standard | Open | Open | Cisco only |
| Max hops | 15 | Unlimited | Unlimited |

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | Multi-router OSPF topology |

---

## 🏆 Skills Demonstrated

- ✅ Configuring OSPFv2 in a multi-router topology
- ✅ Setting OSPF Router IDs and advertising networks with wildcard masks
- ✅ Verifying adjacency formation and the OSPF LSDB
- ✅ Tuning DR/BDR election priority and OSPF interface cost
- ✅ Redistributing a default route into OSPF

---

<div align="center">

**[⬆ Back to Top](#-ospf--open-shortest-path-first-routing-lab)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
