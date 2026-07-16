# 🌐 IP Addressing — Classful Addressing & Class A

### Cybersecurity Student Notes | Networking Course — Lecture 11

> **Source:** Gate Smashers — Class A in IP Addressing (Classful Addressing)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [History of IP Addressing](#1-history-of-ip-addressing)
2. [Classful Addressing — Overview](#2-classful-addressing--overview)
3. [IPv4 Address Structure](#3-ipv4-address-structure)
4. [Class A — Deep Dive](#4-class-a--deep-dive)
   - [4.1 Identifying Class A](#41-identifying-class-a)
   - [4.2 Network ID vs Host ID Split](#42-network-id-vs-host-id-split)
   - [4.3 Number of Networks](#43-number-of-networks)
   - [4.4 Number of Hosts per Network](#44-number-of-hosts-per-network)
   - [4.5 Reserved Addresses in Class A](#45-reserved-addresses-in-class-a)
5. [Default Subnet Mask — Class A](#5-default-subnet-mask--class-a)
6. [Network ID Extraction — AND Operation](#6-network-id-extraction--and-operation)
7. [Special Addresses](#7-special-addresses)
8. [All 5 Classes — Quick Reference](#8-all-5-classes--quick-reference)
9. [🔴 Security Context — IP Classes & Attacks](#9--security-context--ip-classes--attacks)
10. [🧪 Practical Labs](#10--practical-labs)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. History of IP Addressing

### Before 1980s — Fixed Split

```
32-bit IP address:
[──── 8 bits ────][────────── 24 bits ──────────]
   Network Number           Host Number

Problem 1: Only 2⁸ = 256 networks possible → too few
Problem 2: Each network had 2²⁴ = 16,777,216 hosts → too many per network
           Most companies don't need millions of hosts in one network

Example: ARPANET used Network ID = 10 (decimal)
         Host bits: 24 bits → millions of possible hosts
         But no company needed that many!
```

### 1980s — Classful Addressing Born

- Internet demand increased → scalability problem
- Solution: Divide 32-bit space into **5 classes** (A, B, C, D, E)
- Different network/host split ratios for different organization sizes
- **IANA (Internet Assigned Numbers Authority)** manages allocation

```
Timeline:
Pre-1980  → Fixed 8-bit network, 24-bit host
1981      → RFC 791 (IPv4) + Classful addressing introduced
1993      → CIDR (Classless) introduced (classful became obsolete)
Today     → CIDR used, but classful knowledge still required for exams
```

---

## 2. Classful Addressing — Overview

Five classes, each identified by the **leading bits** of the first octet:

```
Class A: 0xxxxxxx  (first bit = 0)
Class B: 10xxxxxx  (first two bits = 10)
Class C: 110xxxxx  (first three bits = 110)
Class D: 1110xxxx  (first four bits = 1110) ← Multicast
Class E: 11110xxx  (first five bits = 11110) ← Reserved/Experimental
```

### First Octet Ranges

| Class | First Bit(s) | Range (first octet) | Purpose              |
| ----- | ------------ | ------------------- | -------------------- |
| **A** | 0            | 0 – 127             | Large organizations  |
| **B** | 10           | 128 – 191           | Medium organizations |
| **C** | 110          | 192 – 223           | Small organizations  |
| **D** | 1110         | 224 – 239           | Multicast            |
| **E** | 11110        | 240 – 255           | Research/Reserved    |

> **Exam shortcut:** Just memorize the first octet ranges. Any IP address — look at first octet → instantly know the class.

---

## 3. IPv4 Address Structure

### 32-bit, 4 Octets, Dotted Decimal

```
Binary:   10000000 00001010 00000010 00000011
          ────────────────────────────────────
Octets:   Octet 1  Octet 2  Octet 3  Octet 4
          ────────────────────────────────────
Decimal:     128  .   10   .   2    .   3
          ────────────────────────────────────
Dotted:          128.10.2.3
```

### Why Dotted Decimal?

- 32 binary bits → hard to read and remember
- Split into 4 groups of 8 (octets) → convert each to decimal
- Range of each octet: 0–255 (2⁸ = 256 values: 0 to 255)
- Total possible IPv4 addresses: 2³² ≈ 4.3 billion

### Binary ↔ Decimal Conversion Quick Reference

| Binary   | Decimal |
| -------- | ------- |
| 00000000 | 0       |
| 10000000 | 128     |
| 11000000 | 192     |
| 11100000 | 224     |
| 11110000 | 240     |
| 11111111 | 255     |

---

## 4. Class A — Deep Dive

### 4.1 Identifying Class A

```
32-bit structure of Class A:
┌─┬───────┬────────────────────────────────────────────┐
│0│ Net   │                    Host                    │
│ │ 7bits │                   24 bits                  │
└─┴───────┴────────────────────────────────────────────┘
 ↑
 First bit ALWAYS = 0 (fixed, cannot change)
 This is the "class identifier bit"

First octet layout:
[ 0 | n | n | n | n | n | n | n ]
  ↑   └──────── 7 network bits ─┘
Fixed
```

**How routers identify Class A:**

- Router receives packet → looks at first bit of dst IP
- First bit = 0 → "This is Class A" → applies Class A routing rules
- No need to inspect further bits for class identification

### 4.2 Network ID vs Host ID Split

```
Class A IP Address:
[──── 8 bits (1st octet) ────][──────── 24 bits (3 octets) ────────]
        Network ID                          Host ID
   (1 fixed + 7 variable)              (all 24 variable)

Example: 64.0.0.1
  64         = Network ID (first octet)
  0.0.1      = Host ID (last three octets)
  Network:   64.0.0.0
  Host:      the ".0.0.1" part (host #1)
```

### 4.3 Number of Networks

```
Network ID = 8 bits, but first bit is FIXED at 0
Variable network bits = 7

Number of possible networks = 2⁷ = 128

BUT: 2 are reserved (not assigned to organizations):
  0.x.x.x   → 0 in network ID = null/this network (IANA reserved)
  127.x.x.x → loopback (127.0.0.1 = localhost) — reserved for diagnostics

Usable networks = 128 - 2 = 126
```

```
Network ID range:
  Minimum: 0000 0000 = 0   → Reserved (null address)
  Maximum: 0111 1111 = 127 → Reserved (loopback)
  Usable:  1 through 126   → 126 organizations can have a Class A network
```

### 4.4 Number of Hosts per Network

```
Host ID = 24 bits (last 3 octets)
All 24 bits are variable

Total host addresses per network = 2²⁴ = 16,777,216

BUT: 2 are reserved:
  First address (all host bits = 0):  Network Address
  Last address  (all host bits = 1):  Broadcast Address

Usable hosts per network = 2²⁴ - 2 = 16,777,214
```

```
Example: Google's Class A network 64.0.0.0

Network address:   64.  0.  0.  0   → represents the network itself
First host:        64.  0.  0.  1   → first usable host
...
Last host:         64.255.255.254   → last usable host
Broadcast address: 64.255.255.255   → sends to ALL hosts in network
```

### 4.5 Reserved Addresses in Class A

#### Network Address (First Address)

```
Purpose: Identifies the network itself
Example: 64.0.0.0

Cannot assign to any host.
Used as: network identifier in routing tables, subnet addresses
The "name" of the network.
```

#### Broadcast Address (Last Address)

```
Purpose: Send packet to ALL hosts in this network simultaneously
Example: 64.255.255.255 (Directed Broadcast)

Broadcast types:
  Limited Broadcast:   255.255.255.255 → all hosts on local network
  Directed Broadcast:  64.255.255.255  → all hosts in 64.0.0.0 network

If you send to 64.255.255.255, every machine in 64.0.0.0 receives it.
Security risk: Smurf attack exploits broadcast addresses!
```

#### Loopback Address

```
127.0.0.0/8 range → Reserved for loopback
127.0.0.1 → Most common loopback ("localhost")

Purpose: Test network stack on same machine without sending to network
Used by: developers, network testing, IPC via localhost

When you ping 127.0.0.1:
  Packet never leaves the machine
  Goes through TCP/IP stack → looped back → received by same machine

Your MERN/PERN app on localhost:3000 → using loopback address
```

---

## 5. Default Subnet Mask — Class A

### What is a Subnet Mask?

> A **subnet mask** separates the Network ID portion from the Host ID portion of an IP address.

### Class A Default Mask

```
Class A Default Mask: 255.0.0.0

Binary:
255       .  0    .  0    .  0
11111111  . 00000000.00000000.00000000

1s mark the NETWORK portion
0s mark the HOST portion

So: 8 bits of network, 24 bits of host → matches Class A structure
```

### CIDR Notation

```
255.0.0.0 = /8 (8 network bits)

Class A:  /8  → 255.0.0.0
Class B:  /16 → 255.255.0.0
Class C:  /24 → 255.255.255.0
```

---

## 6. Network ID Extraction — AND Operation

### How to Find Which Network an IP Belongs To

**Method:** AND the IP address with the Subnet Mask

```
Question: Which network does 64.0.0.8 belong to?

Step 1: Write IP in binary
  64    = 01000000
  0     = 00000000
  0     = 00000000
  8     = 00001000

Step 2: Write Class A mask in binary (255.0.0.0)
  255   = 11111111
  0     = 00000000
  0     = 00000000
  0     = 00000000

Step 3: AND operation (bit by bit)
  01000000.00000000.00000000.00001000   ← IP
  11111111.00000000.00000000.00000000   ← Mask
  ────────────────────────────────────
  01000000.00000000.00000000.00000000   ← Result

Step 4: Convert to decimal
  01000000 = 64
  00000000 = 0
  00000000 = 0
  00000000 = 0

Answer: Network ID = 64.0.0.0
        Host 64.0.0.8 belongs to network 64.0.0.0 ✓
```

### AND Truth Table (reminder)

```
1 AND 1 = 1
1 AND 0 = 0
0 AND 1 = 0
0 AND 0 = 0

Key insight: AND with mask 255 (11111111) → preserves original bits
             AND with mask 0   (00000000) → zeroes out all bits
```

---

## 7. Special Addresses

### Summary of All Reserved Addresses

| Address           | Type               | Purpose                                  |
| ----------------- | ------------------ | ---------------------------------------- |
| `0.0.0.0`         | Null/Unspecified   | "This host, this network" — not assigned |
| `127.0.0.1`       | Loopback           | Localhost — testing on same machine      |
| `127.0.0.0/8`     | Loopback range     | All 127.x.x.x reserved for loopback      |
| `x.0.0.0`         | Network Address    | Identifies the network (first address)   |
| `x.255.255.255`   | Directed Broadcast | Send to all hosts in network x           |
| `255.255.255.255` | Limited Broadcast  | Send to all hosts on local network       |
| `10.x.x.x`        | Private (Class A)  | Internal use, not routed on internet     |
| `169.254.x.x`     | APIPA              | Auto-configured when no DHCP available   |

### Private IP Ranges (RFC 1918)

```
Class A private: 10.0.0.0    – 10.255.255.255  (/8)
Class B private: 172.16.0.0  – 172.31.255.255  (/12)
Class C private: 192.168.0.0 – 192.168.255.255 (/16)

Private IPs:
  → NOT routable on public internet
  → Used inside LANs (your home, office, VirtualBox network)
  → Requires NAT (Network Address Translation) to reach internet

Your VirtualBox Host-Only: 192.168.56.0/24 → Class C private range
```

---

## 8. All 5 Classes — Quick Reference

| Class | 1st bit(s) | Range   | Net bits | Host bits | Networks | Hosts/Net   | Default Mask        | Use         |
| ----- | ---------- | ------- | -------- | --------- | -------- | ----------- | ------------------- | ----------- |
| **A** | 0          | 0–127   | 8        | 24        | 126      | 2²⁴−2 ≈ 16M | 255.0.0.0 (/8)      | Large orgs  |
| **B** | 10         | 128–191 | 16       | 16        | 16,382   | 2¹⁶−2 ≈ 65K | 255.255.0.0 (/16)   | Medium orgs |
| **C** | 110        | 192–223 | 24       | 8         | ~2M      | 2⁸−2 = 254  | 255.255.255.0 (/24) | Small orgs  |
| **D** | 1110       | 224–239 | N/A      | N/A       | N/A      | N/A         | N/A                 | Multicast   |
| **E** | 11110      | 240–255 | N/A      | N/A       | N/A      | N/A         | N/A                 | Reserved    |

### Class Identification at a Glance

```
First octet of IP address:
  0   – 127  → Class A  (0xxxxxxx)
  128 – 191  → Class B  (10xxxxxx)
  192 – 223  → Class C  (110xxxxx)
  224 – 239  → Class D  (1110xxxx)
  240 – 255  → Class E  (11110xxx)

Quick trick:
  < 128  → A
  < 192  → B
  < 224  → C
  < 240  → D
  else   → E
```

---

## 9. 🔴 Security Context — IP Classes & Attacks

### Private IP Ranges — Security Implications

```
Private IPs (10.x.x.x, 172.16-31.x.x, 192.168.x.x):
  Internal networks use these → not directly accessible from internet
  NAT hides internal IPs behind one public IP

Attack: Bypassing NAT
  SSRF (Server-Side Request Forgery):
    Trick a web server into making requests to internal 10.x.x.x addresses
    Attacker → server → internal service (normally inaccessible)

  Example in your MERN/PERN app:
    Endpoint: GET /fetch?url=<user_input>
    Attacker sends: GET /fetch?url=http://192.168.56.1:3000/admin
    Server fetches your internal admin page → exposes it to attacker!
```

### Broadcast Address Attacks

```
Smurf Attack (exploits directed broadcast):
  1. Attacker spoofs victim's IP as source
  2. Sends ICMP echo to network's broadcast address (x.255.255.255)
  3. ALL hosts in that network reply to victim's spoofed IP
  4. Victim receives thousands of ICMP replies = DDoS

Example:
  Attacker spoofs src=10.0.1.5 (victim)
  Sends ICMP to 10.0.255.255 (broadcast)
  1000 hosts all reply to 10.0.1.5 simultaneously = flood

Defense: Block directed broadcast at routers (ip directed-broadcast disabled by default now)
```

### Loopback Exploitation

```
127.0.0.1 → localhost → local services

Attack scenario:
  Web app on server listens on 127.0.0.1:8080 (admin panel)
  "Only accessible locally — it's safe!"

  SSRF attack:
    POST /api/data HTTP/1.1
    Body: {"url": "http://127.0.0.1:8080/admin/delete-all-users"}

  Server thinks "localhost = safe" → executes request
  Attacker bypasses access controls via SSRF!

  Always validate/block localhost in URL inputs on web apps!
```

### IP Class-Based Firewall Bypass

```
Old firewalls used class-based rules:
  "Allow Class A private 10.0.0.0/8 traffic"
  "Block all Class C 192.168.0.0/16 traffic"

Attack: IP fragmentation to confuse class detection
  Send fragmented packets where class bits are in different fragments
  Poorly implemented firewall misclassifies → bypass

Modern firewalls use CIDR + stateful inspection → not vulnerable
```

### Reconnaissance — IP Class Tells You About Target

```
Target IP: 10.20.30.40
  First octet = 10 → Class A private (RFC 1918)
  This is an INTERNAL network address
  You're inside the network or dealing with VPN/tunnel

Target IP: 64.233.160.0
  First octet = 64 → Class A public
  This is Google's public IP range
  Large organization (Class A → up to 16M hosts)

Target IP: 192.168.1.0
  First octet = 192 → Class C private
  Small home/office network
  Likely 254 hosts max

This class identification helps attackers plan their approach:
  Class A target → large org → complex security → APT techniques
  Class C target → small org → simpler security → opportunistic attacks
```

---

## 10. 🧪 Practical Labs

### Lab 1 — Identify IP Classes on Your Network

```bash
# View your own IP address and identify its class
ip addr show

# Your VirtualBox IP: 192.168.56.x
# First octet: 192 → 192-223 range → CLASS C ✓
# Private Class C network

# Check Metasploitable2
ping -c 1 192.168.56.101
# 192.168.56.101 → Class C (192 in range 192-223)

# Check loopback
ping -c 1 127.0.0.1
# 127 → reserved loopback range
# This NEVER leaves your machine

# Check internet IP (what the world sees you as)
curl -s ifconfig.me
# This is your PUBLIC IP (likely Class A or B, assigned by your ISP)
```

### Lab 2 — Subnet Mask & Network ID Extraction

```bash
# See subnet mask of your interfaces
ip addr show | grep "inet "
# Example: inet 192.168.56.102/24
# /24 = 255.255.255.0 = Class C mask

# Python script to extract Network ID
python3 - << 'EOF'
import ipaddress

# Test various IPs
test_ips = [
    ("64.0.0.8",    "255.0.0.0"),      # Class A
    ("130.10.5.20", "255.255.0.0"),    # Class B
    ("192.168.56.101", "255.255.255.0"), # Class C (your lab)
    ("10.0.0.5",    "255.0.0.0"),      # Class A private
]

print(f"{'IP Address':<20} {'Mask':<16} {'Network ID':<20} {'Class'}")
print("-" * 70)

for ip_str, mask_str in test_ips:
    ip = ipaddress.IPv4Address(ip_str)
    mask = ipaddress.IPv4Address(mask_str)

    # AND operation to get network ID
    network_int = int(ip) & int(mask)
    network_id = ipaddress.IPv4Address(network_int)

    # Determine class
    first_octet = int(ip_str.split('.')[0])
    if first_octet < 128: cls = "A"
    elif first_octet < 192: cls = "B"
    elif first_octet < 224: cls = "C"
    elif first_octet < 240: cls = "D"
    else: cls = "E"

    print(f"{ip_str:<20} {mask_str:<16} {str(network_id):<20} {cls}")
EOF
```

### Lab 3 — Broadcast Address Detection and Testing

```bash
# Find broadcast address of your network
ip addr show eth0 | grep "inet "
# inet 192.168.56.102/24 brd 192.168.56.255 scope global eth0
#                         ↑ this is the broadcast address!

# Ping broadcast (sends to ALL hosts on network)
# Use with caution — generates traffic from all live hosts
ping -b 192.168.56.255 -c 3
# -b = allow ping to broadcast address

# In Wireshark, you'll see:
# Multiple ICMP replies from 192.168.56.101, 192.168.56.1, etc.
# This demonstrates broadcast address behavior

# Calculate broadcast address for any IP/mask
python3 - << 'EOF'
import ipaddress

networks = [
    "192.168.56.0/24",
    "10.0.0.0/8",
    "172.16.0.0/16",
    "64.0.0.0/8",
]

for net_str in networks:
    net = ipaddress.IPv4Network(net_str)
    print(f"Network:    {net.network_address}")
    print(f"Broadcast:  {net.broadcast_address}")
    print(f"First host: {list(net.hosts())[0]}")
    print(f"Last host:  {list(net.hosts())[-1]}")
    print(f"Total hosts:{net.num_addresses - 2}")
    print()
EOF
```

### Lab 4 — SSRF via Loopback (Your MERN/PERN App)

```bash
# Test if your Node.js app is vulnerable to SSRF via localhost

# Start your app first (if not running)
# cd /your/app && node server.js

# Test 1: Normal external request (should work)
curl -v "http://localhost:3000/api/data"

# Test 2: SSRF — can your app be tricked to fetch localhost?
# If your app has a URL fetch endpoint:
curl "http://localhost:3000/api/fetch?url=http://127.0.0.1:3000/admin"
# If this returns admin content → SSRF vulnerability!

# Test 3: SSRF to other internal services
curl "http://localhost:3000/api/fetch?url=http://192.168.56.101/admin"
# If successful → your app can reach Metasploitable2's admin page!

# Defense: In your Node.js app, validate URLs
# Reject: localhost, 127.x.x.x, 192.168.x.x, 10.x.x.x, 172.16-31.x.x
cat << 'EOF'
// Node.js SSRF defense example
const { URL } = require('url');

function isPrivateIP(hostname) {
  const privateRanges = [
    /^127\./,           // loopback
    /^10\./,            // Class A private
    /^192\.168\./,      // Class C private
    /^172\.(1[6-9]|2[0-9]|3[01])\./,  // Class B private
    /^0\./,             // reserved
    /^169\.254\./,      // APIPA
    /^localhost$/i,
  ];
  return privateRanges.some(range => range.test(hostname));
}

function validateUrl(userInput) {
  try {
    const url = new URL(userInput);
    if (isPrivateIP(url.hostname)) {
      throw new Error('SSRF attempt blocked: private IP detected');
    }
    return url.href;
  } catch (e) {
    throw new Error('Invalid or blocked URL: ' + e.message);
  }
}
EOF
```

### Lab 5 — Scan by IP Class (Network Reconnaissance)

```bash
# Identify what class your target's IP belongs to
# This informs the scope of the network

TARGET="192.168.56.0/24"   # Your Class C lab network

# Scan the full /24 (Class C — 254 hosts)
nmap -sn $TARGET
# Small network (Class C) → quick scan

# For a Class A private network (large!)
# nmap -sn 10.0.0.0/8 → 16 million hosts → use carefully

# Smarter: scan only active ranges
nmap -sn 10.0.0.0/24    # Scan first /24 of 10.x.x.x

# Identify network class of discovered hosts
nmap -sn 192.168.56.0/24 -oG - | grep "Host:" | awk '{print $2}' | \
python3 - << 'EOF'
import sys
for ip in sys.stdin:
    ip = ip.strip()
    if not ip: continue
    first = int(ip.split('.')[0])
    if first < 128: cls = "A"
    elif first < 192: cls = "B"
    elif first < 224: cls = "C"
    else: cls = "D/E"
    print(f"{ip:<20} Class {cls}")
EOF
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║         IP ADDRESSING — CLASS A EXAM CHEAT SHEET                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS IDENTIFICATION (from first octet)                             ║
║  0   – 127  → Class A   (1st bit = 0)                              ║
║  128 – 191  → Class B   (1st two bits = 10)                        ║
║  192 – 223  → Class C   (1st three bits = 110)                     ║
║  224 – 239  → Class D   (Multicast)                                ║
║  240 – 255  → Class E   (Reserved)                                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS A SPECIFICS                                                   ║
║  First bit:    Fixed = 0                                            ║
║  Network bits: 8 (first octet) — but 1 fixed, 7 variable          ║
║  Host bits:    24 (last 3 octets)                                  ║
║  Range:        0.0.0.0 – 127.255.255.255                           ║
║  Default mask: 255.0.0.0  (/8)                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS A CALCULATIONS                                                ║
║  Total networks:   2⁷ = 128                                        ║
║  Usable networks:  128 - 2 = 126                                   ║
║    (0.x.x.x = null, 127.x.x.x = loopback — both reserved)        ║
║  Hosts per network:  2²⁴ = 16,777,216                              ║
║  Usable hosts:       2²⁴ - 2 = 16,777,214                         ║
║    (first = network addr, last = broadcast addr — both reserved)   ║
╠══════════════════════════════════════════════════════════════════════╣
║  ADDRESS TYPES                                                       ║
║  Network address:   host bits all 0  (e.g., 64.0.0.0)             ║
║  Broadcast address: host bits all 1  (e.g., 64.255.255.255)       ║
║  First host:        network addr + 1 (e.g., 64.0.0.1)            ║
║  Last host:         broadcast - 1    (e.g., 64.255.255.254)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  PRIVATE IP RANGES (RFC 1918)                                        ║
║  Class A: 10.0.0.0/8      (10.x.x.x)                              ║
║  Class B: 172.16.0.0/12   (172.16-31.x.x)                         ║
║  Class C: 192.168.0.0/16  (192.168.x.x)                           ║
║  Private IPs not routable on internet → requires NAT              ║
╠══════════════════════════════════════════════════════════════════════╣
║  NETWORK ID EXTRACTION                                               ║
║  IP AND Subnet Mask = Network ID                                    ║
║  AND with 255 → preserves bits (network portion)                  ║
║  AND with 0   → zeros bits (host portion)                         ║
║  Example: 64.0.0.8 AND 255.0.0.0 = 64.0.0.0 (network)           ║
╠══════════════════════════════════════════════════════════════════════╣
║  SPECIAL ADDRESSES                                                   ║
║  0.0.0.0         → Unspecified/null (not assigned)                 ║
║  127.0.0.1       → Loopback (localhost)                            ║
║  255.255.255.255  → Limited broadcast (local network)              ║
║  x.255.255.255   → Directed broadcast (to network x)              ║
╠══════════════════════════════════════════════════════════════════════╣
║  ALL CLASSES SUMMARY                                                 ║
║  A: /8,  126 networks,   16M hosts  → large orgs                  ║
║  B: /16, 16382 networks, 65K hosts  → medium orgs                 ║
║  C: /24, ~2M networks,   254 hosts  → small orgs                  ║
║  D: Multicast (no network/host split)                              ║
║  E: Reserved/Experimental                                          ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  SSRF → attacker tricks server to fetch 127.0.0.1/10.x.x.x       ║
║  Smurf → spoofed ICMP to broadcast addr → amplified DDoS          ║
║  Private IPs in headers → internal network leakage                 ║
║  Class A public → large org → complex attack surface              ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Class B IP Addressing — 128-191 range
- [ ] Class C IP Addressing — 192-223 range
- [ ] Subnetting — VLSM, CIDR
- [ ] NAT — Network Address Translation
- [ ] ARP — How IP → MAC resolution works

---

_Notes compiled from: Networking Course Lecture 11 — Class A IP Addressing_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
