# 🔀 VLSM in CIDR — Variable Length Subnet Masking

### Cybersecurity Student Notes | Networking Course — Lecture 14

> **Source:** Gate Smashers — Variable Length Subnet Masking in CIDR
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is VLSM?](#1-what-is-vlsm)
2. [The Problem VLSM Solves](#2-the-problem-vlsm-solves)
3. [Reading the Given Network](#3-reading-the-given-network)
4. [The ISP Allocation Problem](#4-the-isp-allocation-problem)
5. [Step-by-Step VLSM Solution](#5-step-by-step-vlsm-solution)
   - [5.1 Split into Two Halves](#51-split-into-two-halves)
   - [5.2 Subnet 1 (S1) — Half](#52-subnet-1-s1--half)
   - [5.3 Split Remaining Half into Two Quarters](#53-split-remaining-half-into-two-quarters)
   - [5.4 Subnet 2 (S2) — One Quarter](#54-subnet-2-s2--one-quarter)
   - [5.5 Subnet 3 (S3) — One Quarter (ISP keeps)](#55-subnet-3-s3--one-quarter-isp-keeps)
6. [Complete VLSM Summary Table](#6-complete-vlsm-summary-table)
7. [Slash Notation After Each Split](#7-slash-notation-after-each-split)
8. [The 0 vs 1 Fix — Exam Trick](#8-the-0-vs-1-fix--exam-trick)
9. [General VLSM Algorithm](#9-general-vlsm-algorithm)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. What is VLSM?

> **VLSM (Variable Length Subnet Masking)** = Creating subnets of **different sizes** from the same parent network block.

```
Regular subnetting:  One parent → equal-sized subnets
VLSM:               One parent → different-sized subnets

Example:
  Parent: /20 (4096 addresses)

  Regular: 4 × /22 (all equal, 1024 each)

  VLSM:    1 × /21 (2048 = half)
            1 × /22 (1024 = quarter)
            1 × /22 (1024 = quarter, kept by ISP)
```

---

## 2. The Problem VLSM Solves

```
Without VLSM:
  Company A needs 2000 hosts → gets /21 (2046 hosts) → OK
  Company B needs 500 hosts  → gets /21 (2046 hosts) → 1500 IPs WASTED
  Company C needs 50 hosts   → gets /21 (2046 hosts) → 1996 IPs WASTED

With VLSM:
  Company A needs 2000 hosts → gets /21 (2046 hosts) ← just right
  Company B needs 500 hosts  → gets /23 (510 hosts)  ← just right
  Company C needs 50 hosts   → gets /26 (62 hosts)   ← just right
  ISP keeps the rest for future allocation

Result: No IP wastage. Efficient allocation.
```

---

## 3. Reading the Given Network

### Lecture's Example Network

```
ISP has: 245.248.128.0/20

Breaking it down:
  IP address:    245.248.128.0
  Prefix:        /20
  Network bits:  20
  Host bits:     32 - 20 = 12
  Total hosts:   2¹² = 4096
  Usable hosts:  4096 - 2 = 4094
```

### Opening the Octets

```
245  .  248  .  128  .  0
 8       8       8     8    = 32 bits total
└──────────────────────────┘
          /20 = 20 network bits

First 2 octets: 8+8 = 16 bits → entirely network (don't touch)
Third octet:    need 20-16 = 4 more network bits from here
Fourth octet:   entirely host bits (8 bits)

Third octet (128) in binary: 1 0 0 0 | 0 0 0 0
                              ↑ ↑ ↑ ↑   ↑ ↑ ↑ ↑
                              N N N N   H H H H
                              (4 net)   (4 host)

Full host portion = last 4 bits of 3rd octet + all 8 bits of 4th octet
                  = 12 bits total ✓
```

```
Bit layout:
Octet:  [───── Oct1 ─────][───── Oct2 ─────][─ Oct3 ─][───── Oct4 ─────]
Bits:   [N N N N N N N N ][N N N N N N N N ][N N N N H H H H][H H H H H H H H]
         ↑── 8 net ──────  ↑── 8 net ──────  ↑ 4net  ↑ 4host  ↑──── 8 host ──
                                                       └──────── 12 host bits ──┘
```

---

## 4. The ISP Allocation Problem

```
ISP has: 245.248.128.0/20  (4096 addresses)

Requirements:
  Organization A → give HALF   → 2048 addresses
  Organization B → give QUARTER → 1024 addresses
  ISP keeps      → QUARTER     → 1024 addresses (for future use)

Visual split:
┌───────────────────────────────────────────┐
│         245.248.128.0/20 (4096)           │
├───────────────────┬───────────────────────┤
│   Half (2048)     │   Half (2048)         │
│   → Org A (S1)    │                       │
│                   ├───────────┬───────────┤
│                   │ Quarter   │ Quarter   │
│                   │ → Org B   │ ISP keeps │
│                   │   (S2)    │   (S3)    │
└───────────────────┴───────────┴───────────┘
```

---

## 5. Step-by-Step VLSM Solution

### 5.1 Split into Two Halves

```
Golden Rule: NEVER disturb network bits.
Work only with HOST bits.

Host bits: 12 (last 4 of oct3 + all 8 of oct4)

To split into 2 equal halves:
  Borrow 1 bit (MSB of host bits)
  Fix this bit to 0 → first half
  Fix this bit to 1 → second half
```

```
Position of MSB of host bits in 3rd octet:
  Oct3:  1 0 0 0 | 0 0 0 0
         N N N N   H H H H
                   ↑
                   This is the MSB of host bits (bit position 4 of oct3)
                   We will fix this bit first
```

### 5.2 Subnet 1 (S1) — Half (→ Organization A)

**Fix MSB of host = 0:**

```
Oct3 fixed portion: 1 0 0 0 [0] _ _ _  (5th bit fixed to 0)
                    ↑ ↑ ↑ ↑  ↑
                    N N N N  [fixed 0] ← this is now also fixed

Oct4: all 8 bits remain as host → vary from 00000000 to 11111111

First address:
  Oct3: 1 0 0 0 [0] 0 0 0 = 128 + 0 = 128
  Oct4: 0 0 0 0 0 0 0 0 = 0
  Address: 245.248.128.0  ← S1 Network ID

Last address:
  Oct3: 1 0 0 0 [0] 1 1 1 = 128+7 = 135
  Oct4: 1 1 1 1 1 1 1 1 = 255
  Address: 245.248.135.255  ← S1 Broadcast

S1 range: 245.248.128.0 → 245.248.135.255
Total:    8 × 256 = 2048 addresses ✓ (half of 4096)
Usable:   2048 - 2 = 2046 hosts
```

```
Why 128 to 135?
  Oct3 ranges: 10000[0]000 to 10000[0]111
               128          to   128+7 = 135
               ↑ 4 network bits fixed + 1 borrowed bit fixed = 5 fixed
               ↑ 3 remaining free bits: 000 to 111 = 8 values
  Each value → 256 hosts (oct4 varies): 8 × 256 = 2048 ✓
```

### 5.3 Split Remaining Half into Two Quarters

The second half (MSB of host = 1) needs to be split into 2 quarters.

**Fix MSB of host = 1 (this defines the second half):**
**Now fix the NEXT host bit to 0 or 1 to split further:**

```
Current fixed bits in oct3:
  1 0 0 0 [1] _ _ _
  N N N N  ↑ fixed (defines 2nd half)
              ↑ next host bit to borrow for S2 vs S3

Fix next bit = 0 → Subnet 2 (S2, first quarter of 2nd half)
Fix next bit = 1 → Subnet 3 (S3, second quarter of 2nd half)
```

### 5.4 Subnet 2 (S2) — One Quarter (→ Organization B)

**MSB of host = 1, next bit = 0:**

```
Oct3: 1 0 0 0 [1][0] _ _
      N N N N  ↑   ↑
               fixed fixed (2 bits borrowed total)
               (2nd half)(1st quarter of 2nd half)

Remaining free bits in oct3: 2 bits (_ _)
Oct4: all 8 bits free

First address:
  Oct3: 1 0 0 0 1 0 0 0 = 128+8 = 136
  Oct4: 0 0 0 0 0 0 0 0 = 0
  Address: 245.248.136.0  ← S2 Network ID

Last address:
  Oct3: 1 0 0 0 1 0 1 1 = 128+8+3 = 139
  Oct4: 1 1 1 1 1 1 1 1 = 255
  Address: 245.248.139.255  ← S2 Broadcast

S2 range: 245.248.136.0 → 245.248.139.255
Total:    4 × 256 = 1024 addresses ✓ (quarter of 4096)
Usable:   1024 - 2 = 1022 hosts
```

```
Why 136 to 139?
  Oct3 ranges: 10001[0]00 to 10001[0]11
               136          to 139
               Fixed: 10001[0] = 128+8 = 136
               Free 2 bits: 00 to 11 = 4 values
               4 × 256 = 1024 ✓
```

### 5.5 Subnet 3 (S3) — One Quarter (ISP Keeps)

**MSB of host = 1, next bit = 1:**

```
Oct3: 1 0 0 0 [1][1] _ _
      N N N N  ↑   ↑
               (2nd half)(2nd quarter)

First address:
  Oct3: 1 0 0 0 1 1 0 0 = 128+8+4 = 140
  Oct4: 0 0 0 0 0 0 0 0 = 0
  Address: 245.248.140.0  ← S3 Network ID

Last address:
  Oct3: 1 0 0 0 1 1 1 1 = 128+8+4+3 = 143
  Oct4: 1 1 1 1 1 1 1 1 = 255
  Address: 245.248.143.255  ← S3 Broadcast

S3 range: 245.248.140.0 → 245.248.143.255
Total:    4 × 256 = 1024 addresses ✓ (quarter of 4096)
Usable:   1024 - 2 = 1022 hosts
```

---

## 6. Complete VLSM Summary Table

| Subnet | Assigned To     | Network ID    | Broadcast       | Range             | Prefix | Total | Usable |
| ------ | --------------- | ------------- | --------------- | ----------------- | ------ | ----- | ------ |
| S1     | Org A (half)    | 245.248.128.0 | 245.248.135.255 | .128.0 – .135.255 | /21    | 2048  | 2046   |
| S2     | Org B (quarter) | 245.248.136.0 | 245.248.139.255 | .136.0 – .139.255 | /22    | 1024  | 1022   |
| S3     | ISP (quarter)   | 245.248.140.0 | 245.248.143.255 | .140.0 – .143.255 | /22    | 1024  | 1022   |

```
Address space visualization:
245.248.128.0                                        245.248.143.255
│←────────────────── /20 (4096 addresses) ──────────────────────────→│
│←──────── S1 /21 (2048) ─────────→│←── S2 /22 ──→│←── S3 /22 ──→│
.128.0              .135.255  .136.0   .139.255 .140.0  .143.255
```

---

## 7. Slash Notation After Each Split

```
Original:     /20  (20 network bits fixed)

After 1st split (create S1 and 2nd half):
  Borrow 1 bit → fixed bits = 21
  S1 prefix:    /21

After 2nd split (create S2 and S3 from 2nd half):
  Borrow 1 more bit → fixed bits = 22
  S2 prefix:    /22
  S3 prefix:    /22

Rule: Each time you borrow 1 host bit, add 1 to the prefix.
  /20 → borrow 1 → /21 (for the half)
  /20 → borrow 2 → /22 (for each quarter)
```

### General Pattern

```
Splits              | Bits borrowed | New prefix | Size relative to parent
─────────────────────────────────────────────────────────────────────
1 equal split (×2)  |      1        | n+1        | 1/2
1 equal split (×4)  |      2        | n+2        | 1/4
1 equal split (×8)  |      3        | n+3        | 1/8
Half then quarter   |    1 then 2   | n+1, n+2   | 1/2 and 1/4 (VLSM)
```

---

## 8. The 0 vs 1 Fix — Exam Trick

The lecture makes an important point: you can fix the borrowed bit to **0 or 1** — both are valid, just assign different subnets.

```
Scenario: Borrowing 1 bit from host

Option A (fix borrowed bit = 0 first):
  First half  (bit=0): 245.248.128.0 → 245.248.135.255  ← S1
  Second half (bit=1): 245.248.136.0 → 245.248.143.255  ← continues

Option B (fix borrowed bit = 1 first):
  First half  (bit=1): 245.248.136.0 → 245.248.143.255  ← S1
  Second half (bit=0): 245.248.128.0 → 245.248.135.255  ← continues

Both options are CORRECT — you just get subnets in different order.
```

### When Exam Options Don't Match

```
If you fix bit=0 and get S1 = 245.248.128.0 – 135.255
But NONE of the exam options show this range...

Try fixing bit=1 first:
  S1 = 245.248.136.0 – 139.255 (if they want quarter first!)

OR read the question again — maybe S2 is what they want.

Exam tip: Try both 0 and 1 if first attempt doesn't match options.
```

---

## 9. General VLSM Algorithm

```
Step 1: Identify parent network (IP/prefix)
Step 2: Calculate total host bits = 32 - prefix

Step 3: Sort requirements by size (largest first — always allocate largest first)

Step 4: For each allocation:
  a. Take the current "available block" (starts as whole parent)
  b. Find minimum prefix that fits: 2^(32-new_prefix) ≥ required_hosts + 2
  c. Assign that /new_prefix block
  d. The next available block starts right after this one

Step 5: Repeat Step 4 for each requirement

Step 6: Remaining space = ISP/reserved
```

### Example: Allocate to 3 orgs from 200.10.0.0/24

```
Requirements:
  Org A: 100 hosts → needs /25 (126 usable) → 128 addresses
  Org B: 60 hosts  → needs /26 (62 usable)  → 64 addresses
  Org C: 30 hosts  → needs /27 (30 usable)  → 32 addresses

Sort by size (largest first): A(100), B(60), C(30)

Allocation (starting from 200.10.0.0):
  Org A: 200.10.0.0/25   → .0   – .127  (128 addresses, 126 usable)
  Org B: 200.10.0.128/26 → .128 – .191  (64 addresses, 62 usable)
  Org C: 200.10.0.192/27 → .192 – .223  (32 addresses, 30 usable)
  Free:  200.10.0.224/27 → .224 – .255  (spare/WAN links)

Total used: 128 + 64 + 32 = 224 out of 256 ✓
```

---

## 10. 🔴 Security Context

### VLSM for Security Zoning

```
Real-world VLSM security design:
  Parent: 10.0.0.0/16 (65,534 usable hosts)

  DMZ (public-facing servers):     10.0.0.0/24   (254 hosts)   ← small, tightly controlled
  Web/App servers:                  10.0.1.0/24   (254 hosts)   ← medium
  Database servers:                 10.0.2.0/25   (126 hosts)   ← restricted access
  Internal users (floor 1):        10.0.4.0/23   (510 hosts)
  Internal users (floor 2):        10.0.6.0/23   (510 hosts)
  Management/Admin:                 10.0.8.0/27   (30 hosts)    ← smallest, highest security
  WAN/Router links:                 10.0.9.0/30   (2 hosts each) ← minimal
```

### Principle of Least Privilege via Subnetting

```
Security benefit of VLSM sizing:
  - DMZ /24 → can have 254 servers MAX → limit blast radius
  - Admin /27 → only 30 admins → limit privileged access
  - Database /25 → isolated → breaching web server doesn't give DB access

Attacker who compromises web server (10.0.1.x):
  Can scan: 10.0.1.0/24 → finds 254 potential hosts
  Can NOT directly reach: 10.0.2.0/25 (DB subnet)
  Must first bypass inter-subnet firewall → harder lateral movement
```

### VLSM Misconfiguration Risks

```
Common mistake: Overlapping subnets

Bad VLSM allocation:
  Subnet 1: 10.0.1.0/24   → 10.0.1.0 – 10.0.1.255
  Subnet 2: 10.0.1.128/25 → 10.0.1.128 – 10.0.1.255  ← OVERLAP!

Result:
  Hosts in Subnet 2 (thought isolated) are also in Subnet 1's range
  Traffic meant for Subnet 2 goes to Subnet 1 router instead
  Security policy for Subnet 1 applies → firewall rules bypassed!

Always verify: subnets must be CONTIGUOUS and NON-OVERLAPPING
```

### Route Summarization Security Benefit

```
VLSM + Route Aggregation:
  Instead of advertising 4 separate routes:
    10.0.0.0/24, 10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24

  Advertise ONE summary route:
    10.0.0.0/22 → covers all four /24s

Security benefit:
  Fewer routes → smaller routing tables → harder to exploit BGP
  Summary route hides internal subnet structure from outside
  Attackers can't infer your internal VLSM design from routing table
```

---

## 11. 🧪 Practical Labs

### Lab 1 — VLSM Calculator (Python)

```python
# Save as vlsm_calculator.py — run: python3 vlsm_calculator.py
import ipaddress
import math

def minimum_prefix(required_hosts: int) -> int:
    """Find smallest prefix that fits required_hosts"""
    host_bits = math.ceil(math.log2(required_hosts + 2))  # +2 for network + broadcast
    return 32 - host_bits

def vlsm_allocate(parent_cidr: str, requirements: list) -> list:
    """
    Allocate subnets using VLSM.
    requirements = [('Name', num_hosts), ...] — already sorted largest first
    """
    parent = ipaddress.IPv4Network(parent_cidr, strict=False)
    available = [parent]
    allocations = []

    for name, hosts_needed in requirements:
        prefix = minimum_prefix(hosts_needed)

        # Find smallest available block that fits
        for i, block in enumerate(available):
            if block.prefixlen <= prefix:
                # Subnet this block to the required prefix
                subnets = list(block.subnets(new_prefix=prefix))
                allocated = subnets[0]

                allocations.append({
                    'name': name,
                    'hosts_needed': hosts_needed,
                    'subnet': str(allocated),
                    'network': str(allocated.network_address),
                    'broadcast': str(allocated.broadcast_address),
                    'first_host': str(list(allocated.hosts())[0]),
                    'last_host': str(list(allocated.hosts())[-1]),
                    'usable': allocated.num_addresses - 2,
                    'prefix': prefix,
                })

                # Replace used block with remaining subnets
                available.pop(i)
                available = subnets[1:] + available
                available.sort(key=lambda x: x.prefixlen, reverse=True)
                break
        else:
            allocations.append({'name': name, 'error': 'Not enough space!'})

    return allocations, available

# ─── Lecture Example ───
print("=" * 65)
print("LECTURE EXAMPLE: 245.248.128.0/20")
print("=" * 65)
parent = "245.248.128.0/20"
requirements = [
    ("Org A (Half)",    2046),
    ("Org B (Quarter)", 1022),
    ("ISP Keep",        1022),
]

allocations, remaining = vlsm_allocate(parent, requirements)
for alloc in allocations:
    if 'error' in alloc:
        print(f"\n  {alloc['name']}: ERROR — {alloc['error']}")
    else:
        print(f"\n  {alloc['name']} (needs {alloc['hosts_needed']} hosts):")
        print(f"    Subnet:      {alloc['subnet']}")
        print(f"    Network ID:  {alloc['network']}")
        print(f"    First host:  {alloc['first_host']}")
        print(f"    Last host:   {alloc['last_host']}")
        print(f"    Broadcast:   {alloc['broadcast']}")
        print(f"    Usable:      {alloc['usable']}")

# ─── Practical Example ───
print("\n\n" + "=" * 65)
print("PRACTICAL EXAMPLE: 192.168.1.0/24 for small office")
print("=" * 65)
parent2 = "192.168.1.0/24"
requirements2 = sorted([
    ("Engineering (50 hosts)",   50),
    ("HR (25 hosts)",            25),
    ("Management (10 hosts)",    10),
    ("WAN Link 1 (2 hosts)",     2),
    ("WAN Link 2 (2 hosts)",     2),
], key=lambda x: x[1], reverse=True)  # sort largest first

allocations2, remaining2 = vlsm_allocate(parent2, requirements2)
for alloc in allocations2:
    if 'error' not in alloc:
        print(f"\n  {alloc['name']}:")
        print(f"    {alloc['subnet']:20s} hosts: {alloc['first_host']} – {alloc['last_host']} ({alloc['usable']} usable)")

if remaining2:
    print(f"\n  Unallocated blocks:")
    for block in remaining2:
        print(f"    {block} ({block.num_addresses} addresses)")
```

### Lab 2 — Verify No Subnet Overlap

```python
# Save as check_overlap.py — run: python3 check_overlap.py
import ipaddress

def check_overlaps(subnets: list) -> list:
    """Check if any subnets overlap"""
    networks = [ipaddress.IPv4Network(s, strict=False) for s in subnets]
    overlaps = []
    for i in range(len(networks)):
        for j in range(i+1, len(networks)):
            if networks[i].overlaps(networks[j]):
                overlaps.append((subnets[i], subnets[j]))
    return overlaps

# Test lecture subnets — should not overlap
lecture_subnets = [
    "245.248.128.0/21",   # S1
    "245.248.136.0/22",   # S2
    "245.248.140.0/22",   # S3
]
print("Lecture subnets overlap check:")
overlaps = check_overlaps(lecture_subnets)
if overlaps:
    print(f"  OVERLAP FOUND: {overlaps} ← SECURITY RISK!")
else:
    print("  No overlaps ✓ — subnets are properly allocated")

# Test bad allocation — intentionally overlapping
bad_subnets = [
    "10.0.1.0/24",
    "10.0.1.128/25",   # overlaps with above!
]
print("\nBad allocation overlap check:")
overlaps = check_overlaps(bad_subnets)
if overlaps:
    for a, b in overlaps:
        print(f"  OVERLAP: {a} overlaps with {b} ← MISCONFIGURATION!")
```

### Lab 3 — Design Security Zones with VLSM

```bash
# Apply VLSM thinking to your VirtualBox lab network
# Parent: 192.168.56.0/24

python3 - << 'EOF'
import ipaddress

parent = ipaddress.IPv4Network("192.168.56.0/24")

# Security zone design using VLSM
zones = [
    ("Parrot OS (Attacker)",   1, "/30"),   # 2 hosts (you + gateway)
    ("Metasploitable2 (Target)", 1, "/30"), # 2 hosts
    ("Future VMs",             14, "/28"),  # 14 hosts
    ("Management/Monitoring",   6, "/29"),  # 6 hosts
    ("Spare",                   0, None),   # rest
]

print("VLSM Security Zone Design for 192.168.56.0/24")
print("=" * 60)

current = list(parent.subnets(new_prefix=30))
# Just show the concept
subnets = [
    ipaddress.IPv4Network("192.168.56.0/30"),   # Attacker zone
    ipaddress.IPv4Network("192.168.56.4/30"),   # Target zone
    ipaddress.IPv4Network("192.168.56.8/29"),   # Management
    ipaddress.IPv4Network("192.168.56.16/28"),  # Future VMs
]

labels = ["Attacker Zone", "Target Zone", "Management", "Future VMs"]
for i, (subnet, label) in enumerate(zip(subnets, labels)):
    hosts = list(subnet.hosts())
    print(f"\nZone {i+1}: {label}")
    print(f"  Subnet:  {subnet}")
    print(f"  Hosts:   {hosts[0]} – {hosts[-1]}")
    print(f"  Usable:  {len(hosts)} hosts")

print("\nFirewall rules needed between zones:")
print("  Attacker Zone → Target Zone:   allow all (pentest lab)")
print("  Target Zone   → Attacker Zone: block all outbound")
print("  Management    → All Zones:     allow SSH (22) only")
print("  Future VMs    → Others:        block until assigned")
EOF
```

### Lab 4 — Subnetting with ipcalc

```bash
# ipcalc handles VLSM calculations easily
sudo apt install ipcalc -y

# Analyze parent network
ipcalc 245.248.128.0/20
echo ""

# Show S1 (/21)
ipcalc 245.248.128.0/21
echo ""

# Show S2 (/22)
ipcalc 245.248.136.0/22
echo ""

# Show S3 (/22)
ipcalc 245.248.140.0/22
echo ""

# Check for overlap between S1 and S2 (should show no conflict)
# All ranges should be within parent 245.248.128.0/20

# Scan your specific subnet only
nmap -sn 192.168.56.0/27   # Only scans .1-.30
nmap -sn 192.168.56.32/27  # Only scans .33-.62
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           VLSM IN CIDR — EXAM CHEAT SHEET                           ║
╠══════════════════════════════════════════════════════════════════════╣
║  VLSM = Variable Length Subnet Masking                              ║
║  Purpose: Create subnets of DIFFERENT sizes from one parent         ║
║  Key: Allocate largest subnet first, then smaller ones              ║
╠══════════════════════════════════════════════════════════════════════╣
║  LECTURE EXAMPLE: 245.248.128.0/20                                  ║
║  Host bits: 12 (3rd octet last 4 bits + all of 4th octet)         ║
║                                                                      ║
║  S1 (Half → /21):                                                   ║
║    Borrow 1 bit (fix MSB of host = 0)                              ║
║    Range: 245.248.128.0 → 245.248.135.255                         ║
║    2048 addresses, 2046 usable                                      ║
║                                                                      ║
║  S2 (Quarter → /22):                                                ║
║    Borrow 2 bits (fix MSB=1, next=0)                               ║
║    Range: 245.248.136.0 → 245.248.139.255                         ║
║    1024 addresses, 1022 usable                                      ║
║                                                                      ║
║  S3 (Quarter → /22):                                                ║
║    Borrow 2 bits (fix MSB=1, next=1)                               ║
║    Range: 245.248.140.0 → 245.248.143.255                         ║
║    1024 addresses, 1022 usable                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  SLASH NOTATION RULE                                                 ║
║  Original /n → borrow k bits → new prefix = n + k                 ║
║  /20 + borrow 1 = /21 (half)                                       ║
║  /20 + borrow 2 = /22 (quarter)                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  GOLDEN RULE (same as subnetting)                                   ║
║  NEVER disturb the network bits                                     ║
║  ONLY borrow from host bits                                         ║
║  MSB of host bits = first bit to borrow                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  0 vs 1 EXAM TRICK                                                  ║
║  Fixing borrowed bit=0 → gets first subnet in range                ║
║  Fixing borrowed bit=1 → gets second subnet in range               ║
║  If your answer doesn't match options → try the other value!       ║
╠══════════════════════════════════════════════════════════════════════╣
║  CONTINUITY CHECK                                                    ║
║  S1 ends at X → S2 starts at X+1                                  ║
║  S2 ends at Y → S3 starts at Y+1                                  ║
║  Lecture: .135.255 → .136.0 → .139.255 → .140.0 ✓                ║
╠══════════════════════════════════════════════════════════════════════╣
║  MINIMUM PREFIX FOR N HOSTS                                          ║
║  Hosts needed:  2 → /30  (4 total)                                 ║
║  Hosts needed:  6 → /29  (8 total)                                 ║
║  Hosts needed: 14 → /28  (16 total)                                ║
║  Hosts needed: 30 → /27  (32 total)                                ║
║  Hosts needed: 62 → /26  (64 total)                                ║
║  Hosts needed: 126 → /25  (128 total)                              ║
║  Hosts needed: 254 → /24  (256 total)                              ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY USES                                                       ║
║  Each security zone gets appropriately sized subnet                 ║
║  DMZ /24, servers /25, admin /27, WAN links /30                   ║
║  Non-overlapping subnets = clean firewall rule separation          ║
║  Route summarization hides internal VLSM from outside              ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Supernetting / Route Aggregation — combining /24s into /22
- [ ] NAT — why private VLSM addresses need translation
- [ ] IPv6 Addressing — 128-bit, /64 standard prefix
- [ ] Routing Algorithms — how routers use subnet info (OSPF, RIP)
- [ ] Access Control Lists (ACLs) — security rules using CIDR notation

---

_Notes compiled from: Networking Course Lecture 14 — VLSM in CIDR (Series Lecture 48)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
