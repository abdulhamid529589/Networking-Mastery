# 🌐 IP Addressing — Class C

### " "Networking Course — Lecture 42

> **Source:** Gate Smashers — Class C in IP Addressing
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Class C — Prefix & Identification](#1-class-c--prefix--identification)
2. [Class C Range — First Octet](#2-class-c-range--first-octet)
3. [Number of IP Addresses in Class C](#3-number-of-ip-addresses-in-class-c)
4. [Network ID vs Host ID Split](#4-network-id-vs-host-id-split)
5. [Number of Networks](#5-number-of-networks)
6. [Number of Hosts per Network](#6-number-of-hosts-per-network)
7. [Default Subnet Mask — Class C](#7-default-subnet-mask--class-c)
8. [Network ID Extraction — Worked Example](#8-network-id-extraction--worked-example)
9. [All Classes Side-by-Side](#9-all-classes-side-by-side)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. Class C — Prefix & Identification

### The Class C Identifier Bits

```
32-bit IPv4 Address (4 octets):
┌──────────────────────┬──────────┬──────────┬──────────┐
│      Octet 1 (8b)    │  Oct 2   │  Oct 3   │  Oct 4   │
├───┬──────────────────┼──────────┼──────────┼──────────┤
│1 1 0│  5 variable    │  8 bits  │  8 bits  │  8 bits  │
│     │  network bits  │ network  │ network  │  HOST    │
└───┴──────────────────┴──────────┴──────────┴──────────┘
 ↑↑↑
 Fixed prefix = "110"
 These 3 positions CANNOT be changed
```

### How to Instantly Identify Class C

```
Rule: If first 3 bits of IP address = 1 1 0 → Class C

In practice: Just check first octet value
  192 to 223 → Class C

Why 192?  110 00000 = 192 (minimum with prefix 110)
Why 223?  110 11111 = 223 (maximum with prefix 110)
```

### Class A / B / C Prefix Comparison

| Class | Fixed Prefix | Fixed Bits | First Octet Range | Variable Bits in Oct 1 |
| ----- | ------------ | ---------- | ----------------- | ---------------------- |
| A     | 0            | 1          | 0–127             | 7                      |
| B     | 10           | 2          | 128–191           | 6                      |
| **C** | **110**      | **3**      | **192–223**       | **5**                  |

---

## 2. Class C Range — First Octet

### Deriving the Range

```
Prefix: 1 1 0 (fixed — cannot change)
Remaining 5 bits in first octet: variable

Minimum (all 5 variable = 0):
  1 1 0 0 0 0 0 0 = 192

Maximum (all 5 variable = 1):
  1 1 0 1 1 1 1 1 = 223

Range: 192 – 223
Total first-octet values: 223 - 192 + 1 = 32 different values
```

### Step-by-Step Binary → Decimal Verification

```
Converting 11000000 (minimum) to decimal:

Position:  7   6   5   4   3   2   1   0
Bits:      1   1   0   0   0   0   0   0

Value = (1×2⁷)+(1×2⁶)+(0×2⁵)+(0×2⁴)+(0×2³)+(0×2²)+(0×2¹)+(0×2⁰)
      = 128 + 64 + 0 + 0 + 0 + 0 + 0 + 0
      = 192 ✓

Converting 11011111 (maximum) to decimal:

Position:  7   6   5   4   3   2   1   0
Bits:      1   1   0   1   1   1   1   1

Value = 128 + 64 + 0 + 16 + 8 + 4 + 2 + 1
      = 223 ✓
```

---

## 3. Number of IP Addresses in Class C

```
Total bits in IPv4:    32
Fixed prefix bits:      3  (cannot change — always "110")
Variable bits:         29  (32 - 3 = 29)

Total Class C addresses = 2²⁹ = 536,870,912 ≈ 537 million

Context:
  Total IPv4 space:  2³² ≈ 4.3 billion
  Class C portion:   2²⁹ = 12.5% of total IPv4 space
  (2²⁹ / 2³² = 1/8 = 12.5%)

Compared to other classes:
  Class A: 2³¹ = 50%   of IPv4 space
  Class B: 2³⁰ = 25%   of IPv4 space
  Class C: 2²⁹ = 12.5% of IPv4 space  ← this lecture
```

---

## 4. Network ID vs Host ID Split

### Class C Split: 24 + 8

```
32-bit IP address:
[────── First 3 octets (24 bits) ──────][── Last octet (8 bits) ──]
           NETWORK ID                           HOST ID
  (identifies which network)          (identifies host in network)

Compare all classes:
  Class A:  8-bit network  + 24-bit host  → few nets, many hosts
  Class B: 16-bit network  + 16-bit host  → balanced
  Class C: 24-bit network  +  8-bit host  → many nets, few hosts
```

```
Example: 194.2.3.4

  194.2.3  → Network ID (first 3 octets)
  4        → Host ID    (last octet only)

  Network:       194.2.3.0
  Host ID value: 4  (this is the 4th machine in network 194.2.3.0)
```

---

## 5. Number of Networks

### Calculation

```
Network ID uses first 3 octets = 24 bits total
Fixed prefix (1 1 0) = 3 bits → cannot vary (not counted)
Variable network bits = 24 - 3 = 21 bits

Number of networks = 2²¹ = 2,097,152 ≈ 20 lakh (20 lac)
```

### Why Exactly 2²¹ Networks? Visual Proof

```
First octet values: 192 to 223 → 32 different values
Second octet values: 0 to 255  → 256 different values
Third octet values:  0 to 255  → 256 different values

Total combinations = 32 × 256 × 256

In powers of 2:
  32  = 2⁵
  256 = 2⁸
  256 = 2⁸

  32 × 256 × 256 = 2⁵ × 2⁸ × 2⁸ = 2²¹ = 2,097,152 ✓
```

### Class C Has the Most Networks of Any Class

```
Class A: 126 networks     (very few — only 1 octet for network)
Class B: 16,384 networks  (moderate)
Class C: ~2,097,152 networks  ← MOST of any unicast class

Why? 3 octets dedicated to network ID
→ Many small organizations can each get their own /24 block
→ ISPs subdivide Class C ranges to assign to customers
```

---

## 6. Number of Hosts per Network

### Calculation

```
Host ID uses last 1 octet = 8 bits
All 8 bits are variable (no prefix in host portion)

Total host addresses per network = 2⁸ = 256

Reserved (cannot assign to any user):
  First address (host bits all 0):  Network Address     (194.2.3.0)
  Last address  (host bits all 1):  Directed Broadcast  (194.2.3.255)

Usable hosts per network = 2⁸ - 2 = 254
```

### Example: ABC Organization 194.2.3.0

```
Network address:     194.2.3.0    ← identifies the network (not assignable)
First usable host:   194.2.3.1    ← first host you can assign
...all usable...
Last usable host:    194.2.3.254  ← last host you can assign
Directed Broadcast:  194.2.3.255  ← broadcasts to ALL hosts (not assignable)

Total addresses in block:     256  (194.2.3.0 to 194.2.3.255)
Usable host addresses:        254  (194.2.3.1 to 194.2.3.254)
```

### Why Minus 2? (Two Addresses Always Reserved)

```
Network Address (194.2.3.0) — host bits all 0:
  This IS the name/identifier of the network.
  Cannot assign to a user — it represents the whole block.
  Used in routing tables: "send 194.2.3.x traffic here."

Directed Broadcast (194.2.3.255) — host bits all 1:
  Used to send one message to ALL hosts in the network at once.
  "If anyone outside ABC org sends a packet to 194.2.3.255,
   every machine inside ABC org receives it."
  Cannot assign to a user — reserved for broadcast function.

Result: 256 - 2 = 254 assignable host addresses per Class C network.
```

### Class C Suitable For

```
254 hosts per network → small to medium organizations:
  Small offices, branches, retail stores
  Home ISP connections (ISP assigns you a /24)
  Small departments within a university
  Startups, SMEs, NGOs

Compare:
  Class A → ~16 million hosts/network → Google, IANA, military
  Class B → ~65,534 hosts/network    → Universities, IRCTC
  Class C → 254 hosts/network        → Small offices, homes
```

---

## 7. Default Subnet Mask — Class C

```
Class C Default Mask: 255.255.255.0  (/24)

Binary:
255       .  255      .  255      .  0
11111111  .  11111111 .  11111111 .  00000000

1s = Network portion (first 24 bits)
0s = Host portion    (last 8 bits)

This matches the Class C network/host split exactly:
  3 octets of 1s → protect the 3 network octets
  1 octet of 0s  → zero out the 1 host octet
```

### AND Logic Recap

```
255 (11111111) AND any octet = that octet unchanged  → PRESERVE
0   (00000000) AND any octet = 0                     → ZERO OUT

So: IP AND 255.255.255.0
  Octets 1, 2, 3: preserved (network ID kept as-is)
  Octet 4:        zeroed    (host portion removed)
  Result = Network Address
```

---

## 8. Network ID Extraction — Worked Example

### Given: IP = 194.2.3.4, determine everything

```
Step 1: Confirm it's Class C
  First octet = 194
  Is 194 in range 192–223? YES → Class C ✓

Step 2: Apply default mask 255.255.255.0

  194   AND  255  = 194  (255 = all 1s → preserves 194)
  2     AND  255  = 2    (255 = all 1s → preserves 2)
  3     AND  255  = 3    (255 = all 1s → preserves 3)
  4     AND  0    = 0    (0   = all 0s → zeroes out)

Step 3: Network ID = 194.2.3.0

Step 4: Derive all key addresses
  Network:         194.2.3.0    (host bits all 0)
  First host:      194.2.3.1    (first assignable)
  Last host:       194.2.3.254  (last assignable)
  Broadcast:       194.2.3.255  (host bits all 1)
  Usable hosts:    254
```

### Verify AND on Octet 4 in Binary

```
Octet 4 of IP:    4 → 0 0 0 0 0 1 0 0
Octet 4 of mask:  0 → 0 0 0 0 0 0 0 0
                      ─────────────────
AND result:           0 0 0 0 0 0 0 0 = 0 ✓

So 4 AND 0 = 0 → host portion is zeroed out → network address ends in .0
```

### Finding Broadcast in Binary

```
Network address:  194.2.3.0
                  = 11000010.00000010.00000011.00000000

Set all HOST bits (last 8) to 1:
                  = 11000010.00000010.00000011.11111111
                  = 194.2.3.255 ← Directed Broadcast ✓
```

### The "255 AND = preserve, 0 AND = zero" Shortcut

```
When masking with 255:  result = original value  (no binary needed)
When masking with 0:    result = 0               (no binary needed)

So 194.2.3.4 AND 255.255.255.0:
  194 AND 255 = 194  ← directly
  2   AND 255 = 2    ← directly
  3   AND 255 = 3    ← directly
  4   AND 0   = 0    ← directly
  Result: 194.2.3.0  (in under 2 seconds, no binary needed!)
```

---

## 9. All Classes Side-by-Side

| Parameter       | Class A      | Class B             | Class C               | Class D   | Class E      |
| --------------- | ------------ | ------------------- | --------------------- | --------- | ------------ |
| Prefix bits     | 0            | 10                  | 110                   | 1110      | 11110        |
| Fixed bits      | 1            | 2                   | 3                     | 4         | 5            |
| 1st octet range | 0–127        | 128–191             | 192–223               | 224–239   | 240–255      |
| Network bits    | 8            | 16                  | 24                    | —         | —            |
| Host bits       | 24           | 16                  | 8                     | —         | —            |
| Total addresses | 2³¹          | 2³⁰                 | 2²⁹                   | 2²⁸       | 2²⁷          |
| % of IPv4 space | 50%          | 25%                 | 12.5%                 | —         | —            |
| Networks        | 126          | 16,384              | **~2,097,152 (~20L)** | —         | —            |
| Hosts/network   | 2²⁴−2 ≈16.7M | 2¹⁶−2 = 65,534      | **2⁸−2 = 254**        | —         | —            |
| Default mask    | /8           | /16                 | **/24**               | —         | —            |
| Suited for      | Huge orgs    | Mid-size orgs       | **Small orgs/homes**  | Multicast | Research     |
| Example use     | ISPs, Google | Universities, IRCTC | Small offices, SMEs   | Streaming | Experimental |

---

## 10. 🔴 Security Context

### Class C Networks as the Most Common Attack Target

```
Class C = 254 usable hosts per /24 block
Most common network type on the internet today:
  → Home networks (your ISP gives you a /24 or smaller)
  → Small office networks
  → Web servers, VPS instances
  → CTF and lab environments (192.168.x.x, 10.x.x.x/24 slices)

Attacker perspective:
  Scanning a Class C /24 network:
  nmap -sn 192.168.1.0/24
  → Only 254 hosts to probe
  → Can finish in seconds with fast scanner
  → Ideal for internal network recon after initial compromise
```

### Private Class C Range — RFC 1918

```
RFC 1918 Private Address Ranges:
  Class A private: 10.0.0.0/8
  Class B private: 172.16.0.0/12
  Class C private: 192.168.0.0/16  ← most common home/lab range

192.168.0.0/16 means:
  192.168.0.0   to   192.168.255.255
  = 256 separate /24 networks
  Each /24 has 254 usable hosts

Your lab machines are almost certainly in this range:
  Metasploitable2: 192.168.56.101 (Class C private)
  Your Parrot OS:  192.168.56.1   (Class C private)
  Host-only NIC:   192.168.56.0/24 VirtualBox subnet
```

### Broadcast-Based Attacks on Class C

```
Class C Directed Broadcast: 194.2.3.255

Smurf Attack on Class C:
  Attacker spoofs victim's IP as source
  Sends ICMP echo request to 194.2.3.255
  ALL 254 hosts in the /24 network reply to victim
  Victim gets flooded with 254 ICMP replies per request

Scale comparison:
  Class C broadcast: 254 hosts reply    → small-medium flood
  Class B broadcast: 65,534 hosts reply → 258× more powerful
  Class A broadcast: 16.7M hosts reply  → catastrophic

Why Class C Smurf still matters:
  Most small networks ARE Class C (/24)
  Amplification factor still significant
  Defense: Block directed broadcasts at border router
           sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1
```

### Network Address Confusion — Common Misconfiguration

```
Class C network: 194.2.3.0/24

Common admin mistake:
  Assigns 194.2.3.0 to a server thinking it's a valid host
  Reality: 194.2.3.0 is the NETWORK ADDRESS — no real host has it!
  Packets sent to 194.2.3.0 go nowhere (or get dropped at router)

Security implication:
  Firewall rule "ALLOW traffic to 194.2.3.0" may be:
  → Silently ineffective (traffic dropped)
  → OR interpreted as ALLOW the whole /24 by some firewall implementations
  → Creates false sense of security or unintended permissive rule

  Always use specific host IPs (194.2.3.1 to .254) in firewall rules
  Never use the network address or broadcast in ACLs
```

### SSRF Targeting Private Class C Ranges

```
SSRF (Server-Side Request Forgery) — most common with Class C:

Scenario:
  Attacker finds SSRF vulnerability in your web app
  Forces server to make internal HTTP requests

Common payloads targeting Class C ranges:
  http://192.168.1.1/admin     → home router admin panel
  http://192.168.0.1/          → default gateway
  http://10.0.0.1/             → cloud internal metadata (Class A private)
  http://169.254.169.254/      → AWS metadata service (link-local)

Why Class C is the focus:
  Most web servers sit in 192.168.x.x/24 (Class C private)
  Internal microservices often on 192.168.x.x
  Docker networks default to 172.17.0.0/16 (Class B private)
  but container IPs are usually /24 slices → Class C behavior

Defense on your MERN/PERN projects:
  Validate and whitelist all URLs your server fetches
  Block requests to RFC 1918 private ranges
  Implement network-level egress filtering
```

### Class C in Your Penetration Testing Workflow

```
Phase 1 — Discover (after gaining access to a Class C network)
  nmap -sn 192.168.56.0/24          → ping sweep, find live hosts
  sudo arp-scan 192.168.56.0/24     → ARP-based discovery (more reliable)
  netdiscover -r 192.168.56.0/24    → passive ARP discovery

Phase 2 — Enumerate
  nmap -sV 192.168.56.101           → service versions on Metasploitable
  nmap -A 192.168.56.101            → aggressive scan
  nmap --script=vuln 192.168.56.101 → known vulnerabilities

Phase 3 — Exploit (authorized lab only)
  msfconsole → use exploit/unix/ftp/vsftpd_234_backdoor
  set RHOSTS 192.168.56.101
  run

Class C /24 = ideal pentest lab subnet size
  Small enough to scan completely in seconds
  Large enough to host multiple VMs (254 slots)
```

---

## 11. 🧪 Practical Labs

### Lab 1 — Class C IP Analyzer (Python)

```python
# Save as class_c_analyzer.py — run: python3 class_c_analyzer.py
import ipaddress

def analyze_classful_ip(ip_str: str) -> dict:
    """Classify IP and compute all network details"""
    parts = ip_str.split('.')
    first  = int(parts[0])
    ip_obj = ipaddress.IPv4Address(ip_str)

    # Determine class and mask
    if first < 128:
        cls, mask_str, net_bits, host_bits = 'A', '255.0.0.0', 8, 24
    elif first < 192:
        cls, mask_str, net_bits, host_bits = 'B', '255.255.0.0', 16, 16
    elif first < 224:
        cls, mask_str, net_bits, host_bits = 'C', '255.255.255.0', 24, 8
    elif first < 240:
        return {'ip': ip_str, 'class': 'D', 'purpose': 'Multicast'}
    else:
        return {'ip': ip_str, 'class': 'E', 'purpose': 'Reserved/Experimental'}

    # AND operation to get network address
    mask_obj    = ipaddress.IPv4Address(mask_str)
    network_int = int(ip_obj) & int(mask_obj)
    network_obj = ipaddress.IPv4Address(network_int)

    # Broadcast = set all host bits to 1
    host_mask      = (1 << host_bits) - 1
    broadcast_int  = network_int | host_mask
    broadcast_obj  = ipaddress.IPv4Address(broadcast_int)

    first_host = ipaddress.IPv4Address(network_int + 1)
    last_host  = ipaddress.IPv4Address(broadcast_int - 1)

    return {
        'ip':               ip_str,
        'class':            cls,
        'first_octet':      first,
        'default_mask':     mask_str,
        'prefix_length':    net_bits,
        'network_address':  str(network_obj),
        'first_host':       str(first_host),
        'last_host':        str(last_host),
        'broadcast':        str(broadcast_obj),
        'total_hosts':      2 ** host_bits,
        'usable_hosts':     2 ** host_bits - 2,
        'is_private':       ip_obj.is_private,
    }

# Test IPs — mix of all classes + lecture example
test_ips = [
    "194.2.3.4",       # Class C PUBLIC (lecture's ABC org example)
    "192.168.56.101",  # Class C PRIVATE (your Metasploitable2)
    "192.168.1.1",     # Class C PRIVATE (typical home router)
    "130.2.3.4",       # Class B PUBLIC  (Oxford, lecture 41)
    "10.0.0.5",        # Class A PRIVATE
    "224.0.0.1",       # Class D — Multicast
]

for ip in test_ips:
    r = analyze_classful_ip(ip)
    print(f"\n{'='*52}")
    print(f"  IP Address:      {r.get('ip')}")
    print(f"  Class:           {r.get('class')}")
    if r.get('class') in ('D', 'E'):
        print(f"  Purpose:         {r.get('purpose')}")
        continue
    print(f"  Default Mask:    {r.get('default_mask')}  (/{r.get('prefix_length')})")
    print(f"  Network:         {r.get('network_address')}")
    print(f"  First Host:      {r.get('first_host')}")
    print(f"  Last Host:       {r.get('last_host')}")
    print(f"  Broadcast:       {r.get('broadcast')}")
    print(f"  Usable Hosts:    {r.get('usable_hosts'):,}")
    print(f"  Private (RFC1918): {r.get('is_private')}")
```

### Lab 2 — Scan Your Class C Lab Subnet

```bash
# Your VirtualBox host-only network is a Class C /24
# This is safe — it's your own isolated lab

# Step 1: Identify your subnet
ip addr show eth0         # or ip addr show enp0s3
# Look for: inet 192.168.56.x/24

# Step 2: Full ping sweep of your /24
nmap -sn 192.168.56.0/24

# Expected output (your lab):
# Host: 192.168.56.1   (VirtualBox host-only adapter / your Parrot)
# Host: 192.168.56.101 (Metasploitable2)

# Step 3: Understand what 254 hosts means at scale
python3 - << 'EOF'
import time

usable = 2**8 - 2   # 254 hosts in /24

# nmap default speed ~100 hosts/sec (ping scan)
nmap_rate = 100
print(f"Class C /24 — {usable} usable hosts")
print(f"nmap ping sweep time: {usable / nmap_rate:.1f} seconds")
print()

# masscan is much faster
masscan_rate = 1_000_000  # 1M packets/sec
print(f"masscan full /24:     {usable / masscan_rate * 1000:.3f} milliseconds!")
print()
print("This is why /24 (Class C) is the standard unit of")
print("pentest scope — small enough to scan completely, fast.")
EOF

# Step 4: Full service scan on Metasploitable
nmap -sV -T4 192.168.56.101
```

### Lab 3 — AND Operation Visualizer

```bash
# Visualize the AND masking operation from the lecture
python3 - << 'EOF'
def show_and_operation(ip: str, mask: str):
    ip_parts   = list(map(int, ip.split('.')))
    mask_parts = list(map(int, mask.split('.')))
    result     = [a & b for a, b in zip(ip_parts, mask_parts)]

    print(f"\nIP:     {ip:<18} = ", end="")
    print("  ".join(f"{o:08b}" for o in ip_parts))

    print(f"Mask:   {mask:<18} = ", end="")
    print("  ".join(f"{o:08b}" for o in mask_parts))

    print(f"        {'AND':<18}   ", end="")
    print("  ".join("─" * 8 for _ in range(4)))

    result_str = ".".join(map(str, result))
    print(f"Result: {result_str:<18} = ", end="")
    print("  ".join(f"{o:08b}" for o in result))
    print(f"\n  → Network Address: {result_str}")
    print(f"  → Note: 255 AND x = x (preserved)")
    print(f"  →       0   AND x = 0 (zeroed out)")

# Lecture examples
print("="*70)
print("EXAMPLE 1: 194.2.3.4 AND 255.255.255.0  (Class C from lecture)")
show_and_operation("194.2.3.4", "255.255.255.0")

print("\n" + "="*70)
print("EXAMPLE 2: 192.168.56.101 AND 255.255.255.0  (your Metasploitable)")
show_and_operation("192.168.56.101", "255.255.255.0")

print("\n" + "="*70)
print("EXAMPLE 3: 130.2.3.4 AND 255.255.0.0  (Class B from lecture 41)")
show_and_operation("130.2.3.4", "255.255.0.0")
EOF
```

### Lab 4 — Directed Broadcast Awareness on Your Lab

```bash
# Demonstrate broadcast address behavior
# (safe — only within your isolated VirtualBox /24)

# Find your broadcast address
ip addr show eth0 | grep "brd"
# Output: inet 192.168.56.x/24 brd 192.168.56.255

# Your Class C broadcast is: 192.168.56.255

# Ping the broadcast (sends ICMP to ALL hosts on your /24)
ping -b -c 2 192.168.56.255
# You should see replies from 192.168.56.1 AND 192.168.56.101

# Open Wireshark simultaneously to observe:
# Filter: icmp
# You'll see: one outgoing request, MULTIPLE incoming replies
# This is EXACTLY the behavior a Smurf attack exploits

# Protect your machine from responding to broadcast pings:
sudo sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1
echo "Your machine will no longer reply to directed broadcasts"

# Verify the protection:
ping -b -c 2 192.168.56.255
# Your Parrot OS should no longer appear in the replies
```

### Lab 5 — Identify All Classes Including Private Ranges

```bash
python3 - << 'EOF'
import ipaddress

# RFC 1918 private ranges — know these for every exam and pentest
private_ranges = {
    "Class A private": ipaddress.IPv4Network("10.0.0.0/8"),
    "Class B private": ipaddress.IPv4Network("172.16.0.0/12"),
    "Class C private": ipaddress.IPv4Network("192.168.0.0/16"),
}

test_ips = [
    "194.2.3.4",       # Class C PUBLIC
    "192.168.56.101",  # Class C PRIVATE
    "192.168.255.254", # Class C PRIVATE (last in range)
    "193.0.0.1",       # Class C PUBLIC (just above private)
    "172.16.5.10",     # Class B PRIVATE
    "10.10.10.10",     # Class A PRIVATE
    "8.8.8.8",         # Class A PUBLIC (Google DNS)
]

print(f"{'IP Address':<22} {'Class':<8} {'Public/Private':<22} {'Range type'}")
print("-" * 75)

for ip_str in test_ips:
    first = int(ip_str.split('.')[0])
    ip    = ipaddress.IPv4Address(ip_str)

    if first < 128:     cls = 'A'
    elif first < 192:   cls = 'B'
    elif first < 224:   cls = 'C'
    elif first < 240:   cls = 'D'
    else:               cls = 'E'

    status    = "PRIVATE (RFC 1918)" if ip.is_private else "PUBLIC"
    range_lbl = ""
    for name, net in private_ranges.items():
        if ip in net:
            range_lbl = name
            break

    print(f"{ip_str:<22} {cls:<8} {status:<22} {range_lbl}")
EOF
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           CLASS C IP ADDRESSING — EXAM CHEAT SHEET                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  IDENTIFICATION                                                       ║
║  Prefix:     First 3 bits = 1 1 0 (fixed)                          ║
║  Range:      192 – 223 (first octet)                               ║
║  Check:      Is first octet in 192–223? → Class C                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  STRUCTURE                                                           ║
║  [110 + 5 bits][8 bits][8 bits][8 bits host]                       ║
║  Network ID:  first 3 octets (24 bits)                             ║
║  Host ID:     last 1 octet   (8 bits)                              ║
║  Default mask: 255.255.255.0  (/24)                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY NUMBERS                                                         ║
║  Total Class C addresses:  2²⁹ ≈ 537 million (12.5% of IPv4)      ║
║  Number of networks:       2²¹ = 2,097,152 (~20 lakh)             ║
║  Host addresses/network:   2⁸  = 256                               ║
║  Usable hosts/network:     2⁸ - 2 = 254                           ║
╠══════════════════════════════════════════════════════════════════════╣
║  HOW 2²¹ NETWORKS?                                                  ║
║  First octet:  32 values  (192–223) = 2⁵                          ║
║  Second octet: 256 values (0–255)   = 2⁸                          ║
║  Third octet:  256 values (0–255)   = 2⁸                          ║
║  32 × 256 × 256 = 2⁵ × 2⁸ × 2⁸ = 2²¹ ✓                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  RESERVED ADDRESSES (same rule as Class A & B)                      ║
║  First address (host=0):   Network Address   → cannot assign       ║
║  Last address (host=1s):   Broadcast Address → cannot assign       ║
║  So: -2 from total hosts → 254 usable                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  WORKED EXAMPLE: 194.2.3.4                                          ║
║  Class:       C (194 is in 192–223)                                ║
║  Mask:        255.255.255.0                                         ║
║  Network:     194.2.3.0   (AND with mask)                          ║
║  First host:  194.2.3.1                                            ║
║  Last host:   194.2.3.254                                          ║
║  Broadcast:   194.2.3.255                                          ║
╠══════════════════════════════════════════════════════════════════════╣
║  AND SHORTCUT                                                        ║
║  IP AND 255 = IP unchanged  (1s preserve all bits)                 ║
║  IP AND 0   = 0             (0s zero out all bits)                 ║
║  Class C: first 3 octets AND 255 = kept, last 1 AND 0 = zeroed    ║
╠══════════════════════════════════════════════════════════════════════╣
║  PRIVATE CLASS C RANGE (RFC 1918)                                   ║
║  192.168.0.0 – 192.168.255.255  (/16 block of /24 subnets)        ║
║  Most common home and lab network range                             ║
║  Your lab: 192.168.56.0/24 (VirtualBox host-only)                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  Class C = 254 hosts → easiest to scan completely (seconds)        ║
║  Broadcast 194.2.3.255 → Smurf attack (254 hosts reply)           ║
║  192.168.x.x = SSRF target when exploiting internal apps           ║
║  Network addr (194.2.3.0) must NEVER appear in firewall host rules ║
╠══════════════════════════════════════════════════════════════════════╣
║  ALL CLASSES QUICK COMPARE                                           ║
║  A: 0–127,   /8,  126 nets,      ~16.7M hosts/net                 ║
║  B: 128–191, /16, 16,384 nets,   65,534 hosts/net                 ║
║  C: 192–223, /24, ~2,097,152 nets, 254 hosts/net  ← THIS LECTURE  ║
║  D: 224–239  Multicast                                              ║
║  E: 240–255  Reserved/Experimental                                  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Class D (Multicast) — 224–239, used for group communication
- [ ] Class E (Reserved) — 240–255, experimental use only
- [ ] Subnetting — dividing a /24 into smaller /25, /26, /27 blocks
- [ ] CIDR — Classless Inter-Domain Routing (beyond classful)
- [ ] NAT — how private Class C addresses reach the public internet
- [ ] IPv6 — why we ran out of IPv4 and what replaced classful addressing

---

_Notes compiled from: Networking Course Lecture 42 — Class C IP Addressing_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
