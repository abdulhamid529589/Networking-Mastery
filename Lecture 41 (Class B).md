# 🌐 IP Addressing — Class B

### Cybersecurity Student Notes | Networking Course — Lecture 12

> **Source:** Gate Smashers — Class B in IP Addressing
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Class B — Prefix & Identification](#1-class-b--prefix--identification)
2. [Class B Range — First Octet](#2-class-b-range--first-octet)
3. [Binary to Decimal Conversion](#3-binary-to-decimal-conversion)
4. [Number of IP Addresses in Class B](#4-number-of-ip-addresses-in-class-b)
5. [Network ID vs Host ID Split](#5-network-id-vs-host-id-split)
6. [Number of Networks](#6-number-of-networks)
7. [Number of Hosts per Network](#7-number-of-hosts-per-network)
8. [Default Subnet Mask — Class B](#8-default-subnet-mask--class-b)
9. [Network ID Extraction — Worked Example](#9-network-id-extraction--worked-example)
10. [All Classes Side-by-Side](#10-all-classes-side-by-side)
11. [🔴 Security Context](#11--security-context)
12. [🧪 Practical Labs](#12--practical-labs)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. Class B — Prefix & Identification

### The Class B Identifier Bits

```
32-bit IPv4 Address (4 octets):
┌──────────────────┬─────────┬──────────────────────────────────┐
│   Octet 1 (8b)   │  Oct 2  │        Octets 3 & 4             │
├──┬───────────────┴─────────┴──────────────────────────────────┤
│1 0│  6 variable  │  8 bits │           16 bits               │
│   │  network bits│         │                                  │
└──┴───────────────┴─────────┴──────────────────────────────────┘
 ↑↑
 Fixed prefix = "10"
 These 2 positions CANNOT be changed
```

### How to Instantly Identify Class B

```
Rule: If first 2 bits of IP address = 1 0 → Class B

In practice: Just check first octet value
  128 to 191 → Class B

Why 128?  10 000000 = 128 (minimum with prefix 10)
Why 191?  10 111111 = 191 (maximum with prefix 10)
```

### Class B vs Class A Comparison

|                          | Class A   | Class B     |
| ------------------------ | --------- | ----------- |
| Fixed prefix bits        | 1 bit (0) | 2 bits (10) |
| First octet range        | 0–127     | 128–191     |
| Variable bits in octet 1 | 7         | 6           |

---

## 2. Class B Range — First Octet

### Deriving the Range

```
Prefix: 1 0 (fixed)
Remaining 6 bits in first octet: variable

Minimum (all 6 variable = 0):
  1 0 0 0 0 0 0 0 = 128

Maximum (all 6 variable = 1):
  1 0 1 1 1 1 1 1 = 191

Range: 128 – 191
Total values: 191 - 128 + 1 = 64 different first-octet values
```

### Step-by-Step Binary → Decimal (Lecture's Method)

```
Converting 10111111 to decimal:

Position:  7  6  5  4  3  2  1  0  ← (right to left, 0-indexed)
Bits:      1  0  1  1  1  1  1  1

Value = (1×2⁷) + (0×2⁶) + (1×2⁵) + (1×2⁴) + (1×2³) + (1×2²) + (1×2¹) + (1×2⁰)
      = 128 + 0 + 32 + 16 + 8 + 4 + 2 + 1
      = 191 ✓
```

---

## 3. Binary to Decimal Conversion

Lecture explains this — master it for all IP calculations:

### Method (Right to Left)

```
Binary number: b₇ b₆ b₅ b₄ b₃ b₂ b₁ b₀

Decimal = b₇×2⁷ + b₆×2⁶ + b₅×2⁵ + b₄×2⁴ + b₃×2³ + b₂×2² + b₁×2¹ + b₀×2⁰
```

### Powers of 2 — Must Memorize

| Power | Value                        |
| ----- | ---------------------------- |
| 2⁰    | 1                            |
| 2¹    | 2                            |
| 2²    | 4                            |
| 2³    | 8                            |
| 2⁴    | 16                           |
| 2⁵    | 32                           |
| 2⁶    | 64                           |
| 2⁷    | 128                          |
| 2⁸    | 256                          |
| 2¹⁴   | 16,384                       |
| 2¹⁶   | 65,536                       |
| 2²⁴   | 16,777,216                   |
| 2³⁰   | 1,073,741,824 (~1 billion)   |
| 2³²   | 4,294,967,296 (~4.3 billion) |

### Lecture's Example: 1011 in binary

```
1 0 1 1
↓ ↓ ↓ ↓
(1×2³) + (0×2²) + (1×2¹) + (1×2⁰)
= 8 + 0 + 2 + 1
= 11
```

---

## 4. Number of IP Addresses in Class B

```
Total bits in IPv4: 32
Fixed prefix bits:   2  (cannot change)
Variable bits:      30  (32 - 2 = 30)

Total Class B addresses = 2³⁰ = 1,073,741,824 ≈ 1 billion

Context:
  Total IPv4 space: 2³² ≈ 4.3 billion
  Class B portion: 2³⁰ = 25% of total IPv4 space
  (2³⁰ / 2³² = 1/4 = 25%)
```

---

## 5. Network ID vs Host ID Split

### Class B Split: 16 + 16

```
32-bit IP address:
[─── First 2 octets (16 bits) ───][─── Last 2 octets (16 bits) ───]
         NETWORK ID                           HOST ID
  (identifies which network)        (identifies host in network)

Compare:
  Class A: 8-bit network  + 24-bit host
  Class B: 16-bit network + 16-bit host  ← balanced split
  Class C: 24-bit network +  8-bit host
```

```
Example: 130.2.3.4

  130.2   → Network ID (first 2 octets)
  3.4     → Host ID    (last 2 octets)

  Network: 130.2.0.0
  Host: the machine with host ID = 3.4 in network 130.2
```

---

## 6. Number of Networks

### Calculation

```
Network ID uses first 2 octets = 16 bits
Fixed prefix (1 0) = 2 bits → cannot be part of network variation
Variable network bits = 16 - 2 = 14 bits

Number of networks = 2¹⁴ = 16,384
```

### Why 16,384? Visual Explanation

```
First octet values: 128 to 191 → 64 different values
Second octet values: 0 to 255 → 256 different values

Total combinations = 64 × 256 = 16,384

In powers of 2:
  64  = 2⁶
  256 = 2⁸
  64 × 256 = 2⁶ × 2⁸ = 2¹⁴ = 16,384 ✓
```

### Are All 16,384 Networks Usable?

Unlike Class A (which reserved 2 network IDs for null and loopback), Class B uses all 16,384 networks — no special reservations like Class A's 0.x.x.x and 127.x.x.x.

```
Class B usable networks ≈ 16,384
(no significant reservations at the network level — minor ones exist but not tested)
```

---

## 7. Number of Hosts per Network

### Calculation

```
Host ID uses last 2 octets = 16 bits
All 16 bits are variable (no fixed prefix in host portion)

Total host addresses per network = 2¹⁶ = 65,536

Reserved (cannot assign to hosts):
  First address (host bits all 0):  Network Address
  Last address  (host bits all 1):  Directed Broadcast Address

Usable hosts per network = 2¹⁶ - 2 = 65,534
```

### Example: Oxford University 130.2.0.0

```
Network:             130.2.0.0     ← identifies Oxford's network
First host:          130.2.0.1     ← first assignable host
...all hosts...
Last host:           130.2.255.254 ← last assignable host
Directed Broadcast:  130.2.255.255 ← sends to ALL hosts in network

Total host addresses:  65,536 (130.2.0.0 to 130.2.255.255)
Usable for hosts:      65,534 (130.2.0.1 to 130.2.255.254)
```

### Why Minus 2?

```
Network Address (130.2.0.0):
  Cannot assign to a host — this IS the network's name/identifier
  Appears in routing tables as the network route
  Used in routing protocols to represent the whole network

Directed Broadcast (130.2.255.255):
  Cannot assign to a host — reserved for broadcast use
  Sending to this address: message reaches EVERY host in 130.2.0.0
  Example: Admin sends maintenance notice to all 65,534 machines at once
```

---

## 8. Default Subnet Mask — Class B

```
Class B Default Mask: 255.255.0.0  (/16)

Binary:
255       .  255      .  0         .  0
11111111  .  11111111 .  00000000  .  00000000

1s = Network portion (first 16 bits)
0s = Host portion    (last 16 bits)

This matches the Class B network/host split exactly.
```

### AND with 255 = Preserve, AND with 0 = Zero Out

```
255 (11111111) AND any octet = that octet unchanged
0   (00000000) AND any octet = 0

So: IP AND 255.255.0.0
  Octets 1&2: preserved (network ID kept)
  Octets 3&4: zeroed    (host portion zeroed)
  Result = Network Address
```

---

## 9. Network ID Extraction — Worked Example

### Given: IP = 130.2.3.4, Class B

```
Step 1: Confirm it's Class B
  First octet = 130
  Is 130 in range 128–191? YES → Class B ✓

Step 2: Apply default mask 255.255.0.0

  130   AND  255  = 130  (255 = all 1s → preserves 130)
  2     AND  255  = 2    (255 = all 1s → preserves 2)
  3     AND  0    = 0    (0   = all 0s → zeroes out)
  4     AND  0    = 0    (0   = all 0s → zeroes out)

Step 3: Network ID = 130.2.0.0

Step 4: Host ID = 3.4 (the zeroed-out portion restated)

Answers:
  Network:   130.2.0.0
  First host: 130.2.0.1
  Last host:  130.2.255.254
  Broadcast:  130.2.255.255
  Host 130.2.3.4 is the 3×256 + 4 = 772nd host in this network
```

### The "255 AND = preserve" Shortcut

```
When masking with 255 (all 1s):
  The result equals the original value.
  No need to write out binary for these octets.

When masking with 0 (all 0s):
  The result is always 0.
  No need to write out binary for these octets either.

So 130.2.3.4 AND 255.255.0.0:
  130 AND 255 = 130  ← directly
  2   AND 255 = 2    ← directly
  3   AND 0   = 0    ← directly
  4   AND 0   = 0    ← directly
  Result: 130.2.0.0  (in 2 seconds, no binary needed!)
```

---

## 10. All Classes Side-by-Side

| Parameter       | Class A      | Class B             | Class C       | Class D   | Class E      |
| --------------- | ------------ | ------------------- | ------------- | --------- | ------------ |
| Prefix bits     | 0            | 10                  | 110           | 1110      | 11110        |
| Fixed bits      | 1            | 2                   | 3             | 4         | 5            |
| 1st octet range | 0–127        | 128–191             | 192–223       | 224–239   | 240–255      |
| Network bits    | 8            | 16                  | 24            | —         | —            |
| Host bits       | 24           | 16                  | 8             | —         | —            |
| Total addresses | 2³¹          | 2³⁰                 | 2²⁹           | 2²⁸       | 2²⁷          |
| % of IPv4 space | 50%          | 25%                 | 12.5%         | —         | —            |
| Networks        | 126          | 16,384              | ~2M           | —         | —            |
| Hosts/network   | 2²⁴−2 ≈16M   | 2¹⁶−2 = 65,534      | 2⁸−2 = 254    | —         | —            |
| Default mask    | /8           | /16                 | /24           | —         | —            |
| Suited for      | Huge orgs    | Mid-size orgs       | Small orgs    | Multicast | Research     |
| Example use     | ISPs, Google | Universities, IRCTC | Small offices | Streaming | Experimental |

---

## 11. 🔴 Security Context

### Class B Networks as Attack Targets

```
Class B = 65,534 hosts per network
Typical Class B organizations: Universities, hospitals, government agencies, mid-size ISPs

Attacker perspective:
  Scanning a Class B /16 network:
  nmap -sn 130.2.0.0/16
  → scans 65,534 hosts
  → finds exposed services across entire organization
  → much wider attack surface than Class C

  University networks (Class B) are prime targets:
  → Weak endpoint security (student devices)
  → Research data (IP theft)
  → Compute resources (cryptomining)
  → Open academic culture = less restrictive firewalls
```

### Broadcast-Based Attacks on Class B

```
Class B Directed Broadcast: 130.2.255.255

Smurf Attack scenario:
  Attacker spoofs victim's IP as source
  Sends ICMP echo request to 130.2.255.255
  ALL 65,534 hosts in Oxford's network reply to victim
  Victim receives massive flood

Scale comparison:
  Class C broadcast: 254 hosts reply → smaller flood
  Class B broadcast: 65,534 hosts reply → 258× more powerful
  Class A broadcast: 16M hosts reply → catastrophic

Defense: Block directed broadcasts at border router
  iptables -A FORWARD -d x.x.255.255 -j DROP
```

### IRCTC / University Style Attacks

```
Real Class B users: IRCTC (Indian railways ticketing), universities

Common attack vectors on these organizations:
  1. Credential stuffing (large user base = valuable accounts)
  2. Account enumeration (65,534 hosts = many users, many passwords to try)
  3. Subdomain/vhost enumeration (multiple services on large network)
  4. VLAN hopping (campus networks often have VLANs)
  5. Insider threat (students/staff have internal access)

Tools:
  nmap -sV 130.2.0.0/16          → service enumeration
  hydra -l admin -P passwords.txt → brute force
  masscan 130.2.0.0/16 -p80,443  → fast port scan
```

### Network Address = Not a Host (Firewall Rule Bypass)

```
Common misconfiguration:
  Admin sets firewall rule: "Allow traffic to 130.2.0.0"
  Thinking: "130.2.0.0 is my server IP"

  Reality: 130.2.0.0 is the NETWORK ADDRESS, not a host!
  No real host has this IP.
  Traffic sent to 130.2.0.0 → goes nowhere or causes confusion.

  Security implication:
  Firewall rule "allow 130.2.0.0" may actually allow all network traffic
  depending on how the firewall interprets unresolvable network addresses.

  Always use specific host IPs in firewall rules.
```

---

## 12. 🧪 Practical Labs

### Lab 1 — Class B IP Analysis Tool (Python)

```python
# Save as class_b_analyzer.py — run: python3 class_b_analyzer.py
import ipaddress

def analyze_ip(ip_str: str) -> dict:
    """Analyze an IP address and classify it"""
    ip = ipaddress.IPv4Address(ip_str)
    first_octet = int(ip_str.split('.')[0])

    # Determine class
    if first_octet < 128:
        cls, mask, net_bits, host_bits = 'A', '255.0.0.0', 8, 24
    elif first_octet < 192:
        cls, mask, net_bits, host_bits = 'B', '255.255.0.0', 16, 16
    elif first_octet < 224:
        cls, mask, net_bits, host_bits = 'C', '255.255.255.0', 24, 8
    elif first_octet < 240:
        return {'class': 'D', 'purpose': 'Multicast'}
    else:
        return {'class': 'E', 'purpose': 'Reserved/Experimental'}

    # Extract network using AND with mask
    mask_addr = ipaddress.IPv4Address(mask)
    network_int = int(ip) & int(mask_addr)
    network_addr = ipaddress.IPv4Address(network_int)

    # Calculate broadcast (set all host bits to 1)
    host_bits_mask = (1 << host_bits) - 1
    broadcast_int = network_int | host_bits_mask
    broadcast_addr = ipaddress.IPv4Address(broadcast_int)

    # First and last host
    first_host = ipaddress.IPv4Address(network_int + 1)
    last_host  = ipaddress.IPv4Address(broadcast_int - 1)

    return {
        'ip': ip_str,
        'class': cls,
        'first_octet': first_octet,
        'default_mask': mask,
        'network_address': str(network_addr),
        'broadcast_address': str(broadcast_addr),
        'first_host': str(first_host),
        'last_host': str(last_host),
        'total_hosts': 2**host_bits,
        'usable_hosts': 2**host_bits - 2,
        'is_private': ip.is_private,
    }

# Test IPs from lecture
test_ips = [
    "64.0.0.8",      # Class A
    "130.2.3.4",     # Class B (Oxford University example)
    "192.168.56.101", # Class C (your lab)
    "10.0.0.5",      # Class A private
    "172.16.5.10",   # Class B private
]

for ip in test_ips:
    result = analyze_ip(ip)
    print(f"\n{'='*50}")
    print(f"IP Address:      {result.get('ip')}")
    print(f"Class:           {result.get('class')}")
    print(f"Default Mask:    {result.get('default_mask', 'N/A')}")
    print(f"Network:         {result.get('network_address', 'N/A')}")
    print(f"First Host:      {result.get('first_host', 'N/A')}")
    print(f"Last Host:       {result.get('last_host', 'N/A')}")
    print(f"Broadcast:       {result.get('broadcast_address', 'N/A')}")
    print(f"Usable Hosts:    {result.get('usable_hosts', 'N/A'):,}")
    print(f"Private Range:   {result.get('is_private', 'N/A')}")
```

### Lab 2 — Scan a Class B Subnet (Safely)

```bash
# Your VirtualBox lab is Class C (/24 = 254 hosts)
# Simulate what scanning a Class B feels like

# Scan your Class C lab fully (safe — it's yours)
nmap -sn 192.168.56.0/24

# Understand what Class B scale means
python3 - << 'EOF'
import time

class_c_hosts = 2**8 - 2     # 254
class_b_hosts = 2**16 - 2    # 65,534
class_a_hosts = 2**24 - 2    # 16,777,214

# nmap scans ~100 hosts/sec on fast network
scan_rate = 100  # hosts per second

print(f"Hosts in /24 (Class C): {class_c_hosts:>12,}")
print(f"Time to scan /24:       {class_c_hosts/scan_rate:.1f} seconds")
print()
print(f"Hosts in /16 (Class B): {class_b_hosts:>12,}")
print(f"Time to scan /16:       {class_b_hosts/scan_rate:.1f} seconds = {class_b_hosts/scan_rate/60:.1f} minutes")
print()
print(f"Hosts in /8  (Class A): {class_a_hosts:>12,}")
print(f"Time to scan /8:        {class_a_hosts/scan_rate:.1f} seconds = {class_a_hosts/scan_rate/3600:.1f} hours")
print()
print("This is why attackers use masscan (>1M packets/sec) for large ranges:")
masscan_rate = 1_000_000
print(f"masscan /16 time:       {class_b_hosts/masscan_rate:.3f} seconds!")
EOF
```

### Lab 3 — Directed Broadcast Awareness

```bash
# Calculate and observe broadcast address behavior

# Your network: 192.168.56.0/24
# Broadcast: 192.168.56.255

# In a real network (ping broadcast = Smurf concept demo)
# On YOUR lab only:
ping -b -c 2 192.168.56.255
# Every host on the network should respond

# In Wireshark, filter: icmp
# You'll see multiple sources responding to your ICMP to broadcast
# This is exactly what Smurf attack exploits (with spoofed source IP)

# Check your broadcast address
ip addr show eth0 | grep "brd"
# Output: inet 192.168.56.x/24 brd 192.168.56.255
#                                   ↑ this is your broadcast address

# Defend against Smurf on your system
sudo sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1
# Now your machine won't respond to broadcast pings
```

### Lab 4 — Network ID Extraction (Multiple Methods)

```bash
# Method 1: Python (quick and clean)
python3 - << 'EOF'
import ipaddress

def extract_network(ip: str, mask: str) -> str:
    interface = ipaddress.IPv4Interface(f"{ip}/{mask}")
    return str(interface.network.network_address)

examples = [
    ("130.2.3.4",  "255.255.0.0"),    # Class B
    ("64.10.5.20", "255.0.0.0"),      # Class A
    ("192.168.56.101", "255.255.255.0"), # Class C
]
for ip, mask in examples:
    print(f"{ip} AND {mask} = {extract_network(ip, mask)}")
EOF

# Method 2: ipcalc (install if needed)
sudo apt install ipcalc -y
ipcalc 130.2.3.4/16
# Shows: Network, Broadcast, First/Last host, Mask

# Method 3: For Metasploitable2
ipcalc 192.168.56.101/24
```

### Lab 5 — Identify Private Class B Addresses

```bash
# RFC 1918 private ranges:
# Class A: 10.0.0.0/8
# Class B: 172.16.0.0/12 (172.16.x.x to 172.31.x.x)
# Class C: 192.168.0.0/16

# Check if an address is private
python3 - << 'EOF'
import ipaddress

test_ips = [
    "130.2.3.4",      # Class B PUBLIC (Oxford University)
    "172.16.5.10",    # Class B PRIVATE (RFC 1918)
    "172.31.255.254", # Class B PRIVATE (last in private range)
    "172.32.0.1",     # Class B PUBLIC (just outside private range)
]

for ip_str in test_ips:
    ip = ipaddress.IPv4Address(ip_str)
    status = "PRIVATE (RFC 1918)" if ip.is_private else "PUBLIC"
    first = int(ip_str.split('.')[0])
    second = int(ip_str.split('.')[1])

    # Class B private: 172.16.0.0 - 172.31.255.255
    is_b_private = (first == 172 and 16 <= second <= 31)

    print(f"{ip_str:<18} Class B: {'✓' if 128<=first<=191 else '✗'} | "
          f"Private: {'✓' if ip.is_private else '✗'} | {status}")
EOF
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           CLASS B IP ADDRESSING — EXAM CHEAT SHEET                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  IDENTIFICATION                                                       ║
║  Prefix:     First 2 bits = 1 0 (fixed)                            ║
║  Range:      128 – 191 (first octet)                               ║
║  Check:      Is first octet in 128–191? → Class B                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  STRUCTURE                                                           ║
║  [10 + 6 bits network][8 bits network][8 bits host][8 bits host]   ║
║  Network ID: first 2 octets (16 bits)                              ║
║  Host ID:    last 2 octets  (16 bits)                              ║
║  Default mask: 255.255.0.0  (/16)                                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY NUMBERS                                                         ║
║  Total Class B addresses:  2³⁰ ≈ 1 billion (25% of IPv4)          ║
║  Number of networks:       2¹⁴ = 16,384                            ║
║  Host addresses/network:   2¹⁶ = 65,536                            ║
║  Usable hosts/network:     2¹⁶ - 2 = 65,534                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  HOW 16,384 NETWORKS?                                               ║
║  First octet: 64 values (128-191)                                  ║
║  Second octet: 256 values (0-255)                                  ║
║  64 × 256 = 16,384 = 2⁶ × 2⁸ = 2¹⁴ ✓                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  RESERVED ADDRESSES (same rule as Class A)                          ║
║  First address (host=0):   Network Address   → cannot assign       ║
║  Last address (host=1s):   Broadcast Address → cannot assign       ║
║  So: -2 from total hosts                                           ║
╠══════════════════════════════════════════════════════════════════════╣
║  WORKED EXAMPLE: 130.2.3.4                                          ║
║  Class:       B (130 in 128-191)                                   ║
║  Mask:        255.255.0.0                                           ║
║  Network:     130.2.0.0   (AND with mask)                          ║
║  First host:  130.2.0.1                                            ║
║  Last host:   130.2.255.254                                        ║
║  Broadcast:   130.2.255.255                                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  AND SHORTCUT                                                        ║
║  IP AND 255 = IP unchanged  (1s preserve all bits)                 ║
║  IP AND 0   = 0             (0s zero out all bits)                 ║
║  Class B: first 2 octets AND 255 = kept, last 2 AND 0 = zeroed    ║
╠══════════════════════════════════════════════════════════════════════╣
║  PRIVATE CLASS B RANGE (RFC 1918)                                   ║
║  172.16.0.0 – 172.31.255.255  (/12)                               ║
║  Not routable on internet → used in corporate LANs                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  Class B = 65,534 hosts → wide attack surface                      ║
║  Broadcast 130.2.255.255 → Smurf attack (65K hosts reply)         ║
║  Universities, hospitals use Class B → high-value targets          ║
║  Private 172.16-31.x.x → SSRF target if behind NAT               ║
╠══════════════════════════════════════════════════════════════════════╣
║  ALL CLASSES QUICK COMPARE                                           ║
║  A: 0-127,   /8,  126 nets,   ~16M hosts/net                      ║
║  B: 128-191, /16, 16384 nets, 65534 hosts/net  ← THIS LECTURE     ║
║  C: 192-223, /24, ~2M nets,   254 hosts/net                       ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Class C IP Addressing — 192-223, /24, 254 hosts
- [ ] Class D (Multicast) and Class E (Reserved)
- [ ] Subnetting — dividing networks into smaller pieces
- [ ] CIDR — Classless Inter-Domain Routing
- [ ] NAT — why private IPs need translation

---

_Notes compiled from: Networking Course Lecture 12 — Class B IP Addressing_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
