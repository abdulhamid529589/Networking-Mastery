# 🌐 Network Layer — Responsibilities & Functionalities

### " "Networking Course — Lecture 10

> **Source:** Gate Smashers — Network Layer Responsibilities
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Network Layer — Position in OSI](#1-network-layer--position-in-osi)
2. [Responsibility 1 — Host-to-Host Delivery](#2-responsibility-1--host-to-host-delivery)
3. [Logical Address (IP Address)](#3-logical-address-ip-address)
4. [Node-to-Node vs Host-to-Host — Key Distinction](#4-node-to-node-vs-host-to-host--key-distinction)
5. [Responsibility 2 — Routing](#5-responsibility-2--routing)
6. [Responsibility 3 — Fragmentation](#6-responsibility-3--fragmentation)
7. [Responsibility 4 — Congestion Control](#7-responsibility-4--congestion-control)
8. [Network Layer Devices](#8-network-layer-devices)
9. [Network Layer Protocols Overview](#9-network-layer-protocols-overview)
10. [🔴 Attack Surface — Network Layer](#10--attack-surface--network-layer)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. Network Layer — Position in OSI

```
OSI MODEL
─────────────────────────────────────────────────────────────
Layer 7 │ Application    │  HTTP, FTP, DNS, SMTP
Layer 6 │ Presentation   │  TLS/SSL, Encoding
Layer 5 │ Session        │  Session management
Layer 4 │ Transport      │  TCP, UDP ← receives from here
─────────────────────────────────────────────────────────────
Layer 3 │ NETWORK        │  ← We are here
         │                │  IP, ICMP, IGMP, Routing protocols
─────────────────────────────────────────────────────────────
Layer 2 │ Data Link      │  Ethernet, ARP ← passes down here
Layer 1 │ Physical       │  Cables, Signals
─────────────────────────────────────────────────────────────

PDU at Network Layer = PACKET
```

### Data Flow

```
Transport Layer  →  [Segment]
                         ↓
Network Layer    →  [IP Header][Segment] = Packet
                         ↓
Data Link Layer  →  [Frame Header][Packet][FCS] = Frame
```

---

## 2. Responsibility 1 — Host-to-Host Delivery

### The Core Job

> The Network Layer is responsible for delivering packets from the **source machine** to the **destination machine**, across one or more intermediate networks.

Also called:

- **Host-to-Host delivery**
- **Source-to-Destination delivery**
- **Machine-to-Machine delivery**

### Why It's Needed — Multi-Network Example

```
Network A (Hub topology)         Network B (Bus topology)
┌─────────────────────┐          ┌──────────────────────┐
│  A1                 │          │  B1                  │
│  A2 ──────► [R1] ──┼──────────┼──► [R2] ──► B2       │
│  A3                 │          │  B3                  │
└─────────────────────┘          └──────────────────────┘
     ^                                           ^
     Source                               Destination

A2 wants to send to B2:
  A2 → R1 : Data Link Layer (node-to-node, within Network A)
  R1 → R2 : Network Layer  (router to router, across internet)
  R2 → B2 : Data Link Layer (node-to-node, within Network B)

Overall A2 → B2 : Network Layer responsibility
```

### The Critical Distinction

```
Data Link Layer  →  Node-to-Node delivery     (one hop at a time)
Network Layer    →  Host-to-Host delivery     (source to final destination)
```

---

## 3. Logical Address (IP Address)

### How Host-to-Host Delivery Works

The Network Layer uses **Logical Addresses (IP Addresses)** to identify source and destination.

```
IP Address = Network ID + Host ID
              ↑               ↑
        Which network    Which machine
        to route to      in that network

Example: 192.168.56.101
  192.168.56  = Network ID  (which subnet/network)
  101         = Host ID     (which machine in that network)
```

### Physical vs Logical Address

| Property | Physical Address (MAC)         | Logical Address (IP)        |
| -------- | ------------------------------ | --------------------------- |
| Layer    | Data Link (L2)                 | Network (L3)                |
| Scope    | Local network only             | Global (internet-wide)      |
| Changes  | Never (burned in NIC)          | Can be assigned/changed     |
| Format   | 48-bit hex (AA:BB:CC:DD:EE:FF) | 32-bit IPv4 or 128-bit IPv6 |
| Used by  | Switches, ARP                  | Routers                     |
| Purpose  | Node-to-node hop               | Source-to-destination       |

### What Happens at Each Hop

```
A2 (src IP: 10.0.1.2) sends to B2 (dst IP: 10.0.2.2):

At A2:
  Packet: [src IP: 10.0.1.2][dst IP: 10.0.2.2][data]
  Frame:  [src MAC: A2][dst MAC: R1][Packet]    ← MAC of next hop

At R1 (router):
  Strips frame → reads IP packet
  Looks at dst IP: 10.0.2.2 → routing table → send to R2
  New frame: [src MAC: R1][dst MAC: R2][Packet]  ← IP unchanged!

At R2 (router):
  Strips frame → reads IP packet
  Looks at dst IP: 10.0.2.2 → it's in my network → send to B2
  New frame: [src MAC: R2][dst MAC: B2][Packet]

At B2:
  Strips frame → reads IP → it's for me!
  Passes packet up to Transport Layer

KEY INSIGHT: IP addresses STAY THE SAME end-to-end
             MAC addresses CHANGE at every hop
```

---

## 4. Node-to-Node vs Host-to-Host — Key Distinction

This distinction is critical for GATE/UGC NET:

```
DELIVERY TYPE    LAYER              SCOPE           ADDRESS USED
─────────────────────────────────────────────────────────────────
Node-to-Node   Data Link (L2)    One hop only      MAC address
Host-to-Host   Network (L3)      Source to dest    IP address
Process-to-Process Transport(L4) App to app        Port number
─────────────────────────────────────────────────────────────────
```

```
Analogy:
  Host-to-Host = City to City (your package goes Delhi → Mumbai)
  Node-to-Node = Each truck/train carrying it one segment at a time

  The final destination (Mumbai) never changes.
  But the vehicle carrying it changes at each transit point.
```

---

## 5. Responsibility 2 — Routing

### What is Routing?

> **Routing** = Deciding the **best path** for a packet to travel from source to destination through a network of routers.

```
A2 ──────────────── R1
                   / | \
                  /  |  \
                R3   R4  R5
                  \  |  /
                   \ | /
                    R2 ──── B2

When packet arrives at R1:
  R1 must decide: send via R3? R4? R5? Which path reaches R2 fastest?
  This decision = ROUTING
```

### Routing Protocols

| Protocol  | Full Name                                  | Type                  | Algorithm       |
| --------- | ------------------------------------------ | --------------------- | --------------- |
| **RIP**   | Routing Information Protocol               | Interior              | Distance Vector |
| **OSPF**  | Open Shortest Path First                   | Interior              | Link State      |
| **BGP**   | Border Gateway Protocol                    | Exterior (ISP-to-ISP) | Path Vector     |
| **EIGRP** | Enhanced Interior Gateway Routing Protocol | Interior (Cisco)      | Hybrid          |
| **IS-IS** | Intermediate System to Intermediate System | Interior              | Link State      |

### Distance Vector vs Link State

```
Distance Vector (RIP):
  Each router knows: "How far am I from each destination?"
  Shares this distance table with NEIGHBOURS only
  "Routing by rumour" — relies on neighbours' info
  Slow convergence, prone to routing loops
  Max hop count: 15 (RIP limitation)

Link State (OSPF):
  Each router knows: complete map of the ENTIRE network topology
  Shares link state info with ALL routers (flooding)
  Each router independently computes shortest path (Dijkstra's algorithm)
  Faster convergence, more accurate, scalable
  No hop count limit
```

### Routing Table

Every router maintains a **routing table** — its decision guide:

```
R1 Routing Table:
┌─────────────────┬──────────────┬───────────┬────────────┐
│ Destination     │ Next Hop     │ Interface │ Metric     │
├─────────────────┼──────────────┼───────────┼────────────┤
│ 10.0.1.0/24     │ direct       │ eth0      │ 0          │
│ 10.0.2.0/24     │ 172.16.0.2   │ eth1      │ 1          │
│ 192.168.0.0/16  │ 172.16.0.5   │ eth1      │ 3          │
│ 0.0.0.0/0       │ 172.16.0.1   │ eth1      │ default    │
└─────────────────┴──────────────┴───────────┴────────────┘

When packet arrives for 10.0.2.5:
  Matches 10.0.2.0/24 → forward to 172.16.0.2 via eth1
```

---

## 6. Responsibility 3 — Fragmentation

### The Problem

```
Different networks have different MTU (Maximum Transmission Unit):
  Ethernet:  MTU = 1500 bytes
  Wi-Fi:     MTU = 2304 bytes
  PPP:       MTU = 1500 bytes
  ATM:       MTU = 48 bytes (cells!)

What happens when a 4000-byte packet enters an Ethernet network?
  Ethernet maximum = 1500 bytes → PACKET TOO LARGE → must fragment!
```

### What is Fragmentation?

> **Fragmentation** = Breaking a large packet into smaller pieces (fragments) so each fits within the MTU of the next network link.

```
Original packet (4000 bytes):
[IP Header][                    Data (3980 bytes)                    ]

After fragmentation at router (MTU=1500):
Fragment 1: [IP Header (fragment info)][Data bytes 0-1479]     = 1500B
Fragment 2: [IP Header (fragment info)][Data bytes 1480-2959]  = 1500B
Fragment 3: [IP Header (fragment info)][Data bytes 2960-3979]  = 1020B
```

### IPv4 Fragmentation Fields in Header

```
IPv4 Header Fragmentation fields:
┌────────────────────────────────────────────────────────┐
│ Identification (16 bits) → same for all fragments of   │
│                            one original packet         │
│ Flags (3 bits):                                        │
│   DF (Don't Fragment) → if set, drop instead of frag  │
│   MF (More Fragments) → 1 if more fragments follow    │
│                         0 if this is the last fragment │
│ Fragment Offset (13 bits) → position of this fragment  │
│                             in original data (÷ 8)    │
└────────────────────────────────────────────────────────┘
```

### Fragment Reassembly

```
Who reassembles fragments?

IPv4: DESTINATION HOST reassembles (not intermediate routers)
      Each fragment travels independently (may take different paths)
      Destination collects all fragments → reassembles using:
        - Same Identification number (belongs to same packet)
        - Fragment Offset (where each piece goes)
        - MF flag = 0 on last fragment

IPv6: NO FRAGMENTATION by routers!
      Source must discover path MTU first (Path MTU Discovery)
      Only source can fragment (DF is effectively always set)
```

### Path MTU Discovery (PMTUD)

```
Source wants to know: what's the smallest MTU along the entire path?

Method:
  1. Source sends packet with DF=1 (Don't Fragment)
  2. If router can't forward (packet > MTU) → drops packet
  3. Router sends ICMP "Fragmentation Needed" message back to source
  4. Source reduces packet size and tries again
  5. Repeats until packet makes it through without fragmentation
```

---

## 7. Responsibility 4 — Congestion Control

### What is Congestion?

```
Normal network:
  [Routers processing packets smoothly]
  Input rate ≈ Output rate → packets flow freely

Congested network:
  [Too many packets → router buffers overflow → packets dropped]
  Input rate >> Output rate → buildup → congestion collapse
```

### Where Congestion Control Happens

| Layer                    | Responsibility                          |
| ------------------------ | --------------------------------------- |
| **Network Layer (L3)**   | Detect and signal congestion            |
| **Transport Layer (L4)** | Actually CONTROL congestion (TCP's job) |

> Network Layer primarily **detects and reports** congestion. Transport Layer (TCP) **controls** it.

### Congestion Control Methods at Network Layer

#### Leaky Bucket

```
Network traffic (irregular bursts):
  |||  |||||  |   |||||||||  |  ||

Leaky bucket (constant output rate):
  | | | | | | | | | | | | | | | |

Like a bucket with a small hole:
  Water (packets) poured in irregularly
  Drips out at constant rate
  Bucket overflows if too much comes too fast → packets dropped
```

#### Token Bucket

```
Tokens generated at constant rate (e.g., 100 tokens/sec)
Each packet needs a token to be sent
Tokens accumulate when no traffic (up to bucket capacity)
Burst allowed (use accumulated tokens)
No tokens → packet waits or dropped

More flexible than leaky bucket — allows controlled bursting
```

#### ICMP for Congestion Signaling

```
Router is overwhelmed:
  Router sends ICMP "Source Quench" message to sender
  Sender reduces its transmission rate

(Note: Source Quench is deprecated in modern networks
 TCP's own congestion control replaced this need)
```

---

## 8. Network Layer Devices

### Router

```
[Network A] ──► [ROUTER] ──► [Network B]

Layers: Physical + Data Link + Network (L1 + L2 + L3)
Function: Connect different networks, decide best path (routing)
Intelligence: HIGH — runs routing algorithms, maintains routing table
Used: Between networks (between your home network and internet)
```

### Switch (L3 Switch)

```
[PC1] ──┐
[PC2] ──┤── [L3 SWITCH] ──► inter-VLAN routing
[PC3] ──┘

Standard switch = L2 (Data Link only)
L3 Switch = L2 + L3 (can route between VLANs)
```

### Bridge

```
[Segment 1] ──► [BRIDGE] ──► [Segment 2]

Layers: Physical + Data Link (L1 + L2 ONLY)
NOT a Network Layer device (no IP routing)
Used: Connect two LAN segments using MAC addresses
```

### Firewall

```
[Internet] ──► [FIREWALL] ──► [Internal Network]

Can operate at L3 (packet filtering based on IP) and above
Network Layer firewall: filters by src/dst IP, protocol
Stateful firewall: tracks connection state
```

### Device Summary

| Device    | Layers   | Addresses Used | Main Function             |
| --------- | -------- | -------------- | ------------------------- |
| Hub       | L1       | None           | Broadcast bits to all     |
| Switch    | L2       | MAC            | Forward frames within LAN |
| L3 Switch | L2+L3    | MAC+IP         | Route between VLANs       |
| Router    | L1+L2+L3 | IP             | Route between networks    |
| Bridge    | L1+L2    | MAC            | Connect LAN segments      |
| Firewall  | L3-L7    | IP+Port        | Filter traffic            |

---

## 9. Network Layer Protocols Overview

| Protocol | Full Name                          | Purpose                                         |
| -------- | ---------------------------------- | ----------------------------------------------- |
| **IPv4** | Internet Protocol v4               | Logical addressing, packet routing              |
| **IPv6** | Internet Protocol v6               | Next-gen addressing (128-bit)                   |
| **ICMP** | Internet Control Message Protocol  | Error reporting, diagnostics (ping, traceroute) |
| **IGMP** | Internet Group Management Protocol | Multicast group membership                      |
| **ARP**  | Address Resolution Protocol        | IP → MAC resolution (technically L2.5)          |
| **OSPF** | Open Shortest Path First           | Link-state routing protocol                     |
| **RIP**  | Routing Information Protocol       | Distance-vector routing                         |
| **BGP**  | Border Gateway Protocol            | Internet routing between ISPs                   |

### ICMP — Special Role

```
ICMP is the "error messenger" of the Network Layer:

Tool         ICMP Message Used
────────────────────────────────────────
ping         Echo Request → Echo Reply
traceroute   TTL Exceeded (Time Exceeded)
PMTUD        Fragmentation Needed
Unreachable  Destination Unreachable
Slow down    Source Quench (deprecated)
```

---

## 10. 🔴 Attack Surface — Network Layer

### IP Spoofing

```
Attacker forges the source IP address in packet header:

Real packet:   [src: 10.0.1.5][dst: target][data]
Spoofed:       [src: 1.2.3.4 ][dst: target][data]  ← fake src

Uses:
  - DDoS amplification (send requests with victim's IP → responses flood victim)
  - Bypass IP-based access controls
  - Hide attacker's identity

Tool: Scapy (can set any src IP)
Defense: Ingress filtering (ISPs block packets with forged src IPs — BCP38)
```

### ICMP Attacks

```
Ping Flood:
  sudo hping3 --icmp --flood target
  Overwhelm target with ICMP echo requests

Smurf Attack (historical):
  Send ICMP echo to broadcast addr with victim's spoofed src IP
  Every host on network replies to victim → amplified flood

Ping of Death:
  Oversized ICMP packet (>65535 bytes) → causes buffer overflow
  Patched in modern systems

ICMP Redirect Attack:
  Attacker sends forged ICMP Redirect to victim
  Victim reroutes traffic through attacker → MitM
```

### Routing Attacks

```
BGP Hijacking:
  Attacker announces false BGP routes
  Internet traffic for victim's IP range rerouted to attacker
  Famous example: Pakistan Telecom hijacked YouTube (2008)

RIP Route Poisoning:
  Inject false RIP updates
  Cause routing loops or redirect traffic

OSPF Attacks:
  Inject fake LSAs (Link State Advertisements)
  Cause suboptimal routing or blackholes
```

### Fragmentation Attacks

```
Teardrop Attack (historical):
  Send overlapping fragments with incorrect offsets
  Reassembly causes buffer overflow → crash

Fragmentation Overlap:
  Send fragments that overlap in content
  Different OSes reassemble differently → bypass IDS
  Attacker puts malicious payload in the "gap"

Tiny Fragment Attack:
  Create very small first fragment
  TCP header split across fragments
  Firewall can't read full TCP header → bypass rules

IP Fragment Flood:
  Send huge number of fragments
  Exhaust target's reassembly buffer
```

### TTL Manipulation

```
Every IP packet has a TTL (Time to Live) field.
Each router decrements TTL by 1.
TTL=0 → router drops packet, sends ICMP Time Exceeded.

Attack:
  Set low TTL → packet dies at a specific router
  Used in: Traceroute (legitimate) and IDS evasion

IDS Evasion:
  Send two versions of packet:
  Version 1 (low TTL): reaches IDS but dies before target
  Version 2 (different content): different TTL, reaches target
  IDS sees one thing, target receives another
```

### IP Tunneling

```
Encapsulate one protocol inside IP:
  [outer IP header][inner IP packet / other protocol]

Used for:
  VPNs (legitimate)
  Covert channels (malicious)
  DNS tunneling: hide data in DNS queries
  ICMP tunneling: use ping packets to exfiltrate data
```

---

## 11. 🧪 Practical Labs

### Lab 1 — Explore Routing on Your Machine

```bash
# View routing table on Parrot OS
ip route show
# or
route -n
netstat -rn

# Output explanation:
# 192.168.56.0/24 dev eth0  → Network A (direct, no router needed)
# 0.0.0.0/0 via 192.168.56.1 → Default route (send everything else to gateway)
# default via 192.168.56.1   → Same as above

# Show routing table on Metasploitable2 (SSH in first)
ssh msfadmin@192.168.56.101
route -n
exit
```

### Lab 2 — Trace the Routing Path (Traceroute)

```bash
# Traceroute to Metasploitable2 (LAN — 1 hop)
traceroute 192.168.56.101

# Traceroute across internet (many hops — WAN routing)
traceroute 8.8.8.8
# Each line = one router hop
# * * * = router doesn't respond to ICMP TTL exceeded

# TCP traceroute (bypasses some firewalls)
traceroute -T -p 443 google.com

# How traceroute works:
# Send packet with TTL=1 → first router drops → sends ICMP Time Exceeded → reveals router 1 IP
# Send packet with TTL=2 → second router drops → reveals router 2 IP
# ...and so on until destination reached

# This maps the exact routing path your packets take!
# Security use: network reconnaissance, finding intermediate nodes
```

### Lab 3 — IP Spoofing with Scapy

```bash
# Craft IP packets with spoofed source (your lab only!)
sudo python3 - << 'EOF'
from scapy.all import *

# Normal packet (real source IP)
normal = IP(dst="192.168.56.101")/ICMP()
print("Normal packet src:", normal[IP].src)

# Spoofed packet (fake source IP)
spoofed = IP(src="10.20.30.40", dst="192.168.56.101")/ICMP()
print("Spoofed packet src:", spoofed[IP].src)

# Send spoofed ping
send(spoofed, verbose=0)
print("Sent spoofed ICMP to Metasploitable2")

# In Wireshark: you'll see ICMP from 10.20.30.40 but
# it came from your Parrot OS machine
# This demonstrates IP spoofing at Network Layer
EOF
```

### Lab 4 — ICMP Attacks (Controlled Lab)

```bash
# Ping flood — ICMP attack on Metasploitable2 (your lab target only!)
sudo hping3 --icmp --flood --rand-source 192.168.56.101
# --icmp = ICMP packets
# --flood = as fast as possible
# --rand-source = random source IPs (simulates distributed attack)

# Monitor effect on Metasploitable2 (SSH in another terminal)
ssh msfadmin@192.168.56.101
top
# Watch CPU usage spike from processing ICMP flood

# Stop with Ctrl+C
# Defense: rate-limit ICMP with iptables
sudo iptables -A INPUT -p icmp --icmp-type echo-request -m limit --limit 1/second -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j DROP
```

### Lab 5 — Fragmentation Analysis

```bash
# Send a large packet and observe fragmentation
# Ethernet MTU = 1500 bytes → anything larger gets fragmented

# Send large ping (fragmented)
ping -c 3 -s 3000 192.168.56.101
# -s 3000 = 3000 byte payload (larger than MTU=1500)

# Capture and see fragments in Wireshark
sudo tcpdump -i eth0 -v "host 192.168.56.101 and icmp" 2>/dev/null
# Look for: id=same for all fragments, frag offset values, MF flag

# In Wireshark display filter:
# ip.flags.mf == 1       ← More Fragments set (not last fragment)
# ip.frag_offset > 0     ← Any fragment that's not the first

# Check your machine's MTU
ip link show eth0 | grep mtu
# Default MTU: 1500 bytes for Ethernet

# Change MTU (reduces fragmentation threshold — for testing)
sudo ip link set eth0 mtu 576    # very small MTU
ping -c 1 -s 1000 192.168.56.101 # now this will fragment
sudo ip link set eth0 mtu 1500   # restore
```

### Lab 6 — BGP Hijacking Awareness (Conceptual)

```bash
# Check current BGP routes to a destination (educational)
# This shows how your traffic actually gets routed on the real internet

# Install looking glass client
sudo apt install curl -y

# Query BGP route for an IP (via public looking glass)
curl -s "https://stat.ripe.net/data/routing-status/data.json?resource=8.8.8.8" | \
  python3 -m json.tool | grep -E "origin|prefix" 2>/dev/null | head -20

# Alternative: use route-views BGP looking glass
# telnet route-views.routeviews.org
# show ip bgp 8.8.8.8

# This shows which AS (Autonomous System) "owns" the route to 8.8.8.8
# BGP hijacking = another AS announces false ownership of these prefixes
```

### Lab 7 — Routing Attack Simulation (ARP + IP Level)

```bash
# Demonstrate Network Layer visibility with combined tools
# This simulates what an attacker sees at the Network Layer

# Start Wireshark with Network Layer filters
sudo wireshark &
# Filter: ip and not arp

# From another terminal, generate diverse Network Layer traffic
ping 8.8.8.8 &                         # ICMP (Network Layer protocol)
curl http://192.168.56.101/ &           # IP/TCP
nmap -sn 192.168.56.0/24 &             # IP/ICMP scan

# In Wireshark, observe:
# Every packet has: src IP, dst IP, TTL, Protocol, Length
# This is ALL Network Layer information
# An attacker with network access sees all of this in cleartext
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           NETWORK LAYER — EXAM CHEAT SHEET                          ║
╠══════════════════════════════════════════════════════════════════════╣
║  POSITION                                                            ║
║  OSI Layer 3 | PDU = PACKET                                        ║
║  Sits between: Transport (L4) above, Data Link (L2) below         ║
╠══════════════════════════════════════════════════════════════════════╣
║  4 RESPONSIBILITIES                                                  ║
║  1. Host-to-Host Delivery  → IP addressing (logical address)       ║
║  2. Routing                → Deciding best path (RIP, OSPF, BGP)  ║
║  3. Fragmentation          → Breaking packets to fit MTU           ║
║  4. Congestion Control     → Leaky/Token Bucket, ICMP signals     ║
╠══════════════════════════════════════════════════════════════════════╣
║  DELIVERY TYPES (CRITICAL FOR EXAM)                                 ║
║  Node-to-Node     = Data Link (L2) = MAC address                  ║
║  Host-to-Host     = Network (L3)   = IP address                   ║
║  Process-to-Process = Transport (L4) = Port number                ║
╠══════════════════════════════════════════════════════════════════════╣
║  IP ADDRESS = Network ID + Host ID                                  ║
║  Network ID → which network to route to                            ║
║  Host ID    → which machine in that network                        ║
║  IP stays same end-to-end, MAC changes at every hop               ║
╠══════════════════════════════════════════════════════════════════════╣
║  ROUTING PROTOCOLS                                                   ║
║  RIP   → Distance Vector, max 15 hops, slow convergence           ║
║  OSPF  → Link State (Dijkstra), fast convergence, no hop limit    ║
║  BGP   → Between ISPs (Exterior), Path Vector algorithm           ║
╠══════════════════════════════════════════════════════════════════════╣
║  FRAGMENTATION                                                       ║
║  MTU = Maximum Transmission Unit (Ethernet = 1500 bytes)           ║
║  Packet > MTU → router fragments it                                ║
║  DF flag = Don't Fragment (if set, drop not fragment)             ║
║  MF flag = More Fragments (1=not last, 0=last fragment)           ║
║  IPv4: routers can fragment; IPv6: only source can fragment        ║
║  Reassembly: always at DESTINATION HOST (not intermediate routers)║
╠══════════════════════════════════════════════════════════════════════╣
║  DEVICES                                                             ║
║  Router    = L1+L2+L3 = main Network Layer device                 ║
║  Bridge    = L1+L2 only (NOT Network Layer)                       ║
║  Hub       = L1 only (NOT Network Layer)                          ║
║  L3 Switch = L2+L3 (can route between VLANs)                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY PROTOCOLS                                                       ║
║  IPv4/IPv6 = Logical addressing + packet delivery                  ║
║  ICMP      = Error messages + diagnostics (ping, traceroute)      ║
║  IGMP      = Multicast group management                           ║
║  ARP       = IP→MAC resolution (technically L2/L2.5)             ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY ATTACKS                                                    ║
║  IP Spoofing    → forge src IP → amplification DDoS, bypass ACL  ║
║  Smurf Attack   → spoofed ICMP to broadcast → amplified flood     ║
║  BGP Hijacking  → false route announcements → reroute traffic     ║
║  Fragmentation  → Teardrop, overlap, tiny fragment attacks        ║
║  ICMP Flood     → ping flood with hping3                          ║
║  TTL Manip.     → IDS evasion, network mapping                   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] IP Addressing — IPv4 classes, subnetting, CIDR notation
- [ ] IPv4 vs IPv6 — header format, differences
- [ ] ARP — Address Resolution Protocol in detail
- [ ] ICMP — All message types, ping, traceroute mechanics
- [ ] Routing Algorithms — Distance Vector, Link State in depth
- [ ] Subnetting — VLSM, CIDR practice problems

---

_Notes compiled from: Networking Course Lecture 10 — Network Layer Responsibilities_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
