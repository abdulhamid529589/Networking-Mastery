# 🌐 IP Addressing — Need for IPv6 (Why IPv6?)

### " "Networking Course — Lecture 52

> **Source:** Gate Smashers — Need for IPv6 / Why IPv6?
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Quick Recap — Where IPv4 Stands](#1-quick-recap--where-ipv4-stands)
2. [What is IPv6? — IPng (IP Next Generation)](#2-what-is-ipv6--ipng-ip-next-generation)
3. [Reason 1 — IPv4 Address Exhaustion (Primary Reason)](#3-reason-1--ipv4-address-exhaustion-primary-reason)
   - [3.1 IPv4 Address Space](#31-ipv4-address-space)
   - [3.2 IoT — The Tipping Point](#32-iot--the-tipping-point)
   - [3.3 IPv6 Address Space — The Solution](#33-ipv6-address-space--the-solution)
4. [Reason 2 — Real-Time Data Transmission Support](#4-reason-2--real-time-data-transmission-support)
5. [Reason 3 — Authentication Support](#5-reason-3--authentication-support)
6. [Reason 4 — Encryption at the Network Layer](#6-reason-4--encryption-at-the-network-layer)
7. [Reason 5 — Better Security](#7-reason-5--better-security)
8. [Reason 6 — Fast Processing at Routers](#8-reason-6--fast-processing-at-routers)
9. [IPv4 vs IPv6 — Side-by-Side Comparison](#9-ipv4-vs-ipv6--side-by-side-comparison)
10. [IPv4 Exhaustion Timeline — How We Got Here](#10-ipv4-exhaustion-timeline--how-we-got-here)
11. [Why This Matters for Cybersecurity](#11-why-this-matters-for-cybersecurity)
12. [🧪 Practical Labs](#12--practical-labs)
13. [Solved Examples](#13-solved-examples)
14. [Exam Cheat Sheet](#14-exam-cheat-sheet)

---

## 1. Quick Recap — Where IPv4 Stands

From Lectures 40–46, IPv4 (the currently dominant IP protocol) has several well-documented structural problems:

```
Classful Addressing (Lectures 40–44):
  → Fixed class sizes → massive address wastage
  → Class D + E: ~537 million addresses permanently reserved

Problems with Classful (Lecture 44):
  → Wastage, inflexibility, maintenance, security

CIDR / Classless (Lecture 45):
  → Partially solved wastage and inflexibility

Subnetting (Lecture 46):
  → Solved maintenance and security within an assigned block

BUT: Even with all these mitigations, the fundamental IPv4 address
     space is still only 2³² ≈ 4.3 billion addresses — and that
     ceiling is now a real-world problem.
```

---

## 2. What is IPv6? — IPng (IP Next Generation)

- **IPv6** = **IP Next Generation** — also written as **IPng**
- It is the **latest version** of the Internet Protocol at the **Network Layer**
- Two IP versions exist: **IPv4** (current dominant standard) and **IPv6** (next generation)

```
IPv4 → 32-bit addressing → 2³² ≈ 4.3 billion addresses
IPv6 → 128-bit addressing → 2¹²⁸ addresses (an astronomically larger space)
```

**Design goal:** IPv6 was designed so that no successor protocol would be needed for the **next 50–60 years** — it is intended to be the definitive, long-term internet protocol for the foreseeable future.

---

## 3. Reason 1 — IPv4 Address Exhaustion (Primary Reason)

> **This is the first and biggest reason for shifting to IPv6.**

### 3.1 IPv4 Address Space

```
IPv4 address: 32 bits
Total possible addresses: 2³² = 4,294,967,296 ≈ 4.3 billion

Sounds like a lot — but consider:
  - Classful waste: Class D + E = ~537 million locked away permanently
  - Private address blocks (RFC 1918) not routable on public internet
  - Historical over-allocation to large organisations (Class A blocks)
  - Actual globally routable public IPv4 addresses: far fewer than 4.3B
```

### 3.2 IoT — The Tipping Point

Historically, internet connections meant **laptops, mobiles, PCs** — a finite and manageable number of devices. The tipping point was the rise of **IoT (Internet of Things)**:

```
Traditional internet devices:
  Laptop, Mobile, PC, Server → finite per household/office

IoT devices (all needing IP addresses):
  AC, Washing Machine, Refrigerator, Smart TV,
  Smart Watch, Smart Home Hub, Security Cameras,
  Industrial Sensors, Medical Devices, Vehicles...
```

- Every IoT device that connects to the internet **needs its own IP address**.
- As smart homes, smart cities, and industrial IoT scale globally, the number of IP-connected devices has exploded far beyond what anyone imagined when IPv4 was designed.
- Even with **CIDR** (Lecture 45) and **NAT (Network Address Translation)** providing some relief, IPv4's 4.3 billion address ceiling was definitively breached.

### 3.3 IPv6 Address Space — The Solution

```
IPv6 address: 128 bits
Total possible addresses: 2¹²⁸

The value is in QUINTILLIONS — so large it is difficult to even conceptualise:
  2¹²⁸ = 340,282,366,920,938,463,463,374,607,431,768,211,456
        ≈ 3.4 × 10³⁸

For context:
  IPv4 (4.3 billion) is like one grain of sand.
  IPv6 (2¹²⁸) is like the number of atoms in the observable universe — many times over.
```

> **Exam tip:** You don't need to memorise the exact IPv6 value — know that it is **128-bit**, produces **2¹²⁸** addresses, is described as being in **quintillions**, and is considered **sufficient for 50–60 years** at minimum.

---

## 4. Reason 2 — Real-Time Data Transmission Support

**IPv4 lacks built-in support for real-time data transmission.**
**IPv6 has dedicated support for this.**

```
Real-time data = live streaming, audio/video that must be delivered
                 IMMEDIATELY with minimal delay.

Examples:
  → Live cricket/football match streams
  → Video conferencing (Zoom, Teams, Google Meet)
  → Live news broadcasts
  → VoIP calls
  → Online gaming
```

- In live streaming, there is an acceptable slight delay, but the content must arrive **near-instantly and in order** — if the ball is bowled in a cricket match, viewers must see it within seconds, not 10 minutes later.
- **IPv4** had no specific fields or support mechanisms in the network layer to prioritise or guarantee real-time delivery of time-sensitive data.
- **IPv6** includes dedicated support for real-time transmission — it has fields and mechanisms that allow the network to prioritise packets that are time-sensitive, enabling much better quality of service for live media.

---

## 5. Reason 3 — Authentication Support

**IPv6 natively supports authentication. IPv4 does not at the network layer.**

```
Authentication = confirming that a message was sent by the claimed sender,
                 not by an impostor.

Example:
  You receive a message claiming to be from "Varun Sir"
  Authentication lets you VERIFY this is genuinely from Varun Sir
  and has not been sent by someone else impersonating him.

Mechanism (conceptual):
  Sender creates a message digest (hash) of the message → sends with message
  Receiver computes the same hash → if they match → message is authenticated ✅
  If they don't match → message was tampered with or from wrong sender ❌
```

- **IPv4:** Authentication, if needed, had to be handled by **upper layers** (Transport, Application) — the network layer itself did not provide it.
- **IPv6:** Authentication is built into the protocol at the network layer itself, through **IPsec (IP Security)**, which is **mandatory** in IPv6 (optional in IPv4). IPsec includes AH (Authentication Header) which verifies both the sender and the integrity of the message.

---

## 6. Reason 4 — Encryption at the Network Layer

**IPv6 can encrypt data at the network layer. IPv4 could not.**

```
Encryption = converting plain text into cipher text so that
             only the intended recipient can read it.

IPv4 approach:
  Encryption was handled by the APPLICATION LAYER only.
  If the application wanted to encrypt, it did so.
  If the application forgot or didn't support encryption → plain text sent.
  The network layer (IPv4) had NO encryption capability of its own.

IPv6 approach:
  If the upper layer (application) sends encrypted data → great.
  If the upper layer does NOT encrypt → IPv6 can encrypt at the NETWORK LAYER.
  Encryption is available as a network-layer capability, not just app-layer.
```

- This is again provided through **IPsec**, specifically the **ESP (Encapsulating Security Payload)** component, which encrypts packet payloads at the network layer.
- In **IPv4**, IPsec is available but **optional**. In **IPv6**, IPsec is **mandatory/built-in** — making encryption a standard feature rather than an afterthought.

---

## 7. Reason 5 — Better Security

```
Combining authentication + encryption + mandatory IPsec:

IPv6 security posture:
  → Packets can be authenticated (sender verified)
  → Packets can be encrypted (payload protected from eavesdropping)
  → IPsec is mandatory, not optional
  → More devices are being connected (IoT) → security is MORE critical, not less

Today's context:
  → E-commerce transactions require security
  → Banking, healthcare, government communications need protection
  → Smart home / IoT devices controlling physical infrastructure need security
```

- With billions more devices connecting due to IoT, the **security surface has expanded enormously**.
- IPv6's built-in security features are a direct response to this — security is designed in, rather than bolted on.

---

## 8. Reason 6 — Fast Processing at Routers

**IPv6 is designed to be processed faster by routers than IPv4.**

### IPv4 Header — Complex and Variable

```
IPv4 header: 20 to 60 bytes (VARIABLE)

Fields that make router processing slow:
  → Variable header length → router must calculate/check header length
  → Checksum field → router must compute checksum at every hop
  → Fragmentation fields → router can fragment packets mid-path
  → Many fields to inspect and process at every intermediate router
```

### IPv6 Header — Fixed and Simplified

```
IPv6 base header: FIXED at 40 bytes (always exactly 40 bytes)

Changes from IPv4:
  → Removed header length field (not needed — it's always 40 bytes)
  → Removed checksum (handled by other layers — eliminates per-hop calculation)
  → Removed mid-path fragmentation (source does fragmentation, not routers)
  → Simplified field set → LESS processing at every router
```

- By fixing the base header at a constant 40 bytes and removing computationally expensive fields (checksum, variable length), **routers can process IPv6 packets significantly faster**.
- Additional information (when needed) is placed in **extension headers** — optional, appended after the base header — so routers that don't need that information don't have to process it.

```
IPv4: Router inspects everything in the header every hop → slow
IPv6: Router processes fixed 40-byte base header → fast
      Extension headers only processed when relevant
```

---

## 9. IPv4 vs IPv6 — Side-by-Side Comparison

| Property               | IPv4                               | IPv6                               |
| ---------------------- | ---------------------------------- | ---------------------------------- |
| **Address length**     | 32 bits                            | 128 bits                           |
| **Total addresses**    | 2³² ≈ 4.3 billion                  | 2¹²⁸ ≈ 3.4 × 10³⁸ (quintillions)   |
| **Address notation**   | Dotted decimal (`192.168.1.1`)     | Hexadecimal groups (`2001:db8::1`) |
| **Header size**        | 20–60 bytes (variable)             | 40 bytes (fixed base)              |
| **Checksum in header** | Yes (computed at every hop)        | No (removed — handled elsewhere)   |
| **Fragmentation**      | At intermediate routers            | Only at source                     |
| **Real-time support**  | No built-in support                | Yes — dedicated fields/mechanisms  |
| **Authentication**     | Optional (via IPsec AH)            | Mandatory (IPsec built in)         |
| **Encryption**         | Optional / application layer only  | Network layer capable (IPsec ESP)  |
| **Security**           | Limited at network layer           | Built-in via mandatory IPsec       |
| **Router processing**  | Slower (variable header, checksum) | Faster (fixed header, no checksum) |
| **Also called**        | —                                  | IPng (IP Next Generation)          |
| **Sufficient for**     | Was sufficient; now exhausted      | Next 50–60 years minimum           |

---

## 10. IPv4 Exhaustion Timeline — How We Got Here

```
1981:   IPv4 defined (RFC 791) — 32-bit, 4.3 billion addresses
        → Seemed more than enough at the time

1980s:  Classful addressing adopted — Class D/E reserved (~537M addresses wasted)
        Large Class A blocks assigned to universities and companies

1993:   CIDR introduced (Lecture 45) — slows the exhaustion rate
        NAT introduced — allows many private hosts to share one public IP

2000s:  Internet boom — mass adoption of laptops, mobiles, servers
        → IPv4 scarcity becomes clear

2010s:  IoT explosion — smart devices, connected appliances
        → Every device needs an IP address
        → IANA allocates last IPv4 /8 blocks (2011)
        → Regional registries run out of IPv4 addresses one by one

2019:   RIPE NCC (Europe) announces exhaustion of IPv4 pool

2020s:  Accelerating IPv6 adoption — Google reports >35% of traffic over IPv6
        Dual-stack deployments common (both IPv4 and IPv6 simultaneously)
```

---

## 11. Why This Matters for Cybersecurity

- **IPv6 attack surface is new and unfamiliar:** Many organisations have deployed IPv6 but haven't secured it properly — firewalls, IDS, and security tools configured only for IPv4 may be completely blind to IPv6 traffic. Attackers can use IPv6 to **bypass IPv4-only security controls** entirely.

- **IPv6 tunnelling attacks:** Techniques like **6to4**, **Teredo**, and **ISATAP** automatically create IPv6 tunnels through IPv4 networks. If your security tools don't inspect these tunnelled packets, an attacker can **exfiltrate data or establish C2 channels** through tunnels that your firewall ignores.

- **Mandatory IPsec is a double-edged sword:** While IPv6's built-in IPsec improves security, the encryption it provides also **prevents security monitoring tools from inspecting packet payloads** — encrypted IPv6 traffic is opaque to passive inspection, which can allow malware C2 or data exfiltration to hide inside legitimate-looking encrypted streams.

- **IPv6 reconnaissance is different:** `nmap` on an IPv6 subnet is fundamentally different — the address space per subnet (`/64`) is `2⁶⁴` addresses, which is far too large to scan exhaustively with traditional host discovery. Attackers instead rely on **multicast-based discovery** (`ff02::1` = all nodes on link, `ff02::2` = all routers), **passive sniffing**, and **DHCPv6 responses** to enumerate hosts.

- **Dual-stack risks:** Most modern systems run **dual-stack** (both IPv4 and IPv6 simultaneously). An attacker who can reach a host via IPv6 may bypass IPv4-based firewall rules that a security team has carefully maintained, since the IPv6 path may be open by default if not explicitly secured.

---

## 12. 🧪 Practical Labs

### Lab 1 — Check Your IPv6 Status

```bash
# Check if your Parrot OS has an IPv6 address assigned
ip addr show eth0 | grep inet6

# Common output types:
#   fe80::...    → link-local IPv6 (always present, scope: link only)
#   2001:db8::   → global unicast IPv6 (publicly routable)

# Check if IPv6 is enabled/disabled on your system
cat /proc/sys/net/ipv6/conf/all/disable_ipv6
# 0 = IPv6 enabled | 1 = IPv6 disabled

# Check if Metasploitable2 has IPv6
ping6 -c 3 fe80::YOUR_INTERFACE_HERE%eth0  # replace with actual link-local

# Enumerate all IPv6-reachable hosts on your lab network (multicast discovery)
ping6 -c 3 ff02::1%eth0  # pings ALL nodes on the LAN — IPv6 equivalent of broadcast
```

### Lab 2 — IPv4 vs IPv6 Address Space Visualisation

```python
# Save as ipv6_scale.py
# Run: python3 ipv6_scale.py

ipv4_space = 2**32
ipv6_space = 2**128

print("="*65)
print("IPv4 vs IPv6 ADDRESS SPACE COMPARISON")
print("="*65)
print(f"\nIPv4 (32-bit):")
print(f"  Total addresses:   {ipv4_space:,}")
print(f"  ≈                  {ipv4_space/1e9:.1f} billion")

print(f"\nIPv6 (128-bit):")
print(f"  Total addresses:   {ipv6_space:,}")
print(f"  Scientific:        {ipv6_space:.2e}")

ratio = ipv6_space // ipv4_space
print(f"\nIPv6 is {ratio:,}× larger than IPv4")
print(f"  ≈ {ratio:.2e} times more addresses")

print(f"\nIf every person on Earth (8 billion) got an equal share:")
print(f"  IPv4 per person:  {ipv4_space // 8_000_000_000:,} addresses")
print(f"  IPv6 per person:  {ipv6_space // 8_000_000_000:,} addresses")

print(f"\nIoT context — estimated connected devices 2030: ~29 billion")
print(f"  IPv4 can support: {ipv4_space:,} total addresses (already exhausted)")
print(f"  IPv6 can support: {ipv6_space:,} addresses")
print(f"  IPv6 per IoT device: {ipv6_space // 29_000_000_000:,} addresses each")
```

### Lab 3 — IPv6 Header vs IPv4 Header Size Comparison

```python
# Save as header_comparison.py
# Run: python3 header_comparison.py

print("="*60)
print("IPv4 vs IPv6 HEADER COMPARISON")
print("="*60)

ipv4_fields = [
    ("Version",              4),
    ("IHL (Header Length)",  4),
    ("Type of Service",      8),
    ("Total Length",        16),
    ("Identification",      16),
    ("Flags",                3),
    ("Fragment Offset",     13),
    ("TTL",                  8),
    ("Protocol",             8),
    ("Header Checksum",     16),
    ("Source IP",           32),
    ("Destination IP",      32),
    ("Options (variable)",  "0–320"),
]

ipv6_fields = [
    ("Version",              4),
    ("Traffic Class",        8),
    ("Flow Label",          20),
    ("Payload Length",      16),
    ("Next Header",          8),
    ("Hop Limit",            8),
    ("Source IP",          128),
    ("Destination IP",     128),
]

print("\nIPv4 Header Fields:")
total_fixed = 0
for name, bits in ipv4_fields:
    if isinstance(bits, int):
        total_fixed += bits
    print(f"  {name:<30} {str(bits):<10} bits")
print(f"  Fixed size: {total_fixed} bits = {total_fixed//8} bytes")
print(f"  With options: up to 60 bytes (VARIABLE)")
print(f"  ⚠ Router must calculate header length at every hop")

print("\nIPv6 Base Header Fields:")
total_v6 = 0
for name, bits in ipv6_fields:
    total_v6 += bits
    print(f"  {name:<30} {bits:<10} bits")
print(f"  Fixed size: {total_v6} bits = {total_v6//8} bytes  (ALWAYS 40 bytes)")
print(f"  ✅ No checksum field → no per-hop checksum computation")
print(f"  ✅ No fragmentation fields → only source fragments, not routers")
print(f"  ✅ Fixed size → routers process instantly, no length calculation needed")
```

### Lab 4 — Detect IPv6 Traffic on Your Lab Network

```bash
# Listen for IPv6 traffic on your lab interface
sudo tcpdump -i eth0 -n ip6

# Common IPv6 traffic you'll see even without explicit IPv6 config:
#   fe80::... (link-local)   → neighbour discovery, router advertisements
#   ff02::1  (all-nodes)     → multicast pings
#   ff02::fb (mDNS multicast)→ service discovery

# See IPv6 neighbour discovery (equivalent of ARP in IPv6)
sudo tcpdump -i eth0 -n icmp6

# Check your IPv6 routing table
ip -6 route show

# Try IPv6 ping to the all-nodes multicast group (your own LAN)
ping6 -c 3 ff02::1%eth0
# Should show responses from all IPv6-enabled devices on your LAN
```

### Lab 5 — IPv6 Security Check — Is Your Firewall IPv6-Aware?

```bash
# Many firewalls are configured for IPv4 only — check if yours covers IPv6

echo "=== IPv4 firewall rules (iptables) ==="
sudo iptables -L -n -v --line-numbers

echo ""
echo "=== IPv6 firewall rules (ip6tables) ==="
sudo ip6tables -L -n -v --line-numbers

# If ip6tables shows no rules but iptables has rules:
# → Your firewall is IPv4-only → IPv6 traffic passes unrestricted
# → This is a common security gap in dual-stack deployments

echo ""
echo "=== Dual-stack status ==="
ip addr | grep -E "inet|inet6" | awk '{print $1, $2}'

# If both inet (IPv4) and inet6 (IPv6) addresses appear for the same interface,
# you are dual-stacked — you MUST secure both or an attacker can use
# the IPv6 path to bypass all your IPv4 iptables rules
```

---

## 13. Solved Examples

### Example 1 — Address Space Calculation

**Question:** How many more IP addresses does IPv6 provide compared to IPv4?

```
IPv4: 2³²  = 4,294,967,296         ≈ 4.3 billion
IPv6: 2¹²⁸ = 340,282,366,920,938,463,463,374,607,431,768,211,456

Ratio: 2¹²⁸ / 2³² = 2^(128-32) = 2⁹⁶

IPv6 provides 2⁹⁶ ≈ 7.9 × 10²⁸ times MORE addresses than IPv4.
```

---

### Example 2 — Identify IPv6 Advantages (Exam-Style)

**Question:** Which of the following is NOT an advantage of IPv6 over IPv4?
(a) Larger address space
(b) Fixed 40-byte base header for faster router processing
(c) Mandatory IPsec for authentication and encryption
(d) Variable header length for flexibility

```
Answer: (d) — Variable header length is a DISADVANTAGE of IPv4.
IPv6 has a FIXED 40-byte base header (not variable).
The other three (a, b, c) are genuine IPv6 advantages.
```

---

### Example 3 — Header Size

**Question:** What is the size of the fixed base header in IPv6? How does this compare to IPv4?

```
IPv6 fixed base header: 40 bytes (always, no exceptions)

IPv4 header: 20 bytes minimum, up to 60 bytes maximum (variable due to Options field)

Key difference:
  IPv4 → variable (20–60 bytes) → routers must calculate header length at each hop
  IPv6 → fixed (40 bytes always) → no length calculation needed → faster processing

Note: Although IPv6's 40-byte base header is larger than IPv4's minimum 20 bytes,
the REMOVAL of checksum and fragmentation processing makes it FASTER to process
at each router despite the larger fixed size.
```

---

## 14. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              NEED FOR IPv6 — EXAM CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  IPv6 IDENTITY                                                         ║
║  IPv6 = IPng = IP Next Generation                                    ║
║  128-bit addressing → 2¹²⁸ addresses (quintillions)                 ║
║  Designed to last 50–60 years without needing a successor            ║
╠══════════════════════════════════════════════════════════════════════╣
║  6 REASONS FOR IPv6 (MEMORISE ALL)                                    ║
║  ─────────────────────────────────────────────────────────          ║
║  1. ADDRESS EXHAUSTION (BIGGEST REASON)                              ║
║     IPv4: 2³² ≈ 4.3 billion — exhausted due to IoT explosion        ║
║     IPv6: 2¹²⁸ ≈ quintillions — sufficient for decades             ║
║     IoT: AC, fridge, washing machine, smart watch all need IPs      ║
║                                                                        ║
║  2. REAL-TIME DATA TRANSMISSION SUPPORT                               ║
║     IPv4: no built-in real-time support                              ║
║     IPv6: dedicated support for live streaming, VoIP, gaming         ║
║                                                                        ║
║  3. AUTHENTICATION SUPPORT                                             ║
║     IPv4: optional, via IPsec (not mandatory)                        ║
║     IPv6: mandatory IPsec — AH (Authentication Header) verifies     ║
║            sender identity and message integrity                      ║
║                                                                        ║
║  4. ENCRYPTION AT NETWORK LAYER                                       ║
║     IPv4: encryption = application layer only                        ║
║     IPv6: network layer encryption via IPsec ESP (mandatory)         ║
║                                                                        ║
║  5. BETTER SECURITY                                                     ║
║     IPv4: limited network-layer security                             ║
║     IPv6: auth + encryption built in → major concern given IoT scale ║
║                                                                        ║
║  6. FAST ROUTER PROCESSING                                             ║
║     IPv4 header: variable (20–60 bytes), has checksum, fragmentation ║
║     IPv6 base header: FIXED 40 bytes, NO checksum, NO router frag   ║
║     → Routers process IPv6 packets much faster                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY NUMBERS                                                            ║
║  IPv4: 32-bit → 2³² ≈ 4.3 billion addresses                        ║
║  IPv6: 128-bit → 2¹²⁸ ≈ quintillions                               ║
║  IPv6 base header: 40 bytes (FIXED)                                 ║
║  IPv4 header: 20–60 bytes (VARIABLE)                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  IPv4 vs IPv6 QUICK COMPARE                                            ║
║  Address bits:   32         vs  128                                  ║
║  Header size:    20–60B var vs  40B fixed                            ║
║  Checksum:       Yes        vs  No (removed for speed)               ║
║  Fragmentation:  Routers    vs  Source only                          ║
║  IPsec:          Optional   vs  Mandatory                            ║
║  Auth/Encrypt:   App layer  vs  Network layer (built-in)             ║
╠══════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICK FOR 6 REASONS:                                           ║
║  "A REST F" → Address space, Real-time, Encryption,                 ║
║               Security, Authentication (IPsec), Fast processing       ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] IPv6 Header Format — all fields in detail (40-byte fixed base header)
- [ ] IPv6 Address Types — unicast, multicast, anycast (no broadcast in IPv6)
- [ ] IPv6 Address Notation — colon-hex format, zero compression rules
- [ ] ICMPv6 — replaces ARP (Neighbour Discovery Protocol, NDP)
- [ ] IPv6 Transition Mechanisms — dual-stack, tunnelling (6to4, Teredo), translation

---

_Notes compiled from: Networking Course Lecture 52 — Need for IPv6_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
