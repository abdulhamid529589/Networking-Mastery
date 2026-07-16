# 🌐 IP Addressing — Classless Addressing (CIDR)

### Cybersecurity Student Notes | Networking Course — Lecture 45

> **Source:** Gate Smashers — Classless Addressing / CIDR
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Quick Recap — Why Classless Addressing Was Needed](#1-quick-recap--why-classless-addressing-was-needed)
2. [Core Concept — No Classes, Only Blocks](#2-core-concept--no-classes-only-blocks)
3. [Block ID vs Host ID](#3-block-id-vs-host-id)
4. [The /n Notation — CIDR Prefix Length](#4-the-n-notation--cidr-prefix-length)
5. [Deriving the Mask from /n](#5-deriving-the-mask-from-n)
6. [Worked Example — 200.10.20.40/28](#6-worked-example--2001020-40-28)
   - [Step 1 — Count Network and Host Bits](#step-1--count-network-and-host-bits)
   - [Step 2 — Number of Hosts](#step-2--number-of-hosts)
   - [Step 3 — Derive the Mask](#step-3--derive-the-mask)
   - [Step 4 — Find the Network (Block) ID — Method 1 (Zero the Host Bits)](#step-4--find-the-network-block-id--method-1-zero-the-host-bits)
   - [Step 5 — Find the Network (Block) ID — Method 2 (AND with Mask)](#step-5--find-the-network-block-id--method-2-and-with-mask)
7. [Three Rules of Classless Addressing (CIDR)](#7-three-rules-of-classless-addressing-cidr)
   - [Rule 1 — Addresses Must Be Contiguous](#rule-1--addresses-must-be-contiguous)
   - [Rule 2 — Number of Addresses Must Be a Power of 2](#rule-2--number-of-addresses-must-be-a-power-of-2)
   - [Rule 3 — First Address Must Be Evenly Divisible by Block Size](#rule-3--first-address-must-be-evenly-divisible-by-block-size)
8. [Classful vs Classless — Side-by-Side Comparison](#8-classful-vs-classless--side-by-side-comparison)
9. [Common /n Values — Quick Reference](#9-common-n-values--quick-reference)
10. [Why This Matters for Cybersecurity](#10-why-this-matters-for-cybersecurity)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Solved Examples](#12-solved-examples)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. Quick Recap — Why Classless Addressing Was Needed

As established in Lecture 44, Classful Addressing has four major problems:

```
1. IP address wastage (Class A: 16.7M hosts/net — far too big)
2. Inflexibility (no option between 254 and 65,534 hosts)
3. Maintenance overhead (large flat networks)
4. Security weaknesses (large broadcast domains)
```

**Classless Addressing (CIDR — Classless Inter-Domain Routing)** was introduced in **1993** as the solution — primarily to fix wastage and inflexibility. Managed by **IANA (Internet Assigned Numbers Authority)**, it allocates exactly what the user needs.

---

## 2. Core Concept — No Classes, Only Blocks

```
Classful:    Fixed classes (A=126 nets/16.7M hosts, B=16384 nets/65534 hosts, C=2M nets/254 hosts)
             → User must accept whichever class is closest to their need

Classless:   NO CLASSES — forget Class A, B, C, D, E entirely
             → Block-based: user requests X addresses, they get exactly X addresses
             → IANA provides a block of exactly the demanded size
```

**Key result:** Wastage of IP addresses is drastically reduced — no more forcing an organisation needing 1,024 hosts into a Class B block of 65,534.

---

## 3. Block ID vs Host ID

In classless addressing, the 32-bit IPv4 address is still divided into two parts — but the split point is **variable** (not fixed by class):

```
32-bit IPv4 Address:
┌─────────────────────────────┬──────────────────┐
│         BLOCK ID            │     HOST ID      │
│  (identifies the network)   │ (identifies host │
│                             │  within network) │
│   n bits (variable)         │  (32 − n) bits   │
└─────────────────────────────┴──────────────────┘

Where n is specified by the /n notation (see Section 4)
```

> **Terminology:** In classless addressing, "Block ID" replaces the classful term "Network ID" — both mean the same thing (the address that identifies the network itself). The terms can be used interchangeably.

---

## 4. The /n Notation — CIDR Prefix Length

### What /n Means

The notation is: **`x.y.z.w/n`**

- `x.y.z.w` = the standard 4-octet IPv4 address (same as always)
- `/n` = **prefix length** — the number of bits used to represent the **Block/Network ID**

```
Example: 200.10.20.40/28

/28 means:
  → 28 bits are used for the BLOCK (Network) ID
  → 32 − 28 = 4 bits are used for the HOST ID
```

### Why /n Is Needed

Without /n, there would be no way to know where the network portion ends and the host portion begins — because unlike classful addressing, there are no fixed class boundaries to tell you. The `/n` carries this information explicitly with every address.

### /n as a Mask

The `/n` value directly tells you the **mask** — a 32-bit value consisting of exactly `n` continuous 1s followed by `(32−n)` continuous 0s:

```
/28 → 28 ones followed by 4 zeros:
      11111111.11111111.11111111.11110000
      = 255.255.255.240
```

---

## 5. Deriving the Mask from /n

**General rule:** Write `n` continuous 1s, then `(32−n)` zeros, across the 32-bit space, then convert each octet to decimal.

```
/n    Binary Mask                              Decimal Mask
──────────────────────────────────────────────────────────
/8    11111111.00000000.00000000.00000000  →  255.0.0.0
/16   11111111.11111111.00000000.00000000  →  255.255.0.0
/24   11111111.11111111.11111111.00000000  →  255.255.255.0
/25   11111111.11111111.11111111.10000000  →  255.255.255.128
/26   11111111.11111111.11111111.11000000  →  255.255.255.192
/27   11111111.11111111.11111111.11100000  →  255.255.255.224
/28   11111111.11111111.11111111.11110000  →  255.255.255.240
/29   11111111.11111111.11111111.11111000  →  255.255.255.248
/30   11111111.11111111.11111111.11111100  →  255.255.255.252
```

**Last-octet binary → decimal (for the /25 to /30 range):**

```
10000000 = 128      → /25
11000000 = 192      → /26
11100000 = 224      → /27
11110000 = 240      → /28  ← our worked example
11111000 = 248      → /29
11111100 = 252      → /30
```

---

## 6. Worked Example — 200.10.20.40/28

### Step 1 — Count Network and Host Bits

```
/28 → 28 bits for Network (Block) ID
      32 − 28 = 4 bits for Host ID
```

### Step 2 — Number of Hosts

```
Host bits = 4
Number of addresses in this block = 2⁴ = 16
```

> **Note:** This is the total number of IP addresses in the block (including the Block ID address and the broadcast address). Usable hosts = 16 − 2 = **14**.

---

### Step 3 — Derive the Mask

```
/28 → 28 continuous 1s, then 4 zeros:

11111111.11111111.11111111.11110000
   255  .   255  .   255  .  240

Mask = 255.255.255.240
```

---

### Step 4 — Find the Network (Block) ID — Method 1 (Zero the Host Bits)

**Concept:** The Block ID is the IP address with all host bits set to 0. The first 28 (network) bits stay exactly as they are.

```
IP Address: 200.10.20.40

Step 1: Keep first 24 bits unchanged (200.10.20 are already full octets):
  200 → 11001000  ← keep all 8 bits
   10 → 00001010  ← keep all 8 bits
   20 → 00010100  ← keep all 8 bits
  (That's 24 bits. We need 28, so we take 4 more from the last octet.)

Step 2: Open the last octet (40) in binary:
  40 → 00101000

Step 3: The first 4 bits of this octet are part of the NETWORK ID (bit 25–28):
  0010 | 1000
  ↑──────┘     ↑──────────────┘
  network bits  host bits (these 4 bits → set to 0000)

Step 4: Set host bits to 0:
  Network portion of last octet: 0010
  Host portion zeroed:           0000
  Last octet result: 00100000 = 32

Step 5: Write the Block (Network) ID:
  200.10.20.32/28
```

**Block ID = `200.10.20.32/28`**

> This is the address by which the entire world identifies the network that `200.10.20.40` belongs to.

---

### Step 5 — Find the Network (Block) ID — Method 2 (AND with Mask)

This is equivalent to Method 1 — perform bitwise AND between the IP address and the mask:

```
IP Address:  200 . 10  . 20  . 40
Mask:        255 . 255 . 255 . 240

AND shortcuts:
  200 AND 255 = 200  (copy as-is — any value AND 255 = same value)
   10 AND 255 = 10   (copy as-is)
   20 AND 255 = 20   (copy as-is)
   40 AND 240 = ?    ← need to calculate this one

40  in binary:  00101000
240 in binary:  11110000

Bitwise AND:
  0 AND 1 = 0
  0 AND 1 = 0
  1 AND 1 = 1
  0 AND 1 = 0
  1 AND 0 = 0
  0 AND 0 = 0
  0 AND 0 = 0
  0 AND 0 = 0
  ──────────────
  Result:   00100000 = 32

Block ID = 200.10.20.32/28  ✅ (same answer as Method 1)
```

> **Use whichever method is faster for you in the exam.** Method 1 (zero the host bits) is often quicker for mental arithmetic. Method 2 (AND with mask) is more rigorous and works universally.

---

## 7. Three Rules of Classless Addressing (CIDR)

For a block of addresses to be **valid** under CIDR, it must satisfy all three rules:

---

### Rule 1 — Addresses Must Be Contiguous

```
All IP addresses in the block must be consecutive — no gaps.

Valid:   200.10.20.32, 200.10.20.33, 200.10.20.34 ... 200.10.20.47
         (32 through 47 — 16 consecutive addresses)

Invalid: 200.10.20.32, 200.10.20.34, 200.10.20.36 ...
         (skipping .33, .35 etc. — not contiguous → NOT a valid CIDR block)
```

---

### Rule 2 — Number of Addresses Must Be a Power of 2

```
Block size must equal 2ⁿ for some integer n.

Valid sizes:   1, 2, 4, 8, 16, 32, 64, 128, 256, 512, 1024 ...
               (2⁰, 2¹, 2², 2³, 2⁴, 2⁵, 2⁶, 2⁷, 2⁸, 2⁹, 2¹⁰ ...)

Example:  Block size = 16 → 2⁴ ✅ valid
          Block size = 17 → NOT a power of 2 ❌ invalid CIDR block

Why? Because host bits = n, and 2ⁿ is always a power of 2.
     There is no integer n such that 2ⁿ = 17.
```

---

### Rule 3 — First Address Must Be Evenly Divisible by Block Size

```
The Block ID (first address of the network) must be divisible by
the block size — i.e., the last (host) bits of the Block ID must ALL be 0.

Our example:
  Block ID last octet:  32 = 00100000
  Block size:           16 = 2⁴
  Host bits (last 4):   0000  ← all zeros ✅ → 32 is divisible by 16 ✓

Invalid example:
  If Block ID were 33:  33 = 00100001
  Last 4 bits:          0001 ← NOT all zeros ❌ → 33 is NOT divisible by 16 ✗
```

**Shortcut to check Rule 3:** Convert the last octet of the Block ID to binary. Look at the last `host_bits` bits. If they are **all zeros**, the block is valid. If any are 1, it is invalid.

```
Block size = 2ⁿ
→ Check the last n bits of the Block ID
→ If all n bits are 0: ✅ divisible — valid CIDR block
→ If any of the n bits are 1: ❌ not divisible — invalid CIDR block
```

---

## 8. Classful vs Classless — Side-by-Side Comparison

| Property                          | Classful Addressing                                   | Classless Addressing (CIDR)                               |
| --------------------------------- | ----------------------------------------------------- | --------------------------------------------------------- |
| **Introduced**                    | 1980s (original)                                      | 1993 (RFC 1517–1520)                                      |
| **Classes**                       | 5 fixed classes (A/B/C/D/E)                           | No classes — block-based                                  |
| **Address notation**              | `x.y.z.w` only                                        | `x.y.z.w/n` — prefix length required                      |
| **Block/network size**            | Fixed per class (254 / 65,534 / 16.7M)                | Variable — any power of 2                                 |
| **Mask type**                     | Default mask per class (implicit)                     | Derived from /n (explicit, carried with every address)    |
| **Flexibility**                   | Low — must fit a class                                | High — exactly the size needed                            |
| **Wastage**                       | High (Class A/B dramatically oversized for most orgs) | Very low (allocate precisely what's needed)               |
| **Managed by**                    | Fixed standard (RFCs defining classes)                | IANA — allocates blocks on demand                         |
| **Class identification question** | First exam question: "Which class?"                   | Never asked — no classes exist                            |
| **Key exam question**             | "Find Network ID, last host, broadcast"               | "Find Block ID, number of hosts, verify rules"            |
| **3 validation rules**            | Not applicable                                        | Contiguous + power of 2 + first address divisible by size |

---

## 9. Common /n Values — Quick Reference

| Prefix  | Network Bits | Host Bits | Block Size | Usable Hosts   | Mask                |
| ------- | ------------ | --------- | ---------- | -------------- | ------------------- |
| /8      | 8            | 24        | 16,777,216 | 16,777,214     | 255.0.0.0           |
| /16     | 16           | 16        | 65,536     | 65,534         | 255.255.0.0         |
| /24     | 24           | 8         | 256        | 254            | 255.255.255.0       |
| /25     | 25           | 7         | 128        | 126            | 255.255.255.128     |
| /26     | 26           | 6         | 64         | 62             | 255.255.255.192     |
| /27     | 27           | 5         | 32         | 30             | 255.255.255.224     |
| **/28** | **28**       | **4**     | **16**     | **14**         | **255.255.255.240** |
| /29     | 29           | 3         | 8          | 6              | 255.255.255.248     |
| /30     | 30           | 2         | 4          | 2              | 255.255.255.252     |
| /31     | 31           | 1         | 2          | 0\*            | 255.255.255.254     |
| /32     | 32           | 0         | 1          | 1 (host route) | 255.255.255.255     |

> **/28 is the worked example in this lecture** — highlighted above.

> **/31** has no usable hosts in classical terms (both addresses are Network ID and Broadcast) — used for point-to-point links per RFC 3021.

> Usable hosts = Block size − 2 (subtract Network/Block ID and Broadcast address).

---

## 10. Why This Matters for Cybersecurity

- **Every nmap scan uses CIDR notation:** When you type `nmap -sn 192.168.56.0/24`, the `/24` is a CIDR prefix length — telling nmap that the network is 24 bits wide and there are 2⁸ = 256 total addresses (254 usable hosts) to probe. Understanding `/n` means understanding exactly what range any tool will operate on.

- **Subnet misconfiguration = attack surface expansion:** A common mistake is over-allocating — e.g., giving a 5-host DMZ a `/24` (254 usable hosts) instead of a `/29` (6 usable hosts). This needlessly expands the blast radius for ARP poisoning, broadcast-based reconnaissance, and lateral movement. CIDR knowledge lets you recognise and flag these misconfigurations.

- **CIDR aggregation and route injection:** BGP (Border Gateway Protocol — the routing protocol of the internet) uses CIDR prefix lengths to advertise routes. A misconfigured or maliciously injected more-specific route (e.g., advertising `/25` instead of `/24`) can **hijack traffic** by making routers prefer the attacker's path — this is called a **BGP prefix hijack**, one of the most impactful attacks on internet infrastructure.

- **Firewall rule precision:** ACLs and firewall rules use CIDR notation to define source/destination ranges. A rule written as `0.0.0.0/0` means "all traffic" — a common critical misconfiguration. Knowing that `/0` means 0 network bits (zero bits identify the network → the entire address space) is essential for correctly reading and writing firewall rules.

- **VLSM and network segmentation design:** The next topic after CIDR is VLSM (Variable Length Subnet Masking) — combining CIDR blocks of different sizes to segment a network efficiently. This is exactly how real security architects design tiered networks (DMZ, internal, management) with appropriately sized subnets and routing boundaries between them.

---

## 11. 🧪 Practical Labs

### Lab 1 — CIDR Block Calculator (Python)

```python
# Save as cidr_calculator.py
# Run: python3 cidr_calculator.py

import ipaddress

def cidr_info(cidr_str: str) -> None:
    """
    Full analysis of a CIDR block:
      - Block (Network) ID
      - Mask
      - Block size & usable hosts
      - First host, Last host, Broadcast
      - CIDR rule validation
    """
    net = ipaddress.IPv4Network(cidr_str, strict=False)
    prefix  = net.prefixlen
    host_bits = 32 - prefix
    block_size = net.num_addresses
    usable = block_size - 2

    first_host = net.network_address + 1
    last_host  = net.broadcast_address - 1

    print(f"\n{'='*55}")
    print(f"  CIDR Block: {cidr_str}")
    print(f"{'─'*55}")
    print(f"  Block (Network) ID:  {net.network_address}/{prefix}")
    print(f"  Broadcast Address:   {net.broadcast_address}")
    print(f"  Subnet Mask:         {net.netmask}")
    print(f"  Prefix Length:       /{prefix}")
    print(f"  Network Bits:        {prefix}")
    print(f"  Host Bits:           {host_bits}")
    print(f"  Block Size:          2^{host_bits} = {block_size}")
    print(f"  Usable Hosts:        {block_size} - 2 = {usable}")
    print(f"  First Host:          {first_host}")
    print(f"  Last Host:           {last_host}")

    # Validate the 3 CIDR rules
    last_octet = int(str(net.network_address).split('.')[-1])
    r3_ok = (last_octet % block_size == 0)

    print(f"\n  --- 3 CIDR RULES VALIDATION ---")
    print(f"  Rule 1 Contiguous:         ✅ Always true for CIDR blocks")
    print(f"  Rule 2 Power of 2:         ✅ {block_size} = 2^{host_bits}")
    print(f"  Rule 3 Divisibility:       "
          f"{'✅' if r3_ok else '❌'} {last_octet} ÷ {block_size} = "
          f"{'integer ✓' if r3_ok else 'NOT integer ✗'}")

# Test cases — from lecture and common exam examples
test_cases = [
    "200.10.20.40/28",    # Lecture worked example (strict=False handles host addr)
    "200.10.20.32/28",    # The corrected block ID
    "192.168.56.0/24",    # Your Metasploitable2 lab subnet
    "10.0.0.0/8",         # Class A equivalent in CIDR
    "172.16.0.0/16",      # Class B equivalent in CIDR
    "192.168.1.0/26",     # 62 hosts
    "10.10.10.128/25",    # 126 hosts — second half of a /24
]

for cidr in test_cases:
    cidr_info(cidr)
```

### Lab 2 — Verify Your Lab Network in CIDR

```bash
# See your actual lab CIDR assignments
ip addr show eth0

# Expected output for your Parrot OS in VirtualBox host-only network:
#   inet 192.168.56.102/24   ← /24 = CIDR prefix length

# Parse manually:
#   /24 → 24 network bits, 8 host bits
#   Block size = 2^8 = 256 addresses
#   Usable hosts = 254
#   Block ID = 192.168.56.0/24
#   Broadcast = 192.168.56.255

# Verify with ipcalc
sudo apt install ipcalc -y
ipcalc 192.168.56.102/24
# Output includes: Network, Broadcast, HostMin, HostMax, Hosts/Net

# Check which CIDR block Metasploitable2 is in
ping -c 1 192.168.56.101
ipcalc 192.168.56.101/24
# Same /24 block → same broadcast domain as Parrot OS
```

### Lab 3 — CIDR Rule 3 Checker (Divisibility)

```python
# Save as cidr_rule3_checker.py
# Run: python3 cidr_rule3_checker.py

def check_rule3(network_address: str, prefix: int) -> None:
    """
    Check CIDR Rule 3: First address of block divisible by block size.
    Uses the binary last-n-bits-must-be-zero shortcut from the lecture.
    """
    host_bits  = 32 - prefix
    block_size = 2 ** host_bits
    last_octet = int(network_address.split('.')[-1])
    binary_lo  = format(last_octet, '08b')

    # Last host_bits bits of the last octet must all be 0
    relevant_bits = binary_lo[-host_bits:] if host_bits <= 8 else binary_lo
    all_zero = all(b == '0' for b in relevant_bits)

    print(f"\nNetwork: {network_address}/{prefix}")
    print(f"  Block size:         {block_size} (2^{host_bits})")
    print(f"  Last octet:         {last_octet} = {binary_lo}")
    print(f"  Last {host_bits} bits:         {relevant_bits}  ← must be all 0s")
    print(f"  Rule 3 (div by {block_size:>3}): {'✅ VALID' if all_zero else '❌ INVALID'} "
          f"({last_octet} {'÷' if all_zero else 'not divisible by'} {block_size} "
          f"{'= ' + str(last_octet // block_size) if all_zero else ''})")

# From the lecture — valid example
check_rule3("200.10.20.32", 28)   # 32 ÷ 16 = 2  ✅

# Invalid — if Block ID were 33
check_rule3("200.10.20.33", 28)   # 33 ÷ 16 = 2.0625  ❌

# More examples
check_rule3("192.168.1.0",  24)   # 0 ÷ 256 = 0  ✅
check_rule3("192.168.1.64", 26)   # 64 ÷ 64 = 1  ✅
check_rule3("192.168.1.65", 26)   # 65 ÷ 64 → ❌
check_rule3("10.10.10.128", 25)   # 128 ÷ 128 = 1  ✅
check_rule3("10.10.10.100", 25)   # 100 ÷ 128 → ❌
```

### Lab 4 — nmap Scan Using CIDR — Applied Understanding

```bash
# Your lab: Parrot OS + Metasploitable2 on 192.168.56.0/24

# Understand what /24 means before scanning:
python3 -c "
import ipaddress
net = ipaddress.IPv4Network('192.168.56.0/24')
print(f'Scanning {net.num_addresses - 2} hosts ({net.network_address+1} – {net.broadcast_address-1})')
"

# Host discovery scan using CIDR notation
sudo nmap -sn 192.168.56.0/24

# Notice: nmap uses the CIDR block rules — it skips .0 (network) and .255 (broadcast)
# and scans the 254 usable host addresses

# Now try a /28 — only 14 hosts, useful for a small isolated DMZ
# (Hypothetical — scan only what actually exists in your lab)
sudo nmap -sn 192.168.56.32/28
# Scans: .33 through .46 (14 usable hosts in this /28 block)
# Notice .32 (block ID) and .47 (broadcast) are not probed as hosts
```

### Lab 5 — Firewall Rule Precision Using CIDR

```bash
# Demonstrate how CIDR precision affects firewall/iptables rules

# Overly broad rule (classful thinking — allow entire /24 to reach SSH):
sudo iptables -A INPUT -s 192.168.56.0/24 -p tcp --dport 22 -j ACCEPT
# Allows 254 hosts — any host on your lab LAN can SSH in

# CIDR-precise rule (allow only /30 — 2 usable hosts, e.g., just your jump box):
sudo iptables -D INPUT -s 192.168.56.0/24 -p tcp --dport 22 -j ACCEPT  # remove old rule
sudo iptables -A INPUT -s 192.168.56.100/30 -p tcp --dport 22 -j ACCEPT
# Now only 192.168.56.101 and 192.168.56.102 can SSH in (2 hosts)
# .100 = block ID, .103 = broadcast → not usable as sources

# View current rules
sudo iptables -L INPUT -n -v --line-numbers

# Clean up lab rules
sudo iptables -F INPUT
```

---

## 12. Solved Examples

### Example 1 — Full CIDR Analysis (from Lecture)

**Given:** `200.10.20.40/28`
**Find:** Number of hosts, mask, Block ID

```
Step 1 — Count bits:
  /28 → 28 network bits, 4 host bits

Step 2 — Number of hosts:
  2⁴ = 16 total addresses → 16 − 2 = 14 usable hosts

Step 3 — Mask:
  28 ones + 4 zeros: 11111111.11111111.11111111.11110000
  = 255.255.255.240

Step 4 — Block ID (zero the host bits):
  200.10.20 → keep as-is (24 bits = fully in network portion)
  40 = 00101000 → take first 4 bits (network): 0010
                → zero last 4 bits (host):     0000
  Last octet = 00100000 = 32

Block ID = 200.10.20.32/28

Step 5 — Verify all 3 CIDR rules:
  Rule 1: Block .32 to .47 — contiguous ✅
  Rule 2: 16 = 2⁴ → power of 2 ✅
  Rule 3: 32 ÷ 16 = 2 (integer) → last 4 bits of 32 = 0000 ✅
```

---

### Example 2 — Find All Key Values for `/26`

**Given:** `172.16.45.100/26`
**Find:** Block ID, mask, number of hosts, first host, last host, broadcast

```
/26 → 26 network bits, 6 host bits
Block size = 2⁶ = 64
Usable hosts = 64 − 2 = 62
Mask = 255.255.255.192 (11111111.11111111.11111111.11000000)

Last octet of IP: 100 = 01100100
Network bits of last octet (first 2): 01
Host bits (last 6): 100100 → set to 000000
Last octet of Block ID: 01000000 = 64

Block ID = 172.16.45.64/26
First host = 172.16.45.65
Last host  = 172.16.45.126
Broadcast  = 172.16.45.127

Rule 3 check: 64 ÷ 64 = 1 ✅
```

---

### Example 3 — Validate a CIDR Block

**Given:** Block `195.10.30.17/28` — is this a valid CIDR block?

```
/28 → host bits = 4, block size = 16

Rule 3: First address = 195.10.30.17
  17 in binary: 00010001
  Last 4 bits: 0001 ← NOT all zeros
  17 ÷ 16 = 1.0625 → NOT an integer

❌ INVALID CIDR block — 17 is not divisible by 16.
   The valid block starting near 17 would be 195.10.30.16/28
   (16 ÷ 16 = 1 → last 4 bits of 16 = 0000 ✅)
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║         CLASSLESS ADDRESSING (CIDR) — EXAM CHEAT SHEET               ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE IDENTITY                                                         ║
║  ─────────────────────────────────────────────────────────          ║
║  Classless Addressing = CIDR (Classless Inter-Domain Routing)        ║
║  Introduced: 1993 (RFC 1517–1520)                                    ║
║  Managed by: IANA (Internet Assigned Number Authority)               ║
║  No classes — user requests X addresses, gets exactly X addresses    ║
╠══════════════════════════════════════════════════════════════════════╣
║  NOTATION: x.y.z.w/n                                                   ║
║  ─────────────────────────────────────────────────────          ║
║  /n = prefix length = number of NETWORK (Block ID) bits             ║
║  Host bits = 32 − n                                                  ║
║  Block size = 2^(32−n)  |  Usable hosts = 2^(32−n) − 2             ║
╠══════════════════════════════════════════════════════════════════════╣
║  FINDING THE MASK FROM /n                                               ║
║  ─────────────────────────────────────────────────────          ║
║  Write n continuous 1s, then (32−n) zeros                           ║
║  /28 → 11111111.11111111.11111111.11110000 = 255.255.255.240        ║
╠══════════════════════════════════════════════════════════════════════╣
║  FINDING BLOCK ID — 2 METHODS (same answer)                            ║
║  ─────────────────────────────────────────────────────          ║
║  Method 1: Keep first n bits of IP address, zero the rest           ║
║  Method 2: AND the IP address with the mask                         ║
║            (X AND 255 = X; X AND 0 = 0; X AND 240 → calculate)     ║
╠══════════════════════════════════════════════════════════════════════╣
║  3 CIDR VALIDATION RULES (ALL must be true)                           ║
║  ─────────────────────────────────────────────────────          ║
║  1. Addresses must be CONTIGUOUS (no gaps in the block)             ║
║  2. Block size must be a POWER OF 2 (2¹, 2², 2³, 2⁴ ...)           ║
║  3. First address (Block ID) must be DIVISIBLE by block size        ║
║     → Shortcut: last (host bits) of Block ID must ALL be 0         ║
╠══════════════════════════════════════════════════════════════════════╣
║  WORKED EXAMPLE (200.10.20.40/28)                                      ║
║  ─────────────────────────────────────────────────────          ║
║  /28 → 28 network bits, 4 host bits                                 ║
║  Block size = 2⁴ = 16  |  Usable = 14                              ║
║  Mask = 255.255.255.240                                             ║
║  40 = 00101000 → zero last 4 → 00100000 = 32                       ║
║  Block ID = 200.10.20.32/28                                         ║
║  Rule 3: 32 ÷ 16 = 2 ✅                                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  COMMON PREFIX QUICK REFERENCE                                         ║
║  ─────────────────────────────────────────────────────          ║
║  /24 → 256 total, 254 usable, mask 255.255.255.0                    ║
║  /25 → 128 total, 126 usable, mask 255.255.255.128                  ║
║  /26 → 64  total,  62 usable, mask 255.255.255.192                  ║
║  /27 → 32  total,  30 usable, mask 255.255.255.224                  ║
║  /28 → 16  total,  14 usable, mask 255.255.255.240 ← LECTURE EX    ║
║  /29 →  8  total,   6 usable, mask 255.255.255.248                  ║
║  /30 →  4  total,   2 usable, mask 255.255.255.252                  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Subnetting — dividing one block into multiple smaller CIDR blocks
- [ ] VLSM — Variable Length Subnet Masking (subnets of different sizes)
- [ ] NAT — how RFC 1918 private addresses share public IPs (consequence of IPv4 exhaustion solved partially by CIDR)
- [ ] IPv6 — the permanent solution (128-bit addresses, no classful waste)
- [ ] BGP — how routers on the internet exchange CIDR prefix routes

---

_Notes compiled from: Networking Course Lecture 45 — Classless Addressing (CIDR)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
