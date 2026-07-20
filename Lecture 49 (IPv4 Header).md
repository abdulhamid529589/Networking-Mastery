# 📦 IPv4 Header — All Fields Explained

### " "Networking Course — Lecture 15

> **Source:** Gate Smashers — IPv4 Header
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [IPv4 — Quick Introduction](#1-ipv4--quick-introduction)
2. [IPv4 Header — Structure Overview](#2-ipv4-header--structure-overview)
3. [Field-by-Field Breakdown](#3-field-by-field-breakdown)
   - [3.1 Version (4 bits)](#31-version-4-bits)
   - [3.2 Header Length / IHL (4 bits)](#32-header-length--ihl-4-bits)
   - [3.3 Type of Service / DSCP (8 bits)](#33-type-of-service--dscp-8-bits)
   - [3.4 Total Length (16 bits)](#34-total-length-16-bits)
   - [3.5 Identification (16 bits)](#35-identification-16-bits)
   - [3.6 Flags (3 bits)](#36-flags-3-bits)
   - [3.7 Fragment Offset (13 bits)](#37-fragment-offset-13-bits)
   - [3.8 TTL — Time to Live (8 bits)](#38-ttl--time-to-live-8-bits)
   - [3.9 Protocol (8 bits)](#39-protocol-8-bits)
   - [3.10 Header Checksum (16 bits)](#310-header-checksum-16-bits)
   - [3.11 Source IP Address (32 bits)](#311-source-ip-address-32-bits)
   - [3.12 Destination IP Address (32 bits)](#312-destination-ip-address-32-bits)
   - [3.13 Options & Padding (variable)](#313-options--padding-variable)
4. [IPv4 Datagram Size Summary](#4-ipv4-datagram-size-summary)
5. [Connectionless Datagram Service](#5-connectionless-datagram-service)
6. [🔴 Security Context — Header Field Attacks](#6--security-context--header-field-attacks)
7. [🧪 Practical Labs](#7--practical-labs)
8. [Exam Cheat Sheet](#8-exam-cheat-sheet)

---

## 1. IPv4 — Quick Introduction

### What is IPv4?

- **IPv4** = Internet Protocol version 4
- Works at **Network Layer (OSI Layer 3)**
- The backbone of the entire internet since 1981
- **Connectionless** — no connection setup before sending
- **Datagram service** — each packet can take any route independently

### Connectionless vs Connection-Oriented

```
Connection-Oriented (e.g., TCP, Virtual Circuit):
  Step 1: Setup connection (reserve bandwidth, establish path)
  Step 2: Send data (all packets follow same path)
  Step 3: Teardown connection
  → More reliable, more overhead

Connectionless (IPv4 Datagram Service):
  Step 1: Just send! No setup.
  Each packet = independent, can take ANY route
  → Less overhead, less reliable (packets may arrive out of order)
  → Upper layer (TCP) handles reliability if needed
```

### The Envelope Analogy

```
Sending data without header:
  Post office gets a letter → no address written on it → WHERE TO SEND?

IPv4 header = the envelope:
  Source address (return address)
  Destination address
  Instructions for delivery
  Information about the content
```

---

## 2. IPv4 Header — Structure Overview

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│ Version │  IHL  │Type of Service│          Total Length         │
│  4 bits │4 bits │   8 bits      │         16 bits               │
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│         Identification          │Flags│    Fragment Offset       │
│          16 bits                │3 bit│       13 bits            │
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│  Time to Live │    Protocol     │        Header Checksum        │
│    8 bits     │    8 bits       │           16 bits             │
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│                       Source Address (32 bits)                  │
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│                    Destination Address (32 bits)                │
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│                    Options (variable) + Padding                 │
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
│                         Payload (Data)                          │
│                     (up to 65,515 bytes)                        │
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

### Header Size

```
Mandatory fields: 13 fields × various bits = 160 bits = 20 bytes
With options:    up to 60 bytes (480 bits) maximum
Payload (data):  0 to 65,515 bytes
Total datagram:  max 65,535 bytes (= 2¹⁶ - 1)
```

### All 13 Fields at a Glance

| #   | Field                  | Size     | Purpose                     |
| --- | ---------------------- | -------- | --------------------------- |
| 1   | Version                | 4 bits   | IPv4 vs IPv6                |
| 2   | IHL (Header Length)    | 4 bits   | Header size in 32-bit words |
| 3   | Type of Service / DSCP | 8 bits   | QoS priority                |
| 4   | Total Length           | 16 bits  | Entire datagram size        |
| 5   | Identification         | 16 bits  | Fragment reassembly ID      |
| 6   | Flags                  | 3 bits   | Fragmentation control       |
| 7   | Fragment Offset        | 13 bits  | Fragment position           |
| 8   | TTL                    | 8 bits   | Loop prevention             |
| 9   | Protocol               | 8 bits   | Upper layer protocol        |
| 10  | Header Checksum        | 16 bits  | Header error detection      |
| 11  | Source IP              | 32 bits  | Sender address              |
| 12  | Destination IP         | 32 bits  | Receiver address            |
| 13  | Options + Padding      | Variable | Extra features              |

---

## 3. Field-by-Field Breakdown

### 3.1 Version (4 bits)

```
Field size: 4 bits
IPv4 value: 0100 (binary) = 4 (decimal)
IPv6 value: 0110 (binary) = 6 (decimal)

Purpose:
  First field a router reads → instantly knows which protocol version
  0100 → "This is an IPv4 packet, apply IPv4 rules"
  0110 → "This is an IPv6 packet, apply IPv6 rules"

Fixed value: ALWAYS 0100 for IPv4 packets
```

### 3.2 Header Length / IHL (4 bits)

**IHL = Internet Header Length**

```
Field size: 4 bits
Values:     0000 to 1111 (0 to 15 in decimal)
Unit:       32-bit words (NOT bytes directly!)
Actual size = IHL value × 4 bytes

Examples:
  IHL = 0101 (5) → 5 × 4 = 20 bytes  ← MINIMUM valid header
  IHL = 1010 (10) → 10 × 4 = 40 bytes
  IHL = 1111 (15) → 15 × 4 = 60 bytes ← MAXIMUM header

Valid range: 20 to 60 bytes
Values 0–4 (i.e., < 5): INVALID (would give < 20 bytes)

Why multiply by 4?
  4 bits → max value 15
  Header must be at least 20 bytes
  15 × 1 = 15 (too small to represent 20)
  15 × 4 = 60 (covers the full range 20–60)
```

```
Exam calculation:
  Given IHL = 7 → 7 × 4 = 28 bytes header
  Given IHL = 3 → 3 × 4 = 12 bytes → INVALID (< 20)
  Given header = 40 bytes → IHL = 40/4 = 10 → binary 1010
```

### 3.3 Type of Service / DSCP (8 bits)

#### Original ToS Field (8 bits)

```
Bits: [P P P | D T R C | 0]
       ↑ ↑ ↑   ↑ ↑ ↑ ↑   ↑
       Precedence         Reserved

Precedence (3 bits): 000=routine to 111=network control
D = Delay:      1 = minimize delay       (VoIP, streaming)
T = Throughput: 1 = maximize throughput  (file transfer)
R = Reliability:1 = maximize reliability (financial data)
C = Cost:       1 = minimize cost        (bulk data)
Last bit:       Reserved (0)
```

#### Modern DSCP (Differentiated Services)

```
Bits: [D D D D D D | E C]
       ↑           ↑ ↑ ↑
       DSCP         ECN
       (6 bits)    (2 bits)

DSCP (6 bits):
  Defines traffic class and QoS treatment
  Different values = different queuing priorities at routers

ECN (Explicit Congestion Notification, 2 bits):
  00 = Not ECN capable
  01 or 10 = ECN capable
  11 = Congestion Encountered → "I'm experiencing congestion!"

  When ECN = 11:
    Router signals congestion WITHOUT dropping packet
    Receiver tells sender → sender reduces speed
    (vs traditional: drop packet = signal congestion)

  Requires BOTH sender AND receiver to agree on ECN beforehand
```

#### QoS Analogy

```
Without ToS/DSCP: All packets equal → traffic jam
  [Email] [VoIP] [FTP] [Video] → same queue → VoIP suffers

With DSCP priority:
  VoIP (delay-sensitive) → DSCP high priority → fast lane
  Email (not urgent)     → DSCP low priority  → normal lane
  FTP  (throughput)      → DSCP medium        → bulk lane
```

### 3.4 Total Length (16 bits)

```
Field size: 16 bits
Maximum value: 2¹⁶ - 1 = 65,535
Minimum value: 20 (header only, no payload)

Total Length = Header + Payload (data)

Header range:   20 to 60 bytes
Payload range:  0 to 65,515 bytes
  (65,535 - 20 minimum header = 65,515 maximum payload)

Purpose:
  Receiver knows exactly how many bytes this datagram contains
  Can detect if datagram was truncated in transit
```

```
Size calculation:
  Total Length field: 65,535 bytes max
  Minimum header:     20 bytes
  Maximum payload:    65,535 - 20 = 65,515 bytes ✓ (lecture value)

  If options used (header = 60 bytes):
  Maximum payload:    65,535 - 60 = 65,475 bytes
```

### 3.5 Identification (16 bits)

```
Field size: 16 bits
Value: Unique ID assigned by sender for each original datagram

Purpose: FRAGMENTATION REASSEMBLY
  When a large datagram is split into fragments:
  ALL fragments from the same original datagram get the SAME identification value

  Receiver uses this ID to:
    Group fragments belonging to the same original datagram
    Reassemble them in the correct order (using Fragment Offset)

Example:
  Original datagram ID = 12345
  After fragmentation → 3 fragments, all have ID = 12345
  Receiver gets ID=12345 fragment 1, fragment 2, fragment 3
  Reassembles them into original

  If two different datagrams are in transit: different IDs
```

_(Detailed fragmentation explanation in next lecture)_

### 3.6 Flags (3 bits)

```
Bits: [Reserved | DF | MF]
         bit 0    bit1  bit2

Reserved (bit 0): Always 0

DF = Don't Fragment (bit 1):
  0 = OK to fragment if needed
  1 = DO NOT fragment — if packet is too large, DROP and send ICMP error
  Used in: Path MTU Discovery

MF = More Fragments (bit 2):
  0 = This is the LAST fragment (or no fragmentation at all)
  1 = More fragments follow — don't reassemble yet

Example:
  First fragment:   DF=0, MF=1 (more coming)
  Middle fragments: DF=0, MF=1 (more coming)
  Last fragment:    DF=0, MF=0 (this is the last one)
```

### 3.7 Fragment Offset (13 bits)

```
Field size: 13 bits
Unit: 8-byte blocks (multiply by 8 to get byte offset)

Purpose:
  Tells receiver WHERE in the original datagram this fragment belongs
  Offset = byte position of this fragment's first byte / 8

Example:
  Original datagram: 4000 bytes of data
  Fragment 1: bytes 0–1479    → offset = 0/8 = 0
  Fragment 2: bytes 1480–2959 → offset = 1480/8 = 185
  Fragment 3: bytes 2960–3999 → offset = 2960/8 = 370

Why divide by 8?
  13 bits → max value 8191
  8191 × 8 = 65,528 → covers full datagram range (65,535)
```

### 3.8 TTL — Time to Live (8 bits)

```
Field size: 8 bits
Value range: 0 to 255
Starting values: typically 64 (Linux), 128 (Windows), 255 (routers)

Purpose: Prevent infinite packet loops

How it works:
  Sender sets TTL = N (e.g., 64)
  Each router that forwards the packet: TTL = TTL - 1
  When TTL reaches 0: router DROPS the packet
                      router sends ICMP "Time Exceeded" back to source

Without TTL:
  Packet gets stuck in routing loop → circulates forever → network congestion

Traceroute uses TTL:
  Send packet with TTL=1 → first router decrements to 0 → drops → sends ICMP
  You see router 1's IP from ICMP message
  Send TTL=2 → second router drops → reveals router 2
  Continue until destination reached
```

```
OS Fingerprinting via TTL:
  Initial TTL = 64  → Linux/Unix
  Initial TTL = 128 → Windows
  Initial TTL = 255 → Cisco IOS/routers

  If you receive packet with TTL=54 → started at 64, went through 10 hops
  64 - 54 = 10 hops traveled
```

### 3.9 Protocol (8 bits)

```
Field size: 8 bits
Purpose: Identifies which upper-layer protocol is encapsulated in payload

Common Protocol Numbers:
  1   → ICMP (Internet Control Message Protocol)
  2   → IGMP (Internet Group Management Protocol)
  6   → TCP  (Transmission Control Protocol)
  17  → UDP  (User Datagram Protocol)
  89  → OSPF (Open Shortest Path First)
  132 → SCTP (Stream Control Transmission Protocol)

Demultiplexing:
  When IP packet arrives at destination, Protocol field says:
  "Pass this payload to TCP handler" (if Protocol=6)
  "Pass this payload to UDP handler" (if Protocol=17)
  "Pass to ICMP handler" (if Protocol=1)
```

### 3.10 Header Checksum (16 bits)

```
Field size: 16 bits
Purpose: Error detection for the HEADER ONLY (not payload)

Why only header?
  If header gets corrupted → wrong destination IP → packet goes to wrong place
  Header checksum catches this error at EACH router

Recalculated at EVERY hop:
  Because TTL changes at every router → checksum changes too
  So every router must recalculate checksum

Process:
  Sender: compute checksum of all header fields, store in this field
  Router: recompute checksum of received header
          if computed ≠ stored → error → DROP packet
          if computed = stored → header intact → forward

Note: Does NOT protect payload (data)
  Upper layers (TCP) have their own checksum for payload
```

### 3.11 Source IP Address (32 bits)

```
Field size: 32 bits
Content: IP address of the sender

Note: CAN BE FORGED! (IP Spoofing attack)
  Unlike MAC address (set by hardware), IP can be manually set to anything
  Attacker can put any source IP → fake identity

Used for: Routing replies back to sender
```

### 3.12 Destination IP Address (32 bits)

```
Field size: 32 bits
Content: IP address of the intended receiver

Used by: Every router along the path to make forwarding decisions
  Router reads dst IP → looks up routing table → forwards accordingly

Note: Broadcast (x.x.x.255), Multicast (224.x.x.x), Unicast addresses all valid
```

### 3.13 Options & Padding (Variable)

```
Options: Extra fields for special purposes
  Record Route: each router adds its IP to track path
  Timestamp:    each router adds timestamp
  Loose Source Routing: sender specifies some routers packet must visit
  Strict Source Routing: sender specifies EXACT path (every hop)
  Security:     classify packet sensitivity level (military use)

Padding:
  Header must be a multiple of 32 bits (4 bytes)
  If options don't fill to a multiple of 32, add zero bits (padding)

  Example: options = 12 bytes → total header = 32 bytes → already multiple of 4 ✓
           options = 10 bytes → total = 30 bytes → add 2 bytes padding → 32 bytes ✓

Without options: header = exactly 20 bytes (no padding needed, 5 × 32 bits)
With options:    header = 24 to 60 bytes (in multiples of 4)
```

---

## 4. IPv4 Datagram Size Summary

```
┌─────────────────────────────────────────────────────────────┐
│                    IPv4 DATAGRAM                            │
├─────────────────────────────────────────┬───────────────────┤
│              HEADER                     │      PAYLOAD      │
│  (min 20 bytes, max 60 bytes)           │  (0–65,515 bytes) │
├─────────────────────────────────────────┴───────────────────┤
│  TOTAL LENGTH: max 65,535 bytes (2¹⁶ - 1)                  │
└─────────────────────────────────────────────────────────────┘

Header minimum: 20 bytes (IHL = 5, no options)
Header maximum: 60 bytes (IHL = 15, full options)
Payload minimum: 0 bytes (header only datagram, rare)
Payload maximum: 65,515 bytes (with minimum 20-byte header)
Total maximum: 65,535 bytes

Note: Most real networks have MTU = 1500 bytes (Ethernet)
So large datagrams get FRAGMENTED before transmission
```

---

## 5. Connectionless Datagram Service

```
Key properties of IPv4:
1. Connectionless: No setup before sending
2. Unreliable:     No guarantee of delivery
3. Best-effort:    Will try its best but no promises
4. Unordered:      Packets may arrive out of order

What handles reliability?
  TCP (Transport Layer) adds:
    Acknowledgments (ACK)
    Retransmission of lost packets
    Ordering

UDP keeps IPv4's unreliability → fast, no overhead

Virtual Circuit (comparison):
  Connection-oriented alternative
  Setup → Data transfer → Teardown
  Resources reserved → guaranteed delivery
  ATM networks used this model
```

---

## 6. 🔴 Security Context — Header Field Attacks

### Attack 1 — IP Spoofing (Source Address Field)

```
Attacker sets source IP = victim's IP:
  Crafted packet: [src: victim][dst: target]

Target replies to victim (not attacker):
  Used in: Amplification DDoS, bypassing IP-based authentication

Defense: Ingress filtering (ISP drops packets with spoofed src IPs — BCP38)
Tool:    Scapy (can set any src IP)

scapy: IP(src="1.2.3.4", dst="target") / ICMP()
```

### Attack 2 — TTL Manipulation

```
OS Fingerprinting:
  Send probe → measure TTL of response
  TTL=54 → started at 64 → Linux system, 10 hops away
  TTL=119 → started at 128 → Windows system, 9 hops away

IDS/Firewall Evasion:
  Send decoy packet with very low TTL → dies before reaching firewall
  Send real malicious packet → reaches target
  IDS logs low-TTL packet → thinks attack failed → ignores

TTL-based traceroute (attacker uses for recon):
  Maps exact network topology between attacker and target
  Reveals intermediate routers, their IPs, response times
```

### Attack 3 — Fragmentation Attacks

```
Tiny Fragment Attack:
  Create fragment so small that TCP header is split across fragments
  Fragment 1: IP header + first 8 bytes of TCP (flags, ports)
  Fragment 2: rest of TCP header

  Firewall reads fragment 1 → TCP flags look normal → ALLOW
  Fragment 2 arrives → contains malicious flags → firewall missed them
  Reassembly at target → full attack payload

Teardrop Attack:
  Send fragments with overlapping offsets:
  Fragment 1: offset=0, length=100 (covers bytes 0-99)
  Fragment 2: offset=50, length=100 (covers bytes 50-149) ← OVERLAP!

  Buggy reassembly → buffer underflow/overflow → crash (historical)
```

### Attack 4 — Protocol Field Abuse

```
IP tunneling:
  Encapsulate one protocol inside another using Protocol field
  IP(Protocol=4)/IP(...) → IP-in-IP tunnel (GRE)

  Covert channel: exfiltrate data inside ICMP packets
  IP(Protocol=1)/ICMP(payload=stolen_data)

  Detection: Deep Packet Inspection (DPI)
  Defense: Block unusual Protocol values at firewall
```

### Attack 5 — Header Checksum Bypass

```
Attacker modifies data in IP header (e.g., dst IP redirect):
  Must also update checksum to match modified header
  Otherwise router drops the packet

  Attackers in MitM position CAN recalculate checksum → undetected

  Checksum only detects ACCIDENTAL errors, not intentional modification
  Use IPSec/TLS for actual security against MitM
```

### Attack 6 — IP Options Exploitation

```
Loose/Strict Source Routing options (historical):
  Attacker specifies path packet must take
  Can force packet to visit attacker-controlled router → intercept

  Most modern routers DROP packets with source routing options
  But legacy systems may still process them

Record Route option abuse:
  Attacker uses Record Route to map internal network topology
  Packet visits routers that add their IPs → attacker reads the path
```

---

## 7. 🧪 Practical Labs

### Lab 1 — Read Real IPv4 Headers with Wireshark

```bash
# Capture traffic and examine IPv4 headers
sudo wireshark &

# Generate traffic
ping -c 5 192.168.56.101
curl http://192.168.56.101

# In Wireshark, click any IP packet:
# Expand "Internet Protocol Version 4":
#   Version: 4
#   Header Length: X bytes (IHL × 4)
#   Differentiated Services Field: 0x00 (DSCP, ECN)
#   Total Length: N
#   Identification: 0xXXXX
#   Flags: DF bit, MF bit
#   Fragment Offset: 0
#   Time to Live: 64 (Linux default)
#   Protocol: 1 (ICMP) or 6 (TCP) or 17 (UDP)
#   Header Checksum: 0xXXXX
#   Source Address: your IP
#   Destination Address: 192.168.56.101

# Filter specific fields:
# ip.ttl < 10          → packets near their TTL limit
# ip.flags.df == 1     → Don't Fragment set
# ip.proto == 6        → TCP packets only
# ip.proto == 17       → UDP packets only
# ip.proto == 1        → ICMP packets only
```

### Lab 2 — Manipulate IPv4 Header with Scapy

```bash
# Craft custom IPv4 packets (your lab only)
sudo python3 - << 'EOF'
from scapy.all import *

# 1. Show default IPv4 header
pkt = IP(dst="192.168.56.101")
pkt.show()

# 2. Examine each field
print(f"\nVersion:     {pkt.version}")
print(f"IHL:         {pkt.ihl} (× 4 = {pkt.ihl*4} bytes)")
print(f"TOS:         {pkt.tos}")
print(f"Total len:   auto-calculated")
print(f"ID:          {pkt.id}")
print(f"Flags:       {pkt.flags}")
print(f"Frag offset: {pkt.frag}")
print(f"TTL:         {pkt.ttl}")
print(f"Protocol:    {pkt.proto}")
print(f"Checksum:    auto-calculated")
print(f"Src IP:      {pkt.src}")
print(f"Dst IP:      {pkt.dst}")

# 3. Custom TTL
low_ttl = IP(dst="192.168.56.101", ttl=5)/ICMP()
print(f"\nCustom TTL=5 packet: TTL={low_ttl.ttl}")
# send(low_ttl)  # uncomment to send

# 4. Set DF bit (don't fragment)
df_pkt = IP(dst="192.168.56.101", flags="DF")/ICMP()
print(f"DF flag set: {df_pkt.flags}")

# 5. IP Spoofing demo
spoofed = IP(src="10.20.30.40", dst="192.168.56.101")/ICMP()
print(f"\nSpoofed src: {spoofed.src} (fake!)")
print(f"Real machine: (your actual IP)")

EOF
```

### Lab 3 — TTL Analysis (OS Fingerprinting)

```bash
# Send pings and observe TTL in responses
# This reveals the OS of the target

# Ping Metasploitable2
ping -c 3 192.168.56.101
# Look at TTL in response:
# Linux default: 64
# Windows default: 128

# More detailed: use nmap for OS fingerprinting
sudo nmap -O 192.168.56.101
# nmap uses TTL + other fingerprints to identify OS

# Calculate how many hops away based on TTL
python3 - << 'EOF'
def guess_os_and_hops(observed_ttl):
    common_starts = [64, 128, 255]
    results = []
    for start in common_starts:
        if start >= observed_ttl:
            hops = start - observed_ttl
            os_guess = {64: "Linux/Mac", 128: "Windows", 255: "Cisco/Router"}[start]
            results.append(f"If initial={start} ({os_guess}): {hops} hops")
    return results

# Example: you receive a ping reply with TTL=54
observed = 54
print(f"Received TTL: {observed}")
for r in guess_os_and_hops(observed):
    print(f"  {r}")
EOF
```

### Lab 4 — Protocol Field Analysis

```bash
# See different Protocol values in real traffic
sudo tcpdump -i eth0 -n -v 2>/dev/null | grep -E "proto (ICMP|TCP|UDP|OSPF)" | head -20

# In Wireshark filter by protocol number:
# ip.proto == 1   → ICMP
# ip.proto == 6   → TCP
# ip.proto == 17  → UDP
# ip.proto == 89  → OSPF

# Generate different protocol traffic
ping -c 2 192.168.56.101      # Protocol 1 (ICMP)
curl http://192.168.56.101    # Protocol 6 (TCP)
nslookup google.com           # Protocol 17 (UDP) to DNS

# Count packets by protocol
sudo tshark -i eth0 -T fields -e ip.proto -c 100 2>/dev/null | \
  sort | uniq -c | sort -rn | head -10
# Shows which protocols are most active on your network
```

### Lab 5 — Implement IPv4 Header Parser (Python)

```python
# Save as ipv4_parser.py — run: python3 ipv4_parser.py
import struct

def parse_ipv4_header(raw_bytes: bytes) -> dict:
    """Parse raw IPv4 header bytes into fields"""
    if len(raw_bytes) < 20:
        return {"error": "Too short for IPv4 header"}

    # Unpack first 20 bytes
    fields = struct.unpack('!BBHHHBBH4s4s', raw_bytes[:20])

    version_ihl = fields[0]
    version = (version_ihl >> 4) & 0xF
    ihl = version_ihl & 0xF
    header_bytes = ihl * 4

    tos = fields[1]
    dscp = (tos >> 2) & 0x3F
    ecn = tos & 0x3

    total_length = fields[2]
    identification = fields[3]

    flags_frag = fields[4]
    flags = (flags_frag >> 13) & 0x7
    df = (flags >> 1) & 1
    mf = flags & 1
    frag_offset = flags_frag & 0x1FFF

    ttl = fields[5]
    protocol = fields[6]
    checksum = fields[7]

    src_ip = '.'.join(str(b) for b in fields[8])
    dst_ip = '.'.join(str(b) for b in fields[9])

    proto_names = {1:'ICMP', 2:'IGMP', 6:'TCP', 17:'UDP', 89:'OSPF'}

    return {
        'Version':         version,
        'IHL (words)':     ihl,
        'Header size':     f"{header_bytes} bytes",
        'DSCP':            dscp,
        'ECN':             ecn,
        'Total Length':    total_length,
        'Payload size':    total_length - header_bytes,
        'Identification':  hex(identification),
        'DF flag':         bool(df),
        'MF flag':         bool(mf),
        'Frag Offset':     frag_offset * 8,
        'TTL':             ttl,
        'Protocol':        f"{protocol} ({proto_names.get(protocol, 'Unknown')})",
        'Checksum':        hex(checksum),
        'Source IP':       src_ip,
        'Destination IP':  dst_ip,
    }

# Example: manually craft a 20-byte IPv4 header
# IP(src=192.168.1.1, dst=192.168.1.2, ttl=64, proto=6/TCP)
raw_header = bytes([
    0x45,             # Version=4, IHL=5
    0x00,             # DSCP=0, ECN=0
    0x00, 0x28,       # Total length = 40 bytes
    0x12, 0x34,       # Identification
    0x40, 0x00,       # Flags=010 (DF), Fragment offset=0
    0x40,             # TTL=64
    0x06,             # Protocol=6 (TCP)
    0x00, 0x00,       # Checksum (not computed here)
    192, 168, 1, 1,   # Source IP
    192, 168, 1, 2,   # Destination IP
])

result = parse_ipv4_header(raw_header)
print("IPv4 Header Analysis:")
print("=" * 40)
for field, value in result.items():
    print(f"  {field:<20}: {value}")
```

---

## 8. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              IPv4 HEADER — EXAM CHEAT SHEET                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  SIZE                                                                ║
║  Mandatory header:  20 bytes (160 bits) — 12 fields (no opt/pad)   ║
║  With options:      up to 60 bytes                                  ║
║  Total datagram:    up to 65,535 bytes (2¹⁶ - 1)                   ║
║  Max payload:       65,515 bytes (65,535 - 20 min header)          ║
╠══════════════════════════════════════════════════════════════════════╣
║  ALL 13 FIELDS                                                       ║
║  Version    4b  → 0100 = IPv4, 0110 = IPv6                        ║
║  IHL        4b  → header size ÷ 4 (min=5, max=15, ×4=20–60 bytes) ║
║  TOS/DSCP   8b  → QoS priority (DTRC bits) + ECN (2 bits)        ║
║  Total Len 16b  → entire datagram (header + data), max 65535      ║
║  ID        16b  → fragment reassembly identification              ║
║  Flags      3b  → Reserved | DF (don't frag) | MF (more frags)   ║
║  Frag Off  13b  → byte position ÷ 8, max 8191 × 8 = 65528        ║
║  TTL        8b  → decremented at each hop, drop when 0            ║
║  Protocol   8b  → ICMP=1, IGMP=2, TCP=6, UDP=17, OSPF=89        ║
║  Checksum  16b  → header-only error detection (recalculated/hop)  ║
║  Src IP    32b  → sender address (can be SPOOFED!)                ║
║  Dst IP    32b  → destination address                             ║
║  Options  var   → source routing, timestamps, security labels     ║
╠══════════════════════════════════════════════════════════════════════╣
║  IHL CALCULATION (EXAM CRITICAL)                                     ║
║  IHL × 4 = header size in bytes                                    ║
║  IHL = 5 → 20 bytes (minimum)                                      ║
║  IHL = 15 → 60 bytes (maximum)                                     ║
║  IHL < 5 → INVALID                                                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  ToS / DSCP BITS                                                     ║
║  Original: PPP | D T R C | 0                                       ║
║    D=Delay, T=Throughput, R=Reliability, C=Cost                    ║
║  Modern DSCP: [6-bit DSCP][2-bit ECN]                              ║
║    ECN=11 → congestion notification (no packet drop)              ║
╠══════════════════════════════════════════════════════════════════════╣
║  TTL INITIAL VALUES                                                  ║
║  Linux/Mac:    64                                                   ║
║  Windows:      128                                                  ║
║  Cisco router: 255                                                  ║
║  Hops traveled = initial_TTL - observed_TTL                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  FLAGS                                                               ║
║  DF=1 → don't fragment → drop if too big → send ICMP error        ║
║  MF=1 → more fragments following → don't reassemble yet           ║
║  MF=0 → last (or only) fragment → reassemble now                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY ATTACKS ON IPv4 HEADER                                     ║
║  IP Spoofing  → forge src IP → DDoS amplification, bypass ACL     ║
║  TTL manip.   → OS fingerprinting, IDS evasion, traceroute        ║
║  Tiny frag    → split TCP header across fragments → bypass FW     ║
║  Teardrop     → overlapping fragments → crash legacy systems      ║
║  Src routing  → force path → intercept traffic (now mostly blocked)║
║  Checksum     → only catches accidents, NOT malicious tampering   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] IPv4 Fragmentation — detailed (Identification, Flags, Fragment Offset)
- [ ] IPv6 Header — simplified header, 128-bit addressing
- [ ] ICMP — all message types, ping, traceroute internals
- [ ] ARP — how IP resolves to MAC (IP→MAC at each hop)
- [ ] IPv4 vs IPv6 — full comparison

---

_Notes compiled from: Networking Course Lecture 15 — IPv4 Header Fields_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
