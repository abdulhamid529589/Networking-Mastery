# 🌐 IP Addressing — Subnetting (Classful)

### " "Networking Course — Lecture 46

> **Source:** Gate Smashers — Subnetting (Classful Addressing)
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Quick Recap — What Subnetting Solves](#1-quick-recap--what-subnetting-solves)
2. [What is Subnetting?](#2-what-is-subnetting)
3. [Purpose of Subnetting — Why We Do It](#3-purpose-of-subnetting--why-we-do-it)
4. [How Subnetting Works — The Core Mechanism](#4-how-subnetting-works--the-core-mechanism)
5. [Worked Example — 200.10.20.0/24 Divided Into 2 Subnets](#5-worked-example--2001020-024-divided-into-2-subnets)
   - [Step 1 — Identify the Starting Network](#step-1--identify-the-starting-network)
   - [Step 2 — Reserve Host Bits for Subnetting](#step-2--reserve-host-bits-for-subnetting)
   - [Step 3 — Calculate Subnet Ranges](#step-3--calculate-subnet-ranges)
   - [Step 4 — Network ID, Broadcast, and Usable Hosts per Subnet](#step-4--network-id-broadcast-and-usable-hosts-per-subnet)
   - [Step 5 — Compare: Before vs After Subnetting](#step-5--compare-before-vs-after-subnetting)
6. [Subnet Mask — Default vs Custom](#6-subnet-mask--default-vs-custom)
7. [How Internal Routing Uses the Subnet Mask](#7-how-internal-routing-uses-the-subnet-mask)
8. [The House-and-Rooms Analogy](#8-the-house-and-rooms-analogy)
9. [Subnetting Rules — How Many Subnets Per Reserved Bit](#9-subnetting-rules--how-many-subnets-per-reserved-bit)
10. [Disadvantage of Subnetting — Extra Routing Step](#10-disadvantage-of-subnetting--extra-routing-step)
11. [Advantages vs Disadvantages — Summary](#11-advantages-vs-disadvantages--summary)
12. [All Key Formulas](#12-all-key-formulas)
13. [Why This Matters for Cybersecurity](#13-why-this-matters-for-cybersecurity)
14. [🧪 Practical Labs](#14--practical-labs)
15. [Solved Examples](#15-solved-examples)
16. [Exam Cheat Sheet](#16-exam-cheat-sheet)

---

## 1. Quick Recap — What Subnetting Solves

From Lecture 44 (Problems with Classful Addressing):

```
Problem:  Large flat networks (esp. Class A/B) are:
           → Hard to maintain
           → Prone to errors (hard to isolate faults)
           → Security risks (large flat broadcast domain)

Solution: SUBNETTING — divide one large network into
          multiple smaller, manageable logical sub-networks
```

> **Important distinction:** CIDR (Lecture 45) solves the **address wastage** problem. Subnetting solves the **maintenance, error, and security** problems from within an already-assigned address block.

---

## 2. What is Subnetting?

**Subnetting = dividing a big network into smaller networks (subnets).**

```
Before subnetting:
  200.10.20.0/24  ←  one large flat network (254 usable hosts)

After subnetting:
  200.10.20.0/25  ←  Subnet 1 (S1): 126 usable hosts
  200.10.20.128/25 ← Subnet 2 (S2): 126 usable hosts
```

- You take a network that was assigned to you (from IANA/classful assignment) and internally divide it into logical segments.
- The **outside world** still sees the original single network address — subnetting is done **internally** within the organisation, at the discretion of the internal network administrator. No external authority needs to be consulted.

---

## 3. Purpose of Subnetting — Why We Do It

### University Example (from Lecture)

```
Large University Network → 200.10.20.0/24 (entire campus)

Divided by subnetting into:
  S1: Examination Department
  S2: Placement Department
  S3: Academics Department
  S4: Co-curricular Department
  S5: Library System
```

Each subnet:

- Has its **own network administrator** responsible only for that department
- Is **isolated** from other departments logically
- Can have its **own security policies** (ACLs, firewall rules)

### Benefits

| Benefit              | Why                                                                                                                                |
| -------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Easy maintenance** | Smaller networks are far easier to manage than one massive flat network                                                            |
| **Fault isolation**  | A problem in S1 (Examinations) doesn't affect S2 (Placement) — easier to detect and repair                                         |
| **Reduced IP waste** | Large Class A/B blocks can be efficiently internally subdivided rather than leaving millions of addresses idle in one flat network |
| **Better security**  | Smaller, separated domains with dedicated admins → better authorization and authentication control                                 |

---

## 4. How Subnetting Works — The Core Mechanism

**The fundamental rule of subnetting:**

```
In subnetting, the NETWORK BITS are NEVER touched.
Only HOST BITS are used to create subnets.
```

For a **Class C network** (e.g., `200.10.20.0`):

```
32-bit address split:
┌──────────────────────────┬──────────┐
│     NETWORK BITS (24)    │ HOST (8) │
│   200  .  10   .   20   │  .  ???  │
│ (DO NOT TOUCH THESE)    │ (use these to subnet) │
└──────────────────────────┴──────────┘

The last 8 bits (last octet) = available for subnetting
```

**How to create subnets:** Take bits from the **MSB (Most Significant Bit) end** of the host portion and **fix** them to create subnet identifiers:

```
1 reserved bit → 2 subnets   (2¹)
2 reserved bits → 4 subnets  (2²)
3 reserved bits → 8 subnets  (2³)
n reserved bits → 2ⁿ subnets
```

> **Always take from the MSB (left) side of the host octet** when reserving bits for subnetting.

---

## 5. Worked Example — 200.10.20.0/24 Divided Into 2 Subnets

### Step 1 — Identify the Starting Network

```
IP Address:  200.10.20.0
First octet: 200 → falls in range 192–223 → Class C
Default mask: 255.255.255.0
Network bits: 24 (first three octets)
Host bits:    8  (last octet)
Usable hosts: 256 − 2 = 254
```

---

### Step 2 — Reserve Host Bits for Subnetting

To divide into **2 subnets**, reserve **1 bit** from the MSB of the host octet:

```
Last octet (8 bits):
┌───┬────────────────────────┐
│ S │  7 remaining host bits  │
└───┴────────────────────────┘
  ↑
  Reserved subnet bit (MSB of last octet)
  S = 0  → Subnet 1 (S1)
  S = 1  → Subnet 2 (S2)
```

---

### Step 3 — Calculate Subnet Ranges

**Subnet 1 (S1): subnet bit = 0 (fixed)**

```
Last octet format: [0][x x x x x x x]
                    ↑ fixed    ↑ 7 variable bits

Minimum: 0 000 0000 = 0
Maximum: 0 111 1111 = 127

S1 range: 200.10.20.0 — 200.10.20.127
```

**Subnet 2 (S2): subnet bit = 1 (fixed)**

```
Last octet format: [1][x x x x x x x]
                    ↑ fixed    ↑ 7 variable bits

Minimum: 1 000 0000 = 128
Maximum: 1 111 1111 = 255

S2 range: 200.10.20.128 — 200.10.20.255
```

---

### Step 4 — Network ID, Broadcast, and Usable Hosts per Subnet

|                            | Subnet 1 (S1)     | Subnet 2 (S2)     |
| -------------------------- | ----------------- | ----------------- |
| **Subnet bit**             | 0 (fixed)         | 1 (fixed)         |
| **Full range**             | `.0` — `.127`     | `.128` — `.255`   |
| **Subnet ID (Network ID)** | `200.10.20.0`     | `200.10.20.128`   |
| **First usable host**      | `200.10.20.1`     | `200.10.20.129`   |
| **Last usable host**       | `200.10.20.126`   | `200.10.20.254`   |
| **Direct Broadcast**       | `200.10.20.127`   | `200.10.20.255`   |
| **Total addresses**        | 128               | 128               |
| **Usable hosts**           | 128 − 2 = **126** | 128 − 2 = **126** |

---

### Step 5 — Compare: Before vs After Subnetting

```
BEFORE subnetting (flat /24 network):
  Network:        200.10.20.0
  Broadcast:      200.10.20.255
  Usable hosts:   254

AFTER subnetting (two /25 subnets):
  S1 usable hosts: 126
  S2 usable hosts: 126
  Total usable:    252

Addresses "lost" to subnetting: 254 − 252 = 2
(Because each subnet now has its own network ID and broadcast address)
```

> **Key point:** Subnetting costs **2 additional reserved addresses** per subnet (each subnet needs its own Network ID and Broadcast address). This is a small price for the management, error isolation, and security benefits gained.

---

## 6. Subnet Mask — Default vs Custom

### Default Mask (Class C)

```
255.255.255.0
→ What the outside world uses
→ Identifies this as a Class C /24 network
→ The external router sees only 200.10.20.0 as a single network
```

### Subnet Mask (After Subnetting)

```
We reserved 1 bit from the host octet for subnetting.
→ That 1 reserved bit must also become a 1 in the mask.

Default Class C mask last octet:   00000000 = 0
Reserved 1 subnet bit as 1:        10000000 = 128

Subnet mask: 255.255.255.128
```

**General formula:**

```
Subnet mask last octet = default last octet + reserved bits set to 1

Reserved 1 bit:  10000000 = 128  → mask last octet = 128
Reserved 2 bits: 11000000 = 192  → mask last octet = 192
Reserved 3 bits: 11100000 = 224  → mask last octet = 224
Reserved 4 bits: 11110000 = 240  → mask last octet = 240
```

---

## 7. How Internal Routing Uses the Subnet Mask

This is the mechanism by which the internal router decides which subnet a packet belongs to:

**Setup:**

```
[External Internet] ──► [External Gateway Router] ──► [Internal Router]
                                                           │         │
                                                          S1        S2
                                                       (.0–.127)  (.128–.255)
```

**Routing logic:**

The internal router applies the **subnet mask** (255.255.255.128) to the destination IP address via AND:

**Case 1: Packet destined for `200.10.20.15`**

```
Destination:   200.10.20.15   → last octet = 15  = 00001111
Subnet mask:   255.255.255.128 → last octet = 128 = 10000000

AND: 00001111
     10000000
     ────────
     00000000  = 0

Result: 200.10.20.0 → this is the Subnet ID of S1 → route to S1 ✅
```

**Case 2: Packet destined for `200.10.20.130`**

```
Destination:   200.10.20.130  → last octet = 130 = 10000010
Subnet mask:   255.255.255.128 → last octet = 128 = 10000000

AND: 10000010
     10000000
     ────────
     10000000  = 128

Result: 200.10.20.128 → this is the Subnet ID of S2 → route to S2 ✅
```

> **This is the entire routing magic of subnetting:** The subnet mask tells the internal router exactly which subnet a destination address belongs to, by ANDing the destination IP with the mask.

---

## 8. The House-and-Rooms Analogy

From the lecture — a perfect mental model for how subnetting looks from inside vs. outside:

```
Your organisation's entire address block = your HOUSE (plot you purchased)
Subnets you create inside              = ROOMS inside the house

From OUTSIDE (the internet):
  External routers know only your house number (200.10.20.0)
  They don't know or care how many rooms you have inside
  → They use the DEFAULT mask (255.255.255.0)

From INSIDE (your internal network):
  Your internal router knows all the rooms (subnets)
  It uses the SUBNET mask (255.255.255.128) to route to the right room
  → Each "room" (subnet) is a separate, manageable domain
```

---

## 9. Subnetting Rules — How Many Subnets Per Reserved Bit

| Bits Reserved | Subnets Created | Remaining Host Bits | Hosts per Subnet | Usable Hosts per Subnet |
| ------------- | --------------- | ------------------- | ---------------- | ----------------------- |
| 1             | 2¹ = **2**      | 7                   | 2⁷ = 128         | 126                     |
| 2             | 2² = **4**      | 6                   | 2⁶ = 64          | 62                      |
| 3             | 2³ = **8**      | 5                   | 2⁵ = 32          | 30                      |
| 4             | 2⁴ = **16**     | 4                   | 2⁴ = 16          | 14                      |
| 5             | 2⁵ = **32**     | 3                   | 2³ = 8           | 6                       |
| 6             | 2⁶ = **64**     | 2                   | 2² = 4           | 2                       |

> **Note:** These examples assume a Class C starting point (8 host bits). For Class B (16 host bits) or Class A (24 host bits), the same principle applies — scale the host-bit count accordingly.

> **Trade-off:** More subnets → fewer hosts per subnet. The bit you "take" from host bits for subnetting reduces the number of hosts available within each subnet.

---

## 10. Disadvantage of Subnetting — Extra Routing Step

Without subnetting, reaching a host involves **3 steps**:

```
Step 1: Find Network ID (which organisation/network)
Step 2: Find Host ID (which specific host)
Step 3: Find Port / Process ID (which process on the host)
```

With subnetting, there is **one additional step**:

```
Step 1: Find Network ID (which organisation/network)
Step 2: Find SUBNET ID (which subnet within the organisation)  ← NEW
Step 3: Find Host ID (which specific host within the subnet)
Step 4: Find Port / Process ID (which process on the host)
```

- This adds **computational overhead** at the internal router level — it must perform an additional mask operation to determine the subnet before looking up the host.
- **Trade-off:** The slight increase in routing computation is worth it for the maintenance, fault isolation, and security benefits.

---

## 11. Advantages vs Disadvantages — Summary

| Advantages                                                     | Disadvantages                                            |
| -------------------------------------------------------------- | -------------------------------------------------------- |
| Easy maintenance — smaller networks per admin                  | Extra routing step increases computation                 |
| Better fault isolation — problems stay contained               | Additional address waste (2 reserved per subnet)         |
| Improved security — smaller broadcast domains, per-subnet ACLs | More complex network administration                      |
| Reduced wastage of large Class A/B address blocks              | Requires careful planning (wrong subnet size = problems) |
| Enables separate network policies per department               |                                                          |

---

## 12. All Key Formulas

```
Given: Class C network (8 host bits), reserving k bits for subnetting:

Number of subnets        = 2ᵏ
Remaining host bits      = 8 − k
Addresses per subnet     = 2^(8−k)
Usable hosts per subnet  = 2^(8−k) − 2

Subnet mask last octet:
  k=1 → 10000000 = 128  → full mask: 255.255.255.128
  k=2 → 11000000 = 192  → full mask: 255.255.255.192
  k=3 → 11100000 = 224  → full mask: 255.255.255.224
  k=4 → 11110000 = 240  → full mask: 255.255.255.240

Total usable hosts after subnetting:
  = 2ᵏ × (2^(8−k) − 2)
  = 2⁸ − 2ᵏ⁺¹
  = 256 − 2ᵏ⁺¹

Example: k=1
  Total usable = 256 − 2² = 256 − 4 = 252  ✓ (matches our worked example)
```

---

## 13. Why This Matters for Cybersecurity

- **Subnet = security boundary:** Each subnet boundary enforced by an internal router is a natural **access control checkpoint**. Firewall rules, ACLs, and IDS sensors can be deployed at each subnet boundary to inspect/filter cross-subnet traffic — this is why well-segmented networks are far more resilient to lateral movement after a breach.

- **Blast radius containment:** If a host in S1 (Examination subnet) is compromised, the attacker's ARP poisoning, network scanning, and lateral movement are naturally contained to S1's broadcast domain. They cannot directly reach S2 (Placement) without traversing the internal router — where detection and blocking controls live.

- **Subnet mask as both routing tool and attack target:** The internal router's subnet mask operation (`destination AND mask = subnet ID`) is the same operation exploited in **subnet scanning** reconnaissance. `nmap -sn 200.10.20.0/25` targets exactly the 126 usable hosts in S1 — understanding subnets means understanding precisely what range any scan will cover.

- **Misconfigured subnet masks = routing failures and security gaps:** A wrong subnet mask on an internal router can cause packets destined for S1 to route to S2 (or be dropped), and can create overlapping address ranges that confuse ACLs — a common real-world misconfiguration that both breaks networks and creates security holes.

- **DHCP scope alignment:** In real networks, the DHCP server must be configured with scopes aligned to each subnet. A misconfigured DHCP scope that spans subnet boundaries can accidentally hand out IPs from S2 to hosts that should be in S1 — breaking both routing and security policies.

---

## 14. 🧪 Practical Labs

### Lab 1 — Subnetting Calculator (Python)

```python
# Save as subnetting_calculator.py
# Run: python3 subnetting_calculator.py

import ipaddress

def subnet_a_network(network_cidr: str, reserved_bits: int) -> None:
    """
    Subnet a classful network by reserving k bits from host portion.
    Shows all subnets, their ranges, IDs, broadcasts, and usable hosts.
    """
    base_net = ipaddress.IPv4Network(network_cidr, strict=True)
    original_prefix = base_net.prefixlen
    new_prefix = original_prefix + reserved_bits
    num_subnets = 2 ** reserved_bits
    host_bits = 32 - new_prefix
    addrs_per_subnet = 2 ** host_bits
    usable_per_subnet = addrs_per_subnet - 2

    print(f"\n{'='*65}")
    print(f"  SUBNETTING: {network_cidr}")
    print(f"{'─'*65}")
    print(f"  Original network:      {network_cidr}")
    print(f"  Original prefix:       /{original_prefix}")
    print(f"  Original usable hosts: {base_net.num_addresses - 2}")
    print(f"\n  Reserved bits (k):     {reserved_bits}")
    print(f"  New prefix:            /{new_prefix}")
    print(f"  Number of subnets:     2^{reserved_bits} = {num_subnets}")
    print(f"  Addresses per subnet:  2^{host_bits} = {addrs_per_subnet}")
    print(f"  Usable hosts/subnet:   {addrs_per_subnet} − 2 = {usable_per_subnet}")
    print(f"  Subnet mask:           {ipaddress.IPv4Network(f'0.0.0.0/{new_prefix}').netmask}")
    print(f"  Total usable (all):    {num_subnets} × {usable_per_subnet} = {num_subnets * usable_per_subnet}")
    print(f"  Addresses 'lost':      {base_net.num_addresses - 2 - num_subnets * usable_per_subnet}")
    print(f"\n{'─'*65}")
    print(f"  {'Subnet':<8} {'Network ID':<22} {'First Host':<22} {'Last Host':<22} {'Broadcast':<20} {'Usable'}")
    print(f"{'─'*65}")

    for i, subnet in enumerate(base_net.subnets(prefixlen_diff=reserved_bits), 1):
        net_id  = subnet.network_address
        bcast   = subnet.broadcast_address
        first_h = net_id + 1
        last_h  = bcast - 1
        print(f"  S{i:<7} {str(net_id)+'/'+str(new_prefix):<22} {str(first_h):<22} {str(last_h):<22} {str(bcast):<20} {usable_per_subnet}")

# Lecture worked example: Class C /24 → 2 subnets (k=1)
subnet_a_network("200.10.20.0/24", reserved_bits=1)

# Divide into 4 subnets (k=2)
subnet_a_network("200.10.20.0/24", reserved_bits=2)

# Divide into 8 subnets (k=3)
subnet_a_network("200.10.20.0/24", reserved_bits=3)

# Your lab subnet - further divide into 4 subnets
subnet_a_network("192.168.56.0/24", reserved_bits=2)
```

### Lab 2 — Replicate the Lecture's AND Routing Logic

```python
# Save as subnet_routing_demo.py
# Run: python3 subnet_routing_demo.py
# Demonstrates how the internal router uses the subnet mask to route packets

def route_packet(dest_ip: str, subnet_mask: str,
                 s1_id: str, s2_id: str) -> None:
    """AND destination IP with subnet mask to find which subnet to route to."""

    def ip_to_int(ip):
        parts = [int(x) for x in ip.split('.')]
        return (parts[0]<<24)|(parts[1]<<16)|(parts[2]<<8)|parts[3]

    def int_to_ip(n):
        return '.'.join(str((n>>(8*i))&0xFF) for i in [3,2,1,0])

    dest_int = ip_to_int(dest_ip)
    mask_int = ip_to_int(subnet_mask)
    result   = dest_int & mask_int
    result_ip = int_to_ip(result)

    dest_last = int(dest_ip.split('.')[-1])
    mask_last = int(subnet_mask.split('.')[-1])
    result_last = dest_last & mask_last

    print(f"\nDestination IP: {dest_ip}")
    print(f"  Last octet:      {dest_last:3d} = {format(dest_last,'08b')}")
    print(f"  Subnet mask oct: {mask_last:3d} = {format(mask_last,'08b')}")
    print(f"  AND result:       {result_last:3d} = {format(result_last,'08b')}")
    print(f"  Resulting subnet ID: {result_ip}")
    if result_ip == s1_id:
        print(f"  → Route to: SUBNET 1 (S1) ✅")
    elif result_ip == s2_id:
        print(f"  → Route to: SUBNET 2 (S2) ✅")
    else:
        print(f"  → No matching subnet found ❌")

print("="*55)
print("INTERNAL ROUTER SUBNET ROUTING DEMO")
print("Network: 200.10.20.0/24  →  S1(.0–.127) + S2(.128–.255)")
print("Subnet mask: 255.255.255.128")
print("="*55)

# From the lecture:
route_packet("200.10.20.15",  "255.255.255.128", "200.10.20.0",   "200.10.20.128")
route_packet("200.10.20.130", "255.255.255.128", "200.10.20.0",   "200.10.20.128")
route_packet("200.10.20.1",   "255.255.255.128", "200.10.20.0",   "200.10.20.128")
route_packet("200.10.20.200", "255.255.255.128", "200.10.20.0",   "200.10.20.128")
```

### Lab 3 — Verify Subnet Boundaries in Your Lab

```bash
# Your VirtualBox lab is already a subnetted network
# Parrot OS and Metasploitable2 are both in 192.168.56.0/24

# See your current subnet assignment
ip addr show eth0 | grep inet

# Verify which subnet Metasploitable2 is in
python3 -c "
import ipaddress
host = ipaddress.IPv4Address('192.168.56.101')
net  = ipaddress.IPv4Network('192.168.56.0/24')
print(f'Host {host} in network {net}: {host in net}')
# Check if you and Metasploitable2 are on the same subnet
my_host = ipaddress.IPv4Address('192.168.56.102')  # adjust to your IP
print(f'Same subnet: {host in net and my_host in net}')
"

# Simulate a 2-subnet scenario with ipcalc
sudo apt install ipcalc -y
ipcalc 192.168.56.0/24          # original /24
ipcalc 192.168.56.0/25          # after subnetting into /25 (S1)
ipcalc 192.168.56.128/25        # S2
```

### Lab 4 — Scan Each Subnet Separately (Blast Radius Demo)

```bash
# Demonstrates how subnetting limits scan/attack scope

# Scan ONLY Subnet 1 (S1) — 192.168.56.0/25 (hosts .1–.126)
echo "=== Scanning Subnet 1 only (S1: .0/25) ==="
sudo nmap -sn 192.168.56.0/25
echo ""
echo "Hosts in S1 range (.1 to .126):"

# Scan ONLY Subnet 2 (S2) — 192.168.56.128/25 (hosts .129–.254)
echo "=== Scanning Subnet 2 only (S2: .128/25) ==="
sudo nmap -sn 192.168.56.128/25
echo ""
echo "Hosts in S2 range (.129 to .254):"

# Key security insight:
echo ""
echo "=== SECURITY INSIGHT ==="
echo "If an attacker compromises a host in S1 (.0/25),"
echo "they can only directly ARP-poison and scan S1's 126 hosts."
echo "To reach S2, they must traverse the internal router —"
echo "where firewall/ACL rules can detect and block them."
```

### Lab 5 — Visualise Subnets with iptables Segmentation

```bash
# Simulate per-subnet security policies using iptables
# (Lab environment only — your VirtualBox isolated network)

# Allow SSH from S1 only (first half of /24)
sudo iptables -A INPUT -s 192.168.56.0/25  -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -s 192.168.56.128/25 -p tcp --dport 22 -j DROP

echo "Firewall policy: S1 (.0/25) can SSH in | S2 (.128/25) cannot"
echo ""

# Show the rules
sudo iptables -L INPUT -n -v --line-numbers

# Test: SSH attempt from Metasploitable2 (192.168.56.101 — in S1)
# ssh user@your-parrot-ip   ← from Metasploitable2 console → should ACCEPT

# Clean up
sudo iptables -F INPUT
```

---

## 15. Solved Examples

### Example 1 — Full Subnetting into 2 (from Lecture)

**Given:** `200.10.20.0/24` — divide into 2 subnets
**Find:** Ranges, subnet IDs, broadcast addresses, usable hosts, subnet mask

```
Step 1: Class C → 8 host bits
Step 2: 2 subnets needed → reserve k=1 bit
Step 3: New prefix = 24 + 1 = /25
Step 4: Addresses per subnet = 2^7 = 128 | Usable = 126
Step 5: Subnet mask = 255.255.255.128

Subnet 1 (S1): subnet bit = 0
  Range:     200.10.20.0   – 200.10.20.127
  Subnet ID: 200.10.20.0
  Broadcast: 200.10.20.127
  Usable:    200.10.20.1  – 200.10.20.126  (126 hosts)

Subnet 2 (S2): subnet bit = 1
  Range:     200.10.20.128 – 200.10.20.255
  Subnet ID: 200.10.20.128
  Broadcast: 200.10.20.255
  Usable:    200.10.20.129 – 200.10.20.254 (126 hosts)

Total usable after subnetting: 126 + 126 = 252
(Original: 254 — lost 2 addresses to the extra network IDs/broadcasts)
```

---

### Example 2 — Divide into 4 Subnets

**Given:** `200.10.20.0/24` — divide into 4 subnets
**Find:** All subnet details

```
4 subnets → reserve k=2 bits
New prefix = /26 | Addresses per subnet = 2^6 = 64 | Usable = 62
Subnet mask = 255.255.255.192

Subnet 1: bit pattern 00 → range .0  – .63   | S1 ID: .0  | Broadcast: .63
Subnet 2: bit pattern 01 → range .64 – .127  | S2 ID: .64 | Broadcast: .127
Subnet 3: bit pattern 10 → range .128 – .191 | S3 ID: .128| Broadcast: .191
Subnet 4: bit pattern 11 → range .192 – .255 | S4 ID: .192| Broadcast: .255

Usable per subnet: 62
Total usable: 4 × 62 = 248
```

---

### Example 3 — Find Subnet for a Given Host

**Given:** Host IP `200.10.20.200`, subnet mask `255.255.255.128`
**Find:** Which subnet does this host belong to?

```
Last octet of host:   200 = 11001000
Last octet of mask:   128 = 10000000

AND: 11001000
     10000000
     ────────
     10000000 = 128

Result: Subnet ID = 200.10.20.128 → Subnet 2 (S2)
Range: 200.10.20.128 – 200.10.20.255
```

---

## 16. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              SUBNETTING — EXAM CHEAT SHEET                           ║
╠══════════════════════════════════════════════════════════════════════╣
║  DEFINITION                                                             ║
║  Subnetting = dividing a large network into smaller logical subnets  ║
║  Done INTERNALLY — outside world still sees one network address      ║
║  Network bits: NEVER touched | Only HOST bits used for subnets       ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FORMULAS (Class C /24 base, reserving k bits)                    ║
║  ─────────────────────────────────────────────────────────          ║
║  Number of subnets        = 2ᵏ                                      ║
║  Remaining host bits      = 8 − k  (for Class C)                    ║
║  Addresses per subnet     = 2^(8−k)                                 ║
║  Usable hosts per subnet  = 2^(8−k) − 2                             ║
║  New prefix length        = 24 + k  (for Class C)                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  SUBNET MASK LAST OCTET                                                ║
║  ─────────────────────────────────────────────────────          ║
║  k=1: 10000000 = 128 → 255.255.255.128                             ║
║  k=2: 11000000 = 192 → 255.255.255.192                             ║
║  k=3: 11100000 = 224 → 255.255.255.224                             ║
║  k=4: 11110000 = 240 → 255.255.255.240                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  WORKED EXAMPLE (200.10.20.0/24 → 2 subnets, k=1)                    ║
║  ─────────────────────────────────────────────────────          ║
║  S1: 200.10.20.0 – .127  | ID: .0  | Broadcast: .127 | Usable: 126 ║
║  S2: 200.10.20.128 – .255| ID: .128| Broadcast: .255 | Usable: 126 ║
║  Subnet mask: 255.255.255.128                                       ║
║  Total usable: 252 (lost 2 vs original 254)                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  INTERNAL ROUTING VIA SUBNET MASK (AND operation)                     ║
║  ─────────────────────────────────────────────────────          ║
║  Dest IP AND subnet mask = Subnet ID → determines which subnet      ║
║  .15  AND 128 = 0   → S1  |  .130 AND 128 = 128 → S2              ║
╠══════════════════════════════════════════════════════════════════════╣
║  ADVANTAGES vs DISADVANTAGES                                           ║
║  ─────────────────────────────────────────────────────          ║
║  ✅ Easy maintenance      | ❌ Extra routing step (computation)      ║
║  ✅ Fault isolation        | ❌ 2 addresses lost per extra subnet    ║
║  ✅ Better security        | ❌ More complex to administer           ║
║  ✅ Less waste in big nets |                                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  ROUTING STEPS: Without subnetting = 3 | With subnetting = 4        ║
║  Network → [Subnet] → Host → Port/Process                           ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Subnetting Class A and Class B — same concept, more host bits available
- [ ] VLSM — Variable Length Subnet Masking (subnets of different sizes in one network)
- [ ] CIDR and Subnetting together — the modern combined approach
- [ ] NAT — how private subnetted networks reach the public internet
- [ ] IPv6 subnetting — the 128-bit equivalent

---

_Notes compiled from: Networking Course Lecture 46 — Subnetting (Classful Addressing)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
