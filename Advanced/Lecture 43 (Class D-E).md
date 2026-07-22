# 🌐 IP Addressing — Class D & Class E

### " "Networking Course — Lecture 43

> **Source:** Gate Smashers — Class D & Class E in IP Addressing
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Quick Recap — Where We Are in Classful Addressing](#1-quick-recap--where-we-are-in-classful-addressing)
2. [Class D — Prefix & Identification](#2-class-d--prefix--identification)
3. [Class D Range — First Octet](#3-class-d-range--first-octet)
4. [Number of IP Addresses in Class D](#4-number-of-ip-addresses-in-class-d)
5. [No Network / No Host in Class D](#5-no-network--no-host-in-class-d)
6. [Purpose of Class D — Multicasting](#6-purpose-of-class-d--multicasting)
7. [Class E — Prefix & Identification](#7-class-e--prefix--identification)
8. [Class E Range — First Octet](#8-class-e-range--first-octet)
9. [Number of IP Addresses in Class E](#9-number-of-ip-addresses-in-class-e)
10. [No Network / No Host in Class E](#10-no-network--no-host-in-class-e)
11. [Purpose of Class E — Military / Experimental](#11-purpose-of-class-e--military--experimental)
12. [Class D vs Class E — Side-by-Side](#12-class-d-vs-class-e--side-by-side)
13. [All Classes Side-by-Side](#13-all-classes-side-by-side)
14. [Why Classful Addressing Was Abandoned](#14-why-classful-addressing-was-abandoned)
15. [🔴 Security Context](#15--security-context)
16. [🧪 Practical Labs](#16--practical-labs)
17. [Solved Examples](#17-solved-examples)
18. [Exam Cheat Sheet](#18-exam-cheat-sheet)

---

## 1. Quick Recap — Where We Are in Classful Addressing

In Classful Addressing, the entire 32-bit IPv4 address space is divided into **5 fixed classes**:

```
32-bit IPv4 Address — 4 octets of 8 bits each:
┌──────────┬──────────┬──────────┬──────────┐
│  Octet 1 │  Octet 2 │  Octet 3 │  Octet 4 │
│  8 bits  │  8 bits  │  8 bits  │  8 bits  │
└──────────┴──────────┴──────────┴──────────┘
            Total = 32 bits
```

The **class is always identified from the first octet alone** — specifically from the leading (fixed) bits.

| Class | Prefix Bits | First Octet Range | Covered?            |
| ----- | ----------- | ----------------- | ------------------- |
| A     | `0`         | 0 – 127           | ✅ Lecture 40       |
| B     | `10`        | 128 – 191         | ✅ Lecture 41       |
| C     | `110`       | 192 – 223         | ✅ Lecture 42       |
| **D** | **`1110`**  | **224 – 239**     | **✅ This lecture** |
| **E** | **`1111`**  | **240 – 255**     | **✅ This lecture** |

> **Key difference from A, B, C:** Class D and Class E have **no concept of Network ID or Host ID**. All addresses in both classes are fully reserved — they cannot be assigned to any organisation or user.

---

## 2. Class D — Prefix & Identification

### The Class D Identifier Bits

```
32-bit IPv4 Address (4 octets):
┌─────────────────────────┬──────────┬──────────┬──────────┐
│       Octet 1 (8b)      │  Oct 2   │  Oct 3   │  Oct 4   │
├──────┬──────────────────┼──────────┼──────────┼──────────┤
│1 1 1 0│  4 variable     │  8 bits  │  8 bits  │  8 bits  │
│      │  bits            │          │          │          │
└──────┴──────────────────┴──────────┴──────────┴──────────┘
  ↑↑↑↑
  Fixed prefix = "1110"
  These 4 positions CANNOT be changed
```

### How to Instantly Identify Class D

```
Rule: If first 4 bits of IP address = 1 1 1 0 → Class D

In practice: Just check first octet value
  224 to 239 → Class D

Why 224?  1110 0000 = 224  (minimum with prefix 1110)
Why 239?  1110 1111 = 239  (maximum with prefix 1110)
```

### Class A / B / C / D Prefix Comparison

| Class | Fixed Prefix | Fixed Bits | First Octet Range | Variable Bits in Oct 1 |
| ----- | ------------ | ---------- | ----------------- | ---------------------- |
| A     | `0`          | 1          | 0 – 127           | 7                      |
| B     | `10`         | 2          | 128 – 191         | 6                      |
| C     | `110`        | 3          | 192 – 223         | 5                      |
| **D** | **`1110`**   | **4**      | **224 – 239**     | **4**                  |

---

## 3. Class D Range — First Octet

### Deriving the Range

```
Prefix: 1 1 1 0 (fixed — cannot change)
Remaining 4 bits in first octet: variable

Minimum (all 4 variable = 0):
  1 1 1 0 0 0 0 0 = 224

Maximum (all 4 variable = 1):
  1 1 1 0 1 1 1 1 = 239

Range: 224 – 239
Total first-octet values: 239 - 224 + 1 = 16 different values
```

### Step-by-Step Binary → Decimal Verification

```
Converting 11100000 (minimum) to decimal:

Position:  7   6   5   4   3   2   1   0
Bits:      1   1   1   0   0   0   0   0

Value = (1×2⁷)+(1×2⁶)+(1×2⁵)+(0×2⁴)+(0×2³)+(0×2²)+(0×2¹)+(0×2⁰)
      = 128 + 64 + 32 + 0 + 0 + 0 + 0 + 0
      = 224 ✓

Converting 11101111 (maximum) to decimal:

Position:  7   6   5   4   3   2   1   0
Bits:      1   1   1   0   1   1   1   1

Value = 128 + 64 + 32 + 0 + 8 + 4 + 2 + 1
      = 239 ✓
```

> **Class D Range: 224.0.0.0 — 239.255.255.255**
> If the first octet of an IP address is between **224 and 239**, it belongs to Class D.

---

## 4. Number of IP Addresses in Class D

```
Total bits in IPv4:    32
Fixed prefix bits:      4  (cannot change — always "1110")
Variable bits:         28  (32 - 4 = 28)

Total Class D addresses = 2²⁸ = 268,435,456 ≈ 250 million

Context:
  Total IPv4 space:   2³² ≈ 4.3 billion
  Class D portion:    2²⁸ ≈ 6.25% of total IPv4 space
  (2²⁸ / 2³² = 1/16 = 6.25%)

Compared to other classes:
  Class A: 2³¹ = 50%    of IPv4 space
  Class B: 2³⁰ = 25%    of IPv4 space
  Class C: 2²⁹ = 12.5%  of IPv4 space
  Class D: 2²⁸ = 6.25%  of IPv4 space  ← this lecture
```

---

## 5. No Network / No Host in Class D

> **There is NO concept of Network ID or Host ID in Class D.**

Unlike Classes A, B, and C — which divide addresses into a **Network ID** portion and a **Host ID** portion — Class D has **no such division whatsoever**.

```
Class A/B/C structure:
  [Network ID bits] + [Host ID bits]
  → Used to identify a specific network and a host within it

Class D structure:
  [1110][28 variable bits]
  → No split. No network. No host.
  → The entire block of 2²⁸ addresses is one reserved pool.
```

This means:

- No organisation can request a Class D address
- You **cannot** calculate number of networks or hosts for Class D
- There is no default subnet mask for Class D
- Class D addresses are never routed to individual users

---

## 6. Purpose of Class D — Multicasting

Class D addresses are reserved exclusively for **multicasting** (also called group broadcasting or group communication).

### Three Types of Network Communication

| Type          | Sender   | Receivers            | Example                      |
| ------------- | -------- | -------------------- | ---------------------------- |
| **Unicast**   | 1 device | 1 specific device    | Direct message, HTTP request |
| **Broadcast** | 1 device | ALL devices on LAN   | ARP request, DHCP discover   |
| **Multicast** | 1 device | A **specific group** | Online lecture, IPTV stream  |

### How Class D Multicasting Works

```
Step 1: A multicast group is created and assigned a Class D address
        e.g., 230.0.0.1 is assigned to "Research Group A"

Step 2: Interested devices JOIN the group
        → Only members receive traffic sent to 230.0.0.1

Step 3: Sender transmits ONE packet to 230.0.0.1
        → Network delivers it to ALL group members
        → Non-members never see it

Result: Efficient one-to-many delivery
        No need to send separate unicast copies to each member
```

### Real-World Uses of Class D Multicast

```
IPTV / Video streaming:     224.0.0.0/4 range used by video on demand
Online research groups:     Collaborative tools that need group delivery
Routing protocols:          OSPF uses 224.0.0.5 and 224.0.0.6
                            RIPv2 uses 224.0.0.9
IGMP (group management):    224.0.0.1 = all multicast hosts
                            224.0.0.2 = all multicast routers
Stock ticker feeds:         Financial data broadcast to subscriber groups
```

### The Major Problem — Address Waste

```
2²⁸ ≈ 250 million IP addresses reserved for multicast groups

Reality:
  Very few multicast groups actually exist in the world
  Groups like research teams are small in number
  Nowhere near 250 million groups have been created

Result:
  Hundreds of millions of IP addresses permanently reserved
  Unavailable for any user or organisation
  This massive waste contributed to IPv4 exhaustion
```

---

## 7. Class E — Prefix & Identification

### The Class E Identifier Bits

```
32-bit IPv4 Address (4 octets):
┌─────────────────────────┬──────────┬──────────┬──────────┐
│       Octet 1 (8b)      │  Oct 2   │  Oct 3   │  Oct 4   │
├──────┬──────────────────┼──────────┼──────────┼──────────┤
│1 1 1 1│  4 variable     │  8 bits  │  8 bits  │  8 bits  │
│      │  bits            │          │          │          │
└──────┴──────────────────┴──────────┴──────────┴──────────┘
  ↑↑↑↑
  Fixed prefix = "1111"
  These 4 positions CANNOT be changed
```

### How to Instantly Identify Class E

```
Rule: If first 4 bits of IP address = 1 1 1 1 → Class E

In practice: Just check first octet value
  240 to 255 → Class E

Why 240?  1111 0000 = 240  (minimum with prefix 1111)
Why 255?  1111 1111 = 255  (maximum with prefix 1111)
```

### Class D vs Class E Prefix — The One Bit Difference

```
Class D prefix: 1 1 1 0  →  bit 4 is 0
Class E prefix: 1 1 1 1  →  bit 4 is 1

That single bit difference shifts the range:
  Class D: 224–239  (first octet bit 4 = 0)
  Class E: 240–255  (first octet bit 4 = 1)
```

---

## 8. Class E Range — First Octet

### Deriving the Range

```
Prefix: 1 1 1 1 (fixed — cannot change)
Remaining 4 bits in first octet: variable

Minimum (all 4 variable = 0):
  1 1 1 1 0 0 0 0 = 240

Maximum (all 4 variable = 1):
  1 1 1 1 1 1 1 1 = 255

Range: 240 – 255
Total first-octet values: 255 - 240 + 1 = 16 different values
```

### Step-by-Step Binary → Decimal Verification

```
Converting 11110000 (minimum) to decimal:

Position:  7   6   5   4   3   2   1   0
Bits:      1   1   1   1   0   0   0   0

Value = (1×2⁷)+(1×2⁶)+(1×2⁵)+(1×2⁴)+(0×2³)+(0×2²)+(0×2¹)+(0×2⁰)
      = 128 + 64 + 32 + 16 + 0 + 0 + 0 + 0
      = 240 ✓

Converting 11111111 (maximum) to decimal:

Position:  7   6   5   4   3   2   1   0
Bits:      1   1   1   1   1   1   1   1

Value = 128 + 64 + 32 + 16 + 8 + 4 + 2 + 1
      = 255 ✓
```

> **Class E Range: 240.0.0.0 — 255.255.255.255**
> If the first octet of an IP address is between **240 and 255**, it belongs to Class E.

---

## 9. Number of IP Addresses in Class E

```
Total bits in IPv4:    32
Fixed prefix bits:      4  (cannot change — always "1111")
Variable bits:         28  (32 - 4 = 28)

Total Class E addresses = 2²⁸ = 268,435,456 ≈ 250 million

Context:
  Total IPv4 space:   2³² ≈ 4.3 billion
  Class E portion:    2²⁸ ≈ 6.25% of total IPv4 space

Why same as Class D?
  Both use exactly 4 fixed prefix bits
  Both have exactly 28 variable bits remaining
  2²⁸ is the same count for both → same number of addresses
```

---

## 10. No Network / No Host in Class E

> **There is NO concept of Network ID or Host ID in Class E.**

Identical situation to Class D — the entire block of `2²⁸` addresses is one reserved pool with no internal division.

```
Class E structure:
  [1111][28 variable bits]
  → No split. No network. No host.
  → All 2²⁸ ≈ 250 million addresses are reserved as a block.
```

- No organisation can request a Class E address
- You **cannot** calculate number of networks or hosts for Class E
- No default subnet mask exists for Class E
- Class E addresses are never routed on the public internet

---

## 11. Purpose of Class E — Military / Experimental

Class E addresses are reserved for **military and experimental/research purposes**.

```
Status: Pre-allocated — never released to the public
        Not available for any commercial or personal use

Officially defined in:
  RFC 1112 (1989) — "Host Extensions for IP Multicasting"
  Later reinforced in RFC 3330 / RFC 5735

Who holds them:
  IANA (Internet Assigned Numbers Authority) controls the block
  Allocated to military and experimental use since the 1980s
  Never re-allocated to general public despite IPv4 shortage

The same waste problem as Class D:
  2²⁸ ≈ 250 million addresses locked away
  Unavailable for billions of users who needed addresses
  A significant factor in accelerating IPv4 exhaustion
```

### Special Note — 255.255.255.255

```
255.255.255.255 falls inside Class E range (first octet = 255)
BUT it has a special separate meaning:

  255.255.255.255 = Limited Broadcast Address
  → Used to broadcast to ALL hosts on the LOCAL network
  → Not forwarded by routers (stays on local segment)
  → Used by DHCP Discover (client has no IP yet, needs to broadcast)

  This is different from Directed Broadcast (e.g., 194.2.3.255)
  which targets all hosts in a SPECIFIC remote network.
```

---

## 12. Class D vs Class E — Side-by-Side

| Property              | Class D                        | Class E                 |
| --------------------- | ------------------------------ | ----------------------- |
| Fixed prefix bits     | `1110`                         | `1111`                  |
| Number of fixed bits  | 4                              | 4                       |
| First octet range     | 224 – 239                      | 240 – 255               |
| Values in first octet | 16                             | 16                      |
| Variable bits         | 28                             | 28                      |
| Total IP addresses    | 2²⁸ ≈ 250 million              | 2²⁸ ≈ 250 million       |
| % of IPv4 space       | ~6.25%                         | ~6.25%                  |
| Network concept       | ❌ None                        | ❌ None                 |
| Host concept          | ❌ None                        | ❌ None                 |
| Default subnet mask   | ❌ None                        | ❌ None                 |
| Reserved for          | Multicasting / Group broadcast | Military / Experimental |
| Available to public   | ❌ No                          | ❌ No                   |
| Example IP            | 230.0.0.1                      | 245.0.1.2               |

### Combined Waste Calculation

```
Class D reserved:  2²⁸ ≈ 268 million addresses
Class E reserved:  2²⁸ ≈ 268 million addresses
                  ──────────────────────────────
Total wasted:          ≈ 536 million addresses
                  ≈ 500 million (approx. as stated in lecture)

Percentage of IPv4 space locked away:
  Class D: 6.25%
  Class E: 6.25%
  Total:   12.5% of all IPv4 addresses — permanently unavailable
```

---

## 13. All Classes Side-by-Side

| Parameter       | Class A      | Class B        | Class C          | Class D       | Class E           |
| --------------- | ------------ | -------------- | ---------------- | ------------- | ----------------- |
| Prefix bits     | `0`          | `10`           | `110`            | `1110`        | `1111`            |
| Fixed bits      | 1            | 2              | 3                | 4             | 4                 |
| 1st octet range | 0 – 127      | 128 – 191      | 192 – 223        | **224 – 239** | **240 – 255**     |
| Network bits    | 8            | 16             | 24               | —             | —                 |
| Host bits       | 24           | 16             | 8                | —             | —                 |
| Total addresses | 2³¹          | 2³⁰            | 2²⁹              | **2²⁸**       | **2²⁸**           |
| % of IPv4 space | 50%          | 25%            | 12.5%            | **6.25%**     | **6.25%**         |
| Networks        | 126          | 16,384         | ~2,097,152       | **—**         | **—**             |
| Hosts/network   | 2²⁴−2 ≈16.7M | 2¹⁶−2 = 65,534 | 2⁸−2 = 254       | **—**         | **—**             |
| Default mask    | /8           | /16            | /24              | **None**      | **None**          |
| Suited for      | Huge orgs    | Mid-size orgs  | Small orgs/homes | **Multicast** | **Military/Exp.** |
| Example IP      | 10.0.0.1     | 172.16.0.1     | 192.168.1.1      | 230.0.0.1     | 245.0.0.1         |

---

## 14. Why Classful Addressing Was Abandoned

Class D and Class E were the final blow to classful addressing's credibility:

```
Problems with Classful Addressing:

1. CLASS D WASTE
   ~250 million addresses reserved for multicast
   Very few actual multicast groups created globally
   → Hundreds of millions of addresses sit unused forever

2. CLASS E WASTE
   ~250 million addresses reserved for military/experimental
   Never released to the public despite IPv4 exhaustion
   → Another ~250 million addresses permanently locked

3. CLASS A/B/C RIGIDITY
   Class A: 16.7 million hosts — no org needs that many
   Class B: 65,534 hosts — most orgs need far fewer
   No in-between option → massive internal waste in every class

4. COMBINED EFFECT
   D + E alone = 12.5% of IPv4 permanently unavailable
   A + B internal waste = billions more addresses squandered
   Internet users growing exponentially through 1980s–90s
   IPv4 exhaustion became inevitable

SOLUTION — 1993: CIDR (Classless Inter-Domain Routing)
  RFC 1517 / 1518 / 1519 / 1520 published
  Prefix length specified explicitly: /8, /18, /24, /27 etc.
  No more fixed classes → allocate exactly what is needed
  Classful addressing used BEFORE 1993
  CIDR / Classless addressing used AFTER 1993 (to present day)
```

---

## 15. 🔴 Security Context

### Class D — Multicast as an Attack Surface

```
Multicast protocol abuse — IGMP Snooping bypass:
  IGMP (Internet Group Management Protocol) manages multicast groups
  Devices join multicast groups using IGMP membership reports
  Attacker on same LAN can:
    → Send spoofed IGMP join requests to subscribe to groups
    → Receive multicast traffic intended only for specific group members
    → Eavesdrop on IPTV streams, financial feeds, routing updates

OSPF Routing Protocol Exploitation:
  OSPF (Open Shortest Path First) uses multicast addresses:
    224.0.0.5 → All OSPF routers
    224.0.0.6 → OSPF designated routers

  Attacker on same segment can:
    → Send crafted OSPF Hello packets to 224.0.0.5
    → Participate in OSPF neighbor relationships
    → Inject false routing information (route poisoning)
    → Redirect traffic through attacker-controlled path (MITM)

  Detection in Wireshark:
    Filter: ospf
    Look for: unexpected OSPF Hello from non-router MAC addresses

Defense:
    Enable OSPF MD5 authentication on all routers
    Implement IGMP snooping on managed switches
    Isolate multicast traffic with VLANs
```

### Class E — Why 255.255.255.255 Matters in Pentesting

```
255.255.255.255 = Limited Broadcast (inside Class E range)
                  Used by: DHCP Discover, DHCP Request
                  Not routed beyond local segment

DHCP Starvation Attack:
  DHCP clients broadcast to 255.255.255.255 to discover servers
  Attacker floods DHCP server with thousands of DISCOVER packets
  Each uses a fake random MAC address
  DHCP server allocates an IP to each → pool exhausted
  Legitimate devices cannot get an IP address → DoS

  Tool: dhcpstarv, yersinia
  Demo (your lab only):
    yersinia -I   (interactive mode)
    → Select DHCP → Launch DHCP starvation

  Defense:
    Enable DHCP Snooping on managed switches
    Rate-limit DHCP requests per port
    Use Port Security to limit MACs per switch port

DHCP Spoofing (Rogue DHCP server):
  Attacker responds to 255.255.255.255 DHCP Discover
  before the real DHCP server responds
  Hands out:
    → Attacker's IP as the Default Gateway → all traffic through attacker
    → Attacker's IP as DNS server → DNS poisoning possible
  Classic on-path (MITM) setup without ARP spoofing needed

  Defense: DHCP Snooping — only trust uplink ports for DHCP offers
```

### The 0.0.0.0/0 and 255.255.255.255 in Firewall Rules

```
Firewall rule misconfiguration — common in small offices:

MISTAKE 1: Allowing 0.0.0.0/0 as source
  "ALLOW any source to reach internal server"
  → Allows the entire internet → completely open

MISTAKE 2: Confusing 255.255.255.255 with a host rule
  Some admins add 255.255.255.255 to block lists thinking
  it is "just another address in the 255.x.x.x range"
  Reality: blocking 255.255.255.255 breaks DHCP discovery
  Clients can no longer obtain IP addresses → network failure

  Always treat 255.255.255.255 as a special reserved address,
  never as a regular host IP in ACL rules.
```

### Scanning and Class D/E Address Space

```
Class D/E ranges are NOT valid unicast destinations
→ Do NOT appear in normal host scanning targets

But misconfigured systems sometimes:
  → Accept packets destined to multicast addresses
  → Respond to pings sent to 224.x.x.x or 240.x.x.x
  → Reveal themselves via passive multicast sniffing

Passive Multicast Discovery (on your lab LAN):
  sudo tcpdump -i eth0 dst net 224.0.0.0/4
  → Reveals all multicast traffic on the LAN
  → Can expose OSPF routers, mDNS (224.0.0.251),
    SSDP (239.255.255.250), and other multicast services

Common Multicast Addresses Worth Knowing:
  224.0.0.1   All hosts on subnet (All Hosts Group)
  224.0.0.2   All routers on subnet
  224.0.0.5   OSPF All Routers
  224.0.0.9   RIPv2 routers
  224.0.0.251 mDNS (used by Bonjour/Avahi — service discovery)
  239.255.255.250  SSDP / UPnP (used by IoT devices)
```

### SSDP Amplification — Class D in DDoS

```
SSDP (Simple Service Discovery Protocol) uses:
  Multicast address: 239.255.255.250
  UDP port: 1900

SSDP Amplification DDoS:
  Attacker spoofs victim IP as source
  Sends small SSDP M-SEARCH packet to 239.255.255.250
  All UPnP-enabled devices (routers, smart TVs, IoT) on LAN respond
  Each response is 30–40× larger than the request
  Victim gets flooded with amplified UDP traffic

Why it matters for your MERN/PERN servers:
  If your server is publicly accessible and UPnP is enabled
  → It can be used as a reflector/amplifier by attackers
  → Disable UPnP on all externally reachable hosts
  → Block UDP 1900 at your border firewall
```

---

## 16. 🧪 Practical Labs

### Lab 1 — Class D & E IP Classifier (Python)

```python
# Save as class_d_e_classifier.py
# Run: python3 class_d_e_classifier.py

import ipaddress

def classify_ip(ip_str: str) -> dict:
    """Full IP classification including Class D and E"""
    parts   = ip_str.split('.')
    first   = int(parts[0])
    ip_obj  = ipaddress.IPv4Address(ip_str)

    # Determine class
    if first < 128:
        cls = 'A'; prefix = '0'; fixed = 1; mask = '255.0.0.0'
        host_bits = 24; reserved = False
    elif first < 192:
        cls = 'B'; prefix = '10'; fixed = 2; mask = '255.255.0.0'
        host_bits = 16; reserved = False
    elif first < 224:
        cls = 'C'; prefix = '110'; fixed = 3; mask = '255.255.255.0'
        host_bits = 8; reserved = False
    elif first < 240:
        cls = 'D'; prefix = '1110'; fixed = 4; mask = None
        host_bits = None; reserved = True
        purpose = 'Multicast / Group Communication'
    else:
        cls = 'E'; prefix = '1111'; fixed = 4; mask = None
        host_bits = None; reserved = True
        purpose = 'Military / Experimental (Reserved)'

    result = {
        'ip':           ip_str,
        'class':        cls,
        'prefix_bits':  prefix,
        'fixed_bits':   fixed,
        'first_octet':  first,
        'is_private':   ip_obj.is_private,
        'is_reserved':  reserved,
    }

    if reserved:
        result['purpose'] = purpose
        result['variable_bits'] = 28
        result['total_ips'] = f"2^28 = {2**28:,} (~250 million)"
    else:
        # Calculate network details
        mask_obj    = ipaddress.IPv4Address(mask)
        network_int = int(ip_obj) & int(mask_obj)
        host_mask   = (1 << host_bits) - 1
        bcast_int   = network_int | host_mask

        result['default_mask']     = f"{mask}  (/{32 - host_bits})"
        result['network_address']  = str(ipaddress.IPv4Address(network_int))
        result['first_host']       = str(ipaddress.IPv4Address(network_int + 1))
        result['last_host']        = str(ipaddress.IPv4Address(bcast_int - 1))
        result['broadcast']        = str(ipaddress.IPv4Address(bcast_int))
        result['usable_hosts']     = 2 ** host_bits - 2

    return result

# ── Test IPs covering all 5 classes ───────────────────────────────────
test_ips = [
    ("194.2.3.4",     "Class C PUBLIC — lecture 42 ABC org example"),
    ("192.168.56.101","Class C PRIVATE — your Metasploitable2"),
    ("130.2.3.4",     "Class B PUBLIC — lecture 41 Oxford example"),
    ("10.0.0.5",      "Class A PRIVATE"),
    ("224.0.0.5",     "Class D — OSPF All Routers multicast"),
    ("239.255.255.250","Class D — SSDP/UPnP multicast"),
    ("230.0.0.1",     "Class D — example from lecture"),
    ("245.0.1.2",     "Class E — example from lecture"),
    ("255.255.255.255","Class E — Limited Broadcast (special)"),
]

for ip_str, label in test_ips:
    r = classify_ip(ip_str)
    print(f"\n{'='*60}")
    print(f"  {label}")
    print(f"{'─'*60}")
    print(f"  IP Address:    {r['ip']}")
    print(f"  Class:         {r['class']}   (prefix: {r['prefix_bits']})")
    print(f"  First Octet:   {r['first_octet']}")
    print(f"  Private:       {r['is_private']}")

    if r['is_reserved']:
        print(f"  Reserved:      YES")
        print(f"  Purpose:       {r['purpose']}")
        print(f"  Total IPs:     {r['total_ips']}")
        print(f"  Network/Host:  ❌ Not applicable")
    else:
        print(f"  Default Mask:  {r['default_mask']}")
        print(f"  Network:       {r['network_address']}")
        print(f"  First Host:    {r['first_host']}")
        print(f"  Last Host:     {r['last_host']}")
        print(f"  Broadcast:     {r['broadcast']}")
        print(f"  Usable Hosts:  {r['usable_hosts']:,}")
```

### Lab 2 — Observe Multicast Traffic on Your Lab Network

```bash
# Capture live multicast traffic on your isolated lab network
# This shows which Class D addresses are actually in use

# Listen for ALL multicast traffic (Class D range: 224.x.x.x - 239.x.x.x)
sudo tcpdump -i eth0 -n dst net 224.0.0.0/4

# Common results you will see even in a small lab:
#   224.0.0.251 → mDNS (Multicast DNS — Avahi/Bonjour service discovery)
#   224.0.0.1   → All hosts group (IGMP membership query)
#   239.255.255.250 → SSDP / UPnP (from Windows or IoT devices)

# Capture ONLY OSPF multicast (if you have a router)
sudo tcpdump -i eth0 -n dst 224.0.0.5 or dst 224.0.0.6

# Capture ONLY mDNS (Avahi — runs on most Linux systems)
sudo tcpdump -i eth0 -n 'udp port 5353'
# mDNS uses 224.0.0.251:5353 — a Class D multicast address

# See which services your own Parrot OS is multicasting
sudo ss -u -l -n | grep 224
# or
netstat -gn
# Shows which multicast groups your machine has joined
```

### Lab 3 — Binary Range Verifier (All 5 Classes)

```bash
python3 - << 'EOF'
# Verify the first-octet ranges for ALL 5 classes from binary

def binary_to_decimal(bits: str) -> int:
    return int(bits, 2)

def show_class_range(cls: str, prefix: str, var_bits: int):
    min_bits = prefix + '0' * var_bits
    max_bits = prefix + '1' * var_bits
    min_dec  = binary_to_decimal(min_bits)
    max_dec  = binary_to_decimal(max_bits)
    count    = max_dec - min_dec + 1
    print(f"Class {cls}:")
    print(f"  Prefix:       {prefix}  ({len(prefix)} fixed bits)")
    print(f"  Min first octet: {min_bits} = {min_dec}")
    print(f"  Max first octet: {max_bits} = {max_dec}")
    print(f"  Range:        {min_dec} – {max_dec}")
    print(f"  Count:        {count} values in first octet")
    print(f"  Variable bits total: 32 - {len(prefix)} = {32 - len(prefix)}")
    print(f"  Total IPs:    2^{32 - len(prefix)} = {2**(32 - len(prefix)):,}")
    print()

print("="*60)
print("CLASSFUL ADDRESSING — RANGE DERIVATION FROM BINARY")
print("="*60 + "\n")

show_class_range('A', '0',    7)   # 1 fixed + 7 variable in oct 1
show_class_range('B', '10',   6)   # 2 fixed + 6 variable in oct 1
show_class_range('C', '110',  5)   # 3 fixed + 5 variable in oct 1
show_class_range('D', '1110', 4)   # 4 fixed + 4 variable in oct 1  ← THIS LECTURE
show_class_range('E', '1111', 4)   # 4 fixed + 4 variable in oct 1  ← THIS LECTURE
EOF
```

### Lab 4 — Passive Multicast Discovery

```bash
# Discover which multicast groups exist on your lab network
# Safe to run — only listening, no packets sent

# Method 1: tcpdump passive capture (30 seconds)
echo "Listening for multicast traffic for 30 seconds..."
sudo timeout 30 tcpdump -i eth0 -n 'dst net 224.0.0.0/4' 2>/dev/null \
  | awk '{print $NF}' | sort | uniq -c | sort -rn

# Method 2: Check multicast group memberships on this machine
echo ""
echo "Multicast groups this machine has joined:"
ip maddr show eth0
# You should see:
#   inet 224.0.0.1 → All Hosts Group (every IP host must join this)

# Method 3: nmap multicast discovery (sends probes to well-known groups)
# Run only on your isolated lab network
sudo nmap --script broadcast-listener -e eth0

# Method 4: Manually ping the All-Hosts multicast group
ping -c 3 -I eth0 224.0.0.1
# All hosts that have joined 224.0.0.1 on the same LAN should reply
# In your lab: Metasploitable2 should respond
```

### Lab 5 — Class D/E Range in nmap — Why They Are Excluded

```bash
python3 - << 'EOF'
import ipaddress

# Demonstrate why nmap skips Class D and E in standard host scans
print("Testing which ranges nmap considers valid unicast hosts:\n")

test_ranges = [
    ("10.0.0.0/8",       "Class A Private"),
    ("172.16.0.0/12",    "Class B Private"),
    ("192.168.0.0/16",   "Class C Private"),
    ("224.0.0.0/4",      "Class D — Multicast"),
    ("240.0.0.0/4",      "Class E — Reserved"),
]

for cidr, label in test_ranges:
    network = ipaddress.IPv4Network(cidr)
    # Check if addresses in this range are usable unicast hosts
    sample  = network.network_address + 1
    is_mc   = sample.is_multicast
    is_rsv  = sample.is_reserved
    is_priv = sample.is_private

    status = []
    if is_mc:  status.append("MULTICAST (Class D)")
    if is_rsv: status.append("RESERVED (Class E)")
    if is_priv:status.append("PRIVATE (RFC 1918)")

    scannable = not (is_mc or is_rsv)
    print(f"{label:<30} ({cidr})")
    print(f"  Sample IP:   {sample}")
    print(f"  Multicast:   {is_mc}")
    print(f"  Reserved:    {is_rsv}")
    print(f"  Private:     {is_priv}")
    print(f"  Host-scannable: {'✅ Yes' if scannable else '❌ No — nmap skips'}")
    print()

print("Conclusion:")
print("  nmap -sn 224.0.0.0/4  → skips all (multicast, not real hosts)")
print("  nmap -sn 240.0.0.0/4  → skips all (reserved, not real hosts)")
print("  nmap -sn 192.168.1.0/24 → scans 254 hosts ✅")
EOF
```

---

## 17. Solved Examples

### Example 1 — Class D Identification

**Given IP:** `239.1.2.3`
**Question:** Which class? How many networks and hosts?

```
Step 1: Check first octet
  First octet = 239

Step 2: Match to range
  Class A: 0–127    → 239 not in range
  Class B: 128–191  → 239 not in range
  Class C: 192–223  → 239 not in range
  Class D: 224–239  → ✅ 239 IS in this range

Answer: Class D

Networks: ❌ Not applicable (no network concept in Class D)
Hosts:    ❌ Not applicable (no host concept in Class D)
Purpose:  Reserved for multicasting
```

---

### Example 2 — Class E Identification

**Given IP:** `245.0.1.2`
**Question:** Which class? How many networks and hosts?

```
Step 1: Check first octet
  First octet = 245

Step 2: Match to range
  Class A: 0–127    → 245 not in range
  Class B: 128–191  → 245 not in range
  Class C: 192–223  → 245 not in range
  Class D: 224–239  → 245 not in range
  Class E: 240–255  → ✅ 245 IS in this range

Answer: Class E

Networks: ❌ Not applicable (no network concept in Class E)
Hosts:    ❌ Not applicable (no host concept in Class E)
Purpose:  Reserved for military / experimental use
```

---

### Example 3 — Binary-First Check (Class D)

**Given IP in binary:** `11101010.00000001.00000010.00000011`
**Question:** Which class?

```
Step 1: Look at first 4 bits of first octet
  1 1 1 0 → matches Class D fixed prefix

Step 2: Convert first octet to decimal (verify range)
  11101010 = 128 + 64 + 32 + 0 + 8 + 2 = 234
  Is 234 in 224–239? ✅ Yes

Answer: Class D
```

---

### Example 4 — Binary-First Check (Class E)

**Given IP in binary:** `11110001.00000000.00000000.00000001`
**Question:** Which class?

```
Step 1: Look at first 4 bits of first octet
  1 1 1 1 → matches Class E fixed prefix

Step 2: Convert first octet to decimal (verify range)
  11110001 = 128 + 64 + 32 + 16 + 1 = 241
  Is 241 in 240–255? ✅ Yes

Answer: Class E
```

---

### Quick Class Identification — Full Decision Flow

```
Given any IP address → look at first octet value:

  0  – 127   → Class A  (prefix: 0)
  128 – 191  → Class B  (prefix: 10)
  192 – 223  → Class C  (prefix: 110)
  224 – 239  → Class D  (prefix: 1110) — Multicast,  no net/host
  240 – 255  → Class E  (prefix: 1111) — Military,   no net/host
```

---

## 18. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║       CLASS D & CLASS E IP ADDRESSING — EXAM CHEAT SHEET            ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS D IDENTIFICATION                                              ║
║  Prefix:    First 4 bits = 1 1 1 0  (fixed)                        ║
║  Range:     224 – 239  (first octet)                               ║
║  Check:     Is first octet in 224–239? → Class D                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS D STRUCTURE                                                   ║
║  [1110 + 4 bits][8 bits][8 bits][8 bits]                           ║
║  Fixed prefix: 4 bits                                              ║
║  Variable bits: 28  (32 - 4)                                       ║
║  Total IPs: 2²⁸ ≈ 250 million ≈ 6.25% of IPv4                    ║
║  Network ID: ❌ None      Host ID: ❌ None                          ║
║  Default mask: ❌ None    Purpose: Multicasting                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS E IDENTIFICATION                                              ║
║  Prefix:    First 4 bits = 1 1 1 1  (fixed)                        ║
║  Range:     240 – 255  (first octet)                               ║
║  Check:     Is first octet in 240–255? → Class E                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS E STRUCTURE                                                   ║
║  [1111 + 4 bits][8 bits][8 bits][8 bits]                           ║
║  Fixed prefix: 4 bits                                              ║
║  Variable bits: 28  (32 - 4)                                       ║
║  Total IPs: 2²⁸ ≈ 250 million ≈ 6.25% of IPv4                    ║
║  Network ID: ❌ None      Host ID: ❌ None                          ║
║  Default mask: ❌ None    Purpose: Military / Experimental         ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS D vs CLASS E — ONE GLANCE                                     ║
║  D: prefix 1110, range 224–239, Multicast                          ║
║  E: prefix 1111, range 240–255, Military/Reserved                  ║
║  Both: 2²⁸ IPs, no network, no host, no mask                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY NUMBERS TO MEMORISE                                             ║
║  2²⁸ = 268,435,456 ≈ 250 million ≈ 25 crore                      ║
║  D + E together = ~500 million = 12.5% of IPv4 wasted             ║
╠══════════════════════════════════════════════════════════════════════╣
║  RANGE DERIVATION TRICK (binary → decimal)                           ║
║  Class D min: 11100000 = 128+64+32 = 224                          ║
║  Class D max: 11101111 = 128+64+32+8+4+2+1 = 239                  ║
║  Class E min: 11110000 = 128+64+32+16 = 240                       ║
║  Class E max: 11111111 = 128+64+32+16+8+4+2+1 = 255               ║
╠══════════════════════════════════════════════════════════════════════╣
║  SPECIAL ADDRESS IN CLASS E RANGE                                    ║
║  255.255.255.255 = Limited Broadcast (not a Class E host)          ║
║  → Sends to ALL hosts on LOCAL segment                             ║
║  → Used by DHCP Discover                                           ║
║  → Not forwarded by routers                                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  Class D used for OSPF (224.0.0.5), mDNS (224.0.0.251),           ║
║    SSDP/UPnP (239.255.255.250) — all attack surfaces              ║
║  SSDP amplification DDoS uses 239.255.255.250 (Class D)            ║
║  DHCP starvation targets 255.255.255.255 (Class E range)           ║
║  255.255.255.255 in firewall ACL as a host = misconfiguration      ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY CLASSFUL ADDRESSING ENDED (1993)                                ║
║  D + E reserved ~500 million addresses permanently                  ║
║  Class A/B/C also suffered internal waste                          ║
║  Solution: CIDR (Classless Inter-Domain Routing)                   ║
║  Classful: used BEFORE 1993                                        ║
║  CIDR/Classless: used AFTER 1993 — still the standard today        ║
╠══════════════════════════════════════════════════════════════════════╣
║  ALL CLASSES QUICK COMPARE                                           ║
║  A: 0–127,   /8,  126 nets,       ~16.7M hosts/net                ║
║  B: 128–191, /16, 16,384 nets,    65,534 hosts/net                ║
║  C: 192–223, /24, ~2,097,152 nets, 254 hosts/net                  ║
║  D: 224–239  Multicast — no net/host ← THIS LECTURE               ║
║  E: 240–255  Reserved  — no net/host ← THIS LECTURE               ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Subnetting — dividing a /24 into /25, /26, /27, /28 blocks
- [ ] CIDR — Classless Inter-Domain Routing (replaces classful after 1993)
- [ ] VLSM — Variable Length Subnet Masking
- [ ] NAT — how private RFC 1918 addresses reach the public internet
- [ ] IGMP — Internet Group Management Protocol (manages Class D groups)
- [ ] IPv6 — why we ran out of IPv4 and what replaced classful addressing

---

_Notes compiled from: Networking Course Lecture 43 — Class D & Class E IP Addressing_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
