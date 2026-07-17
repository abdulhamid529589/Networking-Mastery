# 🌐 IPv6 Header — Structure, Fields & Extension Headers

### Cybersecurity Student Notes | Networking Course — Lecture 53

> **Source:** Gate Smashers — IPv6 Headers
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Quick Recap — Why IPv6?](#1-quick-recap--why-ipv6)
2. [IPv6 Header — Overview & Philosophy](#2-ipv6-header--overview--philosophy)
3. [Base Header — Fixed 40 Bytes](#3-base-header--fixed-40-bytes)
   - [3.1 Version (4 bits)](#31-version-4-bits)
   - [3.2 Traffic Class / Priority (8 bits)](#32-traffic-class--priority-8-bits)
   - [3.3 Flow Label (20 bits)](#33-flow-label-20-bits)
   - [3.4 Payload Length (16 bits)](#34-payload-length-16-bits)
   - [3.5 Next Header (8 bits)](#35-next-header-8-bits)
   - [3.6 Hop Limit (8 bits)](#36-hop-limit-8-bits)
   - [3.7 Source Address (128 bits)](#37-source-address-128-bits)
   - [3.8 Destination Address (128 bits)](#38-destination-address-128-bits)
4. [Extension Headers — Optional Extra Functionality](#4-extension-headers--optional-extra-functionality)
   - [4.1 Routing Header (Type 43)](#41-routing-header-type-43)
   - [4.2 Hop-by-Hop Options Header (Type 0)](#42-hop-by-hop-options-header-type-0)
   - [4.3 Fragment Header (Type 44)](#43-fragment-header-type-44)
   - [4.4 Authentication Header — AH (Type 51)](#44-authentication-header--ah-type-51)
   - [4.5 Destination Options Header (Type 60)](#45-destination-options-header-type-60)
   - [4.6 Encapsulating Security Payload — ESP (Type 50)](#46-encapsulating-security-payload--esp-type-50)
5. [Extension Header Chaining — How Next Header Works](#5-extension-header-chaining--how-next-header-works)
6. [IPv4 Header vs IPv6 Header — Full Comparison](#6-ipv4-header-vs-ipv6-header--full-comparison)
7. [Datagram Service vs Virtual Circuit — Flow Label Context](#7-datagram-service-vs-virtual-circuit--flow-label-context)
8. [Why This Matters for Cybersecurity](#8-why-this-matters-for-cybersecurity)
9. [🧪 Practical Labs](#9--practical-labs)
10. [Solved Examples](#10-solved-examples)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. Quick Recap — Why IPv6?

From the previous lecture (Need for IPv6):

```
IPv4: 32-bit → 2³² ≈ 4.3 billion addresses → EXHAUSTED due to IoT
IPv6: 128-bit → 2¹²⁸ addresses → sufficient for decades

6 reasons for IPv6:
  1. Address exhaustion (primary reason)
  2. Real-time data transmission support
  3. Authentication support (mandatory IPsec)
  4. Encryption at network layer (IPsec ESP)
  5. Better overall security
  6. Faster router processing (fixed header, no checksum, no router fragmentation)
```

Understanding the **header structure** of IPv6 is how all six of those goals are actually implemented in practice.

---

## 2. IPv6 Header — Overview & Philosophy

The IPv6 header was **deliberately redesigned** from scratch — not just a patched-up IPv4 header. The design philosophy:

```
IPv4 header approach:
  → Include everything in the main header
  → Add Options and Padding fields for extras
  → Variable length (20–60 bytes)
  → Routers must process every field at every hop
  → Complex, slow

IPv6 header approach:
  → Base header: fixed, minimal, contains only what EVERY packet needs
  → Extra functionality moved to EXTENSION HEADERS (optional, chained after base)
  → Fixed 40-byte base header always
  → Routers process base header fast; extension headers only when relevant
  → Clean, fast, extensible
```

### IPv6 Header Diagram

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|Version(4) |  Traffic Class(8) |         Flow Label (20)        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|         Payload Length (16)   |  Next Header(8)| Hop Limit (8)|
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                    Source Address (128 bits)                  |
|                                                               |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                                                               |
|                 Destination Address (128 bits)                |
|                                                               |
|                                                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

Total: 4 + 8 + 20 + 16 + 8 + 8 + 128 + 128 = 320 bits = 40 bytes (FIXED)
```

### Field Summary Table

| Field               | Size     | Equivalent in IPv4       |
| ------------------- | -------- | ------------------------ |
| Version             | 4 bits   | Version (same)           |
| Traffic Class       | 8 bits   | Type of Service / DSCP   |
| Flow Label          | 20 bits  | ❌ No equivalent         |
| Payload Length      | 16 bits  | Total Length (partial)   |
| Next Header         | 8 bits   | Protocol field           |
| Hop Limit           | 8 bits   | TTL (Time to Live)       |
| Source Address      | 128 bits | Source IP (32 bits)      |
| Destination Address | 128 bits | Destination IP (32 bits) |

> **Fields removed from IPv4:** IHL (header length), Identification, Flags, Fragment Offset, Header Checksum, Options, Padding — all gone from the base header. Some functionality moved to extension headers; others eliminated entirely.

---

## 3. Base Header — Fixed 40 Bytes

### 3.1 Version (4 bits)

```
Size:  4 bits
Value: 0110 (binary) = 6 (decimal)

  Bit pattern: 0 1 1 0
               │ │ │ └─ LSB
               └─┴─┴─── together = 6

How routers use it:
  → Any packet arriving at a router: router reads first 4 bits
  → 0110 → "this is an IPv6 packet" → process with IPv6 logic
  → 0100 → "this is an IPv4 packet" → process with IPv4 logic

IPv4 value for comparison: 0100 (binary) = 4 (decimal)
```

- The version field is the **first thing a router checks** — it immediately determines how the rest of the header should be interpreted.
- Value is always `0110` for any IPv6 packet, with no exceptions.

---

### 3.2 Traffic Class / Priority (8 bits)

```
Size:  8 bits
Names: Traffic Class  OR  Priority  (same field, two names — both acceptable in exams)

Breakdown of the 8 bits:
  ┌─────────────────┬───────────────┐
  │  DSCP (6 bits)  │  ECN (2 bits) │
  └─────────────────┴───────────────┘

DSCP = Differentiated Services Code Point
  → Sets the PRIORITY of the packet
  → Higher priority packets are forwarded first at congested routers
  → Lower priority packets may be DROPPED if congestion is severe

ECN = Explicit Congestion Notification
  → Allows routers to SIGNAL congestion back to sender
  → Without ECN: packet is simply dropped when router is congested
  → With ECN: router marks the packet → receiver tells sender → sender slows down
  → Avoids unnecessary drops; more efficient congestion management
```

**Real-world use — congestion control:**

```
Scenario: Router is congested. Multiple packets arriving simultaneously.

Without Traffic Class:
  → All packets treated equally → arbitrary dropping → VoIP call breaks up
  → Video stream freezes → background file download also pauses equally

With Traffic Class (Priority set by sender):
  → VoIP packet: high priority → forwarded immediately
  → Video stream: medium priority → forwarded next
  → File download: low priority → can be dropped or delayed without impact
  → User experience preserved for critical real-time traffic
```

> **Security relevance:** An attacker can **forge Traffic Class values** to artificially elevate the priority of their packets — a form of **QoS abuse**. Conversely, they can set low priority on spoofed flooding packets to make them less obviously malicious at first glance.

---

### 3.3 Flow Label (20 bits)

```
Size:  20 bits
Purpose: Real-time data processing / QoS / virtual circuit emulation

Default value: 0 (when not used)
Non-zero value: indicates this packet belongs to a specific "flow"
```

**This is the most unique field in IPv6 — no equivalent in IPv4.**

#### The Problem it Solves

IPv6 is a **datagram service** (just like IPv4) — packets are individually routed and can take **different paths** through the network to reach the same destination:

```
Source ──→ Router A ──→ Router C ──→ Destination
             └────→ Router B ──┘

Packet 1 may go: Source → A → C → Destination
Packet 2 may go: Source → A → B → Destination
Packet 3 may go: Source → A → C → Destination (Router B was busy)

Problem for real-time traffic (VoIP, live video):
  → Packets arrive OUT OF ORDER
  → Different paths have different delays
  → Packet 2 via B might arrive AFTER Packet 3 via C
  → Real-time audio/video must be played back in order
  → Reordering + variable delay = choppy audio, frozen video
```

#### How Flow Label Fixes This

```
Virtual Circuit emulation via Flow Label:

Step 1: Source sets a non-zero Flow Label on all related packets
        (e.g., all packets for this VoIP call get Flow Label = 0xAB123)

Step 2: First packet with that Flow Label arrives at a router
        → Router records: "Flow 0xAB123 goes out interface eth2"
        → This is the RESERVATION — resources allocated for this flow

Step 3: All subsequent packets with Flow Label = 0xAB123
        → Router sees the label → immediately forwards out eth2
        → No need to re-examine full routing table
        → ALL packets follow THE SAME PATH
        → In-order delivery guaranteed
        → Consistent delay (jitter minimised)

Result: Datagram network behaves like a virtual circuit for this flow
        Similar to how TCP provides reliability over IP
```

```
Datagram (normal IPv6):    Source → [different paths] → Destination (out of order)
Virtual Circuit (Flow Label): Source → [one fixed path] → Destination (in order)
```

> **Security relevance:** Flow Label values can be used for **traffic fingerprinting** — an attacker passively monitoring a network can correlate packets belonging to the same session/application by their Flow Label, even if payloads are encrypted. This is a **metadata leakage** risk.

---

### 3.4 Payload Length (16 bits)

```
Size:  16 bits
Range: 0 to 65,535 bytes

What it measures:
  → Length of the DATA that follows the base header
  → Does NOT include the 40-byte base header itself
  → DOES include any extension headers + actual data

Comparison with IPv4:
  IPv4 Total Length: measured the ENTIRE packet (header + data)
  IPv6 Payload Length: measures only what comes AFTER the base header

Maximum standard payload:
  2¹⁶ - 1 = 65,535 bytes ≈ 64 KB
```

#### Jumbograms — Sending More Than 64 KB

```
Standard IPv6 payload: max 65,535 bytes

For larger payloads (e.g., large file transfers, high-resolution video frames):
  → Use JUMBOGRAMS
  → Maximum jumbogram size: 2³² - 1 = 4,294,967,295 bytes ≈ 4 GB per packet!
  → Enabled via the HOP-BY-HOP OPTIONS extension header
  → When jumbogram is used: Payload Length field is set to 0 (signals jumbogram)
  → Actual size stored in the Hop-by-Hop Options header (Jumbo Payload option)

IPv4 had NO equivalent — maximum IPv4 packet size was 65,535 bytes total.
IPv6 jumbograms enable up to 4 GB per packet — critical for
high-performance computing, data centres, supercomputer clusters.
```

> **Security relevance:** **Jumbogram handling bugs** are a known attack surface. Malformed jumbogram headers have historically caused denial-of-service conditions in some IPv6 implementations. When `Payload Length = 0` but no Hop-by-Hop header is present, implementations must handle this gracefully or crash.

---

### 3.5 Next Header (8 bits)

```
Size:  8 bits
Purpose: Identifies what comes AFTER the base header

This is the most architecturally important field in IPv6.
```

#### Two Roles of Next Header

```
Role 1 — No extension headers present:
  Next Header value identifies the upper-layer protocol
  (same role as the "Protocol" field in IPv4)

  Value 6   → TCP
  Value 17  → UDP
  Value 58  → ICMPv6
  Value 89  → OSPF

Role 2 — Extension headers present:
  Next Header value identifies WHICH extension header comes next

  Value 0   → Hop-by-Hop Options header
  Value 43  → Routing header
  Value 44  → Fragment header
  Value 51  → Authentication Header (AH)
  Value 50  → Encapsulating Security Payload (ESP)
  Value 60  → Destination Options header
```

This is what enables the **extension header chain** (covered fully in Section 5).

---

### 3.6 Hop Limit (8 bits)

```
Size:  8 bits
Range: 0 to 255 hops
IPv4 equivalent: TTL (Time to Live)

Function:
  → Set by sender to a maximum hop count (e.g., 64 or 128)
  → Every router that forwards the packet DECREMENTS the value by 1
  → When value reaches 0 → router DROPS the packet + sends ICMPv6 error to source

Why it exists — preventing infinite loops:
  If routing tables malfunction, packets can loop between routers forever:
  Source → Router A → Router B → Router A → Router B → ... (infinite)

  Without Hop Limit: these packets circulate forever, consuming bandwidth
  With Hop Limit: packet is dropped when count reaches 0 → loop terminated
```

```
Example:
  Source sets Hop Limit = 5

  Source → Router 1:  Hop Limit = 5 → 4
  Router 1 → Router 2: Hop Limit = 4 → 3
  Router 2 → Router 3: Hop Limit = 3 → 2
  Router 3 → Router 4: Hop Limit = 2 → 1
  Router 4 → Router 5: Hop Limit = 1 → 0 → DROP + ICMPv6 "Time Exceeded" sent to source
```

> **Difference from IPv4 TTL:**
> In IPv4, the TTL field was originally defined as **seconds** (time-based), but in practice was always used as a **hop count**. IPv6 renamed it **Hop Limit** to officially reflect its actual usage — a pure hop count, never seconds.

> **Security relevance:** Tools like `traceroute6` / `tracert` exploit Hop Limit by deliberately sending packets with Hop Limit = 1, 2, 3... and using the ICMPv6 "Time Exceeded" responses to **map the network path** — useful for recon. **Hop Limit manipulation** can also be used in **TTL-based evasion** to slip packets past IDS/IPS that use TTL values to determine where traffic originated.

---

### 3.7 Source Address (128 bits)

```
Size:  128 bits = 16 bytes = four 32-bit rows in the header diagram

Format: Eight groups of 4 hex digits, separated by colons
  Example: 2001:0db8:85a3:0000:0000:8a2e:0370:7334

Compressed notation rules:
  → Leading zeros in each group can be omitted:   0db8 → db8
  → One consecutive sequence of all-zero groups can be replaced by ::
  Example: 2001:db8:85a3::8a2e:370:7334

Types of IPv6 source addresses:
  fe80::/10  → Link-Local (auto-configured, not routable off-link)
  fc00::/7   → Unique Local (private, like IPv4 RFC 1918)
  2000::/3   → Global Unicast (publicly routable)
  ::1        → Loopback (equivalent to IPv4 127.0.0.1)
```

> **Why 128 bits appear as 4 rows in diagrams:**
> Header diagrams are drawn 32 bits wide (one row = 32 bits). 128 ÷ 32 = 4 rows. The address is not split into 4 pieces — it is a single contiguous 128-bit value, just displayed across 4 rows for readability.

---

### 3.8 Destination Address (128 bits)

```
Size:  128 bits (same as source, same format)

Same notation rules apply.

One key difference from IPv4:
  IPv4 had BROADCAST addresses (255.255.255.255, subnet broadcast)
  IPv6 has NO broadcast — replaced by MULTICAST

  IPv6 multicast addresses:
    ff02::1  → All nodes on the local link (equivalent of IPv4 broadcast)
    ff02::2  → All routers on the local link
    ff02::5  → All OSPF routers
    ff02::fb → mDNS (multicast DNS)
```

> **Security relevance:** Since there is no IPv4-style broadcast in IPv6, **broadcast amplification attacks** (a classic DDoS technique) cannot be done the same way. However, **multicast abuse** is possible — sending packets to `ff02::1` reaches every IPv6 host on the LAN segment, which can be used for **network discovery** or **flooding**.

---

## 4. Extension Headers — Optional Extra Functionality

Extension headers are the key innovation that keeps the IPv6 base header lean. Instead of putting everything in the main header (IPv4's approach), extra functionality is **chained after the base header** using extension headers.

```
Packet structure WITHOUT extension headers:
  [ Base Header (40B) ][ Data ]

Packet structure WITH extension headers:
  [ Base Header (40B) ][ Ext Header 1 ][ Ext Header 2 ][ ... ][ Data ]

Rules:
  → Extension headers appear in a DEFINED ORDER (not arbitrary)
  → Each extension header has its own "Next Header" field pointing to the next one
  → The last extension header's "Next Header" points to the upper-layer protocol
  → Must follow order: Hop-by-Hop → Routing → Fragment → AH → ESP → Destination → Data
```

### Correct Extension Header Order

```
1. Hop-by-Hop Options     (must be FIRST if present — every router reads this)
2. Routing Header         (source routing — path control)
3. Fragment Header        (fragmentation info)
4. Authentication Header  (AH — integrity and authentication)
5. ESP Header             (encryption)
6. Destination Options    (only destination reads this)
7. Upper-layer data       (TCP, UDP, ICMPv6, etc.)
```

---

### 4.1 Routing Header (Type 43)

```
Next Header value: 43
Purpose: Source routing — sender specifies the exact path packets must follow

How it works:
  Sender includes a list of intermediate routers in the Routing Header
  Packet MUST visit those routers in that order before reaching destination

Analogy:
  Normal routing: You send a courier to Dhaka — courier company decides the route
  Source routing: You send a courier to Dhaka and specify it MUST go through
                  Sylhet → Comilla → Dhaka (you decide the exact route)
```

```
Routing Header structure (simplified):
  ┌────────────────┬──────────────┬────────────────┬─────────────┐
  │ Next Header    │ Hdr Ext Len  │ Routing Type   │ Seg Left    │
  ├────────────────┴──────────────┴────────────────┴─────────────┤
  │         Router Address 1 (128 bits)                           │
  │         Router Address 2 (128 bits)                           │
  │         ...                                                   │
  │         Router Address N (128 bits)                           │
  └───────────────────────────────────────────────────────────────┘

  Seg Left = "Segments Left" — how many listed routers still to visit
```

> **Security relevance — Type 0 Routing Header (RH0) Attack:**
> The original Routing Header (Type 0) was **deprecated in RFC 5095 (2007)** because it enabled a devastating attack:
>
> - Attacker could use RH0 to make a packet **bounce back and forth** between two routers repeatedly
> - A single packet could generate **exponential traffic amplification** (DDoS amplification)
> - Example: Attacker → Router A (via RH0) → Router B → Router A → Router B → ... many times
> - **Fix:** Most routers now drop RH0 headers. Type 2 Routing Header (for Mobile IPv6) is still used.
>
> **Practical test on your lab:**
>
> ```bash
> # Check if your router/firewall drops RH0 (it should)
> # Craft a packet with RH0 using Scapy:
> # scapy (in Parrot OS)
> # >>> pkt = IPv6(dst="target", routing_type=0) / IPv6ExtHdrRouting() / TCP()
> # >>> send(pkt)
> # A properly secured router will drop this silently
> ```

---

### 4.2 Hop-by-Hop Options Header (Type 0)

```
Next Header value: 0
Purpose: Carry information that EVERY intermediate router must read and process

Key rule: If present, Hop-by-Hop MUST be the FIRST extension header
          (immediately after the base header)

Why? Because this header is read at EVERY HOP along the path —
     unlike other extension headers which routers skip unless relevant.

Main uses:
  1. Jumbo Payload Option → enables jumbograms (payloads > 65,535 bytes)
  2. Router Alert Option  → tells routers "you need to process this packet's data"
                            (used by RSVP, MLD — multicast listener discovery)
  3. Pad1 / PadN Options  → alignment padding
```

```
Hop-by-Hop flow:
  Source → Router 1 → Router 2 → Router 3 → Destination
            ↓           ↓           ↓
          reads       reads       reads       reads
          HbH hdr    HbH hdr    HbH hdr    HbH hdr

  All other extension headers:
  Source → Router 1 → Router 2 → Router 3 → Destination
            ↓           ↓           ↓
          SKIPS       SKIPS       SKIPS       READS
          (only destination or specific nodes read them)
```

> **Security relevance:** The **Router Alert option** (in Hop-by-Hop) has been abused in DoS attacks — crafting packets with malformed Router Alert options can cause routers to spend excessive CPU processing them. Known as **Hop-by-Hop header abuse** / **Router Alert flooding**.

---

### 4.3 Fragment Header (Type 44)

```
Next Header value: 44
Purpose: Carries fragmentation information when a packet must be divided

Key IPv6 fragmentation rule — THE BIGGEST CHANGE FROM IPv4:

  IPv4: ANY intermediate router can fragment a packet if needed
        → Fragmentation happens transparently along the path
        → Destination reassembles

  IPv6: ONLY THE SOURCE can fragment
        → Intermediate routers CANNOT fragment
        → If a packet is too large for a link, router sends ICMPv6
          "Packet Too Big" error back to source
        → Source then re-sends with smaller packet size (Path MTU Discovery)
```

```
Fragment Header structure:
  ┌────────────────┬──────────────┬─────────────────────────┬───┬──┐
  │ Next Header(8) │ Reserved (8) │  Fragment Offset (13)   │Res│ M│
  ├────────────────┴──────────────┴─────────────────────────┴───┴──┤
  │                  Identification (32 bits)                       │
  └────────────────────────────────────────────────────────────────┘

  Fragment Offset: position of this fragment in the original packet
  M flag (More): 1 = more fragments follow, 0 = this is the last fragment
  Identification: unique ID shared by all fragments of the same original packet
```

> **Security relevance — IPv6 Fragmentation Attacks:**
>
> - **Overlapping Fragment Attack:** Send fragments with overlapping offsets — some IDS/firewall implementations reassemble incorrectly, allowing malicious content to bypass inspection. RFC 5722 mandates dropping overlapping fragments.
> - **Tiny Fragment Attack:** Send a fragment so small that the TCP header is split across two fragments — some firewalls only inspect the first fragment for port numbers, so the second fragment carries the real payload unseen.
> - **Fragment Header with no following data:** Sending a packet with a Fragment Header but the "More" flag = 0 and offset = 0 is an atomic fragment — can confuse some middlebox implementations.
>
> **Test on your Metasploitable2 lab:**
>
> ```bash
> # Using Scapy to craft fragmented IPv6 packets:
> from scapy.all import *
>
> # Send fragmented ping to Metasploitable2
> send(fragment6(IPv6(dst="METASPLOITABLE2_IPv6")/ICMPv6EchoRequest(), fragSize=200))
>
> # Capture and inspect on Metasploitable2:
> sudo tcpdump -i eth0 -n ip6 and icmp6
> ```

---

### 4.4 Authentication Header — AH (Type 51)

```
Next Header value: 51
Purpose: Data integrity + sender authentication
         Protects against: spoofing, replay attacks, data tampering

What AH provides:
  ✅ Authentication  — confirms packet came from claimed sender
  ✅ Integrity       — confirms packet was not modified in transit
  ❌ Confidentiality — does NOT encrypt (payload is still readable)

How it works (conceptual):
  Sender:
    1. Computes HMAC (Hash-based Message Authentication Code) over:
       - Selected IPv6 header fields (those that don't change in transit)
       - Extension headers
       - Payload data
    2. Puts the HMAC digest in the AH header
    3. Sends packet

  Receiver:
    1. Receives packet
    2. Computes HMAC independently using shared key
    3. Compares with HMAC in AH header
    4. Match → packet is authentic and unmodified ✅
    5. No match → packet was tampered with or spoofed ❌ → DISCARD
```

```
AH Header structure:
  ┌────────────────┬──────────────┬──────────────────────────────┐
  │ Next Header(8) │ Payload Len  │         Reserved (16)         │
  ├────────────────┴──────────────┴──────────────────────────────┤
  │                    SPI — Security Parameter Index (32)        │
  ├───────────────────────────────────────────────────────────────┤
  │                    Sequence Number (32)                        │
  ├───────────────────────────────────────────────────────────────┤
  │              Integrity Check Value — ICV (variable)           │
  │                    (the HMAC digest)                           │
  └───────────────────────────────────────────────────────────────┘

  SPI: identifies which security association (key/algorithm) to use
  Sequence Number: prevents replay attacks (each packet has unique increasing number)
  ICV: the actual authentication digest (HMAC-SHA256, etc.)
```

> **Security relevance — Replay Attack Prevention:**
> The **Sequence Number** in AH is critical — if an attacker captures a legitimate authenticated packet and re-sends it later, the receiver sees the sequence number has already been used → packet rejected. Without this, an attacker could replay a legitimate "transfer money" packet repeatedly.
>
> **Important limitation of AH:**
> AH authenticates the IPv6 header but must **skip mutable fields** (fields routers are allowed to change in transit, like Hop Limit). This means AH provides partial header protection — a subtlety relevant for security exams.

---

### 4.5 Destination Options Header (Type 60)

```
Next Header value: 60
Purpose: Carries options that ONLY THE DESTINATION NODE reads

Comparison with Hop-by-Hop:
  Hop-by-Hop (Type 0): every router along the path reads it
  Destination Options (Type 60): only final destination reads it
  → Intermediate routers completely ignore and skip this header

Use case:
  Sender wants to include extra information for the receiver's eyes only
  Example: application-specific processing hints, padding for alignment,
           Mobile IPv6 home address option
```

```
Why this matters for security:
  → Intermediate nodes (routers, firewalls) CANNOT inspect Destination Options
  → An attacker can potentially hide information in Destination Options
    that bypasses perimeter security inspection
  → Security tools must reassemble the full extension header chain to analyse
    Destination Options — many don't bother → potential blind spot
```

---

### 4.6 Encapsulating Security Payload — ESP (Type 50)

```
Next Header value: 50
Purpose: ENCRYPTION + authentication of packet payload

What ESP provides (unlike AH):
  ✅ Confidentiality  — ENCRYPTS the payload (no one in the middle can read it)
  ✅ Authentication   — verifies sender identity
  ✅ Integrity        — detects tampering
  ✅ Replay protection — sequence number (same as AH)

AH vs ESP comparison:
  AH:  authenticates but does NOT encrypt → payload visible
  ESP: encrypts AND authenticates → payload hidden
  Both: part of IPsec, mandatory in IPv6
```

```
ESP Header and Trailer structure:
  ┌───────────────────────────────────────────────────────────────┐
  │               SPI — Security Parameter Index (32)             │  ← ESP Header
  ├───────────────────────────────────────────────────────────────┤
  │                    Sequence Number (32)                        │
  ├═══════════════════════════════════════════════════════════════╡
  │                   ENCRYPTED PAYLOAD                            │  ← Encrypted
  │               (TCP/UDP header + data, all encrypted)           │
  ├───────────────────────────────────────────────────────────────┤
  │                    Padding (0–255 bytes)                       │  ← ESP Trailer
  ├───────────────────────────────────────────────────────────────┤
  │            Pad Length (8) │    Next Header (8)                 │
  ├───────────────────────────────────────────────────────────────┤
  │              Integrity Check Value — ICV (variable)            │  ← ESP Auth
  └───────────────────────────────────────────────────────────────┘

  Everything between ESP Header and ESP Auth trailer is ENCRYPTED.
  Even TCP/UDP ports are hidden from intermediate inspection.
```

> **Security relevance — ESP as a double-edged sword:**
>
> - **Defensive:** ESP encrypted traffic cannot be read by man-in-the-middle attackers, ISPs, or surveillance — strong privacy and confidentiality.
> - **Offensive:** ESP encrypted traffic also **cannot be inspected by your IDS/IPS/DLP tools** — malware can use ESP-encrypted IPv6 to **exfiltrate data** or maintain **C2 channels** that your security monitoring is completely blind to.
> - **Forensics challenge:** When investigating an incident involving IPsec/ESP traffic, you need access to the keys (from endpoint key stores) to decrypt captures — raw packet captures are useless without them.

---

## 5. Extension Header Chaining — How Next Header Works

This is the architectural heart of IPv6's extensibility.

```
Example: Packet using Routing + Fragment + Authentication + Data (TCP)

[ Base Header ]──────────────────────────────────────────────────────
  Next Header = 43 (Routing)        ← "After me comes a Routing header"

[ Routing Header ]───────────────────────────────────────────────────
  Next Header = 44 (Fragment)       ← "After me comes a Fragment header"

[ Fragment Header ]──────────────────────────────────────────────────
  Next Header = 51 (AH)             ← "After me comes an AH header"

[ Authentication Header ]────────────────────────────────────────────
  Next Header = 6 (TCP)             ← "After me comes TCP data"

[ TCP Segment (Data) ]───────────────────────────────────────────────
  (no Next Header — this is the payload)
```

```
Next Header value lookup table:

  Value  | Meaning
  -------|--------------------------------------------------------------
   0     | Hop-by-Hop Options Header
   6     | TCP (upper layer — no more extension headers)
   17    | UDP (upper layer)
   43    | Routing Header
   44    | Fragment Header
   50    | ESP — Encapsulating Security Payload
   51    | AH — Authentication Header
   58    | ICMPv6 (upper layer)
   59    | No Next Header (packet ends here — no upper layer data)
   60    | Destination Options Header
   89    | OSPF (upper layer)
```

**Value 59 — "No Next Header":**

```
  Special value meaning: "the payload ends here, nothing follows"
  Used when there is no upper-layer data after the last extension header
  Routers must not try to process anything after seeing value 59
```

---

## 6. IPv4 Header vs IPv6 Header — Full Comparison

| Property                 | IPv4 Header                         | IPv6 Base Header                               |
| ------------------------ | ----------------------------------- | ---------------------------------------------- |
| **Size**                 | 20–60 bytes (VARIABLE)              | 40 bytes (FIXED — always)                      |
| **Number of fields**     | 12–13 fields (with options)         | 8 fields (clean, minimal)                      |
| **Version**              | 4 bits → value 0100 (4)             | 4 bits → value 0110 (6)                        |
| **Header Length field**  | IHL — 4 bits (needed, var length)   | ❌ Removed (always 40B, not needed)            |
| **QoS/Priority**         | Type of Service (8 bits)            | Traffic Class (8 bits) — same idea             |
| **Real-time support**    | ❌ No dedicated field               | ✅ Flow Label (20 bits)                        |
| **Total/Payload Length** | Total Length (16 bits incl. header) | Payload Length (16 bits, excl. base)           |
| **Identification**       | 16 bits (for fragmentation)         | ❌ Moved to Fragment extension header          |
| **Flags**                | 3 bits (DF, MF flags)               | ❌ Moved to Fragment extension header          |
| **Fragment Offset**      | 13 bits                             | ❌ Moved to Fragment extension header          |
| **TTL / Hop Limit**      | TTL — 8 bits (was "seconds")        | Hop Limit — 8 bits (honest hop count)          |
| **Protocol / Next Hdr**  | Protocol (8 bits)                   | Next Header (8 bits)                           |
| **Checksum**             | ✅ 16 bits — computed every hop     | ❌ Removed entirely (other layers handle it)   |
| **Source Address**       | 32 bits                             | 128 bits                                       |
| **Destination Address**  | 32 bits                             | 128 bits                                       |
| **Options**              | Variable, in main header (slow)     | ❌ Moved to Extension Headers (optional, fast) |
| **Padding**              | Variable, in main header            | ❌ Removed / handled in ext headers            |
| **Fragmentation by**     | Any intermediate router             | Source ONLY                                    |
| **Checksum recomputed**  | At every hop (slow)                 | Never (removed)                                |
| **Router processing**    | Slow — variable header, checksum    | Fast — fixed header, no checksum               |

---

## 7. Datagram Service vs Virtual Circuit — Flow Label Context

Understanding this distinction is essential for grasping why Flow Label exists.

```
DATAGRAM SERVICE (how IPv4 and IPv6 normally work):
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  → No connection established before sending
  → Each packet independently routed
  → Packets may take different paths
  → Packets may arrive out of order
  → No resource reservation
  → Analogy: postal service — each letter independently routed

  Source                    Destination
    │──Pkt1──→ Router A ──→ Router C ──→│
    │──Pkt2──→ Router A ──→ Router B ──→│  (different path!)
    │──Pkt3──→ Router A ──→ Router C ──→│

VIRTUAL CIRCUIT (what Flow Label emulates):
  ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  → Path established before sending (or path decided per-flow)
  → All packets follow THE SAME PATH
  → Packets arrive in order
  → Resources reserved along the path
  → Analogy: telephone call — dedicated circuit for duration

  Source                    Destination
    │──Pkt1──→ Router A ──→ Router C ──→│
    │──Pkt2──→ Router A ──→ Router C ──→│  (same path enforced by Flow Label)
    │──Pkt3──→ Router A ──→ Router C ──→│
```

```
Why datagram is normally preferred:
  ✅ Resilient — if one path fails, packets reroute automatically
  ✅ Efficient — uses all available paths
  ✅ Simple — no connection setup overhead

Why virtual circuit is needed for real-time:
  ✅ In-order delivery — no reordering jitter
  ✅ Predictable latency — consistent path = consistent delay
  ✅ Resource reservation — bandwidth guaranteed
  ❌ Less resilient — if the chosen path fails, ALL packets affected
```

Flow Label gives IPv6 the **best of both worlds** — normally use datagram, switch to virtual circuit mode for time-sensitive flows.

---

## 8. Why This Matters for Cybersecurity

### IPv6 Header Manipulation Attacks

```bash
# Understanding IPv6 fields lets you craft, inspect, and detect attack packets

# Key attacks that abuse specific IPv6 header fields:

Field             Attack
──────────────────────────────────────────────────────────────────────
Version           Malformed version fields to confuse dual-stack parsers
Traffic Class     QoS abuse — forging priorities to starve other traffic
Flow Label        Traffic fingerprinting — correlating encrypted sessions
Payload Length    Length mismatch attacks — causes parser confusion / DoS
Next Header       Invalid/unknown Next Header values — parser exploits
Hop Limit         Traceroute for network mapping; TTL-based IDS evasion
Source Address    IPv6 spoofing (harder than IPv4 but possible)
Destination       Multicast abuse (ff02::1 flooding)
```

### Extension Header Abuse

```
Extension headers are a primary attack surface in IPv6:

1. Routing Header (Type 43 / RH0):
   → Deprecated due to traffic amplification DoS (RFC 5095)
   → Test: does your firewall drop RH0 packets?

2. Hop-by-Hop abuse:
   → Router Alert flooding → exhausts router CPU
   → Unknown options with "01" action bits → can cause packet copying/forwarding exploits

3. Overlapping fragments (Fragment Header):
   → Bypass IDS/IPS — inject malicious payload via overlapping reassembly

4. Excessively long extension header chains:
   → "Extension Header Chain" DoS — craft packet with 100+ chained headers
   → Forces routers to parse huge chains → CPU exhaustion

5. ESP encryption hiding malware C2:
   → Malware uses IPv6 + ESP to create encrypted tunnel
   → DLP, IDS, proxy tools are blind to ESP-encrypted payloads
```

### Practical Security Checks

```bash
# 1. Does your firewall drop Type 0 Routing Headers?
sudo ip6tables -L -n | grep "rt"
# Should see a DROP rule for routing type 0

# 2. Are you logging IPv6 extension header anomalies?
sudo tcpdump -i eth0 -n "ip6" -w /tmp/ipv6_capture.pcap
# Open in Wireshark: filter ip6.nxt to inspect Next Header values

# 3. Check for unexpected IPv6 extension headers in your web traffic
# Wireshark filter examples:
# ipv6.nxt == 0    → Hop-by-Hop (rare in normal traffic — investigate)
# ipv6.nxt == 43   → Routing Header (very rare — almost always suspicious)
# ipv6.nxt == 44   → Fragment (normal, but check for tiny/overlapping fragments)
# ipv6.nxt == 50   → ESP (encrypted — cannot inspect payload)
# ipv6.nxt == 51   → AH (authenticated — can inspect but not decrypt)
```

---

## 9. 🧪 Practical Labs

### Lab 1 — Inspect a Real IPv6 Packet Header with Wireshark

```bash
# Step 1: Start a packet capture on your Parrot OS interface
sudo tcpdump -i eth0 -w /tmp/ipv6_lab.pcap ip6 &

# Step 2: Generate some IPv6 traffic
ping6 -c 5 ipv6.google.com   # or any IPv6 destination
curl -6 https://ipv6.google.com

# Step 3: Stop capture
fg  # bring tcpdump to foreground
# Ctrl+C to stop

# Step 4: Open in Wireshark
wireshark /tmp/ipv6_lab.pcap &

# In Wireshark, expand "Internet Protocol Version 6" to see:
#   Version: 6
#   Traffic Class: 0x00
#   Flow Label: 0x?????
#   Payload Length: ???
#   Next Header: 58 (ICMPv6) or 6 (TCP) etc.
#   Hop Limit: 64 (typical Linux default)
#   Source: your IPv6 address
#   Destination: target IPv6 address

# Useful Wireshark filters:
# ipv6                    → all IPv6 traffic
# ipv6.nxt == 58          → ICMPv6 packets
# ipv6.nxt == 44          → fragmented packets
# ipv6.hlim < 5           → low hop limit (suspicious / traceroute)
# ipv6.flow != 0          → packets using Flow Label
```

---

### Lab 2 — Craft Custom IPv6 Packets with Scapy (Parrot OS)

```python
# Save as ipv6_craft.py
# Run: sudo python3 ipv6_craft.py
# Purpose: Understand each header field by crafting packets

from scapy.all import *

TARGET = "::1"  # loopback for safe testing on your own machine

print("="*60)
print("IPv6 HEADER CRAFTING LAB")
print("="*60)

# ── Lab 2a: Basic IPv6 packet — inspect all fields ──────────────
pkt_basic = IPv6(
    version=6,          # always 6
    tc=0,               # Traffic Class — no priority
    fl=0,               # Flow Label — not used
    plen=None,          # auto-calculated by Scapy
    nh=58,              # Next Header: 58 = ICMPv6
    hlim=64,            # Hop Limit
    src="::1",          # source (loopback)
    dst=TARGET          # destination
) / ICMPv6EchoRequest(id=0x1234, seq=1, data=b"IPv6 Lab Test")

print("\n[Lab 2a] Basic IPv6/ICMPv6 packet:")
pkt_basic.show()

# ── Lab 2b: Set Flow Label (real-time traffic simulation) ────────
pkt_flow = IPv6(
    fl=0xABCDE,         # Non-zero Flow Label → marks as real-time flow
    hlim=64,
    dst=TARGET
) / ICMPv6EchoRequest(data=b"Flow Label test")

print("\n[Lab 2b] Packet with non-zero Flow Label:")
pkt_flow.show()

# ── Lab 2c: Vary Hop Limit — simulate traceroute behaviour ───────
print("\n[Lab 2c] Sending packets with Hop Limit 1 through 5:")
for ttl in range(1, 6):
    pkt = IPv6(dst="2001:db8::1", hlim=ttl) / ICMPv6EchoRequest()
    print(f"  Hop Limit = {ttl}: {pkt.summary()}")

# ── Lab 2d: Traffic Class / Priority setting ─────────────────────
pkt_priority = IPv6(
    tc=0xB8,            # 0xB8 = DSCP 46 (Expedited Forwarding) — highest QoS
    dst=TARGET,
    hlim=64
) / ICMPv6EchoRequest(data=b"High priority packet")

print("\n[Lab 2d] High-priority packet (Traffic Class = 0xB8 / EF):")
pkt_priority.show()

# ── Lab 2e: Send and capture response ───────────────────────────
print("\n[Lab 2e] Sending basic packet to loopback and waiting for reply:")
response = sr1(pkt_basic, timeout=2, verbose=0)
if response:
    print(f"  Got reply! Type: {response.getlayer(ICMPv6EchoReply)}")
    print(f"  Responding Hop Limit: {response[IPv6].hlim}")
else:
    print("  No reply received.")
```

---

### Lab 3 — IPv6 Extension Header Analysis with Scapy

```python
# Save as ipv6_ext_headers.py
# Run: sudo python3 ipv6_ext_headers.py
# Purpose: Craft packets with extension headers, observe chaining

from scapy.all import *

TARGET = "::1"

print("="*60)
print("EXTENSION HEADER LAB")
print("="*60)

# ── Lab 3a: Hop-by-Hop Options Header ───────────────────────────
pkt_hbh = (
    IPv6(dst=TARGET) /
    IPv6ExtHdrHopByHop(options=[PadN(optdata=b'\x00\x00')]) /
    ICMPv6EchoRequest(data=b"Hop-by-Hop test")
)
print("\n[Lab 3a] Packet with Hop-by-Hop Options Header:")
pkt_hbh.show()
print(f"  Next Header in base header: {pkt_hbh[IPv6].nh}")
print(f"  (Should be 0 = Hop-by-Hop)")

# ── Lab 3b: Fragment Header ──────────────────────────────────────
# Create a large payload that will be fragmented
large_data = b"A" * 2000
pkt_large = IPv6(dst=TARGET) / ICMPv6EchoRequest(data=large_data)
fragments = fragment6(pkt_large, fragSize=500)

print(f"\n[Lab 3b] Fragmentation of large packet ({len(large_data)} bytes payload):")
print(f"  Original packet size: {len(pkt_large)} bytes")
print(f"  Number of fragments created: {len(fragments)}")
for i, frag in enumerate(fragments):
    fh = frag.getlayer(IPv6ExtHdrFragment)
    if fh:
        print(f"  Fragment {i+1}: offset={fh.offset}, M={fh.m}, ID={hex(fh.id)}")

# ── Lab 3c: Destination Options Header ──────────────────────────
pkt_dst = (
    IPv6(dst=TARGET) /
    IPv6ExtHdrDestOpt(options=[PadN(optdata=b'\x00\x00\x00\x00')]) /
    ICMPv6EchoRequest(data=b"Dest Options test")
)
print("\n[Lab 3c] Packet with Destination Options Header:")
pkt_dst.show()

# ── Lab 3d: Chained extension headers ───────────────────────────
pkt_chained = (
    IPv6(dst=TARGET) /
    IPv6ExtHdrHopByHop(options=[PadN(optdata=b'\x00\x00')]) /
    IPv6ExtHdrDestOpt(options=[PadN(optdata=b'\x00\x00')]) /
    ICMPv6EchoRequest(data=b"Chained headers test")
)
print("\n[Lab 3d] Packet with chained HbH + DestOpt headers:")
print(f"  Layer stack: {pkt_chained.summary()}")
```

---

### Lab 4 — IPv6 Security Audit on Metasploitable2

```bash
# Prerequisite: Metasploitable2 running in VirtualBox on same host-only network

# Step 1: Find Metasploitable2's IPv6 address
# On Metasploitable2:
ifconfig | grep inet6
# Look for the link-local address: fe80::...

# Step 2: Discover all IPv6 hosts on your lab network from Parrot OS
ping6 -c 3 ff02::1%eth0
# Shows all IPv6-enabled devices responding on your LAN

# Step 3: IPv6-specific nmap scan of Metasploitable2
# Get Metasploitable2's link-local IPv6 address first, then:
sudo nmap -6 -sV -sC fe80::METASPLOITABLE_ADDRESS%eth0 --open

# Note: -6 flag required for IPv6 scanning
# Link-local addresses require the interface suffix (%eth0)

# Step 4: Check if Metasploitable2 responds to IPv6 traffic differently
# Does it have services open on IPv6 that aren't on IPv4?
sudo nmap -6 -p- fe80::METASPLOITABLE_ADDRESS%eth0

# Step 5: Analyse IPv6 extension headers in traffic to Metasploitable2
sudo tcpdump -i eth0 -n "ip6 and host fe80::METASPLOITABLE_ADDRESS" -v
# -v (verbose) shows extension headers if present

# Step 6: Check your own web projects for IPv6 support
# If running a Node.js/Express app:
#   Is it listening on :: (all IPv6 interfaces) or 0.0.0.0 (IPv4 only)?
# Check with:
sudo ss -tlnp | grep -E ":::|\[::\]"
# Entries with ::: or [::] are IPv6-listening services
```

---

### Lab 5 — IPv6 Header Field Calculator (Python)

```python
# Save as ipv6_header_calc.py
# Run: python3 ipv6_header_calc.py
# Utility to decode/display any IPv6 header field values

def decode_traffic_class(tc_byte):
    """Decode 8-bit Traffic Class into DSCP + ECN"""
    dscp = (tc_byte >> 2) & 0x3F  # top 6 bits
    ecn  = tc_byte & 0x03          # bottom 2 bits

    dscp_names = {
        0:  "Default/Best Effort (BE)",
        8:  "Class Selector 1 (CS1) — low priority",
        16: "Class Selector 2 (CS2)",
        24: "Class Selector 3 (CS3)",
        32: "Class Selector 4 (CS4)",
        40: "Class Selector 5 (CS5)",
        46: "Expedited Forwarding (EF) — highest, real-time",
        48: "Class Selector 6 (CS6)",
        56: "Class Selector 7 (CS7) — network control",
        10: "AF11 — Assured Forwarding class 1, low drop",
        12: "AF12 — Assured Forwarding class 1, med drop",
        14: "AF13 — Assured Forwarding class 1, high drop",
    }
    ecn_names = {0: "Not ECN-capable", 1: "ECT(1)", 2: "ECT(0)", 3: "Congestion Experienced (CE)"}

    return dscp, ecn, dscp_names.get(dscp, f"DSCP {dscp} (custom)"), ecn_names[ecn]

def next_header_name(value):
    """Decode Next Header / Protocol number"""
    mapping = {
        0: "Hop-by-Hop Options", 6: "TCP", 17: "UDP", 41: "IPv6-in-IPv4 (6in4 tunnel)",
        43: "Routing Header", 44: "Fragment Header", 50: "ESP (Encapsulating Security Payload)",
        51: "AH (Authentication Header)", 58: "ICMPv6", 59: "No Next Header",
        60: "Destination Options", 89: "OSPF", 132: "SCTP"
    }
    return mapping.get(value, f"Unknown ({value})")

print("="*65)
print("IPv6 HEADER FIELD DECODER")
print("="*65)

# Example: decode a packet you captured
version_bits = "0110"
tc = 0x00
flow_label = 0x00000
payload_len = 64
next_hdr = 58
hop_limit = 64

print(f"\nVersion field: {version_bits} (binary) = {int(version_bits, 2)} (decimal) → IPv{int(version_bits, 2)}")

dscp, ecn, dscp_name, ecn_name = decode_traffic_class(tc)
print(f"\nTraffic Class: 0x{tc:02X} ({tc:08b})")
print(f"  DSCP ({dscp}): {dscp_name}")
print(f"  ECN  ({ecn}):  {ecn_name}")

print(f"\nFlow Label: 0x{flow_label:05X} ({'not in use' if flow_label == 0 else 'ACTIVE — real-time flow'})")
print(f"\nPayload Length: {payload_len} bytes")
print(f"  Total packet size: {40 + payload_len} bytes (40B base header + {payload_len}B payload)")
print(f"\nNext Header: {next_hdr} → {next_header_name(next_hdr)}")
print(f"\nHop Limit: {hop_limit}")
print(f"  Typical values: 64 (Linux/macOS), 128 (Windows), 255 (routing protocols)")
print(f"  If you see low values (< 5): possible traceroute/recon in progress")

print(f"\nBase header size: 40 bytes = 320 bits (ALWAYS FIXED)")
print(f"  4+8+20+16+8+8+128+128 = {4+8+20+16+8+8+128+128} bits = {(4+8+20+16+8+8+128+128)//8} bytes ✅")
```

---

## 10. Solved Examples

### Example 1 — Bit Count Verification

**Question:** Verify that the IPv6 base header is exactly 40 bytes (320 bits) by adding up all field sizes.

```
Field               Size (bits)
──────────────────────────────────
Version                 4
Traffic Class           8
Flow Label             20
Payload Length         16
Next Header             8
Hop Limit               8
Source Address        128
Destination Address   128
──────────────────────────────────
TOTAL                 320 bits = 320 ÷ 8 = 40 bytes ✅
```

---

### Example 2 — Next Header Chain Reading

**Question:** A packet has the following structure. What is in the packet?

```
Base Header:      Next Header = 0
Extension 1:      Next Header = 44    (this is which header type?)
Extension 2:      Next Header = 58    (this is which header type?)
Data:             ICMPv6 content
```

```
Answer:
  Next Header = 0  in base header   → Hop-by-Hop Options header follows
  Next Header = 44 in Extension 1   → Fragment header follows
  Next Header = 58 in Extension 2   → ICMPv6 data follows (58 = ICMPv6)

Packet chain:
  [ IPv6 Base Header ] → [ Hop-by-Hop Options ] → [ Fragment Header ] → [ ICMPv6 ]

This is a FRAGMENTED ICMPv6 packet with hop-by-hop processing required.
Could be a legitimate jumbo ICMPv6 echo (ping6 with large payload),
OR could be a fragmentation-based attack — worth inspecting in Wireshark.
```

---

### Example 3 — Exam Style (2-mark)

**Question:** What is the purpose of the Flow Label field in IPv6? How does it differ from Traffic Class?

```
Flow Label (20 bits):
  Purpose: Real-time data processing — converts datagram service to
           virtual circuit by marking related packets with the same label.
           Routers use the label to forward all packets of a flow along
           the SAME PATH, ensuring in-order delivery and low jitter.
  Use case: VoIP, live video streaming, online gaming.

Traffic Class (8 bits):
  Purpose: Congestion control and QoS priority — DSCP + ECN.
           Sets relative priority of the packet.
           High-priority packets forwarded first; low-priority can be dropped.
  Use case: Differentiating critical vs. background traffic.

Key difference:
  Traffic Class: controls PRIORITY (which packet goes first at a congested router)
  Flow Label: controls PATH (which router to go to — same path for all flow packets)
  They solve DIFFERENT problems and work at different levels of QoS.
```

---

### Example 4 — Security (5-mark)

**Question:** A security engineer captures IPv6 traffic and finds packets with Next Header = 43. Should this be flagged? Explain.

```
Next Header = 43 → Routing Header

This SHOULD be flagged and investigated. Reasons:

1. Routing Header Type 0 (RH0) was deprecated in RFC 5095 (2007)
   due to traffic amplification attacks. Any RH0 packet today is
   either a misconfiguration or an active attack — should be dropped.

2. Even for valid Routing Header types (e.g., Type 2 for Mobile IPv6),
   Routing Header use is rare in normal traffic. Its presence warrants
   investigation: Is it Mobile IPv6? Is it source routing? By whom?

3. Source routing (which Routing Header enables) can be used to:
   - Bypass firewall rules (route traffic through trusted intermediate nodes)
   - Probe internal network topology
   - Conduct traffic amplification (RH0 specifically)

Recommended action:
  → Drop all RH0 packets at the firewall (ip6tables -m rt --rt-type 0 -j DROP)
  → Investigate any RH2+ packets — confirm they are legitimate Mobile IPv6
  → Alert on any Routing Header packets not matching expected Mobile IPv6 flows
  → Log source addresses of all Routing Header senders for threat intelligence
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           IPv6 HEADER — EXAM CHEAT SHEET                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  BASE HEADER — ALWAYS 40 BYTES (320 BITS) — FIXED                    ║
║  ─────────────────────────────────────────────────────────          ║
║  Field              Size    Key Point                                ║
║  Version            4 bit   Value = 0110 (binary) = 6               ║
║  Traffic Class      8 bit   DSCP(6) + ECN(2) — priority + congestion║
║  Flow Label        20 bit   Real-time / virtual circuit emulation    ║
║  Payload Length    16 bit   Data after base header (NOT incl. 40B)  ║
║  Next Header        8 bit   What follows: ext header or upper layer  ║
║  Hop Limit          8 bit   Like TTL — decrements per hop, drop @ 0 ║
║  Source Address   128 bit   Sender's IPv6 address                   ║
║  Destination      128 bit   Receiver's IPv6 address                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  EXTENSION HEADERS (OPTIONAL, CHAINED VIA NEXT HEADER)               ║
║  ─────────────────────────────────────────────────────────          ║
║  Type  Value  Purpose                                                ║
║  HbH     0    Every router reads — jumbograms, router alert         ║
║  Routing 43   Source routing — sender specifies path                ║
║  Frag    44   Fragmentation — SOURCE ONLY (not intermediate routers)║
║  AH      51   Authentication + Integrity (NO encryption)            ║
║  ESP     50   Encryption + Authentication (full security)           ║
║  DstOpt  60   Only destination reads — opaque to intermediate nodes ║
╠══════════════════════════════════════════════════════════════════════╣
║  NEXT HEADER VALUES (MEMORISE THESE)                                  ║
║  0=HbH  43=Routing  44=Fragment  50=ESP  51=AH  60=DstOpt           ║
║  6=TCP  17=UDP  58=ICMPv6  59=No Next Header  89=OSPF               ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY DIFFERENCES FROM IPv4 HEADER                                     ║
║  REMOVED: IHL, Identification, Flags, Fragment Offset, Checksum     ║
║  REMOVED: Options, Padding (→ moved to Extension Headers)           ║
║  RENAMED: TTL → Hop Limit  |  Protocol → Next Header                ║
║  ADDED:   Flow Label (20 bit) — NO equivalent in IPv4               ║
║  CHANGED: Source/Dest 32bit → 128bit                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  FLOW LABEL — THE UNIQUE FIELD                                        ║
║  Converts DATAGRAM service → VIRTUAL CIRCUIT behaviour              ║
║  All packets same flow label → same path → in-order delivery        ║
║  Used for: VoIP, live streaming, gaming — real-time traffic         ║
║  Value 0 = not in use  |  Non-zero = active flow                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  JUMBOGRAMS                                                            ║
║  Normal payload max: 65,535 bytes (16-bit Payload Length)           ║
║  Jumbogram max: ~4 GB (via Hop-by-Hop extension header)             ║
║  When jumbogram: Payload Length field = 0 (signals jumbogram)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  AH vs ESP                                                             ║
║  AH (51):  Auth ✅  Integrity ✅  Encrypt ❌  → verifies, doesn't hide║
║  ESP (50): Auth ✅  Integrity ✅  Encrypt ✅  → verifies AND hides    ║
║  Both are IPsec — MANDATORY in IPv6, optional in IPv4               ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY EXAM TRAPS                                                   ║
║  Routing Header Type 0 (RH0): DEPRECATED — drop at firewall         ║
║  Fragment Header: ONLY SOURCE fragments in IPv6 (not routers)       ║
║  Hop Limit: hop COUNT (not seconds, unlike IPv4 TTL original intent) ║
║  Checksum: REMOVED from IPv6 base header (handled at other layers)  ║
║  Broadcast: does NOT exist in IPv6 → replaced by Multicast          ║
╠══════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS                                                         ║
║  Base header fields: "VeTra-FLo PayNext HoSo-De"                   ║
║  → Version, Traffic class, Flow label, Payload length,              ║
║     Next header, Hop limit, Source, Destination                     ║
║                                                                        ║
║  Extension header order: "H-R-F-A-E-D"                              ║
║  → Hop-by-Hop, Routing, Fragment, AH, ESP, Destination              ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] IPv6 Address Types — Unicast, Multicast, Anycast (no broadcast)
- [ ] IPv6 Address Notation — colon-hex format, zero compression rules (`::`)
- [ ] ICMPv6 — replaces ARP via NDP (Neighbour Discovery Protocol)
- [ ] IPv6 Autoconfiguration — SLAAC (Stateless Address Autoconfiguration)
- [ ] IPv6 Transition Mechanisms — Dual-stack, 6to4 tunnelling, Teredo, NAT64
- [ ] IPsec in depth — AH and ESP configuration on Linux (Parrot OS)

---

_Notes compiled from: Networking Course — IPv6 Headers (Gate Smashers)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
