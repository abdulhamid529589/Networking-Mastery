# 🌐 Types of Computer Networks

### Cybersecurity Student Notes | Networking Course — Lecture 02

> **Source:** Hindi lecture transcript — Types of Computer Networks
> **Exam Relevance:** GATE · University Exams · Interviews · CompTIA ITF+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Overview — Why Networks are Classified](#1-overview--why-networks-are-classified)
2. [PAN — Personal Area Network](#2-pan--personal-area-network)
3. [LAN — Local Area Network](#3-lan--local-area-network)
4. [CAN — Campus Area Network](#4-can--campus-area-network)
5. [MAN — Metropolitan Area Network](#5-man--metropolitan-area-network)
6. [WAN — Wide Area Network](#6-wan--wide-area-network)
7. [Master Comparison Table](#7-master-comparison-table)
8. [Visual Range Diagram](#8-visual-range-diagram)
9. [🔴 Attack Surface by Network Type](#9--attack-surface-by-network-type)
10. [🧪 Practical Labs — Test in Your Setup](#10--practical-labs--test-in-your-setup)
11. [Exam & Interview Cheat Sheet](#11-exam--interview-cheat-sheet)

---

## 1. Overview — Why Networks are Classified

The **primary basis of classification** is **distance (range)**. As distance increases:

```
Distance ↑  →  Hardware complexity ↑
               Cost ↑
               Maintenance difficulty ↑
               Error/noise chances ↑
               Transmission speed ↓ (comparatively)
```

### The 5 Types (Small → Large)

```
PAN ──► LAN ──► CAN ──► MAN ──► WAN
 10m    1km    5km    50km   Unlimited
```

| Abbreviation | Full Form                 |
| ------------ | ------------------------- |
| **PAN**      | Personal Area Network     |
| **LAN**      | Local Area Network        |
| **CAN**      | Campus Area Network       |
| **MAN**      | Metropolitan Area Network |
| **WAN**      | Wide Area Network         |

---

## 2. PAN — Personal Area Network

### What is it?

The **smallest** type of network. Connects devices around a **single person** — within a room or personal space.

```
[Your Phone] ←──Bluetooth──► [Your Laptop]
[Your Laptop] ←──Bluetooth──► [Your Smartwatch]
[Your Phone]  ←──Hotspot──►  [Your Tablet]
```

### Key Parameters

| Parameter              | Value / Detail                                                |
| ---------------------- | ------------------------------------------------------------- |
| **Range**              | Up to **10 meters** (some say up to 100m for USB/IR variants) |
| **Technology**         | Bluetooth, USB, Infrared (IR), Hotspot (Wi-Fi Direct)         |
| **Transmission Speed** | High (within short range, signal is clean)                    |
| **Ownership**          | **Private** — you own and operate it yourself                 |
| **Maintenance**        | **Very Easy** — built-in hardware, no extra setup             |
| **Cost**               | **Very Low** — no routers/extra hardware needed               |
| **Error Rate**         | **Very Low** — short distance = less noise                    |

### Real-World Examples

- Connecting wireless earbuds to your phone (Bluetooth)
- Sharing your phone hotspot with a friend
- Connecting a wireless mouse/keyboard to your laptop
- Smartwatch syncing with phone

### 🔴 PAN Security Threats

| Attack                      | How                                             | Tool                   |
| --------------------------- | ----------------------------------------------- | ---------------------- |
| **Bluejacking**             | Sending unsolicited messages via Bluetooth      | BlueZ tools            |
| **Bluesnarfing**            | Unauthorized access to phone data via Bluetooth | Bluesnarfer            |
| **MITM via Bluetooth**      | Intercepting Bluetooth pairing                  | btlejack               |
| **Evil Hotspot**            | Fake hotspot impersonates legitimate one        | hostapd                |
| **Bluetooth Eavesdropping** | Sniffing Bluetooth traffic                      | Ubertooth One hardware |

---

## 3. LAN — Local Area Network

### What is it?

Connects devices within a **single building or small area** — your home, school, or office floor.

```
[PC 1] ─┐
[PC 2] ─┼──[Switch]──[Router]──► Internet
[PC 3] ─┘
[Laptop] ──(Wi-Fi)──[Router]──► Internet
```

### Key Parameters

| Parameter              | Value / Detail                                         |
| ---------------------- | ------------------------------------------------------ |
| **Range**              | Up to **1 kilometer** (within a building/campus block) |
| **Technology**         | Ethernet (IEEE 802.3), Wi-Fi (IEEE 802.11)             |
| **Transmission Speed** | **High** — 100 Mbps to 10 Gbps (Ethernet)              |
| **Ownership**          | **Private** — your home, your school, your office      |
| **Maintenance**        | **Easy** — manageable number of devices                |
| **Cost**               | **Low** — basic switch/router needed                   |
| **Error Rate**         | **Low** — short distance, controlled environment       |

### Real-World Examples

- Home Wi-Fi network
- Your VirtualBox **Host-Only Network** (Parrot OS ↔ Metasploitable2)
- A school computer lab
- Office floor network

### Technologies Deep Dive

| Technology | Standard                 | Speed          | Medium                |
| ---------- | ------------------------ | -------------- | --------------------- |
| Ethernet   | IEEE 802.3               | 10Mbps–10Gbps  | UTP/Fiber cable       |
| Wi-Fi      | IEEE 802.11a/b/g/n/ac/ax | 11Mbps–9.6Gbps | Radio waves           |
| Token Ring | IEEE 802.5               | 4–16 Mbps      | (Legacy, mostly dead) |

### 🔴 LAN Security Threats

| Attack                     | Layer | Tool                   | How                          |
| -------------------------- | ----- | ---------------------- | ---------------------------- |
| **ARP Spoofing**           | L2    | arpspoof, Ettercap     | Fake ARP replies → MitM      |
| **MAC Flooding**           | L2    | macof                  | Overflows switch CAM table   |
| **VLAN Hopping**           | L2    | Scapy                  | Bypass VLAN segmentation     |
| **DHCP Starvation**        | L3    | Yersinia               | Exhaust DHCP pool            |
| **Rogue DHCP Server**      | L3    | Yersinia               | Fake DHCP → redirect traffic |
| **Wi-Fi Deauth**           | L2    | aircrack-ng (aireplay) | Disconnect clients from AP   |
| **Evil Twin AP**           | L2    | hostapd-wpe            | Fake Wi-Fi access point      |
| **LLMNR/NBT-NS Poisoning** | L3    | Responder              | Capture NTLMv2 hashes        |

---

## 4. CAN — Campus Area Network

### What is it?

Connects **multiple LANs within a campus** — university campus, large corporate complex, military base. Multiple buildings connected together.

```
[Building A LAN] ─┐
[Building B LAN] ─┼──[Core Switch/Router]──[Campus Network]
[Building C LAN] ─┘
[Library LAN]    ─┘
```

### Key Parameters

| Parameter              | Value / Detail                                                 |
| ---------------------- | -------------------------------------------------------------- |
| **Range**              | **1 km – 5 km** (within a campus boundary)                     |
| **Technology**         | Ethernet, Fiber Optic backbone, Wi-Fi                          |
| **Transmission Speed** | **High to Moderate**                                           |
| **Ownership**          | **Private** — university/company owns it                       |
| **Maintenance**        | **Moderate** — more buildings = more devices = more management |
| **Cost**               | **Moderate** — fiber between buildings costs more              |
| **Error Rate**         | **Moderate**                                                   |

### Real-World Examples

- Stanford University's internal network
- Microsoft's campus network (multiple buildings in Redmond)
- Your university's network (if multiple buildings are connected)
- Hospital complex networks

### 🔴 CAN Security Threats

CAN inherits all LAN threats PLUS:

| Attack                      | Why More Dangerous in CAN                           |
| --------------------------- | --------------------------------------------------- |
| **Lateral Movement**        | Attacker on one building's LAN can pivot to another |
| **Rogue Devices**           | Hard to monitor all physical ports across buildings |
| **Fiber Tap**               | Physical interception of inter-building fiber       |
| **Domain Credential Theft** | Centralized AD/LDAP → single point of failure       |

---

## 5. MAN — Metropolitan Area Network

### What is it?

Covers an **entire city or metropolitan area**. Larger than CAN, smaller than WAN. Can be owned publicly or privately.

```
[Kolkata North Area] ──────┐
[Kolkata South Area] ──────┼──[MAN Backbone]──[City-wide Network]
[Kolkata East Area]  ──────┘
[Airport Network]    ──────┘
```

### Key Parameters

| Parameter              | Value / Detail                                                                           |
| ---------------------- | ---------------------------------------------------------------------------------------- |
| **Range**              | **5 km – 50 km** (city-wide)                                                             |
| **Technology**         | **FDDI** (Fiber Distributed Data Interface), **ATM** (Asynchronous Transfer Mode), WiMAX |
| **Transmission Speed** | **Moderate** (varies by infrastructure)                                                  |
| **Ownership**          | **Public or Private** — govt or private ISP can operate                                  |
| **Maintenance**        | **Difficult** — spread across city, many towers/devices                                  |
| **Cost**               | **High** — city-scale infrastructure                                                     |
| **Error Rate**         | **Moderate to High**                                                                     |

### Technologies Deep Dive

| Technology | Full Form                                       | Note                                     |
| ---------- | ----------------------------------------------- | ---------------------------------------- |
| **FDDI**   | Fiber Distributed Data Interface                | Fiber optic ring topology, 100 Mbps      |
| **ATM**    | Asynchronous Transfer Mode                      | Fixed 53-byte cells, used for voice+data |
| **WiMAX**  | Worldwide Interoperability for Microwave Access | Wireless MAN, IEEE 802.16                |

### Real-World Examples

- City-wide cable TV/internet network (your ISP infrastructure)
- Smart city surveillance networks
- Municipal government networks (e.g., across Chittagong city divisions)
- Metro rail communication networks

### 🔴 MAN Security Threats

| Attack                    | How                                           |
| ------------------------- | --------------------------------------------- |
| **BGP Hijacking**         | Rerouting city-level traffic                  |
| **Tower/Node Compromise** | Physical access to repeater nodes             |
| **ISP-level MitM**        | Interception at ISP routing equipment         |
| **Wiretapping**           | Physical fiber tapping at distribution points |

---

## 6. WAN — Wide Area Network

### What is it?

The **largest** type of network. Connects countries and continents. **The Internet is the world's largest WAN.**

```
[Bangladesh] ──submarine cable──► [USA]
[India]      ──satellite link──►  [UK]
[Japan]      ──fiber link──►      [Germany]
         All connected = Internet (WAN)
```

### Key Parameters

| Parameter              | Value / Detail                                                           |
| ---------------------- | ------------------------------------------------------------------------ |
| **Range**              | **Unlimited** — country to country, worldwide                            |
| **Technology**         | Leased lines, Dial-up, DSL, Satellite, Submarine fiber cables            |
| **Transmission Speed** | **Comparatively Lower** (due to massive distance + hops)                 |
| **Ownership**          | **Mostly Public** — ISPs like Airtel, Jio, GP, Robi operate the backbone |
| **Maintenance**        | **Very Difficult** — global scale, thousands of devices                  |
| **Cost**               | **Very High**                                                            |
| **Error Rate**         | **High** — more distance = more noise, more hops                         |

### WAN Technologies

| Technology          | Description                                      | Era              |
| ------------------- | ------------------------------------------------ | ---------------- |
| **Dial-up**         | Used telephone lines (PSTN), max 56Kbps          | 1990s            |
| **DSL**             | Digital Subscriber Line, uses phone line, faster | 2000s            |
| **Leased Lines**    | Dedicated line from ISP (expensive, reliable)    | Corporate        |
| **MPLS**            | Multi-Protocol Label Switching, enterprise WAN   | Current          |
| **Satellite**       | For remote areas, high latency                   | Current          |
| **Submarine Cable** | Undersea fiber optic between continents          | Current backbone |

### The Internet & WWW Connection

```
WAN = Internet infrastructure (physical)
WWW = World Wide Web = application/content layer on top of Internet
Internet ⊃ WWW (Internet is bigger — includes email, FTP, VoIP etc.)
```

### Historical Context (from lecture)

- **1990s:** Internet concept emerged from telephone networks (PSTN)
- **PCO → Local calls**
- **STD (Subscriber Trunk Dialing) → Inter-state calls** = LAN equivalent
- **ISD (International Subscriber Dialing) → International calls** = WAN equivalent
- Early internet piggy-backed on telephone infrastructure (dial-up modems)

### 🔴 WAN Security Threats

| Attack                       | Description                        | Example                                  |
| ---------------------------- | ---------------------------------- | ---------------------------------------- |
| **BGP Hijacking**            | Corrupting internet routing tables | Pakistan Telecom hijacked YouTube (2008) |
| **DNS Hijacking**            | Redirecting DNS at ISP level       | Nation-state attacks                     |
| **DDoS**                     | Flooding WAN links with traffic    | Mirai botnet                             |
| **Submarine Cable Tap**      | NSA PRISM-style interception       | Physical tap on undersea cables          |
| **ISP Surveillance**         | ISP logs/intercepts your traffic   | Government-ordered wiretapping           |
| **IP Spoofing**              | Forging source IP on WAN packets   | Amplification DDoS                       |
| **Man-in-the-Middle at ISP** | SSL stripping at ISP level         | Requires ISP cooperation                 |

---

## 7. Master Comparison Table

| Parameter       | PAN            | LAN             | CAN             | MAN               | WAN               |
| --------------- | -------------- | --------------- | --------------- | ----------------- | ----------------- |
| **Full Form**   | Personal Area  | Local Area      | Campus Area     | Metropolitan Area | Wide Area         |
| **Range**       | ~10m           | ~1 km           | ~5 km           | ~50 km            | Unlimited         |
| **Example**     | Bluetooth      | Home Wi-Fi      | University      | City network      | Internet          |
| **Technology**  | Bluetooth, USB | Ethernet, Wi-Fi | Ethernet, Fiber | FDDI, ATM, WiMAX  | Leased lines, DSL |
| **Speed**       | High           | High            | High–Moderate   | Moderate          | Comparatively Low |
| **Ownership**   | Private        | Private         | Private         | Public/Private    | Mostly Public     |
| **Maintenance** | Very Easy      | Easy            | Moderate        | Difficult         | Very Difficult    |
| **Cost**        | Very Low       | Low             | Moderate        | High              | Very High         |
| **Error Rate**  | Very Low       | Low             | Moderate        | Moderate–High     | High              |

---

## 8. Visual Range Diagram

```
  [You]
    │
    │◄── 10m ──►  PAN  (Bluetooth, Hotspot)
    │
    │◄──────── 1km ────────►  LAN  (Home/Office Wi-Fi, Ethernet)
    │
    │◄──────────────── 5km ──────────────────►  CAN  (University Campus)
    │
    │◄──────────────────────────── 50km ──────────────────────────────►  MAN  (City)
    │
    │◄──────────────────────────────── UNLIMITED ──────────────────────────────────►  WAN  (Internet)
```

---

## 9. 🔴 Attack Surface by Network Type

### PAN Attacks — Your Lab

```bash
# Scan for Bluetooth devices (on Parrot OS)
hcitool scan
hcitool inq

# Get device info
hcitool info <BD_ADDR>

# Bluetooth vulnerability scanner
sudo apt install bluez
bluetoothctl
  scan on
  devices
```

### LAN Attacks — VirtualBox Lab (Metasploitable2)

```bash
# ARP Spoofing — MitM in your LAN
sudo arpspoof -i eth0 -t 192.168.56.101 192.168.56.1

# MAC Flooding — exhaust switch CAM table
sudo macof -i eth0

# DHCP Starvation with Yersinia
sudo yersinia -G   # GUI mode

# Rogue DHCP — must disable yours first
# (Lab only — never on real network)
```

### Wi-Fi LAN Attacks (Monitor Mode)

```bash
# Enable monitor mode on wireless adapter
sudo airmon-ng start wlan0

# Scan for networks
sudo airodump-ng wlan0mon

# Deauth attack (disconnect clients) — on your own test AP only
sudo aireplay-ng --deauth 10 -a <AP_MAC> wlan0mon

# WPA handshake capture
sudo airodump-ng -c <channel> --bssid <AP_MAC> -w capture wlan0mon

# Crack WPA (wordlist attack) — your own test AP only
sudo aircrack-ng -w /usr/share/wordlists/rockyou.txt capture-01.cap
```

### MAN/WAN Level — Understanding Tools

```bash
# Trace the route your packets take across WAN hops
traceroute google.com
traceroute -T -p 443 google.com   # TCP traceroute

# DNS query path
dig google.com +trace

# BGP route lookup (educational)
whois -h whois.radb.net -- '-i origin AS<number>'
```

---

## 10. 🧪 Practical Labs — Test in Your Setup

### Lab 1 — Identify Your Network Type

```bash
# On Parrot OS — see your network interfaces
ip addr show
ip route show

# eth0 / enp0s3 → Your LAN interface (VirtualBox NAT or Host-Only)
# wlan0          → Your wireless LAN interface

# Check which type of network you're on
# VirtualBox Host-Only = LAN (isolated private network)
# VirtualBox NAT = Simulated WAN (goes through your host to internet)
```

### Lab 2 — Simulate PAN with Hotspot Attack

```bash
# Create a fake access point (rogue AP) on your Parrot OS
# This simulates PAN/LAN-level evil twin attack

sudo apt install hostapd dnsmasq

# hostapd config (evil_twin.conf)
cat << EOF > /tmp/evil_twin.conf
interface=wlan0
driver=nl80211
ssid=FreeWiFi_Test
channel=6
EOF

# dnsmasq config — fake DHCP
cat << EOF > /tmp/dnsmasq.conf
interface=wlan0
dhcp-range=10.0.0.10,10.0.0.50,255.255.255.0,12h
EOF

# Run the fake AP (your own test environment only)
sudo hostapd /tmp/evil_twin.conf &
sudo dnsmasq -C /tmp/dnsmasq.conf
```

### Lab 3 — Your MERN/PERN App — Network Type Context

Your web app when running locally:

```
localhost:3000     → IPC (same machine, not networking)
192.168.56.1:3000  → LAN (accessible on host-only VirtualBox network)
0.0.0.0:3000       → LAN/WAN depending on firewall rules
Deployed on Vercel → WAN (internet-accessible)
```

```bash
# See what interfaces your Node.js app binds to
ss -tulnp | grep node
# or
netstat -tulnp | grep node

# Test LAN accessibility from Metasploitable2
# (SSH into Metasploitable2 first)
curl http://192.168.56.1:3000/api/health

# Scan your own Node.js app like an attacker would
nmap -sV -p 3000 192.168.56.1
```

### Lab 4 — Metasploitable2 as a LAN Target

```bash
# Full service enumeration of your LAN target
nmap -sV -sC -A -p- 192.168.56.101 -oN metasploitable_full.txt

# Identify which network type this simulates
# VirtualBox Host-Only network = isolated LAN
# Both machines on 192.168.56.0/24 subnet = LAN range

# Common open services on Metasploitable2:
# Port 21  → FTP (vsftpd 2.3.4 — backdoored!)
# Port 22  → SSH
# Port 23  → Telnet
# Port 80  → HTTP (DVWA)
# Port 139 → Samba (SMB)
# Port 3306 → MySQL
```

---

## 11. Exam & Interview Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              TYPES OF NETWORKS — EXAM CHEAT SHEET                   ║
╠══════════════╦═══════════╦═══════════╦═══════════╦══════════════════╣
║   Parameter  ║   PAN     ║    LAN    ║    MAN    ║      WAN         ║
╠══════════════╬═══════════╬═══════════╬═══════════╬══════════════════╣
║ Range        ║  ~10m     ║  ~1km     ║  ~50km    ║  Unlimited       ║
║ Technology   ║ Bluetooth ║ Ethernet  ║ FDDI/ATM  ║ Leased/DSL       ║
║ Speed        ║  High     ║  High     ║ Moderate  ║  Low (relative)  ║
║ Ownership    ║ Private   ║ Private   ║ Pub/Priv  ║  Public mostly   ║
║ Maintenance  ║ Very Easy ║  Easy     ║ Difficult ║  Very Difficult  ║
║ Cost         ║ Very Low  ║  Low      ║  High     ║  Very High       ║
║ Error Rate   ║ Very Low  ║  Low      ║ Moderate  ║  High            ║
╚══════════════╩═══════════╩═══════════╩═══════════╩══════════════════╝

KEY FACTS TO MEMORIZE:
✓ PAN → Bluetooth → ~10m → Private → Very Easy → Very Low cost
✓ LAN → Ethernet/Wi-Fi → ~1km → Private → Easy
✓ CAN → Multi-building → ~5km → Private → Moderate (between LAN and MAN)
✓ MAN → FDDI + ATM → City-wide → Public/Private → Difficult
✓ WAN → Leased Lines → Worldwide → Public → Very Difficult

INTERNET vs WWW:
✓ Internet = WAN (physical infrastructure)
✓ WWW = Application running ON the Internet
✓ Internet ⊃ WWW (Email, FTP, VoIP etc. are also Internet but not WWW)

TELEPHONE NETWORK PARALLEL:
✓ PCO  = Local calls   → PAN/LAN equivalent
✓ STD  = Inter-state   → MAN equivalent
✓ ISD  = International → WAN equivalent
✓ 1990s: Internet was born using telephone (PSTN) infrastructure

SECURITY QUICK MAP:
✓ PAN attacks  → Bluejacking, Bluesnarfing, Evil Hotspot
✓ LAN attacks  → ARP Spoof, MAC Flood, DHCP Starve, Evil Twin
✓ CAN attacks  → Lateral movement, Rogue devices
✓ MAN attacks  → BGP local hijack, node compromise
✓ WAN attacks  → BGP hijack, DDoS, DNS hijacking, submarine tap
```

---

## 📚 Topics Coming Next (Based on Series)

- [ ] Network Topologies (Bus, Star, Ring, Mesh, Hybrid)
- [ ] OSI Model — Layer by Layer Deep Dive
- [ ] IP Addressing — IPv4, Subnetting, CIDR
- [ ] TCP vs UDP — Detailed comparison
- [ ] DNS — How name resolution works (and DNS attacks)
- [ ] HTTP / HTTPS — Request/Response cycle (and web attacks)

---

_Notes compiled from: Networking Course Lecture 02 — Types of Computer Networks_
_Source language: Hindi lecture transcript_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
