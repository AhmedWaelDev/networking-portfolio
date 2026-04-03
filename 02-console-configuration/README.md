<div align="center">

# 🖥️ Device Configuration via Console Port

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-00ADEF?style=for-the-badge)
![Topic](https://img.shields.io/badge/Topic-Device%20Management-orange?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)

> Perform initial Cisco device configuration using a console cable — the foundation of all network device setup.

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

This lab demonstrates how to perform **initial device configuration** on Cisco routers and switches using a **console cable connection**. The console port provides out-of-band management — working even when the device has no IP address or the network is down.

---

## 🗺️ Network Topology

```
[PC]──[Console Cable]──[Router / Switch]
  RS-232                Console Port
  (Blue cable in Packet Tracer)
```

> 📸 See [`screenshots/topology.png`](screenshots/topology.png) for the full Packet Tracer topology.

---

## 🛠️ Technologies Used

| Technology | Purpose |
|:---|:---|
| Console Port (RS-232/RJ-45) | Physical local management access |
| Cisco IOS CLI | Command line configuration |
| Terminal Emulation | PC Terminal app in Packet Tracer |
| Global Configuration Mode | Making persistent changes |
| NVRAM | Storing startup configuration |

---

## 📝 Lab Steps

### Step 1 — Connect via Console Cable

In Packet Tracer:
1. Select the **Console (light blue)** cable
2. Connect **PC RS-232 port** → **Router/Switch Console port**

### Step 2 — Open Terminal on the PC

```
PC → Desktop Tab → Terminal → OK (default settings)
Baud: 9600 | Data bits: 8 | Parity: None | Stop bits: 1
```

### Step 3 — Enter Privileged Mode & Set Hostname

```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
R1(config)#
```

### Step 4 — Secure Privileged EXEC Mode

```cisco
R1(config)# enable secret Str0ngP@ss
```

> 🔐 Always use `enable secret` (MD5 hash) — **never** use `enable password` (plaintext).

### Step 5 — Secure the Console Line

```cisco
R1(config)# line console 0
R1(config-line)# password Cons0leP@ss
R1(config-line)# login
R1(config-line)# exec-timeout 5 0
R1(config-line)# exit
```

### Step 6 — Secure VTY Lines

```cisco
R1(config)# line vty 0 4
R1(config-line)# password VtyP@ss
R1(config-line)# login
R1(config-line)# transport input ssh
R1(config-line)# exit
```

### Step 7 — Encrypt All Passwords

```cisco
R1(config)# service password-encryption
```

### Step 8 — Configure Login Banner (MOTD)

```cisco
R1(config)# banner motd #
WARNING: Unauthorized access is prohibited. Disconnect immediately.
#
```

### Step 9 — Assign Management IP (Switches only)

```cisco
Switch(config)# interface vlan 1
Switch(config-if)# ip address 192.168.1.2 255.255.255.0
Switch(config-if)# no shutdown
Switch(config)# ip default-gateway 192.168.1.1
```

### Step 10 — Save Configuration ⚠️

```cisco
R1# copy running-config startup-config
```

> ⚠️ **Critical:** If you don't save, all configuration is lost on reboot!

---

## ✅ Verification

| Command | Purpose |
|:---|:---|
| `show running-config` | View current active configuration |
| `show startup-config` | View saved config in NVRAM |
| `show version` | IOS version and device hardware info |
| `show users` | Active console and VTY sessions |

---

## 💡 Key Concepts

<details>
<summary><b>IOS CLI Modes</b></summary>

```
Router>          ← User EXEC Mode       (view only)
Router#          ← Privileged EXEC Mode (full view + commands)
Router(config)#  ← Global Config Mode   (make changes)
Router(config-if)#  ← Interface Config Mode
Router(config-line)# ← Line Config Mode
```

Move between modes:
```cisco
Router> enable          → enters Privileged EXEC
Router# configure terminal → enters Global Config
Router(config)# exit    → goes back one level
Router(config)# end     → jumps back to Privileged EXEC
```

</details>

<details>
<summary><b>Console vs VTY Access</b></summary>

| Feature | Console | VTY (Telnet/SSH) |
|:---|:---:|:---:|
| Connection Type | Physical cable | Network (remote) |
| Works without IP? | ✅ Yes | ❌ No |
| Used for initial setup | ✅ Yes | ❌ No |
| Supports SSH? | N/A | ✅ Yes |
| Out-of-band management | ✅ Yes | ❌ No |

</details>

<details>
<summary><b>Password Types in Cisco IOS</b></summary>

| Command | Storage | Security |
|:---|:---|:---|
| `enable password` | Plaintext in config | ❌ Weak |
| `enable secret` | MD5 hash | ✅ Strong |
| `service password-encryption` | Type 7 cipher | ⚠️ Weak but hides plaintext |

**Rule:** Always use `enable secret`. If both are set, `enable secret` takes precedence.

</details>

---

## 📸 Screenshots

| Screenshot | Description |
|:---|:---|
| [`topology.png`](screenshots/topology.png) | Console cable connection topology |
| [`terminal-session.png`](screenshots/terminal-session.png) | Terminal window open on PC |
| [`hostname-config.png`](screenshots/hostname-config.png) | Hostname and password configuration |
| [`show-run.png`](screenshots/show-run.png) | Final running configuration output |

---

## 🏆 Skills Demonstrated

- ✅ Connecting to Cisco devices using a console rollover cable
- ✅ Navigating all Cisco IOS CLI modes
- ✅ Configuring hostnames, passwords, and login banners
- ✅ Setting console and VTY line security
- ✅ Saving and verifying device configuration

---

<div align="center">

**[⬆ Back to Top](#%EF%B8%8F-device-configuration-via-console-port)** &nbsp;|&nbsp; **[🏠 Back to Main Portfolio](../README.md)**

</div>
