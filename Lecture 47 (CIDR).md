# 🔀 Subnetting in CIDR — Classless Inter-Domain Routing

### Cybersecurity Student Notes | Networking Course — Lecture 13 (Series Lecture 47)

> **Source:** Gate Smashers — Subnetting in CIDR
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [CIDR vs Classful — Key Difference](#1-cidr-vs-classful--key-difference)
2. [CIDR Slash Notation](#2-cidr-slash-notation)
3. [Reading a CIDR Address](#3-reading-a-cidr-address)
4. [Finding Network ID from Host ID](#4-finding-network-id-from-host-id)
5. [Subnetting in CIDR — Core Concept](#5-subnetting-in-cidr--core-concept)
6. [Worked Example — 195.10.20.128/26](#6-worked-example--1951020128-26)
   - [6.1 Setup — Identify Bits](#61-setup--identify-bits)
   - [6.2 Create Subnet 1 (S1)](#62-create-subnet-1-s1)
   - [6.3 Create Subnet 2 (S2)](#63-create-subnet-2-s2)
   - [6.4 Slash Notation After Subnetting](#64-slash-notation-after-subnetting)
7. [Subnet Summary Table](#7-subnet-summary-table)
8. [General Subnetting Formula](#8-general-subnetting-formula)
9. [More Examples](#9-more-examples)
10. [VLSM — Variable Length Subnet Masking](#10-vlsm--variable-length-subnet-masking)
11. [🔴 Security Context — Subnetting & Network Segmentation](#11--security-context--subnetting--network-segmentation)
12. [🧪 Practical Labs](#12--practical-labs)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. CIDR vs Classful — Key Difference

### Classful (Old — 1981–1993)

```
IP address class determined by first octet value.
Fixed network/host split:
  Class A: /8   (8 network bits)
  Class B: /16  (16 network bits)
  Class C: /24  (24 network bits)

Problem: Inflexible. Company needing 500 hosts gets entire Class B (65,534 hosts) — wasteful!
```

### CIDR (Current — since 1993)

```
No classes. Network size defined by /prefix_length.
Can have ANY prefix length: /8, /12, /19, /26, /30...

Company needing 500 hosts → /23 (510 hosts) — efficient!
Company needing 50 hosts  → /26 (62 hosts)  — efficient!

Key idea: network bits are counted by the slash number, not by IP class.
```

---

## 2. CIDR Slash Notation

```
IP_address / prefix_length

Example: 195.10.20.128/26

  195.10.20.128  = the IP address (or network address)
  /26            = 26 bits are NETWORK bits (the prefix)

Derivation:
  Total bits = 32
  Network bits = 26
  Host bits = 32 - 26 = 6
```

### Slash → Subnet Mask Conversion

```
/prefix → write (prefix) ones, then zeros to fill 32 bits

/26: 11111111.11111111.11111111.11000000
      ↑ 8 ones  ↑ 8 ones  ↑ 8 ones  ↑ 2 ones, 6 zeros

Convert to decimal:
  11111111 = 255
  11111111 = 255
  11111111 = 255
  11000000 = 128+64 = 192

Subnet mask = 255.255.255.192
```

### Common CIDR Prefix Reference

| Prefix | Subnet Mask     | Host Bits | Total Hosts | Usable Hosts |
| ------ | --------------- | --------- | ----------- | ------------ |
| /24    | 255.255.255.0   | 8         | 256         | 254          |
| /25    | 255.255.255.128 | 7         | 128         | 126          |
| /26    | 255.255.255.192 | 6         | 64          | 62           |
| /27    | 255.255.255.224 | 5         | 32          | 30           |
| /28    | 255.255.255.240 | 4         | 16          | 14           |
| /29    | 255.255.255.248 | 3         | 8           | 6            |
| /30    | 255.255.255.252 | 2         | 4           | 2            |

---

## 3. Reading a CIDR Address

### 195.10.20.128/26 — Full Breakdown

```
IP: 195.10.20.128
    └─Oct1─┘└Oct2┘└Oct3┘└Oct4─┘

Binary of each octet:
  195 = 11000011
  10  = 00001010
  20  = 00010100
  128 = 10000000

Full 32-bit binary:
  11000011.00001010.00010100.10000000
  └──────────────── 26 ──────────────┘└── 6 ──┘
         NETWORK BITS                  HOST BITS

/26 means: first 26 bits = network, last 6 bits = host

Since 8+8+8 = 24 bits fit in first 3 octets:
  First 3 octets  → entirely network bits (don't touch!)
  4th octet       → first 2 bits = network, last 6 bits = host
```

### Visualizing the Split in 4th Octet

```
4th octet: 1 0 0 0 0 0 0 0  = 128
            ↑ ↑ ↑ ↑ ↑ ↑ ↑ ↑
            │ │ │ │ │ │ │ │
            N N H H H H H H   (N=network, H=host)
            ↑ ↑
         2 more network bits from /26 (24 already done in first 3 octets)
         (these 2 bits = the "10" prefix coming from the network portion)
```

---

## 4. Finding Network ID from Host ID

Sometimes the given IP is a **host IP** (not the network address). You must find the network ID first.

### Example: Given 195.10.20.129/26

```
Step 1: Open 4th octet in binary
  129 = 10000001

Step 2: Count host bits from /26
  32 - 26 = 6 host bits → last 6 bits are host bits

Step 3: Set all host bits to 0 to get Network ID
  129 = 1 0 | 0 0 0 0 0 1   ← last 6 bits are host
              └─ set to 0 ──┘
  Result: 1 0 0 0 0 0 0 0 = 128

Step 4: Network ID = 195.10.20.128  ← same as our /26 example!
  (129 is just the first host in that network)
```

### Golden Rule

```
To find Network ID from any IP:
  Set all HOST BITS to 0 → that gives the Network ID

To find Broadcast Address from any IP:
  Set all HOST BITS to 1 → that gives the Broadcast Address
```

---

## 5. Subnetting in CIDR — Core Concept

### What is Subnetting?

> Dividing one larger network into multiple smaller sub-networks (subnets).

### The Golden Rule of Subnetting

```
NEVER disturb the NETWORK BITS.
ONLY borrow from the HOST BITS.
```

### How to Create Subnets

```
Original: /26 → 6 host bits → 64 addresses

To create 2 subnets:
  Borrow 1 bit from host bits → fix it
  1 bit borrowed → 2¹ = 2 subnets created
  Remaining host bits = 6 - 1 = 5 → each subnet has 2⁵ = 32 addresses

To create 4 subnets:
  Borrow 2 bits → 2² = 4 subnets
  Remaining host bits = 4 → each subnet has 2⁴ = 16 addresses

To create 8 subnets:
  Borrow 3 bits → 2³ = 8 subnets
  Remaining = 3 host bits → 2³ = 8 addresses each

General formula:
  Bits borrowed = n → subnets = 2ⁿ
  New prefix = original prefix + n
  Hosts per subnet = 2^(remaining_host_bits) - 2
```

---

## 6. Worked Example — 195.10.20.128/26

### 6.1 Setup — Identify Bits

```
Network: 195.10.20.128/26

Total bits:   32
Network bits: 26
Host bits:    32 - 26 = 6

4th octet (128) in binary: 1 0 0 0 0 0 0 0
                            N N H H H H H H
                            ↑ ↑ ← 2 network bits (complete /26)
                                └──────────── 6 host bits

Total hosts in /26: 2⁶ = 64
Usable hosts:       64 - 2 = 62
```

### 6.2 Create Subnet 1 (S1)

**Fix the most significant host bit (MSB of host portion) to 0:**

```
4th octet bit pattern (before subnetting):
  N  N  H  H  H  H  H  H
  1  0  0  0  0  0  0  0  ← original 128, host bits = 000000

Fix MSB of host to 0:
  N  N  [0] H  H  H  H  H   ← now 3 network bits, 5 host bits
  1  0   0  ?  ?  ?  ?  ?

Subnet 1 (S1): MSB of host = 0
  Range of remaining 5 host bits: 00000 to 11111

  First address (all host=0):
    1 0 0 | 0 0 0 0 0 = 128 → 195.10.20.128  ← S1 Network ID

  Last address (all host=1):
    1 0 0 | 1 1 1 1 1 = 128+31 = 159 → 195.10.20.159  ← S1 Broadcast

  First usable host: 195.10.20.129
  Last usable host:  195.10.20.158
  Usable hosts: 2⁵ - 2 = 30
```

```
S1 visual:
10 [0] 00000  = 128  ← Network ID
10 [0] 00001  = 129  ← First host
10 [0] 00010  = 130
...
10 [0] 11110  = 158  ← Last host
10 [0] 11111  = 159  ← Broadcast
         ↑ fixed to 0 for S1
```

### 6.3 Create Subnet 2 (S2)

**Fix the MSB of host portion to 1:**

```
Subnet 2 (S2): MSB of host = 1
  N  N  [1] H  H  H  H  H

  First address (remaining host=0):
    1 0 1 | 0 0 0 0 0 = 128+32 = 160 → 195.10.20.160  ← S2 Network ID

  Last address (remaining host=1):
    1 0 1 | 1 1 1 1 1 = 128+32+31 = 191 → 195.10.20.191  ← S2 Broadcast

  First usable host: 195.10.20.161
  Last usable host:  195.10.20.190
  Usable hosts: 2⁵ - 2 = 30
```

```
S2 visual:
10 [1] 00000  = 160  ← Network ID
10 [1] 00001  = 161  ← First host
...
10 [1] 11110  = 190  ← Last host
10 [1] 11111  = 191  ← Broadcast
         ↑ fixed to 1 for S2
```

### 6.4 Slash Notation After Subnetting

```
Original network: /26 (26 fixed bits)
Borrowed 1 bit from host → now 27 bits fixed

New prefix: /27

S1: 195.10.20.128/27
S2: 195.10.20.160/27

Verification:
  /27 → 32 - 27 = 5 host bits → 2⁵ = 32 addresses per subnet ✓
  Two /27 subnets = 32 + 32 = 64 addresses = original /26 ✓
```

---

## 7. Subnet Summary Table

|                  | Original Network | Subnet 1 (S1)    | Subnet 2 (S2)    |
| ---------------- | ---------------- | ---------------- | ---------------- |
| **CIDR**         | 195.10.20.128/26 | 195.10.20.128/27 | 195.10.20.160/27 |
| **Network ID**   | 195.10.20.128    | 195.10.20.128    | 195.10.20.160    |
| **First Host**   | 195.10.20.129    | 195.10.20.129    | 195.10.20.161    |
| **Last Host**    | 195.10.20.190    | 195.10.20.158    | 195.10.20.190    |
| **Broadcast**    | 195.10.20.191    | 195.10.20.159    | 195.10.20.191    |
| **Total addr**   | 64               | 32               | 32               |
| **Usable hosts** | 62               | 30               | 30               |
| **Subnet mask**  | 255.255.255.192  | 255.255.255.224  | 255.255.255.224  |

```
Address space view:
195.10.20.128 ─────────────────────────────── 195.10.20.191
│← ─── ─── ─── ─── /26 (64 addresses) ─── ─── ─── ─── ─── →│
│←  ─── /27 (32) ──→│← ─── /27 (32) ─── →│
128              159 160                191
└───── S1 ──────────┘└───────── S2 ─────────┘
```

---

## 8. General Subnetting Formula

```
Given: Network X.X.X.X/n, want to create 2^k subnets

Bits to borrow: k
New prefix:     n + k
Subnets created: 2^k
Addresses per subnet: 2^(32-n-k)
Usable hosts per subnet: 2^(32-n-k) - 2

Total usable hosts:
  Before: 2^(32-n) - 2
  After:  2^k × (2^(32-n-k) - 2) = 2^(32-n) - 2^(k+1)

  Loss = 2^(k+1) - 2 additional addresses wasted on network/broadcast IDs
  (Each subnet wastes 2 addresses instead of the original 1 network + 1 broadcast)
```

### Quick Reference: Subnetting /26

| Bits Borrowed | Subnets | New Prefix | Hosts/Subnet | Usable/Subnet |
| ------------- | ------- | ---------- | ------------ | ------------- |
| 1             | 2       | /27        | 32           | 30            |
| 2             | 4       | /28        | 16           | 14            |
| 3             | 8       | /29        | 8            | 6             |
| 4             | 16      | /30        | 4            | 2             |

---

## 9. More Examples

### Example 2: 192.168.1.0/24 → 4 subnets

```
Original: /24, 8 host bits, 256 addresses, 254 usable hosts
Want: 4 subnets → borrow 2 bits (2² = 4)
New prefix: /26
Host bits remaining: 6 → 64 addresses per subnet, 62 usable

Subnet 1: 192.168.1.0/26    → 192.168.1.0   – 192.168.1.63
Subnet 2: 192.168.1.64/26   → 192.168.1.64  – 192.168.1.127
Subnet 3: 192.168.1.128/26  → 192.168.1.128 – 192.168.1.191
Subnet 4: 192.168.1.192/26  → 192.168.1.192 – 192.168.1.255

Pattern: each subnet's network ID = previous broadcast + 1
  0, 64, 128, 192 → jumps of 64 (= 2⁶ = addresses per subnet)
```

### Example 3: 10.0.0.0/8 → 2 subnets

```
Original: /8 Class A, 24 host bits
Borrow 1 bit → /9, 2 subnets, 2²³ = 8,388,608 addresses each

Subnet 1: 10.0.0.0/9    → 10.0.0.0   – 10.127.255.255
Subnet 2: 10.128.0.0/9  → 10.128.0.0 – 10.255.255.255
```

### The "Jump Size" Shortcut

```
Jump size between subnet Network IDs = 2^(host_bits_remaining)

For /26 → /27: jump = 2⁵ = 32
  128, 160, (192, 224 if continued)

For /24 → /26: jump = 2⁶ = 64
  0, 64, 128, 192

For /24 → /27: jump = 2⁵ = 32
  0, 32, 64, 96, 128, 160, 192, 224
```

---

## 10. VLSM — Variable Length Subnet Masking

VLSM = subnets of **different sizes** from the same parent network.

```
Problem: You have 192.168.1.0/24 and need:
  Dept A: 100 hosts
  Dept B: 50 hosts
  Dept C: 25 hosts
  WAN links: 2 hosts each (×3)

Fixed-size subnetting wastes addresses.
VLSM allocates exactly what's needed.

Step 1: Sort by size (largest first)
  Dept A: 100 hosts → need 2^7=128 → /25 (126 usable)
  Dept B: 50 hosts  → need 2^6=64  → /26 (62 usable)
  Dept C: 25 hosts  → need 2^5=32  → /27 (30 usable)
  WAN×3:  2 hosts   → need 2^2=4   → /30 (2 usable)

Step 2: Allocate sequentially
  Dept A: 192.168.1.0/25   → .0   – .127
  Dept B: 192.168.1.128/26 → .128 – .191
  Dept C: 192.168.1.192/27 → .192 – .223
  WAN1:   192.168.1.224/30 → .224 – .227
  WAN2:   192.168.1.228/30 → .228 – .231
  WAN3:   192.168.1.232/30 → .232 – .235
  Free:   192.168.1.236/30 → .236 – .255 (spare)
```

---

## 11. 🔴 Security Context — Subnetting & Network Segmentation

### Subnetting as a Security Tool

```
Security principle: Network Segmentation
  Divide network into subnets → limit attacker's lateral movement
  Compromise one subnet → cannot easily reach others

Without segmentation:
  [PCs] [Servers] [IoT] [CCTV] all on 192.168.1.0/24
  Attacker compromises a PC → can reach ALL servers, IoT, CCTV

With segmentation (subnetting):
  192.168.1.0/26    → PCs subnet
  192.168.1.64/26   → Server subnet
  192.168.1.128/26  → IoT subnet
  192.168.1.192/26  → Management subnet

  Firewall/ACL rules between subnets:
    PCs can reach Servers (ports 80, 443 only)
    PCs CANNOT reach IoT or Management subnets
    IoT devices cannot reach any other subnet (internet-only)
```

### VLANs + Subnets (Common in Real Networks)

```
VLAN 10 + 192.168.10.0/24 → HR Department
VLAN 20 + 192.168.20.0/24 → Engineering
VLAN 30 + 192.168.30.0/24 → Finance

Attacker on VLAN 10 (HR):
  Cannot directly reach VLAN 20 or 30
  Traffic must pass through router/firewall with ACLs

VLAN Hopping Attack bypass attempt:
  Double-tagging 802.1Q frames → bypass VLAN isolation
  Defense: Use non-default native VLAN, disable trunk auto-negotiation
```

### Subnet Scanning — Attacker Perspective

```
Attacker discovers target: 195.10.20.128/26
  Knows: 64 addresses in this block (128 to 191)
  Scans: nmap -sn 195.10.20.128/26

  If network is subnetted into /27:
    S1: 195.10.20.128/27 → 30 hosts
    S2: 195.10.20.160/27 → 30 hosts

  Attacker sees: some hosts at .128-.158, others at .160-.190
  Can infer: two /27 subnets exist

  Security lesson: Subnetting alone doesn't hide your network
  Attacker just needs to scan the range.
```

### CIDR Notation in Firewall Rules

```
Firewall rules use CIDR to allow/deny ranges efficiently:

Allow internal users:
  iptables -A INPUT -s 192.168.1.0/24 -j ACCEPT

Block specific subnet:
  iptables -A INPUT -s 10.10.20.0/27 -j DROP  ← blocks 30 hosts in one rule

Without CIDR, you'd need 30 individual rules!

Mistake: Wrong prefix length
  iptables -A INPUT -s 192.168.1.0/23 -j ACCEPT
  Accidentally allows 192.168.0.0 – 192.168.1.255 (512 hosts, not 256!)
  Always calculate prefix carefully.
```

### Subnet Exhaustion Attack

```
DHCP + IPAM attacks:
  If attacker gets inside network → requests many IPs (DHCP starvation)
  Small subnets (/29 = 6 usable, /30 = 2 usable) exhaust quickly
  Result: Legitimate hosts can't get IP → DoS

Defense:
  Use appropriately sized subnets (not too small for expected growth)
  Enable DHCP snooping on switches
  Set DHCP rate limits per port
```

---

## 12. 🧪 Practical Labs

### Lab 1 — Subnet Calculator in Python (Full Implementation)

```python
# Save as subnet_calculator.py — run: python3 subnet_calculator.py
import ipaddress

def subnet_info(cidr: str) -> dict:
    """Get complete info about a network"""
    net = ipaddress.IPv4Network(cidr, strict=False)
    hosts = list(net.hosts())
    return {
        'cidr': cidr,
        'network': str(net.network_address),
        'broadcast': str(net.broadcast_address),
        'first_host': str(hosts[0]) if hosts else 'N/A',
        'last_host': str(hosts[-1]) if hosts else 'N/A',
        'total': net.num_addresses,
        'usable': len(hosts),
        'mask': str(net.netmask),
        'prefix': net.prefixlen,
        'host_bits': 32 - net.prefixlen,
    }

def create_subnets(parent_cidr: str, new_prefix: int) -> list:
    """Split a network into subnets of given prefix length"""
    parent = ipaddress.IPv4Network(parent_cidr, strict=False)
    return list(parent.subnets(new_prefix=new_prefix))

# ─── Lecture Example ───
print("=" * 60)
print("LECTURE EXAMPLE: 195.10.20.128/26 → 2 subnets (/27)")
print("=" * 60)

parent = "195.10.20.128/26"
info = subnet_info(parent)
print(f"\nParent Network:")
for k, v in info.items():
    print(f"  {k:<15}: {v}")

subnets = create_subnets(parent, 27)
print(f"\nSubnets created (borrowing 1 bit → /27):")
for i, subnet in enumerate(subnets, 1):
    s = subnet_info(str(subnet))
    print(f"\n  Subnet S{i}: {subnet}")
    print(f"    Network ID:    {s['network']}")
    print(f"    First host:    {s['first_host']}")
    print(f"    Last host:     {s['last_host']}")
    print(f"    Broadcast:     {s['broadcast']}")
    print(f"    Usable hosts:  {s['usable']}")

# ─── General examples ───
print("\n" + "=" * 60)
print("MORE EXAMPLES")
print("=" * 60)

examples = [
    ("192.168.1.0/24", 26, "4 subnets of /26"),
    ("10.0.0.0/8",     10, "4 subnets of /10"),
    ("172.16.0.0/16",  18, "4 subnets of /18"),
]

for parent_cidr, new_pfx, desc in examples:
    print(f"\n{parent_cidr} → {desc}:")
    subnets = create_subnets(parent_cidr, new_pfx)
    for i, s in enumerate(subnets[:4], 1):
        net = ipaddress.IPv4Network(str(s))
        hosts = list(net.hosts())
        print(f"  S{i}: {s}  hosts: {hosts[0]} – {hosts[-1]}  ({len(hosts)} usable)")
```

### Lab 2 — Subnet Your VirtualBox Network

```bash
# Your VirtualBox network: 192.168.56.0/24
# Let's subnet it into 4 /26 subnets

python3 - << 'EOF'
import ipaddress

parent = ipaddress.IPv4Network("192.168.56.0/24")
subnets = list(parent.subnets(new_prefix=26))

print("Subnetting 192.168.56.0/24 into 4 × /26 subnets")
print("(Your VirtualBox network segmented for security)")
print()
for i, subnet in enumerate(subnets, 1):
    hosts = list(subnet.hosts())
    label = ["Management", "Attacker VMs", "Vulnerable VMs", "Spare"][i-1]
    print(f"Subnet {i} ({label}): {subnet}")
    print(f"  Network:    {subnet.network_address}")
    print(f"  Hosts:      {hosts[0]} – {hosts[-1]}")
    print(f"  Broadcast:  {subnet.broadcast_address}")
    print(f"  Usable:     {len(hosts)}")
    print()

print("Your Parrot OS:       192.168.56.102 → Subnet 2 (Attacker VMs)")
print("Metasploitable2:      192.168.56.101 → Subnet 2 (Attacker VMs)")
print("To isolate them, put Metasploitable2 in Subnet 3 (Vulnerable VMs)")
EOF
```

### Lab 3 — Implement Network Segmentation with iptables

```bash
# Simulate subnet-based segmentation using iptables
# Scenario: Two /27 subnets from 192.168.56.0/26
#   Management subnet: 192.168.56.0/27  (IPs .1-.30)
#   User subnet:       192.168.56.32/27 (IPs .33-.62)

# View current rules
sudo iptables -L -n -v

# Block traffic between hypothetical subnets (drop all cross-subnet traffic)
# This simulates having a firewall between subnets
sudo iptables -A FORWARD -s 192.168.56.0/27 -d 192.168.56.32/27 -j DROP
sudo iptables -A FORWARD -s 192.168.56.32/27 -d 192.168.56.0/27 -j DROP

# Allow only specific port (e.g., allow web from user subnet to management)
sudo iptables -I FORWARD 1 -s 192.168.56.32/27 -d 192.168.56.0/27 -p tcp --dport 80 -j ACCEPT

# View new rules
sudo iptables -L FORWARD -n -v

# Clean up after testing
sudo iptables -F FORWARD
```

### Lab 4 — Scan a Specific Subnet

```bash
# After subnetting, scan each /27 separately to understand scope

# Subnet 1 of your lab (/27 simulation)
nmap -sn 192.168.56.0/27
# Expected: finds hosts .1-.30 range only

# Subnet 2 (where Metasploitable2 lives in this simulation)
nmap -sn 192.168.56.96/27
# Expected: finds .97-.126

# Full scan of parent /24 to find all
nmap -sn 192.168.56.0/24

# Show what nmap sees as network boundaries
nmap --iflist | grep -E "DEV|^eth"
# Shows your network interfaces and their subnets
```

### Lab 5 — Subnetting Quiz Script

```python
# Save as subnet_quiz.py — run: python3 subnet_quiz.py
# Practice subnetting for exams

import ipaddress
import random

def generate_question():
    """Generate a random subnetting question"""
    # Random /24, /26, /27 network
    prefix = random.choice([24, 25, 26, 27])
    a = random.randint(10, 220)
    b = random.randint(0, 255)
    c = random.randint(0, 255)
    # Make valid network address
    net = ipaddress.IPv4Network(f"{a}.{b}.{c}.0/{prefix}", strict=False)
    hosts = list(net.hosts())

    borrow = random.randint(1, min(3, 32-prefix-2))  # borrow 1-3 bits
    new_prefix = prefix + borrow
    subnets = list(net.subnets(new_prefix=new_prefix))

    return {
        'parent': str(net),
        'borrow': borrow,
        'new_prefix': new_prefix,
        'num_subnets': 2**borrow,
        'subnets': subnets[:4],  # show first 4
    }

print("=" * 55)
print("SUBNETTING PRACTICE QUIZ")
print("=" * 55)

for q_num in range(3):
    q = generate_question()
    print(f"\nQuestion {q_num+1}:")
    print(f"  Network: {q['parent']}")
    print(f"  Borrow {q['borrow']} bit(s) to create {q['num_subnets']} subnets")
    print(f"  New prefix: /{q['new_prefix']}")
    input("  Press ENTER to see answer...")

    print(f"\n  Answer — {q['num_subnets']} subnets of /{q['new_prefix']}:")
    for i, subnet in enumerate(q['subnets'], 1):
        net = ipaddress.IPv4Network(str(subnet))
        hosts = list(net.hosts())
        print(f"    S{i}: {subnet}  →  hosts: {hosts[0]} – {hosts[-1]}  ({len(hosts)} usable)")
    print()
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           SUBNETTING IN CIDR — EXAM CHEAT SHEET                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  CIDR NOTATION                                                       ║
║  X.X.X.X/n → n = number of network bits                            ║
║  Host bits = 32 - n                                                 ║
║  Total addresses = 2^(32-n)                                        ║
║  Usable hosts    = 2^(32-n) - 2                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  SUBNETTING STEPS                                                    ║
║  1. Identify host bits: 32 - prefix                                ║
║  2. Borrow k bits (from MSB of host bits)                          ║
║  3. Subnets created: 2^k                                           ║
║  4. New prefix: original + k                                       ║
║  5. Hosts per subnet: 2^(remaining_host_bits) - 2                 ║
║  6. Jump size = 2^(remaining_host_bits)                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  GOLDEN RULE: NEVER DISTURB NETWORK BITS                            ║
║  Only borrow from HOST bits                                         ║
║  Fix borrowed bit to 0 → Subnet 1                                  ║
║  Fix borrowed bit to 1 → Subnet 2                                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  FINDING NETWORK ID FROM HOST IP                                     ║
║  Set all HOST BITS to 0 → Network Address                          ║
║  Set all HOST BITS to 1 → Broadcast Address                        ║
║  Example: 195.10.20.129/26                                         ║
║    129 = 10|000001 → host bits = 000001                            ║
║    Set to 0: 10|000000 = 128                                       ║
║    Network ID = 195.10.20.128 ✓                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  LECTURE EXAMPLE ANSWERS                                             ║
║  195.10.20.128/26 → borrow 1 bit → /27                            ║
║  S1: 195.10.20.128/27  (.128 – .159)  30 usable hosts             ║
║  S2: 195.10.20.160/27  (.160 – .191)  30 usable hosts             ║
║  Jump size: 2^5 = 32                                               ║
╠══════════════════════════════════════════════════════════════════════╣
║  COMMON PREFIX TABLE                                                 ║
║  /24 → 256 addr, 254 usable  │ /27 → 32 addr, 30 usable          ║
║  /25 → 128 addr, 126 usable  │ /28 → 16 addr, 14 usable          ║
║  /26 → 64 addr,  62 usable   │ /30 → 4 addr,  2 usable           ║
╠══════════════════════════════════════════════════════════════════════╣
║  SLASH AFTER SUBNETTING                                              ║
║  Borrow 1 bit: new slash = original + 1                            ║
║  Borrow 2 bits: new slash = original + 2                           ║
║  Original /26, borrow 1 → /27                                      ║
║  Original /24, borrow 2 → /26 (4 subnets)                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY USES                                                       ║
║  Segment networks → limit lateral movement                         ║
║  VLAN + subnet per department → isolate breach                     ║
║  /30 for WAN links → only 2 hosts, minimal waste                  ║
║  Firewall rules with CIDR → block/allow entire subnets            ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] VLSM (Variable Length Subnet Masking) — detailed examples
- [ ] Supernetting / Route Aggregation — combining smaller networks
- [ ] NAT (Network Address Translation) — private to public IP
- [ ] IPv6 Addressing — 128-bit, prefix notation
- [ ] Routing Algorithms — RIP, OSPF detail

---

_Notes compiled from: Networking Course Lecture 13 — Subnetting in CIDR (Series Lecture 47)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
