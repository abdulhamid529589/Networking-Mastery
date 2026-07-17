# 🌐 VLSM — Variable Length Subnet Masking

### Cybersecurity Student Notes | Networking Course — Lecture 46

> **Source:** Gate Smashers — VLSM (Variable Length Subnet Masking)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is VLSM?](#1-what-is-vlsm)
2. [Why VLSM? (vs. Fixed-Length Subnetting)](#2-why-vlsm-vs-fixed-length-subnetting)
3. [Key Terminology](#3-key-terminology)
4. [Golden Rules Before You Start](#4-golden-rules-before-you-start)
5. [Step-by-Step VLSM Calculation](#5-step-by-step-vlsm-calculation)
   - [5.1 Understand the Requirements](#51-understand-the-requirements)
   - [5.2 Fix Bits to Create Subnets of Different Sizes](#52-fix-bits-to-create-subnets-of-different-sizes)
   - [5.3 Calculate Ranges for Each Subnet](#53-calculate-ranges-for-each-subnet)
   - [5.4 Calculate Subnet Masks](#54-calculate-subnet-masks)
6. [Full Summary Table of the Worked Example](#6-full-summary-table-of-the-worked-example)
7. [Usable Hosts Formula](#7-usable-hosts-formula)
8. [Quick Reference Rules](#8-quick-reference-rules)
9. [More Worked Examples](#9-more-worked-examples)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. What is VLSM?

**VLSM (Variable Length Subnet Masking)** is the technique of dividing a single IP network into **subnets of different sizes**, rather than all-equal subnets, so that the IP address space is allocated efficiently based on actual department/zone requirements.

```
Fixed-length subnetting (previous lecture):
  One large network → split into equal-sized subnets
  e.g.,  /24 → four equal /26 subnets (64 IPs each)
  Problem: A department needing 100 IPs and one needing 10 IPs
           both get 64 — massive waste of the 54 unused in the small one.

VLSM:
  One large network → split into variable-sized subnets
  e.g.,  /24 → one /25 (128 IPs) + two /26 (64 IPs each)
  The big department gets the large subnet, the small ones get small subnets.
  Addresses are allocated only as needed — far less waste.
```

**Hard drive analogy (from the lecture):** Just as Windows lets you partition a 1 TB hard drive into one 500 GB partition and two 250 GB partitions (rather than forcing four equal 250 GB partitions), VLSM lets you partition a network into unequal subnets based on what each department actually needs.

---

## 2. Why VLSM? (vs. Fixed-Length Subnetting)

| Feature                | Fixed-Length Subnetting          | VLSM                                                |
| ---------------------- | -------------------------------- | --------------------------------------------------- |
| Subnet sizes           | All equal                        | Variable — each sized to requirement                |
| IP address waste       | High (unused IPs in small depts) | Minimal                                             |
| Complexity             | Simple                           | Slightly more complex                               |
| Real-world use         | Legacy / very simple networks    | Modern networks (OSPF, EIGRP, BGP all support VLSM) |
| Subnet mask per subnet | Same for all subnets             | Different per subnet (hence "variable")             |

> **Prerequisite:** This lecture assumes you have already watched the basic subnetting video (Lecture 45). 70–80% of the mechanics are the same — VLSM just adds the ability to fix a different number of bits for different subnets.

---

## 3. Key Terminology

| Term                       | Meaning                                                                                            |
| -------------------------- | -------------------------------------------------------------------------------------------------- |
| **Network address**        | First address of any subnet — reserved as the subnet ID, not assignable to hosts                   |
| **Broadcast address**      | Last address of any subnet — reserved for direct broadcast, not assignable to hosts                |
| **Usable host addresses**  | All addresses between network and broadcast = total block size − 2                                 |
| **Subnet mask**            | Shows how many bits are fixed (network + subnet bits) in a /n notation, e.g. /25 = 255.255.255.128 |
| **Fixed bits**             | Bits at the start of the host portion that are "locked" to define a subnet boundary                |
| **Host bits**              | Remaining free bits after fixing — used for assigning addresses to individual hosts                |
| **CIDR notation**          | Shorthand for subnet mask using a slash, e.g. /25 means 25 consecutive 1-bits in the mask          |
| **Default mask (Class C)** | 255.255.255.0 = /24 = 24 fixed bits already (the network portion)                                  |

---

## 4. Golden Rules Before You Start

```
┌───────────────────────────────────────────────────────────────┐
│  RULE 1: NEVER touch the network bits                          │
│    Only the host bits (rightmost octet in Class C) are yours  │
│    to subdivide. The first 24 bits of a Class C address are   │
│    the network ID — changing them moves you to a different     │
│    network entirely.                                           │
│                                                                 │
│  RULE 2: Always subtract 2 from each subnet's block size       │
│    First address = Subnet/Network ID (reserved)               │
│    Last address  = Broadcast address (reserved)               │
│    Usable hosts  = Block size − 2                              │
│                                                                 │
│  RULE 3: Largest subnet first (best practice)                  │
│    Allocate the biggest required subnet from the start of the  │
│    address space, then carve smaller subnets from what's left. │
│    This keeps addressing clean and avoids fragmentation.       │
│                                                                 │
│  RULE 4: Total usable = 256 − (2 × number of subnets)         │
│    Every subnet wastes exactly 2 addresses (ID + broadcast).   │
│    With n subnets: usable = 256 − 2n                           │
└───────────────────────────────────────────────────────────────┘
```

---

## 5. Step-by-Step VLSM Calculation

### Worked Example from Lecture

```
Network:      200.10.20.0  (Class C)
Total IPs:    256  (0–255),  usable: 254  (1–254)

Requirements:
  Department 1 (Academic): needs > 100 hosts  →  needs ≥ 128-block subnet
  Department 2:            needs > 50 hosts   →  needs ≥ 64-block subnet
  Department 3:            needs > 50 hosts   →  needs ≥ 64-block subnet
```

---

### 5.1 Understand the Requirements

Sort requirements **largest to smallest** before doing anything:

| Priority | Requirement | Minimum Block Size Needed        |
| -------- | ----------- | -------------------------------- |
| 1st      | > 100 hosts | 128 addresses (gives 126 usable) |
| 2nd      | > 50 hosts  | 64 addresses (gives 62 usable)   |
| 3rd      | > 50 hosts  | 64 addresses (gives 62 usable)   |

Total addresses needed: 128 + 64 + 64 = 256 → exactly fits within a /24.

---

### 5.2 Fix Bits to Create Subnets of Different Sizes

The Class C network `200.10.20.0` gives us 8 **host bits** to work with (the last octet). Start from the MSB (leftmost host bit) and fix bits to carve out subnets:

```
Host octet, 8 bits:  [ b7 | b6 | b5 | b4 | b3 | b2 | b1 | b0 ]
                       MSB                                   LSB

Step 1 — Create S1 (needs 128-block):
  Fix b7 = 0
  This locks the top half of the address space (0–127) as Subnet 1.
  Remaining free bits for hosts: b6 b5 b4 b3 b2 b1 b0 (7 bits → 2^7 = 128 addresses)

Step 2 — The other half (b7 = 1, i.e., 128–255) is still unallocated.
  We now need to divide it into two equal 64-blocks for S2 and S3.
  Fix b7 = 1 AND b6 = 0  →  S2  (128–191)
  Fix b7 = 1 AND b6 = 1  →  S3  (192–255)
  Remaining free bits for hosts in each: b5 b4 b3 b2 b1 b0 (6 bits → 2^6 = 64 addresses each)
```

**Visual breakdown:**

```
Address space (0–255):

0       ←──────────── 128 addresses ────────────→ 127
│  S1:  b7 = 0  (1 bit fixed beyond /24)                │

128     ←── 64 addresses ──→ 191
│  S2:  b7=1, b6=0  (2 bits fixed beyond /24)   │

192     ←── 64 addresses ──→ 255
│  S3:  b7=1, b6=1  (2 bits fixed beyond /24)   │
```

---

### 5.3 Calculate Ranges for Each Subnet

**Subnet 1 (S1) — 1 bit fixed (b7 = 0):**

```
Binary range:
  Smallest: 0 0000000  =  0
  Largest:  0 1111111  =  127

Range:      200.10.20.0  →  200.10.20.127
Block size: 128
Network ID: 200.10.20.0     (first — reserved)
Broadcast:  200.10.20.127   (last — reserved)
Usable:     200.10.20.1  →  200.10.20.126   (126 hosts ✓ satisfies > 100)
```

**Subnet 2 (S2) — 2 bits fixed (b7=1, b6=0):**

```
Binary range:
  Smallest: 1 0 000000  =  128
  Largest:  1 0 111111  =  191

Range:      200.10.20.128  →  200.10.20.191
Block size: 64
Network ID: 200.10.20.128   (first — reserved)
Broadcast:  200.10.20.191   (last — reserved)
Usable:     200.10.20.129  →  200.10.20.190  (62 hosts ✓ satisfies > 50)
```

**Subnet 3 (S3) — 2 bits fixed (b7=1, b6=1):**

```
Binary range:
  Smallest: 1 1 000000  =  192
  Largest:  1 1 111111  =  255

Range:      200.10.20.192  →  200.10.20.255
Block size: 64
Network ID: 200.10.20.192   (first — reserved)
Broadcast:  200.10.20.255   (last — reserved)
Usable:     200.10.20.193  →  200.10.20.254  (62 hosts ✓ satisfies > 50)
```

---

### 5.4 Calculate Subnet Masks

The subnet mask tells a router **how many bits are fixed** (network + subnet bits).

```
Class C default: 24 bits fixed already (/24 = 255.255.255.0)

S1: fixed 1 extra bit beyond /24  →  24+1 = 25 fixed bits
    /25 mask:  11111111.11111111.11111111.10000000
             = 255.255.255.128

S2: fixed 2 extra bits beyond /24 →  24+2 = 26 fixed bits
    /26 mask:  11111111.11111111.11111111.11000000
             = 255.255.255.192

S3: same as S2, 2 extra bits fixed →  26 fixed bits
    /26 mask:  255.255.255.192
```

**How the last octet is built:**

```
S1 mask last octet:  1 0000000  =  128   (/25)
S2 mask last octet:  1 1000000  =  192   (/26)
S3 mask last octet:  1 1000000  =  192   (/26)
```

---

## 6. Full Summary Table of the Worked Example

| Subnet | Network Address   | Broadcast Address | Usable Range                  | Usable Hosts | Subnet Mask         | CIDR |
| ------ | ----------------- | ----------------- | ----------------------------- | ------------ | ------------------- | ---- |
| S1     | 200.10.20.**0**   | 200.10.20.**127** | 200.10.20.1 – 200.10.20.126   | **126**      | 255.255.255.**128** | /25  |
| S2     | 200.10.20.**128** | 200.10.20.**191** | 200.10.20.129 – 200.10.20.190 | **62**       | 255.255.255.**192** | /26  |
| S3     | 200.10.20.**192** | 200.10.20.**255** | 200.10.20.193 – 200.10.20.254 | **62**       | 255.255.255.**192** | /26  |

**Total usable check:**

```
Formula: usable = 256 − 2n   (n = number of subnets = 3)
         usable = 256 − 6 = 250

Verify: 126 + 62 + 62 = 250 ✓
```

---

## 7. Usable Hosts Formula

```
For a single subnet:
  Usable hosts = 2^h − 2
  Where h = number of free host bits in that subnet

For the whole address space across n subnets:
  Total usable = Total IPs − 2n   =   256 − 2n   (for a /24 base network)
```

| Subnet | Free host bits (h) | Block size (2^h) | Usable (2^h − 2) |
| ------ | ------------------ | ---------------- | ---------------- |
| S1 /25 | 7                  | 128              | 126              |
| S2 /26 | 6                  | 64               | 62               |
| S3 /26 | 6                  | 64               | 62               |

---

## 8. Quick Reference Rules

```
┌─────────────────────────────────────────────────────────────────┐
│  RULE 1: Sort requirements largest → smallest                    │
│    Always allocate the biggest subnet first, from the bottom    │
│    (lowest address) of the space, then move upward.            │
│                                                                   │
│  RULE 2: Determine block size needed                             │
│    Find the smallest power of 2 that is ≥ (required hosts + 2) │
│    e.g., > 100 hosts needed → block size = 128 (2^7)            │
│          > 50 hosts needed  → block size = 64  (2^6)            │
│                                                                   │
│  RULE 3: Fix host bits to create the block boundary              │
│    Block 128 → fix MSB (1 bit)                                  │
│    Block 64  → fix 2 MSBs                                       │
│    Block 32  → fix 3 MSBs ... and so on                         │
│                                                                   │
│  RULE 4: Range = from fixed-bits + all-0 to fixed-bits + all-1  │
│    Network ID = fixed pattern + 000...0                          │
│    Broadcast  = fixed pattern + 111...1                          │
│                                                                   │
│  RULE 5: Subnet mask = 24 + (extra bits fixed)                   │
│    For Class C base (/24):                                       │
│    Fix 1 extra → /25 → 255.255.255.128                           │
│    Fix 2 extra → /26 → 255.255.255.192                           │
│    Fix 3 extra → /27 → 255.255.255.224                           │
│    Fix 4 extra → /28 → 255.255.255.240                           │
│                                                                   │
│  RULE 6: Total usable = 256 − 2n  (n = number of subnets)       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. More Worked Examples

### Example 2 — Four Subnets of Mixed Sizes

```
Network:  192.168.1.0/24
Requirements:
  Dept A: > 60 hosts  →  block 128 (/25)
  Dept B: > 28 hosts  →  block 64  (/26) [next power of 2 above 30]
  Dept C: > 12 hosts  →  block 32  (/27)
  Dept D: > 12 hosts  →  block 32  (/27)

Total: 128 + 64 + 32 + 32 = 256 → fits exactly

Subnets:

S-A: 192.168.1.0/25
     Range: 192.168.1.0 – 192.168.1.127
     Usable: 192.168.1.1 – 192.168.1.126 (126 hosts)
     Mask: 255.255.255.128

S-B: 192.168.1.128/26
     Range: 192.168.1.128 – 192.168.1.191
     Usable: 192.168.1.129 – 192.168.1.190 (62 hosts)
     Mask: 255.255.255.192

S-C: 192.168.1.192/27
     Range: 192.168.1.192 – 192.168.1.223
     Usable: 192.168.1.193 – 192.168.1.222 (30 hosts)
     Mask: 255.255.255.224

S-D: 192.168.1.224/27
     Range: 192.168.1.224 – 192.168.1.255
     Usable: 192.168.1.225 – 192.168.1.254 (30 hosts)
     Mask: 255.255.255.224

Total usable: 256 − 2×4 = 256 − 8 = 248
Verify: 126 + 62 + 30 + 30 = 248 ✓
```

### Example 3 — Quick Subnet Mask Lookup

```
Given block size → find /n notation and mask last octet:

Block 128  → /25 → last octet: 10000000 = 128 → 255.255.255.128
Block 64   → /26 → last octet: 11000000 = 192 → 255.255.255.192
Block 32   → /27 → last octet: 11100000 = 224 → 255.255.255.224
Block 16   → /28 → last octet: 11110000 = 240 → 255.255.255.240
Block 8    → /29 → last octet: 11111000 = 248 → 255.255.255.248
Block 4    → /30 → last octet: 11111100 = 252 → 255.255.255.252
             (/30 is the smallest practical subnet — 2 usable hosts,
              used for point-to-point router links)
```

---

## 10. 🔴 Security Context

### VLSM Is an Addressing Technique — But It Has Direct Security Implications

VLSM is purely an IP addressing and routing mechanism, but how you subnet your network has profound effects on your attack surface, your ability to apply access controls, and how easily an attacker can move laterally.

### 10.1 Subnetting as Network Segmentation (Defense in Depth)

```
If all 254 hosts in 200.10.20.0/24 share the same flat network:
  → A compromised host can reach all 253 others directly
  → No internal boundary for a firewall or ACL to enforce
  → ARP broadcast traffic floods every host (performance + sniffing risk)

With VLSM into S1/S2/S3:
  → A compromised host in S2 (200.10.20.128/26) CANNOT directly reach
    S1 (200.10.20.0/25) without passing through the router
  → The router can apply Access Control Lists (ACLs) at each subnet boundary
  → A host in a compromised subnet is isolated from the others
  → This is the practical implementation of "network segmentation" /
    "micro-segmentation" that every security architecture framework
    (NIST SP 800-53, ISO 27001, CIS Controls) recommends
```

### 10.2 Common Real-World Subnet Layout for Security

```
Typical university / organization segmentation using VLSM:

  200.10.20.0/25    → Student / General users  (largest, least trusted)
  200.10.20.128/26  → Staff / Faculty          (medium trust)
  200.10.20.192/26  → Servers / DMZ            (small, highest protection)

Firewall / router between these subnets enforces:
  - Students cannot initiate connections to servers
  - Staff can reach servers but only on allowed ports
  - External internet can only reach DMZ, never internal subnets
  - Servers cannot initiate connections back to student/staff subnets
    (reduces lateral movement if a server is compromised)
```

### 10.3 VLSM and ACL Alignment

```
Each subnet boundary is a natural ACL enforcement point:
  - Routers/firewalls inspect source/destination IP and match against
    subnet masks to decide which subnet a packet belongs to
  - The subnet mask IS what the router uses to match routing table
    entries — which is exactly why VLSM-designed networks naturally
    align with per-subnet ACL rules
  - Poorly designed subnets (e.g., putting servers and untrusted users
    in the same /24) make it impossible to write a meaningful ACL
    that distinguishes them by IP
```

### 10.4 Subnet Scanning (Attacker's Perspective — Your Own Lab Only)

```
Understanding VLSM also helps you understand how attackers enumerate
networks. On your own Metasploitable2 lab:

  - A /25 subnet attacker scans 128 addresses (faster, noisier)
  - A /26 subnet attacker scans 64 addresses
  - Knowing subnet boundaries from leaked subnet masks allows an
    attacker to identify which subnet class a discovered IP is in,
    inferring its likely role (e.g., a .192/26 subnet is often servers)

Tools you can run on your OWN lab:
  nmap -sn 192.168.56.0/25   # discover hosts in a /25
  nmap -sn 192.168.56.128/26 # discover hosts in a /26
```

| Mechanism                    | Purpose                                | Security Benefit                |
| ---------------------------- | -------------------------------------- | ------------------------------- |
| VLSM into separate subnets   | Logical network segmentation           | Limits lateral movement         |
| ACLs at subnet boundaries    | Traffic filtering between subnets      | Enforces least-privilege        |
| Firewall between subnets     | Stateful inspection at borders         | Deep packet filtering           |
| /30 subnets for router links | Minimizes exposed IPs on transit links | Reduces attack surface          |
| Private RFC 1918 addressing  | Hides internal topology from internet  | Prevents direct external attack |

---

## 11. 🧪 Practical Labs

### Lab 1 — VLSM Calculator from Scratch in Python

```python
# Save as vlsm_calc.py — run: python3 vlsm_calc.py

import ipaddress

def vlsm_subnets(base_network: str, requirements: list[int]) -> list[dict]:
    """
    Given a base network (e.g. '200.10.20.0/24') and a list of required
    host counts (sorted largest to smallest), returns VLSM subnet allocations.
    """
    requirements = sorted(requirements, reverse=True)
    network = ipaddress.IPv4Network(base_network, strict=True)
    available = list(network.subnets(new_prefix=network.prefixlen))

    results = []
    for req in requirements:
        # Find the smallest block that satisfies req + 2 (ID + broadcast)
        prefix_needed = 32 - (req + 2 - 1).bit_length()
        # Walk available subnets and pick the first that fits
        for candidate in sorted(available, key=lambda n: n.prefixlen, reverse=True):
            if candidate.prefixlen <= prefix_needed:
                # Expand or use directly
                while candidate.prefixlen < prefix_needed:
                    subs = list(candidate.subnets())
                    available.append(subs[1])
                    candidate = subs[0]
                available.remove(candidate)
                hosts = list(candidate.hosts())
                results.append({
                    "required": req,
                    "subnet": str(candidate),
                    "network_id": str(candidate.network_address),
                    "broadcast": str(candidate.broadcast_address),
                    "first_usable": str(hosts[0]) if hosts else "N/A",
                    "last_usable": str(hosts[-1]) if hosts else "N/A",
                    "usable_hosts": len(hosts),
                    "mask": str(candidate.netmask),
                })
                break

    return results

# Reproduce the lecture's example
reqs = [100, 50, 50]
subnets = vlsm_subnets("200.10.20.0/24", reqs)

print("=" * 65)
print(f"VLSM for 200.10.20.0/24 | Requirements: {reqs}")
print("=" * 65)
for i, s in enumerate(subnets, 1):
    print(f"\nSubnet {i} (needs >{s['required']} hosts):")
    print(f"  Network:      {s['subnet']}")
    print(f"  Network ID:   {s['network_id']}")
    print(f"  Broadcast:    {s['broadcast']}")
    print(f"  Usable range: {s['first_usable']} – {s['last_usable']}")
    print(f"  Usable hosts: {s['usable_hosts']}")
    print(f"  Subnet mask:  {s['mask']}")

total_usable = sum(s['usable_hosts'] for s in subnets)
n = len(subnets)
print(f"\nTotal usable: {total_usable}  |  Formula check: 256 − 2×{n} = {256 - 2*n}")
```

### Lab 2 — Verify VLSM Subnets Don't Overlap

```python
# Save as vlsm_overlap_check.py — run: python3 vlsm_overlap_check.py
import ipaddress

subnets = [
    "200.10.20.0/25",
    "200.10.20.128/26",
    "200.10.20.192/26",
]

networks = [ipaddress.IPv4Network(s, strict=True) for s in subnets]

print("Overlap check:")
overlap_found = False
for i, a in enumerate(networks):
    for j, b in enumerate(networks):
        if i >= j:
            continue
        if a.overlaps(b):
            print(f"  ❌ OVERLAP between {a} and {b}")
            overlap_found = True

if not overlap_found:
    print("  ✓ No overlaps — VLSM allocation is valid")

print("\nSubnet details:")
for n in networks:
    hosts = list(n.hosts())
    print(f"  {str(n):22} | hosts: {len(hosts):3} | "
          f"{n.network_address} – {n.broadcast_address} | mask {n.netmask}")
```

### Lab 3 — Configure VLSM Subnets on Your Own Linux Lab (Parrot OS)

```bash
# Assign each VLSM subnet to a virtual interface for testing
# (purely educational — on your own machine only)

# Create virtual interfaces for each subnet
sudo ip link add veth-s1 type dummy
sudo ip link add veth-s2 type dummy
sudo ip link add veth-s3 type dummy

# Assign IPs from each VLSM subnet
sudo ip addr add 200.10.20.1/25  dev veth-s1   # S1 — first usable of /25
sudo ip addr add 200.10.20.129/26 dev veth-s2  # S2 — first usable of /26
sudo ip addr add 200.10.20.193/26 dev veth-s3  # S3 — first usable of /26

sudo ip link set veth-s1 up
sudo ip link set veth-s2 up
sudo ip link set veth-s3 up

# Verify the subnets are recognized correctly
ip addr show veth-s1
ip addr show veth-s2
ip addr show veth-s3

# Routing table now shows the three separate VLSM subnets
ip route show

# Test: can S1 reach S2 directly? (No — different subnet, needs routing)
ping -c 2 -I veth-s1 200.10.20.129  # should fail without routing

# Clean up
sudo ip link del veth-s1
sudo ip link del veth-s2
sudo ip link del veth-s3
```

### Lab 4 — Scan Your Own VLSM Subnets with Nmap (Own Lab Only)

```bash
# Understand how attackers enumerate VLSM-segmented networks
# Run ONLY on your own Metasploitable2 / home lab

# Ping sweep the /25 subnet
nmap -sn 200.10.20.0/25 --open

# Ping sweep the /26 subnets
nmap -sn 200.10.20.128/26 --open
nmap -sn 200.10.20.192/26 --open

# With Metasploitable2 on 192.168.56.101 (adjust per your config)
nmap -sn 192.168.56.0/24  # find active hosts first
nmap -A 192.168.56.101    # detailed scan of Metasploitable2
```

### Lab 5 — Subnetting Practice Script (Self-Quiz Generator)

```python
# Save as vlsm_quiz.py — run: python3 vlsm_quiz.py
# Great for exam prep — generates random VLSM problems and checks your answers

import ipaddress, random

def generate_vlsm_question():
    base = f"192.168.{random.randint(0,255)}.0/24"
    r1 = random.randint(50, 100)
    r2 = random.randint(20, 49)
    r3 = random.randint(5, 19)
    return base, [r1, r2, r3]

def solve_block_size(hosts_needed):
    n = 1
    while (2**n - 2) < hosts_needed:
        n += 1
    return 2**n

base, reqs = generate_vlsm_question()
print(f"Network: {base}")
print(f"Requirements: S1 needs {reqs[0]} hosts, S2 needs {reqs[1]}, S3 needs {reqs[2]}")
print("\nWhat is the block size and /prefix for each subnet?")
input("Press Enter to see answer...")

for i, r in enumerate(sorted(reqs, reverse=True), 1):
    block = solve_block_size(r)
    prefix = 32 - block.bit_length() + 1
    print(f"  S{i} (needs {r} hosts) → block size {block} → /{prefix}")
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              VLSM — EXAM CHEAT SHEET                                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHAT IS VLSM?                                                        ║
║  Variable Length Subnet Masking — divide a network into subnets of   ║
║  DIFFERENT sizes to match actual per-department host requirements     ║
╠══════════════════════════════════════════════════════════════════════╣
║  GOLDEN PROCESS (5 steps)                                             ║
║  1. Sort requirements largest → smallest                              ║
║  2. Find block size = smallest 2^h ≥ (hosts needed + 2)              ║
║  3. Allocate largest subnet first (from lowest available address)    ║
║  4. Remaining space → carve next subnet from where last one ended    ║
║  5. Repeat until all requirements are met                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  HOST BIT → BLOCK SIZE → PREFIX → LAST OCTET MASK                    ║
║  h=7 → 128 → /25 → 255.255.255.128                                   ║
║  h=6 →  64 → /26 → 255.255.255.192                                   ║
║  h=5 →  32 → /27 → 255.255.255.224                                   ║
║  h=4 →  16 → /28 → 255.255.255.240                                   ║
║  h=3 →   8 → /29 → 255.255.255.248                                   ║
║  h=2 →   4 → /30 → 255.255.255.252  (point-to-point links)           ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FORMULAS                                                         ║
║  Usable hosts per subnet = 2^h − 2                                   ║
║  Total usable for whole /24 = 256 − 2n   (n = number of subnets)    ║
║  Subnet mask /n → prefix n = 24 + (extra bits fixed in last octet)   ║
╠══════════════════════════════════════════════════════════════════════╣
║  SUBNET RANGES — WORKED EXAMPLE (200.10.20.0/24)                     ║
║  S1 /25: 200.10.20.0   – 200.10.20.127  (usable: .1 – .126, 126)    ║
║  S2 /26: 200.10.20.128 – 200.10.20.191  (usable: .129 – .190, 62)   ║
║  S3 /26: 200.10.20.192 – 200.10.20.255  (usable: .193 – .254, 62)   ║
╠══════════════════════════════════════════════════════════════════════╣
║  THINGS THAT DON'T CHANGE                                             ║
║  First address of any subnet = Network ID (reserved, not assignable) ║
║  Last address of any subnet  = Broadcast  (reserved, not assignable) ║
║  Never touch network bits (first 24 bits of a Class C address)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                       ║
║  VLSM naturally implements network segmentation — each subnet is a   ║
║  containment boundary where router ACLs / firewalls can enforce      ║
║  least-privilege traffic rules and limit lateral movement            ║
║  /30 subnets are standard for point-to-point router links (only      ║
║  2 usable IPs — one per router interface, minimal exposure)          ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] VLSM numericals — full problem set with different class networks
- [ ] Supernetting / CIDR — the reverse: combining small networks into larger ones
- [ ] Routing protocols and how they carry subnet mask info (OSPF, EIGRP vs. older RIPv1)
- [ ] IP header structure — TTL, fragmentation, checksum
- [ ] Network Address Translation (NAT) — how private RFC 1918 addressing and subnetting interact with the internet

---

_Notes compiled from: Networking Course Lecture 46 — VLSM (Variable Length Subnet Masking)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
