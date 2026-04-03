<div align="center">

# 📡 WLC — Wireless LAN Controller Lab

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Wireless%20Networking-teal?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Configure a Cisco Wireless LAN Controller to centrally manage Lightweight Access Points using CAPWAP — the enterprise standard for Wi-Fi deployment.

</div>

---

## 📋 Table of Contents
- [Objective](#-objective)
- [Network Topology](#-network-topology)
- [Technologies Used](#-technologies-used)
- [Architecture Overview](#-architecture-overview)
- [Lab Steps](#-lab-steps)
- [WLC GUI Reference](#-wlc-gui-reference)
- [Verification](#-verification)
- [Key Concepts](#-key-concepts)
- [Screenshots](#-screenshots)
- [Skills Demonstrated](#-skills-demonstrated)

---

## 🎯 Objective

This lab demonstrates how to configure a **Cisco Wireless LAN Controller (WLC)** to manage **Lightweight Access Points (LAPs)** using the **CAPWAP** protocol. Centralized wireless management handles authentication, roaming, RF tuning, and SSID configuration for all APs from a single dashboard.

---

## 🗺️ Network Topology



> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|:---|:---|
| Cisco WLC | Central wireless management controller |
| Lightweight AP (LAP) | Access point with no local config — managed by WLC |
| CAPWAP | Tunnel protocol between LAP and WLC |
| DHCP | IP assignment for APs and wireless clients |
| WPA2 / PSK | Wireless security (encryption + authentication) |
| SSID / WLAN | Wireless network name and policy profile |
| Management VLAN | Dedicated VLAN for WLC and AP management |

---

## 🏗️ Architecture Overview

### Autonomous AP vs Lightweight AP

| Feature | Autonomous AP | Lightweight AP + WLC |
|:---|:---:|:---:|
| Configuration | Per device (CLI/SSH) | Centralized on WLC |
| Management | Individual login per AP | Single WLC dashboard |
| Seamless Roaming | ❌ | ✅ WLC manages sessions |
| Firmware Updates | Per AP manually | Push from WLC to all APs |
| Scalability | Small networks | Enterprise (100s of APs) |
| Config backup | Manual | Automatic via WLC |

### CAPWAP Tunnel Architecture

```
Control Plane:  LAP ←──UDP 5246──→ WLC  (management, config, firmware)
Data Plane:     LAP ←──UDP 5247──→ WLC  (client traffic encapsulated)
```

**AP Boot Sequence:**
1. AP gets IP via DHCP
2. AP discovers WLC (via DHCP option 43, DNS, or broadcast)
3. AP establishes CAPWAP control tunnel
4. AP downloads its configuration from WLC
5. AP begins serving clients — **no local config stored**

---

## 📝 Lab Steps

### Step 1 — Configure DHCP for AP Management

```cisco
Router(config)# ip dhcp pool WIRELESS-MGMT
Router(config-dhcp)# network 192.168.100.0 255.255.255.0
Router(config-dhcp)# default-router 192.168.100.1
Router(config-dhcp)# dns-server 8.8.8.8
Router(config-dhcp)# exit
Router(config)# ip dhcp excluded-address 192.168.100.1 192.168.100.20
```

### Step 2 — Configure Trunk Port to WLC

```cisco
SW1(config)# interface GigabitEthernet0/1
SW1(config-if)# switchport mode trunk
SW1(config-if)# switchport trunk allowed vlan 10,30,99,100
```

### Step 3 — Access the WLC Web GUI

```
Browser → https://192.168.100.X
Username: admin
Password: Cisco1234  (Packet Tracer default — change in production!)
```

### Step 4 — Create a WLAN (SSID)

In the WLC GUI:
1. **WLANs** → **Create New** → **Go**
2. Set **Profile Name**: `Corp-WiFi`
3. Set **SSID**: `Corp-WiFi`
4. Set **Status**: Enabled
5. **Security tab** → Layer 2: `WPA+WPA2`
6. **PSK**: Enter your passphrase
7. Click **Apply**

### Step 5 — Verify AP Registration

**WLC GUI:** Wireless → Access Points

| Column | Expected Value |
|:---|:---|
| AP Name | LAP-1, LAP-2... |
| Status | **Registered** ✅ |
| IP Address | From DHCP pool |
| Controller | Your WLC IP |

### Step 6 — Connect a Wireless Client

In Packet Tracer:
1. Click PC/Laptop → **PC Wireless** tab
2. Select your SSID from the list
3. Enter PSK passphrase
4. Confirm the device receives an IP via DHCP

### Step 7 — Verify End-to-End Connectivity

```
PC> ping 192.168.100.1     ← Gateway ping
PC> ping 192.168.1.10      ← Wired network ping
```

---

## 🖥️ WLC GUI Reference

| Section | What You Can Do |
|:---|:---|
| **Monitor** | Dashboard: AP count, client count, RF utilization |
| **WLANs** | Create/edit SSIDs, security policies, QoS |
| **Wireless** | AP inventory, radio settings, RF groups |
| **Security** | AAA, RADIUS integration, ACLs |
| **Management** | WLC system settings, NTP, SNMP, upgrades |
| **Controller** | Physical ports, interfaces, VLANs |

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show ip dhcp binding` | Confirm APs and clients received IPs |
| `show cdp neighbors` | Verify WLC visible on switch |
| `show interfaces trunk` | Trunk carrying wireless VLANs |
| WLC GUI → Monitor | AP status, client associations, signal strength |

---

## 💡 Key Concepts

<details>
<summary><b>How CAPWAP Discovery Works</b></summary>

When a LAP boots, it finds the WLC using this discovery sequence (in order):

1. **DHCP Option 43** — DHCP server provides WLC IP in option 43
2. **DNS** — Lookup for `CISCO-CAPWAP-CONTROLLER.local`
3. **Broadcast** — Sends CAPWAP Discovery Request to 255.255.255.255
4. **Previously known WLC** — Stored in AP memory from last connection
5. **Manual** — Administrator configures WLC IP on AP via CLI

</details>

<details>
<summary><b>WPA2 vs WPA3</b></summary>

| Feature | WPA2 | WPA3 |
|:---|:---:|:---:|
| Encryption | AES-CCMP | AES-GCMP-256 |
| Handshake | 4-way | SAE (Simultaneous Authentication of Equals) |
| Brute force resistance | Moderate | Strong (forward secrecy) |
| Open network security | ❌ | ✅ OWE (Opportunistic Wireless Encryption) |
| CCNA exam focus | ✅ Primary | Awareness only |

</details>

<details>
<summary><b>Split MAC Architecture Explained</b></summary>

In traditional autonomous APs, **all 802.11 functions run on the AP itself**.

With WLC + LAP, MAC functions are split:

| Function | Handled By |
|:---|:---|
| Beacons, probe responses | LAP (real-time, must be local) |
| Authentication (802.1X) | WLC |
| Association, reassociation | WLC |
| Encryption/decryption | LAP |
| Roaming decisions | WLC |
| Configuration management | WLC |

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | WLC + LAP + switch topology |


---

## 🏆 Skills Demonstrated

- ✅ Understanding WLC + Lightweight AP centralized architecture
- ✅ Configuring WLANs and SSIDs with WPA2 on Cisco WLC GUI
- ✅ Verifying AP registration and client association
- ✅ Configuring DHCP and VLANs to support wireless infrastructure
- ✅ Understanding CAPWAP control and data tunnel operation

---

<div align="center">

**[⬆ Back to Top](#-wlc--wireless-lan-controller-lab)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
