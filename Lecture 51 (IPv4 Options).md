# 🌐 IPv4 — Options & Padding Field

### Cybersecurity Student Notes | Networking Course — Lecture 51

> **Source:** Gate Smashers — IPv4 Options and Padding Field
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Where Options & Padding Fit in IPv4 Header](#1-where-options--padding-fit-in-ipv4-header)
2. [Options Field — Overview](#2-options-field--overview)
3. [Option 1 — Record Route](#3-option-1--record-route)
4. [Option 2 — Source Routing](#4-option-2--source-routing)
5. [Strict vs Loose Source Routing](#5-strict-vs-loose-source-routing)
6. [Padding Field](#6-padding-field)
7. [IPv4 Header Size — Min, Max & the Options Connection](#7-ipv4-header-size--min-max--the-options-connection)
8. [Options in IPv4 vs IPv6](#8-options-in-ipv4-vs-ipv6)
9. [All IPv4 Header Fields — Side-by-Side](#9-all-ipv4-header-fields--side-by-side)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. Where Options & Padding Fit in IPv4 Header

### IPv4 Header Structure

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
├───────┬───────┬───────────────────────┬───────────────────────────┤
│  Ver  │  IHL  │    Type of Service    │       Total Length        │ ← Row 1
├───────┴───────┴───────────────────────┴───────────────────────────┤
│         Identification                │ Flags │  Fragment Offset  │ ← Row 2
├───────────────────────────────────────┴───────┴───────────────────┤
│  Time to Live │    Protocol           │     Header Checksum       │ ← Row 3
├───────────────────────────────────────────────────────────────────┤
│                       Source Address                              │ ← Row 4
├───────────────────────────────────────────────────────────────────┤
│                     Destination Address                           │ ← Row 5
├───────────────────────────────────────────────────────────────────┤
│                  Options (0 to 40 bytes)          │   Padding     │ ← Row 6+
└───────────────────────────────────────────────────┴───────────────┘
↑ Fixed 20 bytes (rows 1–5) ─────────────────────────────────────── ↑
                                         Variable 0–40 bytes (row 6) ↑
```

### Key Point

```
IPv4 Header:
  Minimum size: 20 bytes  ← Options = 0 bytes (not used)
  Maximum size: 60 bytes  ← Options = 40 bytes (fully used)

"Options" is the ONLY variable-length part of the IPv4 header.
All other fields are fixed-size.
```

---

## 2. Options Field — Overview

### What are Options?

```
Options are OPTIONAL (not required in every packet).
Some packets include them, others do not.

Size range:  0 to 40 bytes
  0 bytes  → no options used → header = 20 bytes (minimum)
  40 bytes → all options used → header = 60 bytes (maximum)

Why variable?
  Not every packet needs routing control, diagnostics, or recording.
  Including optional features only when needed saves header space
  in the vast majority of everyday packets.
```

### Main Options Available in IPv4

| Option Name       | Purpose                                          |
| ----------------- | ------------------------------------------------ |
| Record Route      | Record IP addresses of all routers on the path   |
| Source Routing    | Source defines the exact/partial path in advance |
| Timestamp         | Record time at each router hop                   |
| Security          | Military/classified security level tagging       |
| Stream Identifier | Used in streaming applications (legacy)          |

> **Exam focus:** Record Route and Source Routing are the two most tested options.

---

## 3. Option 1 — Record Route

### What It Does

```
As a packet travels from Source → Routers → Destination,
each router along the path APPENDS its own IP address
into the Options field of the packet.

Source S ──► R1 ──► R2 ──► R3 ──► Destination D

After R1: Options field contains [IP of R1]
After R2: Options field contains [IP of R1, IP of R2]
After R3: Options field contains [IP of R1, IP of R2, IP of R3]

When packet reaches D: destination knows the FULL PATH taken.
```

### How Many Routers Can Be Recorded?

```
Options field max size = 40 bytes
IPv4 address size      =  4 bytes (32 bits)

Maximum routers recordable = 40 ÷ 4 = 10 routers

BUT — the option also needs a few overhead bytes
(option type, length, pointer fields)
So in practice: up to 9 router addresses can be stored.

Exam answer: 9 routers (safe answer) or 10 (if question ignores overhead)
```

### Visual — Record Route in Action

```
Packet leaves S:
  Options = [ _ _ _ _ _ _ _ _ _ _ ] ← 10 empty slots

Passes through R1 (IP: 10.0.0.1):
  Options = [ 10.0.0.1 | _ _ _ _ _ _ _ _ _ ] ← R1 writes itself

Passes through R2 (IP: 10.0.0.2):
  Options = [ 10.0.0.1 | 10.0.0.2 | _ _ _ _ _ _ _ _ ] ← R2 appends

Passes through R3 (IP: 10.0.0.3):
  Options = [ 10.0.0.1 | 10.0.0.2 | 10.0.0.3 | _ _ _ _ _ _ _ ] ← R3 appends

Arrives at D:
  Destination reads the full path taken: R1 → R2 → R3
```

### Why Record Route is Rarely Used Today

```
Privacy / Security concern:
  ISPs and network operators do NOT want their internal
  router IP addresses exposed to arbitrary internet traffic.

  Recording router IPs reveals:
  → Internal network topology
  → Router IP addresses (attack targets)
  → Routing policies (competitive intelligence)

Result:
  Most ISPs and enterprise networks BLOCK or IGNORE the Record Route option.
  IPv6 completely REMOVED this option from the standard.
```

---

## 4. Option 2 — Source Routing

### What It Does

```
Instead of letting routers decide the path (normal routing),
the SOURCE defines the path the packet must take — in ADVANCE.

Normal routing:
  S sends packet → Each router independently decides next hop
  → Packet follows "best" route at each step

Source routing:
  S says: "My packet MUST go S → R1 → R2 → R3 → D"
  → This path is embedded in the packet's Options field
  → Intermediate routers follow the SOURCE's instructions
```

### Real-World Analogy

> Source routing is like booking a train journey and specifying
> "I want to travel Delhi → Agra → Bhopal → Mumbai" —
> you define every stop in advance rather than letting
> the railway company route you however they want.

### Who Can Use Source Routing?

```
NOT available to regular users.
Only NETWORK ADMINISTRATORS can configure source routing
(through network tools, routing software, or specialized devices).

Why restricted?
  → If any user could dictate routing paths,
    they could bypass firewalls and security zones
  → Massive security risk (see Security Context section)
```

---

## 5. Strict vs Loose Source Routing

### Strict Source Routing (SSR)

```
Definition:
  The SOURCE defines THE COMPLETE PATH — every single router
  from source to destination is listed explicitly.
  The packet MUST follow this exact sequence. No deviation allowed.

Example:
  S specifies: S → R1 → R2 → R3 → D
  R4, R5, R6 exist on the network but packet CANNOT pass through them.
  If R2 is down → packet is DROPPED (cannot take alternate route).

Diagram:
  [S] → [R1] → [R2] → [R3] → [D]
          ↑      ↑      ↑
         must   must   must
         pass   pass   pass

Exam keyword: COMPLETE / EXACT path pre-defined
```

### Loose Source Routing (LSR)

```
Definition:
  The SOURCE defines SOME WAYPOINTS (key routers it must pass through)
  but does NOT define the complete path.
  Between specified waypoints, routers can choose their own path.

Example:
  S specifies: must pass through R1 and R3
  Between S and R1: any path allowed
  Between R1 and R3: any path allowed (may go via R2, R4, or R5)
  After R3: any path to D allowed

Diagram:
  [S] →→→→→→ [R1] →→→→→→→→→→ [R3] →→→→→ [D]
        any          any route          any
        route       (via R2/R4/R5)     route

Exam keyword: PARTIAL path pre-defined, routers fill in the gaps
```

### Strict vs Loose Comparison

| Property           | Strict Source Routing (SSR)  | Loose Source Routing (LSR)     |
| ------------------ | ---------------------------- | ------------------------------ |
| Path specification | Complete — every hop defined | Partial — only waypoints given |
| Flexibility        | None — exact path mandatory  | Flexible between waypoints     |
| Failure behavior   | Drop if any hop unreachable  | Can reroute between waypoints  |
| Control level      | Maximum control              | Moderate control               |
| Use case           | Testing specific paths       | Traffic engineering, VPN       |
| Exam keyword       | EXACT / TOTAL path           | PARTIAL / SOME routers listed  |

---

## 6. Padding Field

### What is Padding?

```
The IPv4 header size must ALWAYS be a multiple of 4 bytes.
Why? Because the IHL (Internet Header Length) field counts
header length in units of 4-byte words.

IHL = header size ÷ 4
So header must be exactly divisible by 4.

Problem:
  Options can be ANY size from 0 to 40 bytes.
  20 (fixed) + options bytes may NOT always be a multiple of 4.

Solution: PADDING
  Add extra zero bytes (0x00) until the total header size
  becomes a multiple of 4.
```

### Padding Example

```
Fixed header:     20 bytes
Options used:      3 bytes
Total so far:     23 bytes

Is 23 a multiple of 4?  23 ÷ 4 = 5.75  → NO ✗

Add padding:       1 byte of zeros (0x00)
New total:        24 bytes

Is 24 a multiple of 4?  24 ÷ 4 = 6  → YES ✓

IHL = 24 ÷ 4 = 6  (IHL field stores value 6)
```

### Padding Rule

```
Padding bytes needed = (4 - (options_size % 4)) % 4

Examples:
  Options = 0 bytes → 20 + 0 = 20 → multiple of 4 ✓ → padding = 0
  Options = 1 byte  → 20 + 1 = 21 → not multiple → padding = 3
  Options = 2 bytes → 20 + 2 = 22 → not multiple → padding = 2
  Options = 3 bytes → 20 + 3 = 23 → not multiple → padding = 1
  Options = 4 bytes → 20 + 4 = 24 → multiple of 4 ✓ → padding = 0
  Options = 8 bytes → 20 + 8 = 28 → multiple of 4 ✓ → padding = 0
  Options = 40 bytes→ 20+40 = 60  → multiple of 4 ✓ → padding = 0

Padding is all ZEROS (0x00) — receiver ignores it.
```

### Visual Example

```
Header with 3 bytes of options (not multiple of 4):

├──────────────────── 20 bytes fixed ──────────────────────┤
├── 3 bytes options ──┤─ 1 byte padding ─┤
                       (= 0x00)

Total = 20 + 3 + 1 = 24 bytes = 6 × 4 ✓
IHL = 6
```

---

## 7. IPv4 Header Size — Min, Max & the Options Connection

### Complete Size Reference

```
Component              Min        Max
─────────────────────────────────────────
Fixed fields           20 bytes   20 bytes  (always present)
Options field           0 bytes   40 bytes  (variable)
Padding                 0 bytes    3 bytes  (to align to 4-byte boundary)
─────────────────────────────────────────
Total header            20 bytes   60 bytes

IHL field range:
  IHL = 5  → header = 5 × 4 = 20 bytes (minimum, no options)
  IHL = 15 → header = 15 × 4 = 60 bytes (maximum, full options)
```

### IHL Field Connection

```
IHL (Internet Header Length) is a 4-bit field in the IPv4 header.
It stores the header length in 32-bit (4-byte) words.

IHL = 5  → 5 × 4 = 20 bytes  ← minimum (no options)
IHL = 6  → 6 × 4 = 24 bytes
IHL = 7  → 7 × 4 = 28 bytes
...
IHL = 15 → 15 × 4 = 60 bytes ← maximum (40 bytes of options)

How to find options size from IHL:
  Options size = (IHL × 4) - 20

Example: IHL = 8
  Header size = 8 × 4 = 32 bytes
  Options size = 32 - 20 = 12 bytes
```

---

## 8. Options in IPv4 vs IPv6

### Why IPv6 Removed Options

```
IPv4 Options → Variable header size (20–60 bytes)
               Routers must parse options to find payload
               Slows down router processing
               Security risks (source routing abuse)
               Complexity in implementation

IPv6 Solution → Options COMPLETELY REMOVED from main header
                IPv6 uses "Extension Headers" instead
                Main IPv6 header is ALWAYS 40 bytes (fixed)
                Extension headers added between IPv6 header and payload
                Routers skip extension headers they don't understand

Specifically removed from IPv6:
  ✗ Record Route   → security concern (topology exposure)
  ✗ Source Routing → severe security vulnerability
                     (can bypass firewalls — see Security section)
  ✗ Timestamp      → replaced by better mechanisms

IPv6 equivalent of options:
  Hop-by-Hop Options Header
  Routing Header (limited, type 0 deprecated)
  Fragment Header
  Destination Options Header
```

---

## 9. All IPv4 Header Fields — Side-by-Side

| Field               | Size           | Purpose                                     |
| ------------------- | -------------- | ------------------------------------------- |
| Version             | 4 bits         | IPv4 = 4                                    |
| IHL                 | 4 bits         | Header length in 4-byte words (5–15)        |
| Type of Service     | 8 bits         | QoS / priority (DSCP)                       |
| Total Length        | 16 bits        | Entire packet size (header + data)          |
| Identification      | 16 bits        | Groups fragments of same datagram           |
| Flags               | 3 bits         | DF, MF bits for fragmentation control       |
| Fragment Offset     | 13 bits        | Position of fragment data (in 8-byte units) |
| Time to Live (TTL)  | 8 bits         | Max hops; decremented by each router        |
| Protocol            | 8 bits         | Upper layer: TCP=6, UDP=17, ICMP=1          |
| Header Checksum     | 16 bits        | Error detection for header only             |
| Source Address      | 32 bits        | Sender's IP address                         |
| Destination Address | 32 bits        | Recipient's IP address                      |
| **Options**         | **0–40 bytes** | **Record Route, Source Routing, etc.**      |
| **Padding**         | **0–3 bytes**  | **Align header to 4-byte boundary**         |

---

## 10. 🔴 Security Context

### Source Routing as an Attack Vector

```
Source Routing was designed for network troubleshooting
and traffic engineering. But it became one of the most
dangerous features in IPv4 history.

Why dangerous?
  Normally a packet goes:
    External Attacker → Internet → Firewall → Internal Server
    (firewall BLOCKS malicious traffic)

  With Source Routing enabled:
    External Attacker → Internet → Internal Server → Firewall
    (attacker ROUTES packet THROUGH the target, bypassing firewall!)

  The attacker specifies the route in reverse — forcing the
  return traffic to pass through the attacker's machine,
  enabling man-in-the-middle attacks.
```

### Attack — Source Routing Firewall Bypass

```
Normal traffic flow:
  [Attacker] ──► [Firewall] ──► [Internal Server]
                    BLOCKED ✗

Source routing attack:
  Attacker sends packet with source route:
    Go to [Internal Server] first, then route response via [Attacker]

  [Attacker] ──► [Internal Server] ──► [Attacker]
  Firewall sees outbound reply traffic → may ALLOW it ✓

  Attacker intercepts the response and gains data
  from behind the firewall without being blocked!

This is why:
  RFC 1812: Routers SHOULD NOT process source-routed packets by default
  Most modern routers and firewalls DROP source-routed packets
  IPv6 Routing Header Type 0 (equivalent) was DEPRECATED (RFC 5095)
```

### Attack — Record Route for Network Reconnaissance

```
If Record Route is accepted by routers:
  Attacker sends a probe packet with Record Route option enabled
  → Collects IP addresses of ALL routers on the path
  → Builds a map of internal network topology

Information gained:
  → How many hops to target
  → Private IP addresses of internal routers
  → Which ISPs and transit networks are used
  → Potential attack targets (routers with known vulnerabilities)

Defense:
  Most ISPs and enterprises DROP packets with Record Route option
  Configure: no ip record-route (Cisco IOS)
  iptables: -m ipv4options --ssrr -j DROP  (source route)
            -m ipv4options --lsrr -j DROP  (loose source route)
            -m ipv4options --rr   -j DROP  (record route)
```

### Attack — Loose Source Routing for Traffic Interception

```
Step 1: Attacker learns victim uses path: A → R1 → R2 → B

Step 2: Attacker crafts packet with LSR:
        "Must pass through attacker's machine, then reach B"

Step 3: Packet travels: A → Attacker → R2 → B
        Attacker intercepts and reads all traffic!

This is a CLASSIC man-in-the-middle attack using IPv4 options.

Real-world impact:
  Used in BGP session hijacking
  Used to intercept VoIP calls on misconfigured networks
  Exploited in several documented corporate espionage cases
```

### Attack — IP Options Denial of Service

```
Processing IP options requires EXTRA CPU TIME at each router.
Options force the router out of the fast-path (hardware switching)
into slow-path (software/CPU processing).

Attack:
  Flood router with packets containing unusual IP options
  → Each packet requires special CPU processing
  → Router CPU spikes to 100% → legitimate traffic dropped
  → Effective DoS against the router itself

Affected: Cisco, Juniper routers (historical vulnerabilities)
CVE examples:
  CVE-2007-0480: Cisco IOS DoS via crafted IP options
  CVE-2010-0564: HP ProCurve DoS via IP options packets

Defense:
  Drop all packets with IP options at the border
  iptables -m ipv4options --any-opt -j DROP
  Cisco: ip options drop
```

### Detecting IP Options Abuse in Your Lab

```bash
# Detect packets with IP options using Wireshark
# Filter: ip.options   → shows all packets with any IP option set

# Specific option filters:
# ip.opt.type == 7    → Record Route option
# ip.opt.type == 131  → Loose Source Route
# ip.opt.type == 137  → Strict Source Route

# Using tcpdump:
sudo tcpdump -i eth0 'ip[0] & 0x0f > 5'
# ip[0] = first byte of IP header = version (4 bits) + IHL (4 bits)
# & 0x0f = mask to get IHL only
# > 5 means IHL > 5 = header > 20 bytes = OPTIONS ARE PRESENT

# Alert on source-routed packets:
sudo tcpdump -i eth0 'ip[0] & 0x0f > 5' -w options_detected.pcap
```

### Block IP Options on Your Systems

```bash
# Block all packets with IP options at the firewall
# (good security practice — most legitimate traffic doesn't use options)

# iptables (Linux)
sudo iptables -A INPUT  -m ipv4options --any-opt -j DROP
sudo iptables -A FORWARD -m ipv4options --any-opt -j DROP

# Specifically block source routing (most dangerous)
sudo iptables -A INPUT  -m ipv4options --ssrr -j DROP  # strict source route
sudo iptables -A INPUT  -m ipv4options --lsrr -j DROP  # loose source route

# Disable source routing at kernel level (Linux)
# Prevents your system from FORWARDING source-routed packets
sudo sysctl -w net.ipv4.conf.all.accept_source_route=0
sudo sysctl -w net.ipv4.conf.default.accept_source_route=0

# Make permanent:
echo "net.ipv4.conf.all.accept_source_route=0" | sudo tee -a /etc/sysctl.conf
echo "net.ipv4.conf.default.accept_source_route=0" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

echo "Source routing disabled. Verify:"
sysctl net.ipv4.conf.all.accept_source_route
# Expected output: net.ipv4.conf.all.accept_source_route = 0
```

---

## 11. 🧪 Practical Labs

### Lab 1 — IPv4 Options Parser (Python)

```python
# Save as ipv4_options_parser.py — run: python3 ipv4_options_parser.py
# Parses and explains common IPv4 option type codes

OPTION_TYPES = {
    0:   ("End of Options List",        "Marks end of options"),
    1:   ("No Operation (NOP)",          "Padding / alignment between options"),
    7:   ("Record Route (RR)",           "Each router records its IP address"),
    68:  ("Internet Timestamp",          "Each router records timestamp"),
    130: ("Security (DoD)",              "Military security classification"),
    131: ("Loose Source Route (LSR)",    "Source specifies some waypoints"),
    136: ("Stream ID",                   "Legacy streaming identifier"),
    137: ("Strict Source Route (SSR)",   "Source specifies complete path"),
    148: ("Router Alert",               "Routers must examine this packet"),
}

def parse_ihl_and_options(ihl_value: int):
    """Given IHL value, calculate header and options sizes"""
    header_size  = ihl_value * 4
    options_size = header_size - 20
    padding_needed = (4 - (options_size % 4)) % 4 if options_size % 4 != 0 else 0

    print(f"\n{'='*55}")
    print(f"  IHL value:         {ihl_value}")
    print(f"  Header size:       {ihl_value} × 4 = {header_size} bytes")
    print(f"  Fixed fields:      20 bytes")
    print(f"  Options size:      {header_size} - 20 = {options_size} bytes")
    print(f"  Max routers (RR):  {options_size // 4} addresses "
          f"({'~' + str(options_size//4 - 1) + ' usable' if options_size >= 4 else 'N/A'})")
    print(f"  Is multiple of 4:  {'✓ YES' if header_size % 4 == 0 else '✗ NO'}")

def explain_option(type_code: int):
    name, desc = OPTION_TYPES.get(type_code, ("Unknown", "Not a standard option"))
    print(f"\n  Option type {type_code} (0x{type_code:02X}):")
    print(f"    Name:    {name}")
    print(f"    Purpose: {desc}")

    security_note = ""
    if type_code == 131:
        security_note = "⚠️  SECURITY RISK — can bypass firewalls (firewall bypass attack)"
    elif type_code == 137:
        security_note = "🚨 HIGH RISK — full path control, enables MITM attacks"
    elif type_code == 7:
        security_note = "⚠️  PRIVACY RISK — leaks internal router topology"
    if security_note:
        print(f"    Security: {security_note}")

def padding_calc(options_bytes: int):
    total = 20 + options_bytes
    pad   = (4 - (total % 4)) % 4
    print(f"\n  Options = {options_bytes} bytes:")
    print(f"    20 (fixed) + {options_bytes} (options) = {total} bytes")
    if pad == 0:
        print(f"    {total} is multiple of 4 ✓ → padding = 0 bytes")
    else:
        print(f"    {total} is NOT multiple of 4 → add {pad} byte(s) of padding")
        print(f"    {total} + {pad} = {total + pad} bytes = {(total+pad)//4} × 4 ✓")
        print(f"    IHL = {(total+pad)//4}")

# Run analysis
print("IHL RANGE ANALYSIS")
for ihl in [5, 6, 8, 10, 15]:
    parse_ihl_and_options(ihl)

print("\n\nOPTION TYPE CODES")
for code in [7, 131, 137, 68]:
    explain_option(code)

print("\n\nPADDING CALCULATIONS")
for opt_size in [0, 1, 2, 3, 4, 8, 12, 40]:
    padding_calc(opt_size)
```

### Lab 2 — Detect IP Options with tcpdump + Wireshark

```bash
# Packets with IP options have IHL > 5
# First byte of IP header: top 4 bits = version, bottom 4 bits = IHL
# We check if lower 4 bits > 5

# Real-time detection:
sudo tcpdump -i eth0 'ip[0] & 0x0f > 5' -n -v
# -n = no DNS resolution
# -v = verbose (shows option details)

# Capture to file for analysis:
sudo tcpdump -i eth0 'ip[0] & 0x0f > 5' -w ip_options_capture.pcap

# Open in Wireshark and use these filters:
echo ""
echo "Wireshark filters for IP options:"
echo "  ip.options                    → any packet with options"
echo "  ip.opt.type == 7              → Record Route"
echo "  ip.opt.type == 131            → Loose Source Route (BLOCK THIS!)"
echo "  ip.opt.type == 137            → Strict Source Route (BLOCK THIS!)"
echo "  ip.hdr_len > 20              → header larger than 20 bytes = options present"
echo ""
echo "In Wireshark, expand: Internet Protocol → Options"
echo "You will see each option type, length, and data parsed automatically."
```

### Lab 3 — Craft a Packet with IP Options Using Scapy

```python
# Save as craft_ip_options.py
# Run: sudo python3 craft_ip_options.py
# Educational demo — send to your own Metasploitable2 ONLY

from scapy.all import IP, ICMP, IPOption_RR, IPOption_LSRR, send, sr1

target = "192.168.56.101"  # Your Metasploitable2

print("=" * 55)
print("DEMO 1: Packet WITH Record Route option")
print("=" * 55)

# Build ICMP packet with Record Route option
pkt_rr = IP(dst=target, options=IPOption_RR()) / ICMP()

print(f"Normal packet header size:  20 bytes")
print(f"With Record Route option:   {len(pkt_rr[IP]) - len(pkt_rr[IP].payload)} bytes")
print(f"IHL value:                  {pkt_rr[IP].ihl}")
print()
pkt_rr[IP].show()

print()
print("=" * 55)
print("DEMO 2: Packet WITHOUT options (normal)")
print("=" * 55)

pkt_normal = IP(dst=target) / ICMP()
print(f"Normal packet IHL: {pkt_normal[IP].ihl} (= 5 × 4 = 20 bytes)")
print(f"Has options:       {'Yes' if pkt_normal[IP].ihl > 5 else 'No'}")
print()

# Compare IHL values
print("COMPARISON:")
print(f"  Without options: IHL = {pkt_normal[IP].ihl} → {pkt_normal[IP].ihl * 4} bytes")
print(f"  With RR option:  IHL = {pkt_rr[IP].ihl} → {pkt_rr[IP].ihl * 4} bytes")
print(f"  Options size:    {(pkt_rr[IP].ihl - pkt_normal[IP].ihl) * 4} bytes")

# To actually send (uncomment):
# print("\nSending packet with Record Route to Metasploitable2...")
# response = sr1(pkt_rr, timeout=2, verbose=False)
# if response and IP in response and response[IP].options:
#     print(f"Response options: {response[IP].options}")
# else:
#     print("No options in response (target likely ignores RR)")
```

### Lab 4 — Verify Source Routing is Disabled

```bash
# Check current status of source routing on your system
echo "=== Source Routing Status ==="
echo ""

for iface in $(ip link show | grep "^[0-9]" | awk -F: '{print $2}' | tr -d ' '); do
    val=$(sysctl -n net.ipv4.conf.${iface}.accept_source_route 2>/dev/null)
    if [ ! -z "$val" ]; then
        if [ "$val" = "0" ]; then
            status="✓ DISABLED (secure)"
        else
            status="✗ ENABLED (vulnerable!)"
        fi
        echo "  $iface: accept_source_route = $val → $status"
    fi
done

echo ""
echo "  all:     $(sysctl -n net.ipv4.conf.all.accept_source_route)"
echo "  default: $(sysctl -n net.ipv4.conf.default.accept_source_route)"
echo ""

# Disable if enabled:
echo "Disabling source routing on all interfaces..."
sudo sysctl -w net.ipv4.conf.all.accept_source_route=0
sudo sysctl -w net.ipv4.conf.default.accept_source_route=0
echo "Done. Source routing now disabled."
echo ""

# Add iptables rule to drop source-routed packets from network
echo "Adding iptables rules to DROP source-routed packets..."
sudo iptables -A INPUT -m ipv4options --ssrr -j DROP
sudo iptables -A INPUT -m ipv4options --lsrr -j DROP
sudo iptables -L INPUT | grep -i "source\|ssrr\|lsrr"
echo "Rules applied."
```

### Lab 5 — Padding Calculator Tool

```bash
python3 - << 'EOF'
def ipv4_padding_needed(options_bytes: int) -> dict:
    """Calculate padding needed for given options size"""
    total_without_padding = 20 + options_bytes
    remainder = total_without_padding % 4
    padding   = (4 - remainder) % 4
    total     = total_without_padding + padding
    ihl       = total // 4

    return {
        'options_bytes':  options_bytes,
        'raw_total':      total_without_padding,
        'padding_needed': padding,
        'final_total':    total,
        'ihl_value':      ihl,
        'valid':          total % 4 == 0,
    }

print(f"{'Options':>8} {'Raw Total':>10} {'Padding':>8} {'Final':>7} {'IHL':>5} {'Valid':>6}")
print("-" * 50)

for opt in range(0, 44, 1):
    r = ipv4_padding_needed(opt)
    if r['padding_needed'] > 0 or opt in [0, 4, 8, 12, 20, 36, 40]:
        mark = "← boundary" if opt in [0, 40] else ("← padding!" if r['padding_needed'] > 0 else "")
        print(f"{r['options_bytes']:>8} {r['raw_total']:>10} {r['padding_needed']:>8} "
              f"{r['final_total']:>7} {r['ihl_value']:>5} {'✓' if r['valid'] else '✗':>6}  {mark}")

print()
print("Key insight: Options of 0, 4, 8, 12, 16... (multiples of 4) need NO padding.")
print("Other sizes need 1, 2, or 3 bytes of padding to reach next multiple of 4.")
EOF
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║         IPv4 OPTIONS & PADDING — EXAM CHEAT SHEET                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  OPTIONS FIELD BASICS                                                ║
║  Size:     0 to 40 bytes (variable — only optional field)           ║
║  Header:   20B (no options) to 60B (full 40B options)              ║
║  Used by:  Some packets — not all packets include options           ║
╠══════════════════════════════════════════════════════════════════════╣
║  OPTION 1 — RECORD ROUTE                                            ║
║  Each router on the path appends its OWN IP to the packet           ║
║  Max IP addresses stored = 40 ÷ 4 = 10 (exam: 9 if overhead asked) ║
║  IPv4 address = 4 bytes (32 bits)                                   ║
║  Disabled by most ISPs — exposes internal topology (security risk)  ║
║  REMOVED in IPv6                                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  OPTION 2 — SOURCE ROUTING                                          ║
║  Source defines the path in advance (not routers)                   ║
║  Only network admins can use it (not regular users)                 ║
║                                                                      ║
║  Strict (SSR): COMPLETE path — every hop listed — no deviation      ║
║  Loose (LSR):  PARTIAL path — some waypoints — gaps filled by router║
╠══════════════════════════════════════════════════════════════════════╣
║  STRICT vs LOOSE — ONE-LINE MEMORY                                   ║
║  Strict = TOTAL route locked in  (exam keyword: complete/exact)     ║
║  Loose  = SOME waypoints only    (exam keyword: partial/some)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  PADDING                                                             ║
║  Purpose: Make total header size a multiple of 4 bytes              ║
║  Content: All zero bits (0x00) — receiver ignores them              ║
║  Size:    0 to 3 bytes (only as much as needed)                     ║
║                                                                      ║
║  Example: 20 + 3 options = 23 bytes → add 1 byte pad → 24 ✓       ║
║  Formula: padding = (4 - (options_size % 4)) % 4                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  IHL CONNECTION                                                      ║
║  IHL = header size ÷ 4                                              ║
║  IHL = 5  → header = 20B (no options)                              ║
║  IHL = 15 → header = 60B (max options)                             ║
║  Options size = (IHL × 4) - 20                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  IPv4 vs IPv6 OPTIONS                                                ║
║  IPv4: Options in MAIN HEADER → variable size (20–60B)             ║
║  IPv6: NO OPTIONS in main header → always fixed 40B                ║
║        Uses EXTENSION HEADERS instead                               ║
║        Source Routing (Type 0) deprecated in IPv6 (RFC 5095)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY                                                            ║
║  Source routing → firewall bypass, MITM attacks                     ║
║  Record route  → network topology reconnaissance                    ║
║  Both should be BLOCKED at border routers                           ║
║  Linux defense: sysctl net.ipv4.conf.all.accept_source_route=0     ║
║  tcpdump detection: 'ip[0] & 0x0f > 5' (IHL > 5 = options present)║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK NUMBERS                                                       ║
║  IPv4 address = 4 bytes = 32 bits                                   ║
║  Max options  = 40 bytes                                            ║
║  Max routers in Record Route = 40 ÷ 4 = 10 (or ~9 with overhead)  ║
║  Header min = 20B (IHL=5) │ Header max = 60B (IHL=15)             ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] IPv4 Header — Complete field reference (Version, TTL, Protocol, Checksum)
- [ ] IPv6 Header — Fixed 40-byte header, extension headers explained
- [ ] ICMP — Error and control messages (Ping, Traceroute, Fragmentation Needed)
- [ ] IPv6 Extension Headers — How IPv6 handles what IPv4 put in Options
- [ ] Path MTU Discovery — Using DF bit and ICMP to find minimum MTU

---

_Notes compiled from: Networking Course Lecture 51 — IPv4 Options and Padding_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
