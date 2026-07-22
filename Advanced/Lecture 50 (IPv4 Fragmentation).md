# 🌐 IPv4 Fragmentation

### " "Networking Course — Lecture 43

> **Source:** Gate Smashers — Fragmentation in IPv4
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Fragmentation?](#1-what-is-fragmentation)
2. [The Three Fragmentation Fields in IPv4 Header](#2-the-three-fragmentation-fields-in-ipv4-header)
3. [Identification Field](#3-identification-field)
4. [Flag Field — 3 Bits](#4-flag-field--3-bits)
5. [Fragment Offset Field](#5-fragment-offset-field)
6. [MTU — Maximum Transmission Unit](#6-mtu--maximum-transmission-unit)
7. [Worked Numerical — 3000 Byte Datagram, MTU 500](#7-worked-numerical--3000-byte-datagram-mtu-500)
8. [Step-by-Step Fragment Table](#8-step-by-step-fragment-table)
9. [How Reassembly Works at Destination](#9-how-reassembly-works-at-destination)
10. [All Classes Side-by-Side](#10-key-formulas--quick-calculation-rules)
11. [🔴 Security Context](#11--security-context)
12. [🧪 Practical Labs](#12--practical-labs)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. What is Fragmentation?

### Core Concept

```
IPv4 is a DATAGRAM SERVICE.

What does datagram service mean?
  → Packets can travel different routes to the same destination
  → Different links along the way may have different size limits
  → When a packet is too large for the next link → FRAGMENT it

Fragmentation = Breaking one large datagram into smaller pieces
                so each piece fits within the link's size limit (MTU)
```

### Where Does Fragmentation Happen?

```
[Source] ──────► [Router] ──────► [Next Network] ──────► [Destination]
            large datagram   link MTU = 500 bytes
            arrives (3000B)  router splits into 7 fragments
                             each fragment ≤ 500 bytes
                             all 7 forwarded independently
                                                    destination reassembles
```

### Why is IPv4 Unique Here?

```
IPv4:   Fragmentation can happen AT ROUTERS along the path
        → Any intermediate router can fragment if needed
        → Destination reassembles the original datagram

IPv6:   Fragmentation NOT allowed at routers
        → Only the SOURCE can fragment
        → Uses Path MTU Discovery instead
        → Routers send ICMPv6 "Packet Too Big" if needed
```

---

## 2. The Three Fragmentation Fields in IPv4 Header

These three fields occupy the **second row** of the IPv4 header:

```
IPv4 Header — Row 2 (32 bits total):
┌─────────────────────┬──────────┬──────────────────────────────┐
│  Identification     │  Flags   │      Fragment Offset         │
│     16 bits         │  3 bits  │          13 bits             │
└─────────────────────┴──────────┴──────────────────────────────┘
        ↑                  ↑                    ↑
   Who do I          Allowed to         How many data bytes
   belong to?        fragment?          came before me?
   (all fragments    Last or not?
   of same datagram
   share this value)
```

---

## 3. Identification Field

### Purpose

```
When a datagram is fragmented into multiple pieces,
ALL fragments carry the SAME identification value.

At the destination, this is how reassembly works:
  "All fragments with ID = 4500 belong to the same datagram"
  "All fragments with ID = 4501 belong to a different datagram"

Without identification → destination cannot know which
fragments go together!
```

### Properties

```
Size:    16 bits
Range:   0 to 2¹⁶ - 1  =  0 to 65,535
Assigned by: The ORIGINAL SENDER (source host)
Changes: Does NOT change across fragments — all fragments
         of the same datagram share the SAME identification value
```

### Analogy

> Identification is like a **tracking number on a package**.
> When a courier splits your large parcel into 7 boxes,
> all 7 boxes carry the same tracking number.
> At delivery, the recipient groups all boxes by tracking number
> and reassembles your original item.

---

## 4. Flag Field — 3 Bits

```
Flag field layout:
┌─────┬─────────────────┬─────────────────┐
│  0  │   DF (bit 1)    │   MF (bit 2)    │
│ MSB │  Do Not Fragment│  More Fragments  │
│     │   (bit 1)       │   (bit 0)       │
└─────┴─────────────────┴─────────────────┘
  ↑
  Reserved — always 0, cannot be changed
```

### Bit 0 (MSB) — Reserved

```
Value: always 0
Purpose: reserved for future use
Action: never change this bit
```

### Bit 1 — DF (Do Not Fragment)

```
DF = 0 → Fragmentation IS ALLOWED
          Routers along the path may fragment this datagram
          Normal operation for most traffic

DF = 1 → Fragmentation is NOT ALLOWED
          If datagram is too large for a link:
            → Router DROPS the datagram
            → Router sends ICMP "Fragmentation Needed" back to source

When do we use DF = 1?
  → Path MTU Discovery (find smallest MTU on the path)
  → When we want to know the maximum packet size that fits end-to-end
  → Applications that cannot handle reassembly (some real-time apps)
```

### Bit 2 — MF (More Fragments)

```
MF = 1 → More fragments FOLLOW after this one
          "I am NOT the last fragment"
          → Set on ALL fragments EXCEPT the last

MF = 0 → This is the LAST fragment (or only fragment)
          "No more fragments after me"
          → Set ONLY on the last fragment

Memory trick:
  MF = 1 → "More coming!" (middle / early packets)
  MF = 0 → "I'm the last one" (final packet or unfragmented)
```

### Flag Summary Table

| DF  | MF  | Meaning                                                    |
| --- | --- | ---------------------------------------------------------- |
| 0   | 0   | Not fragmented (single complete datagram) OR last fragment |
| 0   | 1   | Fragment — more fragments follow                           |
| 1   | 0   | Do not fragment — this is the only/last                    |
| 1   | 1   | Do not fragment + more follow (invalid state)              |

---

## 5. Fragment Offset Field

### Purpose

```
Fragment Offset answers:
  "How many bytes of DATA came before this fragment?"

This tells the destination EXACTLY WHERE in the original
datagram this fragment's data belongs → enables correct reassembly.
```

### Properties

```
Size:    13 bits
Unit:    NOT bytes — measured in units of 8 bytes (scale of 8)
Range:   0 to 2¹³ - 1 = 0 to 8191
         (representing byte positions 0 to 65,528)
```

### Why Scale of 8?

```
Fragment offset is 13 bits.
If we measured in bytes:  max offset = 2¹³ - 1 = 8191 bytes
But IP payload can be up to 65,515 bytes → 8191 is not enough!

Solution: measure in 8-byte units
  13 bits × 8 = effectively 16 bits of byte addressing
  Max offset = 8191 × 8 = 65,528 bytes ✓ — enough for any datagram

This means: every fragment's data must be a MULTIPLE OF 8 bytes
(except the last fragment which can be any size)
```

### How to Calculate Offset

```
Fragment offset = (total data bytes sent before this fragment) ÷ 8

Example:
  480 bytes sent before this fragment
  Offset = 480 ÷ 8 = 60

At destination, to recover byte position:
  Byte position = offset × 8 = 60 × 8 = 480 ✓
```

---

## 6. MTU — Maximum Transmission Unit

### Definition

```
MTU = Maximum Transmission Unit

The largest packet size (in bytes) that a particular
network link or protocol can transmit in a single frame.

MTU is set by the DATA LINK LAYER of each network.
Different links can have different MTUs.

Common MTU values:
  Ethernet:        1500 bytes  (most common)
  PPPoE (DSL):     1492 bytes
  IPv4 minimum:     576 bytes  (all IPv4 nodes must handle)
  This lecture:     500 bytes  (exam example)
```

### Header is Part of the MTU

```
CRITICAL POINT:
  MTU = 500 bytes total
  = 20 bytes (IP header) + 480 bytes (data/payload)

  You CANNOT send 500 bytes of pure data — the header takes 20 bytes!
  Usable data per fragment = MTU - IP header = 500 - 20 = 480 bytes

General formula:
  Data per fragment = MTU - Header size
                    = MTU - 20        (for standard IPv4 header)
```

---

## 7. Worked Numerical — 3000 Byte Datagram, MTU 500

### Problem Statement

```
A datagram of 3000 bytes arrives at a router:
  Total size:    3000 bytes
  IP Header:       20 bytes
  IP Payload:    2980 bytes  (pure data from upper layer)

The router must forward this datagram to a link with:
  MTU = 500 bytes

Find:
  (a) Number of fragments
  (b) Total Length of each fragment
  (c) MF bit value of each fragment
  (d) Fragment Offset of each fragment
```

### Step 1 — Find Data Per Fragment

```
MTU             = 500 bytes
IP Header       =  20 bytes
Data/fragment   = 500 - 20 = 480 bytes

Each fragment carries 480 bytes of payload
(this value must be a multiple of 8 → 480 ÷ 8 = 60 ✓)
```

### Step 2 — Find Number of Fragments

```
Total payload  = 2980 bytes
Data/fragment  =  480 bytes

Number of fragments = ⌈2980 ÷ 480⌉ = ⌈6.208...⌉ = 7 fragments

Verify data distribution:
  Fragments 1–6: 480 bytes each  → 6 × 480 = 2880 bytes
  Fragment 7:    2980 - 2880     = 100 bytes  (last, smaller)

✓ 2880 + 100 = 2980 bytes total payload ✓
```

### Step 3 — Total Length per Fragment

```
Total Length = IP Header + Data payload in that fragment

Fragments 1 to 6:  20 + 480 = 500 bytes
Fragment 7:        20 + 100 = 120 bytes
```

### Step 4 — MF Bit per Fragment

```
All fragments EXCEPT the last → MF = 1 ("more coming")
Last fragment only            → MF = 0 ("I am the last")

Fragment 1: MF = 1
Fragment 2: MF = 1
Fragment 3: MF = 1
Fragment 4: MF = 1
Fragment 5: MF = 1
Fragment 6: MF = 1
Fragment 7: MF = 0  ← last fragment
```

### Step 5 — Fragment Offset per Fragment

```
Offset = (data bytes sent before this fragment) ÷ 8

Fragment 1: 0 bytes before it         → 0 ÷ 8 = 0
Fragment 2: 480 bytes before it        → 480 ÷ 8 = 60
Fragment 3: 480+480 = 960 before it   → 960 ÷ 8 = 120
Fragment 4: 480×3 = 1440 before it    → 1440 ÷ 8 = 180
Fragment 5: 480×4 = 1920 before it    → 1920 ÷ 8 = 240
Fragment 6: 480×5 = 2400 before it    → 2400 ÷ 8 = 300
Fragment 7: 480×6 = 2880 before it    → 2880 ÷ 8 = 360

Verify last offset recovers correct byte position:
  360 × 8 = 2880 bytes before fragment 7
  Fragment 7 carries 100 bytes
  2880 + 100 = 2980 = total payload ✓
```

---

## 8. Step-by-Step Fragment Table

### Complete Answer Table

| Fragment | Data (bytes) | Total Length | MF    | Offset | Identification |
| -------- | ------------ | ------------ | ----- | ------ | -------------- |
| P1       | 480          | 500          | 1     | 0      | same (e.g. X)  |
| P2       | 480          | 500          | 1     | 60     | same (X)       |
| P3       | 480          | 500          | 1     | 120    | same (X)       |
| P4       | 480          | 500          | 1     | 180    | same (X)       |
| P5       | 480          | 500          | 1     | 240    | same (X)       |
| P6       | 480          | 500          | 1     | 300    | same (X)       |
| P7       | 100          | 120          | **0** | 360    | same (X)       |

### Visual Timeline

```
Original datagram (3000 bytes):
├──────── 20B header ────────┤──────────────── 2980B payload ─────────────────────┤

After fragmentation at router (MTU = 500):

P1: ├─20B─┤──────480B data──────┤  offset=0,   MF=1, len=500
P2: ├─20B─┤──────480B data──────┤  offset=60,  MF=1, len=500
P3: ├─20B─┤──────480B data──────┤  offset=120, MF=1, len=500
P4: ├─20B─┤──────480B data──────┤  offset=180, MF=1, len=500
P5: ├─20B─┤──────480B data──────┤  offset=240, MF=1, len=500
P6: ├─20B─┤──────480B data──────┤  offset=300, MF=1, len=500
P7: ├─20B─┤──100B─┤             offset=360, MF=0, len=120

Note: All 7 fragments share the SAME Identification value.
```

---

## 9. How Reassembly Works at Destination

```
Step 1: Destination receives fragments (possibly out of order)
        → Groups them by Identification value

Step 2: For each fragment group:
        → Sort by Fragment Offset (lowest first = P1)
        → Find the fragment with MF = 0 → that is the last one

Step 3: Verify completeness
        → Check offsets form a continuous sequence: 0, 60, 120, ...
        → No gaps allowed

Step 4: Reconstruct
        → Place each fragment's data at: offset × 8
        → P1 data at byte 0
        → P2 data at byte 480
        → P3 data at byte 960
        → ... and so on
        → Remove the extra headers (keep only original header)
        → Original 2980 byte payload is recovered

Step 5: If any fragment is LOST:
        → Entire original datagram is discarded
        → Upper layer (TCP) detects loss and requests retransmission
        → UDP: data is simply lost (no recovery)
```

---

## 10. Key Formulas & Quick Calculation Rules

### Essential Formulas

```
Data per fragment  = MTU - IP Header Size (usually 20 bytes)

Number of fragments = ⌈Total Payload ÷ Data per fragment⌉
                    (ceiling — always round UP)

Total Length of fragment n (not last) = MTU
Total Length of last fragment          = 20 + remaining payload bytes

MF bit:
  All fragments except last → MF = 1
  Last fragment only        → MF = 0

Fragment Offset of fragment n:
  = (data bytes in ALL previous fragments) ÷ 8
  = (n-1) × data_per_fragment ÷ 8   [for equal-size fragments]

Recover byte position from offset:
  byte_position = offset × 8
```

### Data per Fragment Must Be Multiple of 8

```
Why? Fragment offset counts in 8-byte units.
If data per fragment is NOT a multiple of 8,
the offset calculation breaks for subsequent fragments.

Check: 480 ÷ 8 = 60 ✓ (whole number → valid)
       479 ÷ 8 = 59.875 ✗ (not valid — adjust MTU or header size)

In exams: data per fragment is always engineered to be multiple of 8.
In real networks: MTU values (1500, 576, etc.) ensure this automatically.
```

### Quick Number Line for This Problem

```
Offset × 8 = byte start position of that fragment's data:

0   × 8 =    0  ← P1 starts here
60  × 8 =  480  ← P2 starts here
120 × 8 =  960  ← P3 starts here
180 × 8 = 1440  ← P4 starts here
240 × 8 = 1920  ← P5 starts here
300 × 8 = 2400  ← P6 starts here
360 × 8 = 2880  ← P7 starts here, ends at 2880+100=2980 ✓
```

---

## 11. 🔴 Security Context

### Fragmentation as an Attack Vector

```
IPv4 fragmentation was designed for performance,
but it introduced several serious security vulnerabilities
that are actively exploited today.

As a cybersecurity student, you need to understand
BOTH how fragmentation works AND how it can be weaponized.
```

### Attack 1 — Teardrop Attack (CVE-1997-0947)

```
How it works:
  Attacker sends overlapping fragments (bad offset values)
  Fragment 1: offset=0,  data covers bytes 0-479
  Fragment 2: offset=30, data starts at byte 240 (OVERLAPS with F1!)

  Vulnerable OS tries to reassemble overlapping fragments
  → Buffer overflow or kernel panic
  → System crash (Blue Screen of Death on old Windows)

Affected:
  Windows 3.1, 95, NT 3.51, NT 4.0
  Linux kernel < 2.0.32 (1997)

Defense:
  Patch OS (all modern OSes handle this safely)
  Firewall: drop fragments with overlapping offsets
  IDS rule: alert on overlapping fragment offsets
```

### Attack 2 — Ping of Death (CVE-1996-1454)

```
How it works:
  ICMP ping packet has max legal size: 65,535 bytes
  Attacker sends fragments that, when REASSEMBLED, exceed 65,535 bytes
  Individual fragments look valid and pass firewall inspection
  Reassembly at destination → buffer overflow → crash

  Each fragment ≤ MTU → looks legitimate to packet filters
  But last_offset × 8 + last_data_size > 65,535 → overflow

Detection:
  Check: (offset × 8 + data_length) > 65,535 for any fragment
  If yes → malicious, DROP

Defense:
  Patch OS / modern systems immune
  Firewall / IDS signature for oversized reassembled datagrams
```

### Attack 3 — Fragment Evasion (IDS/Firewall Bypass)

```
The BIG modern threat:

Most firewalls and IDS inspect packets at the APPLICATION layer.
They need complete data to detect attacks like SQL injection, XSS, etc.

If an attacker FRAGMENTS the malicious payload:
  Fragment 1: "SELECT * FROM us"  (offset=0,  MF=1)
  Fragment 2: "ers WHERE 1=1"     (offset=2,  MF=0)

  Neither fragment alone looks malicious
  Firewall may pass both without reassembling → attack gets through!

Stateless firewall:  only sees individual fragments → BLIND to payload
Stateful firewall:   reassembles fragments first → sees full attack
Deep packet inspection (DPI): reassembles AND inspects → detects attack

Defense for your MERN/PERN apps:
  Use a reverse proxy (nginx, Cloudflare) that reassembles before app
  Enable DPI on your firewall / WAF
  Drop tiny fragments (offset > 0 with small data sizes — suspicious)
```

### Attack 4 — IP Fragment Flooding (DoS)

```
How it works:
  Attacker sends massive stream of fragments
  All with MF=1 but never sends the last fragment (MF=0)
  Victim keeps allocated memory waiting for reassembly
  Memory buffer fills up → legitimate traffic dropped → DoS

  Reassembly timeout exists (usually 15-30 seconds)
  but attacker sends faster than timeout can clean up

Detection in Wireshark:
  Filter: ip.flags.mf == 1    → see all non-final fragments
  Watch for: thousands of fragments from same source, never completed

Defense:
  Limit fragment reassembly buffer size
  Aggressive timeout for incomplete fragment sets
  Rate-limit fragments per source IP
  Firewall rule: drop fragments from untrusted sources
```

### Attack 5 — Tiny Fragment Attack

```
How it works:
  Attacker sends first fragment so small (8 bytes of data)
  that the TCP header is split across two fragments

  Fragment 1: IP header + first 8 bytes of TCP header (src/dst port)
  Fragment 2: IP header + rest of TCP header (flags, seq, etc.)

  Packet filters that inspect TCP ports see only fragment 1 → pass it
  Fragment 2 follows and completes the connection with dangerous flags

  Classic example: attacker hides RST or SYN flags in fragment 2
  → firewall rules based on TCP flags are bypassed

Defense:
  Drop all fragments where offset=1 (8 bytes of data = only port info)
  iptables: -m ipv4options --fragment -j DROP  (fragmented packets)
  Modern firewalls automatically handle this
```

### Checking Fragmentation in Your Lab

```bash
# Check if a path fragments packets
# (tracepath does MTU discovery)
tracepath google.com
# Look for "pmtu XXXX" lines showing MTU reductions

# Send an oversized packet with DF=1 (path MTU discovery)
ping -M do -s 1473 192.168.56.101
# -M do = DF bit set, -s = data size
# If too large: "Frag needed and DF set" error → reveals MTU

# Send a large fragmented ping (forces fragmentation)
ping -s 2000 192.168.56.101
# Packet will be fragmented since 2000 + 28 > 1500 (Ethernet MTU)

# Observe fragmentation in Wireshark:
# Filter: ip.flags.mf == 1    → see all non-final fragments
# Filter: ip.frag_offset > 0  → see all subsequent fragments
# Filter: ip.flags.df == 1    → see packets with DF bit set
```

---

## 12. 🧪 Practical Labs

### Lab 1 — Fragment Calculator (Python)

```python
# Save as fragment_calc.py — run: python3 fragment_calc.py
import math

def fragment_datagram(total_size: int, header_size: int, mtu: int):
    """
    Calculate fragmentation details for a datagram.
    total_size:  complete datagram size in bytes (header + payload)
    header_size: IP header size in bytes (usually 20)
    mtu:         maximum transmission unit of outgoing link
    """
    payload      = total_size - header_size
    data_per_frag = mtu - header_size

    # data_per_frag must be multiple of 8
    if data_per_frag % 8 != 0:
        data_per_frag = (data_per_frag // 8) * 8
        print(f"  [!] Adjusted data/fragment to {data_per_frag} (multiple of 8)")

    num_fragments = math.ceil(payload / data_per_frag)

    print(f"\n{'='*60}")
    print(f"FRAGMENTATION ANALYSIS")
    print(f"{'='*60}")
    print(f"  Original datagram:  {total_size} bytes")
    print(f"  IP Header:          {header_size} bytes")
    print(f"  Total Payload:      {payload} bytes")
    print(f"  Link MTU:           {mtu} bytes")
    print(f"  Data per fragment:  {data_per_frag} bytes")
    print(f"  Fragments needed:   {num_fragments}")
    print()
    print(f"{'Frag':<6} {'Data(B)':<10} {'TotalLen':<10} {'MF':<4} {'Offset':<8} {'BytePos'}")
    print("-" * 55)

    bytes_sent = 0
    for i in range(1, num_fragments + 1):
        is_last  = (i == num_fragments)
        data     = payload - bytes_sent if is_last else data_per_frag
        mf       = 0 if is_last else 1
        offset   = bytes_sent // 8
        tot_len  = header_size + data
        byte_pos = offset * 8   # verify: should equal bytes_sent

        print(f"P{i:<5} {data:<10} {tot_len:<10} {mf:<4} {offset:<8} {byte_pos}")
        bytes_sent += data

    print(f"\nTotal payload reassembled: {bytes_sent} bytes {'✓' if bytes_sent == payload else '✗ ERROR'}")

# Lecture example
fragment_datagram(total_size=3000, header_size=20, mtu=500)

print()

# Your own test cases
fragment_datagram(total_size=4020, header_size=20, mtu=1500)  # Ethernet MTU
fragment_datagram(total_size=8000, header_size=20, mtu=576)   # Minimum IPv4 MTU
```

### Lab 2 — Observe Real Fragmentation in Wireshark

```bash
# Step 1: Start Wireshark on your lab interface
# Filter: ip.flags.mf == 1 or ip.frag_offset > 0

# Step 2: Trigger fragmentation using ping with oversized packet
# Default Ethernet MTU = 1500 bytes
# ping data + ICMP(8B) + IP(20B) = total → if > 1500, fragmented

# Packet that fits (no fragmentation):
ping -c 1 -s 1400 192.168.56.101
# 1400 + 8 (ICMP) + 20 (IP) = 1428 < 1500 → NO fragmentation

# Packet that forces fragmentation:
ping -c 1 -s 3000 192.168.56.101
# 3000 + 8 + 20 = 3028 > 1500 → FRAGMENTED into 3 parts

# In Wireshark you will see:
#   Fragment 1: Total Length=1500, MF=1, Offset=0
#   Fragment 2: Total Length=1500, MF=1, Offset=185
#   Fragment 3: Total Length=~60,  MF=0, Offset=370

# Verify offset math:
python3 - << 'EOF'
data_per_frag = 1500 - 20  # 1480
payload = 3000 + 8         # ICMP data + ICMP header
print(f"Payload: {payload} bytes")
print(f"Data/fragment: {data_per_frag} bytes")
import math
n = math.ceil(payload / data_per_frag)
print(f"Fragments: {n}")
for i in range(n):
    offset = (i * data_per_frag) // 8
    print(f"P{i+1} offset = {i * data_per_frag} // 8 = {offset}")
EOF
```

### Lab 3 — DF Bit and Path MTU Discovery

```bash
# Test DF bit (Do Not Fragment) behavior

# Set DF=1 and send a packet that requires fragmentation
# If MTU is too small → ICMP "Fragmentation Needed" is returned

# Test with DF=1 (no fragmentation allowed):
ping -M do -c 1 -s 1473 192.168.56.101
# -M do = DF bit set (do not fragment)
# -s 1473 = data size (1473 + 8 ICMP + 20 IP = 1501 > MTU 1500)
# Expected: "Frag needed and DF set (mtu = 1500)" error message

# Test with DF=0 (fragmentation allowed — default):
ping -M dont -c 1 -s 1473 192.168.56.101
# -M dont = DF=0, fragmentation allowed
# Expected: success — router fragments the packet

# Observe in Wireshark:
# DF=1 case: ICMP Type 3, Code 4 ("Fragmentation Needed") in reply
# DF=0 case: Fragments with MF=1 followed by MF=0

echo ""
echo "Wireshark filters for this lab:"
echo "  ip.flags.df == 1        → packets with DF bit set"
echo "  icmp.type == 3          → ICMP Destination Unreachable"
echo "  icmp.code == 4          → specifically 'Fragmentation Needed'"
```

### Lab 4 — Detect Fragment-Based Attacks with Scapy

```python
# Save as detect_fragments.py
# Run: sudo python3 detect_fragments.py
# Requires: pip install scapy

from scapy.all import sniff, IP

suspicious = []

def inspect_fragment(pkt):
    if IP in pkt:
        ip = pkt[IP]
        frag_offset = ip.frag          # fragment offset (in 8-byte units)
        mf_bit      = ip.flags & 0x1   # More Fragments bit
        df_bit      = (ip.flags >> 1) & 0x1

        # Rule 1: Tiny fragment attack (first fragment too small)
        # Legitimate first fragment should carry full TCP header (≥20 bytes)
        if frag_offset == 0 and mf_bit == 1 and len(pkt) < 68:
            alert = f"[!] TINY FRAGMENT from {ip.src} → possible evasion attack"
            print(alert)
            suspicious.append(('tiny_fragment', ip.src))

        # Rule 2: Oversized reassembly check
        # (frag_offset × 8) + data_length should not exceed 65535
        data_len = len(ip.payload)
        reassembled_end = frag_offset * 8 + data_len
        if reassembled_end > 65535:
            print(f"[!] PING OF DEATH from {ip.src} → reassembled size {reassembled_end}")
            suspicious.append(('ping_of_death', ip.src))

        # Rule 3: Log all fragments for monitoring
        if mf_bit == 1 or frag_offset > 0:
            print(f"[i] Fragment: src={ip.src} offset={frag_offset} "
                  f"MF={mf_bit} DF={df_bit} size={len(pkt)}")

print("Monitoring fragments... (Ctrl+C to stop)")
print("Trigger with: ping -c 1 -s 3000 <any-ip>")
print()
sniff(filter="ip", prn=inspect_fragment, store=0)
```

### Lab 5 — Fragment a Custom Packet with Scapy

```python
# Save as send_fragments.py
# Run: sudo python3 send_fragments.py
# This demonstrates how fragmentation works at packet level
# ONLY run against your own Metasploitable2 in your lab

from scapy.all import IP, ICMP, Raw, send, fragment

# Create a large ICMP packet (will be fragmented)
target = "192.168.56.101"
large_data = b"X" * 3000   # 3000 bytes of data

packet = IP(dst=target) / ICMP() / Raw(load=large_data)

print(f"Original packet size: {len(packet)} bytes")
print(f"Fragmenting with MTU=500 (data/frag = 480 bytes)...\n")

# Fragment the packet
frags = fragment(packet, fragsize=480)

print(f"Number of fragments created: {len(frags)}")
print()
print(f"{'Frag':<6} {'Size':<8} {'MF':<4} {'Offset':<8} {'Data'}")
print("-" * 45)

for i, frag in enumerate(frags, 1):
    mf     = frag[IP].flags & 1
    offset = frag[IP].frag
    size   = len(frag)
    data   = size - 20
    print(f"P{i:<5} {size:<8} {mf:<4} {offset:<8} {data} bytes")

# Send the fragments (comment out if you just want to see the analysis)
# print("\nSending fragments...")
# send(frags, verbose=False)
# print("Done.")
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║         IPv4 FRAGMENTATION — EXAM CHEAT SHEET                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY FRAGMENTATION?                                                  ║
║  Different links have different MTUs                                ║
║  If datagram > next link MTU → router FRAGMENTS it                  ║
║  Destination REASSEMBLES original datagram                          ║
║  IPv4: routers CAN fragment │ IPv6: only source can fragment        ║
╠══════════════════════════════════════════════════════════════════════╣
║  THREE FIELDS INVOLVED (row 2 of IPv4 header)                       ║
║  Identification:    16 bits — same value for ALL fragments of       ║
║                               the same datagram                     ║
║  Flags:             3 bits — bit0=reserved(0), DF, MF              ║
║  Fragment Offset:  13 bits — data bytes before this ÷ 8            ║
╠══════════════════════════════════════════════════════════════════════╣
║  FLAG BITS                                                           ║
║  DF=0 → fragmentation ALLOWED (normal)                             ║
║  DF=1 → do NOT fragment (drop if too large, send ICMP error)       ║
║  MF=1 → more fragments follow (all packets except last)            ║
║  MF=0 → last (or only) fragment                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FORMULAS                                                        ║
║  Data/fragment   = MTU - 20 (IP header)                            ║
║  Num fragments   = ⌈Total Payload ÷ Data per fragment⌉             ║
║  Last fragment   = Total Payload - (Data/frag × (N-1))             ║
║  Total Length    = 20 + data in that fragment                       ║
║  Offset          = (cumulative data before) ÷ 8                    ║
║  Recover bytes   = offset × 8                                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  OFFSET IS IN UNITS OF 8 BYTES (scale of 8)                         ║
║  Data per fragment MUST be a multiple of 8                          ║
║  (except last fragment which can be any size)                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  WORKED EXAMPLE: 3000B datagram, MTU=500                            ║
║  Header = 20B → Payload = 2980B                                    ║
║  Data/frag = 500-20 = 480B                                         ║
║  Fragments = ⌈2980/480⌉ = 7                                        ║
║                                                                      ║
║  P1: len=500, MF=1, offset=0                                       ║
║  P2: len=500, MF=1, offset=60    (480÷8)                          ║
║  P3: len=500, MF=1, offset=120   (960÷8)                          ║
║  P4: len=500, MF=1, offset=180  (1440÷8)                          ║
║  P5: len=500, MF=1, offset=240  (1920÷8)                          ║
║  P6: len=500, MF=1, offset=300  (2400÷8)                          ║
║  P7: len=120, MF=0, offset=360  (2880÷8) ← LAST, 100B data       ║
╠══════════════════════════════════════════════════════════════════════╣
║  FRAGMENTATION ATTACKS                                               ║
║  Teardrop:         overlapping offsets → buffer overflow (legacy)   ║
║  Ping of Death:    reassembled size > 65535 bytes                   ║
║  Fragment Evasion: split attack payload across frags to bypass IDS  ║
║  Fragment Flood:   MF=1 fragments with no terminator → memory DoS  ║
║  Tiny Fragment:    8-byte first fragment → hides TCP flags from FW  ║
╠══════════════════════════════════════════════════════════════════════╣
║  REASSEMBLY FACTS                                                    ║
║  Done at: DESTINATION only (not at intermediate routers)           ║
║  Uses: Identification + Offset to rebuild original order            ║
║  If any fragment lost → entire datagram discarded                   ║
║  TCP: detects loss, retransmits │ UDP: data permanently lost        ║
╠══════════════════════════════════════════════════════════════════════╣
║  WIRESHARK FILTERS                                                   ║
║  ip.flags.mf == 1        → non-final fragments                     ║
║  ip.frag_offset > 0      → subsequent fragments                    ║
║  ip.flags.df == 1        → DF bit set (no fragment allowed)        ║
║  icmp.code == 4          → "Fragmentation Needed" ICMP message     ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] IPv4 Header — All fields complete (Total Length, TTL, Protocol, Checksum)
- [ ] ICMP — Error messages including "Fragmentation Needed" (Type 3, Code 4)
- [ ] Path MTU Discovery — How sources find the minimum MTU end-to-end
- [ ] IPv6 — Why fragmentation was removed from routers
- [ ] Subnetting — Dividing IP address space (follows addressing series)

---

_Notes compiled from: Networking Course Lecture 43 — Fragmentation in IPv4_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
