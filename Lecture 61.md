# 🌐 NAT — Network Address Translation

### " "Networking Course — Network Layer

> **Source:** Gate Smashers — NAT (Network Address Translation) — Lecture 61

> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is NAT?](#1-what-is-nat)
2. [Why NAT is Needed — The IPv4 Exhaustion Problem](#2-why-nat-is-needed--the-ipv4-exhaustion-problem)
3. [Private vs Public IP Addresses](#3-private-vs-public-ip-addresses)
4. [How NAT Works — Step by Step](#4-how-nat-works--step-by-step)
   - [4.1 Outbound Translation (Private → Public)](#41-outbound-translation-private--public)
   - [4.2 Inbound Translation (Public → Private)](#42-inbound-translation-public--private)
   - [4.3 The NAT Translation Table](#43-the-nat-translation-table)
   - [4.4 The Hostel Analogy](#44-the-hostel-analogy)
5. [Types of NAT](#5-types-of-nat)
   - [5.1 Static NAT](#51-static-nat)
   - [5.2 Dynamic NAT](#52-dynamic-nat)
   - [5.3 PAT — Port Address Translation (NAT Overload)](#53-pat--port-address-translation-nat-overload)
6. [NAT Translation Table — Full Entry Format](#6-nat-translation-table--full-entry-format)
7. [NAT and IPv6 — Why NAT May Become Obsolete](#7-nat-and-ipv6--why-nat-may-become-obsolete)
8. [Security Implications of NAT](#8-security-implications-of-nat)
   - [8.1 NAT as an Accidental Firewall](#81-nat-as-an-accidental-firewall)
   - [8.2 NAT Weaknesses and Bypass Techniques](#82-nat-weaknesses-and-bypass-techniques)
   - [8.3 NAT Traversal in Penetration Testing](#83-nat-traversal-in-penetration-testing)
9. [🧪 Practical Labs](#9--practical-labs)
   - [Lab 1 — Observe NAT Translation in Your Own Network](#lab-1--observe-nat-translation-in-your-own-network)
   - [Lab 2 — Inspect NAT with Wireshark (Before and After Translation)](#lab-2--inspect-nat-with-wireshark-before-and-after-translation)
   - [Lab 3 — Configure NAT on Linux (iptables — Masquerade)](#lab-3--configure-nat-on-linux-iptables--masquerade)
   - [Lab 4 — NAT and Your MERN/PERN Web Project](#lab-4--nat-and-your-mernpern-web-project)
   - [Lab 5 — Port Forwarding Through NAT](#lab-5--port-forwarding-through-nat)
10. [Solved Examples](#10-solved-examples)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. What is NAT?

```
NAT — Network Address Translation

  Full name:    Network Address Translation
  Abbreviation: NAT
  RFC:          RFC 1631 (original), RFC 3022 (updated)
  Layer:        Network Layer (Layer 3) — implemented at the router/gateway
  Purpose:      TRANSLATE IP addresses as packets cross between
                a private network and the public internet

  Key definition:
    NAT translates PRIVATE IP addresses into PUBLIC IP addresses
    (for outbound traffic) and PUBLIC IP addresses back into
    PRIVATE IP addresses (for inbound reply traffic).

  In one sentence:
    "Your home/office has many private devices — NAT lets them all
     share ONE public IP address to communicate with the internet."

  Translation direction:
    Outbound: Private IP  ──NAT──►  Public IP   (leaving the network)
    Inbound:  Public IP   ──NAT──►  Private IP  (returning replies)
```

```
Where NAT sits in the network:

  Private Network (LAN)         NAT Router/Gateway          Internet
  ┌─────────────────────┐      ┌───────────────────┐      ┌──────────────┐
  │  10.10.0.1  Host A  │      │  Private:10.10.0.0 │      │              │
  │  10.10.0.2  Host B  │─────►│  Public: 125.12.31.7│─────►│   Internet  │
  │  10.10.0.3  Host C  │      │                   │      │              │
  │  10.10.0.4  Host D  │      │  Performs NAT     │      │              │
  └─────────────────────┘      └───────────────────┘      └──────────────┘
        Many private IPs                ONE public IP

  The internet only ever sees the ONE public IP (125.12.31.7).
  The internal private IPs (10.10.x.x) are completely invisible outside.
```

---

## 2. Why NAT is Needed — The IPv4 Exhaustion Problem

```
THE CORE PROBLEM — IPv4 ADDRESS SPACE IS LIMITED:

  IPv4 uses 32-bit addresses.
  Total possible IPv4 addresses: 2³² = 4,294,967,296 (~4.3 billion)

  This sounds large — but consider:
    → Billions of smartphones (often multiple per person)
    → Billions of laptops and desktops
    → Smart TVs, IoT devices, printers, routers
    → Cloud servers, data centres
    → Each device needs a unique IP to communicate on the internet

  4.3 billion addresses is nowhere near enough for the modern world.

HISTORICAL CONTEXT:
  Early internet (1980s–1990s):
    → Dial-up connections: you got an IP when you connected,
      returned it when you disconnected → IP reuse was possible
    → Few devices per person or organisation
    → Dynamic IP assignment (DHCP) helped rotate IPs efficiently

  Modern world:
    → Always-on connections with dedicated IPs
    → Multiple devices per person (phone + laptop + tablet + smart home)
    → Organisations need IPs for ALL devices simultaneously
    → Running out of public IPs became a real crisis → exhaustion

HOW NAT SOLVES THIS:
  Instead of giving EVERY device a unique PUBLIC IP:
    → Give an organisation ONE (or a few) public IP addresses
    → Let ALL internal devices share that public IP via NAT
    → Inside the organisation: use PRIVATE IP addresses (free, unlimited reuse)
    → Outside the organisation: only the one public IP is visible

  Result:
    → A university with 10,000 students uses ONE public IP
    → 10,000 different private IPs internally
    → Internet only needs to know about ONE address
    → Conserves the scarce public IPv4 address space massively
```

---

## 3. Private vs Public IP Addresses

```
PRIVATE IP ADDRESS RANGES (RFC 1918):

  Three ranges are reserved for private networks — NOT routable on the internet:

  ┌───────────────────────┬───────────────────┬──────────────────┬───────────────────┐
  │  Range                │  CIDR Notation    │  Class           │  Total Addresses  │
  ├───────────────────────┼───────────────────┼──────────────────┼───────────────────┤
  │  10.0.0.0 –           │  10.0.0.0/8       │  Class A         │  2²⁴ = 16,777,216 │
  │  10.255.255.255       │                   │                  │                   │
  ├───────────────────────┼───────────────────┼──────────────────┼───────────────────┤
  │  172.16.0.0 –         │  172.16.0.0/12    │  Class B         │  2²⁰ = 1,048,576  │
  │  172.31.255.255       │                   │                  │                   │
  ├───────────────────────┼───────────────────┼──────────────────┼───────────────────┤
  │  192.168.0.0 –        │  192.168.0.0/16   │  Class C         │  2¹⁶ = 65,536     │
  │  192.168.255.255      │                   │                  │                   │
  └───────────────────────┴───────────────────┴──────────────────┴───────────────────┘

KEY RULES:
  ✓ Private IPs CAN be reused in different organisations
    → Your home: 192.168.1.10
    → Your neighbour's home: also 192.168.1.10 — NO CONFLICT
    → A university across the world: 10.10.0.5 — NO CONFLICT
    (They are all PRIVATE — isolated within each network)

  ✗ Private IPs CANNOT be routed on the internet
    → A router on the internet receiving a packet with source = 10.x.x.x
      will DROP it — private addresses have no meaning on the public internet
    → This is why NAT MUST translate private → public before sending

  ✓ Public IPs MUST be globally unique
    → Assigned by IANA (Internet Assigned Numbers Authority)
    → Your ISP gives you one (or a small block)
    → Only public IPs can be routed on the internet

ANALOGY:
  Private IP = Room number inside a hotel (101, 202...)
    → Room 101 exists in thousands of hotels worldwide — no conflict
    → Each hotel's room numbering is INTERNAL — the outside world doesn't care
  Public IP = The hotel's street address
    → Every hotel has a UNIQUE street address for the outside world to find it
    → NAT = the hotel's reception: routes incoming mail to the right room
```

---

## 4. How NAT Works — Step by Step

### 4.1 Outbound Translation (Private → Public)

```
SCENARIO:
  Host A inside a university wants to access facebook.com (25.25.25.10)
  Host A's private IP:  10.10.0.2
  University public IP: 125.12.31.7
  Destination server:   25.25.25.10

ORIGINAL PACKET (generated by Host A):
  ┌─────────────────────┬────────────────────────────────────────┐
  │  Source IP          │  10.10.0.2   (Host A's private IP)    │
  │  Destination IP     │  25.25.25.10 (Facebook server)        │
  └─────────────────────┴────────────────────────────────────────┘

PACKET ARRIVES AT THE NAT ROUTER:
  The NAT router knows:
  → Source IP = 10.10.0.2 = PRIVATE IP → cannot be sent on internet
  → Must replace the source IP with the public IP before sending out

NAT TRANSLATION (outbound):
  NAT CUTS the private source IP: 10.10.0.2
  NAT PASTES the public IP:       125.12.31.7

TRANSLATED PACKET (sent out to internet):
  ┌─────────────────────┬────────────────────────────────────────┐
  │  Source IP          │  125.12.31.7 (university's public IP)  │
  │  Destination IP     │  25.25.25.10 (Facebook server)         │
  └─────────────────────┴────────────────────────────────────────┘

INTERNET sees this packet:
  → Source = 125.12.31.7 → valid public IP → routable → packet delivered
  → Facebook server processes the request and will send reply to 125.12.31.7
  → Internet has NO IDEA that 10.10.0.2 was the real origin
```

---

### 4.2 Inbound Translation (Public → Private)

```
REPLY FROM FACEBOOK SERVER:

REPLY PACKET (from Facebook server):
  ┌─────────────────────┬────────────────────────────────────────┐
  │  Source IP          │  25.25.25.10 (Facebook server)         │
  │  Destination IP     │  125.12.31.7 (university's public IP)  │
  └─────────────────────┴────────────────────────────────────────┘

PACKET ARRIVES BACK AT THE NAT ROUTER:
  The NAT router sees Destination = 125.12.31.7 (its own public IP)
  It must figure out WHICH internal host this reply belongs to
  → It looks up the TRANSLATION TABLE (see 4.3)
  → Translation table says: 125.12.31.7 talking to 25.25.25.10
    came from 10.10.0.2 internally

NAT TRANSLATION (inbound):
  NAT REPLACES destination IP: 125.12.31.7 → 10.10.0.2

FINAL PACKET (delivered to Host A):
  ┌─────────────────────┬────────────────────────────────────────┐
  │  Source IP          │  25.25.25.10 (Facebook server)         │
  │  Destination IP     │  10.10.0.2   (Host A's private IP)    │
  └─────────────────────┴────────────────────────────────────────┘

HOST A receives the reply:
  → Delivered correctly to Host A's private IP
  → Host A gets its Facebook page, unaware of the translation that happened
```

---

### 4.3 The NAT Translation Table

```
NAT TRANSLATION TABLE:

  When a packet leaves the private network, NAT records the session
  in a TRANSLATION TABLE so it can route the reply back correctly.

  Basic NAT table (IP-only NAT):
  ┌──────────────────────┬──────────────────────┐
  │  Private IP          │  Destination IP      │
  ├──────────────────────┼──────────────────────┤
  │  10.10.0.2           │  25.25.25.10         │
  │  10.10.0.3           │  31.13.72.36         │
  │  10.10.0.4           │  25.25.25.10         │
  └──────────────────────┴──────────────────────┘

  Problem: What if multiple internal hosts connect to THE SAME server?
  (All three rows could be going to 25.25.25.10 — how does NAT tell them apart?)

  Solution: Add PORT NUMBERS to the translation table (this is PAT/NAT Overload):
  ┌──────────────────┬─────────────────┬─────────────────┬─────────────────┐
  │  Private IP      │  Private Port   │  Public IP      │  Public Port    │
  ├──────────────────┼─────────────────┼─────────────────┼─────────────────┤
  │  10.10.0.2       │  52341          │  125.12.31.7    │  52341          │
  │  10.10.0.3       │  48123          │  125.12.31.7    │  48123          │
  │  10.10.0.4       │  61200          │  125.12.31.7    │  61200          │
  └──────────────────┴─────────────────┴─────────────────┴─────────────────┘

  With port numbers: NAT can uniquely identify EACH connection
  → Reply comes in for 125.12.31.7:52341 → NAT knows → send to 10.10.0.2:52341
  → Reply comes in for 125.12.31.7:48123 → NAT knows → send to 10.10.0.3:48123

  This is how one public IP can serve THOUSANDS of private devices simultaneously.
  Each device's connections get unique port numbers → unique NAT table entries.
```

---

### 4.4 The Hostel Analogy

```
THE HOSTEL / HOTEL ANALOGY (from the lecture):

  UNIVERSITY WITH 10 HOSTELS:
    → Each hostel has rooms: 101, 102, 201, 202...
    → Room 101 in Hostel 1 AND Room 101 in Hostel 2 can both exist
      → No conflict because they are INTERNAL room numbers (private IPs)
    → The outside world doesn't know individual room numbers
    → Outside world knows only: "Gate Smashers University, Block 5" (public IP)

  SENDING A LETTER (packet) FROM HOSTEL ROOM:
    → A student in Room 10.10.0.2 writes a letter to facebook.com
    → The letter's return address CANNOT be "Room 10.10.0.2"
      → Post office (internet) won't know where to deliver the reply
    → The HOSTEL RECEPTION (NAT router) changes the return address
      to the university's official postal address: 125.12.31.7
    → Facebook replies to the university's address
    → Hostel reception looks at which student was expecting a reply
      → Delivers it to Room 10.10.0.2

  WORLDWIDE HOTEL ANALOGY:
    → Thousands of hotels worldwide all have "Room 101"
    → No conflict — each hotel's Room 101 is PRIVATE to that hotel
    → Hotel's official address (public IP) is unique worldwide
    → NAT = the reception desk routing mail to the right room

  WHAT IF TWO STUDENTS WRITE TO FACEBOOK AT THE SAME TIME?
    → Both letters go out from the same public address: 125.12.31.7
    → Reception tracks: letter A = from Room 2, letter B = from Room 3
    → Port numbers = the "reference number" on each letter
    → When Facebook replies, reception checks the reference number
      → Delivers reply A to Room 2, reply B to Room 3
```

---

## 5. Types of NAT

### 5.1 Static NAT

```
STATIC NAT (One-to-One mapping):

  One SPECIFIC private IP is permanently mapped to ONE specific public IP.
  The mapping never changes — it is manually configured.

  Example:
    Private IP: 10.10.0.10  ←permanent mapping→  Public IP: 125.12.31.10

  Use case:
    → A web server inside your organisation that must always be reachable
      from the internet at a fixed public IP
    → The server's private IP always translates to the same public IP

  Diagram:
    10.10.0.10  ──Static NAT──►  125.12.31.10  (always, fixed)
    10.10.0.11  ──Static NAT──►  125.12.31.11  (always, fixed)

  Limitations:
    → Requires ONE public IP per private device that needs static mapping
    → Does NOT conserve public IP addresses (still one-to-one)
    → Used only for servers that need a permanent, fixed public-facing IP
```

---

### 5.2 Dynamic NAT

```
DYNAMIC NAT (Pool-based mapping):

  A POOL of public IP addresses is defined.
  When an internal host wants to communicate externally:
    → NAT dynamically assigns it one available public IP from the pool
  When the session ends:
    → The public IP is returned to the pool for another host to use

  Example:
    Pool of public IPs: 125.12.31.1 to 125.12.31.10  (10 addresses)
    Internal hosts: 10.10.0.1 to 10.10.0.100           (100 devices)

    At any given moment:
    → 10.10.0.5 → assigned 125.12.31.3 (for this session)
    → 10.10.0.20 → assigned 125.12.31.7 (for this session)
    → 10.10.0.55 → waiting (all 10 pool addresses in use → connection fails)

  Limitations:
    → Still requires multiple public IPs (the pool)
    → Maximum concurrent connections = size of pool
    → NOT as efficient as PAT (see below)
    → If pool is exhausted → new connections are rejected
```

---

### 5.3 PAT — Port Address Translation (NAT Overload)

```
PAT — PORT ADDRESS TRANSLATION (also called NAT Overload):

  The most common form of NAT — used in virtually every home router.
  MANY private IPs all share ONE single public IP.
  Differentiation is done using PORT NUMBERS.

  How it works:
    → Every outbound connection gets a unique source port number
    → NAT table stores: Private IP + Private Port → Public IP + Public Port
    → All connections share the SAME public IP but have DIFFERENT port numbers

  Example:
    All three devices share public IP: 125.12.31.7

    10.10.0.2:52341  ──PAT──►  125.12.31.7:52341  (to facebook.com)
    10.10.0.3:48123  ──PAT──►  125.12.31.7:48123  (to google.com)
    10.10.0.4:61200  ──PAT──►  125.12.31.7:61200  (to facebook.com)

    When reply comes to 125.12.31.7:52341 → NAT knows → it's for 10.10.0.2
    When reply comes to 125.12.31.7:48123 → NAT knows → it's for 10.10.0.3

  Capacity:
    → Port numbers are 16-bit → 0 to 65,535 possible port values
    → Ports 0–1023 reserved → ~64,000 ports available
    → ONE public IP can theoretically support ~64,000 simultaneous sessions
    → One public IP for an entire home network (10–20 devices) is more than enough

  THIS is what your home router does — PAT, not basic NAT.
  Your home has ONE public IP (from your ISP).
  Every device shares it via PAT.
```

---

## 6. NAT Translation Table — Full Entry Format

```
COMPLETE PAT TRANSLATION TABLE FORMAT:

  ┌──────────────────┬───────────────┬─────────────────┬───────────────┬───────────────────┐
  │  Private IP      │ Private Port  │  Public IP      │ Public Port   │  Destination      │
  ├──────────────────┼───────────────┼─────────────────┼───────────────┼───────────────────┤
  │  10.10.0.2       │  52341        │  125.12.31.7    │  52341        │  25.25.25.10:443  │
  │  10.10.0.2       │  52342        │  125.12.31.7    │  52342        │  25.25.25.10:443  │
  │  10.10.0.3       │  48123        │  125.12.31.7    │  48123        │  31.13.72.36:80   │
  │  10.10.0.4       │  61200        │  125.12.31.7    │  61200        │  25.25.25.10:443  │
  └──────────────────┴───────────────┴─────────────────┴───────────────┴───────────────────┘

NOTE: The SAME private IP (10.10.0.2) can have MULTIPLE entries
  → Each browser tab, each app connection = new port number = new table entry
  → The port number is what uniquely identifies each connection

HOW THE TABLE IS USED:

  Outbound (private → public):
    Packet arrives from 10.10.0.2:52341 destined for 25.25.25.10
    → NAT creates entry, replaces source IP:port with 125.12.31.7:52341
    → Packet leaves the network looking like it came from 125.12.31.7

  Inbound (public → private):
    Reply arrives for 125.12.31.7:52341 from 25.25.25.10
    → NAT looks up destination port 52341 in translation table
    → Finds: 52341 maps to 10.10.0.2:52341
    → NAT replaces destination IP:port with 10.10.0.2:52341
    → Packet delivered to the correct internal host

TABLE ENTRY LIFECYCLE:
  → Entry created when outbound packet first leaves
  → Entry maintained while session is active (traffic flowing)
  → Entry removed when session ends (FIN/RST for TCP, or timeout for UDP)
  → Timeout: TCP ~2 hours, UDP ~60 seconds (router-dependent)
```

---

## 7. NAT and IPv6 — Why NAT May Become Obsolete

```
THE IPv6 SOLUTION:

  IPv4:  32-bit addresses → 2³²  = ~4.3 billion addresses → EXHAUSTED
  IPv6: 128-bit addresses → 2¹²⁸ = 340,282,366,920,938,463,463,374,607,431,768,211,456
                                   ≈ 340 undecillion addresses

  This is an astronomically large number.
  Every grain of sand on Earth could have billions of IPv6 addresses.

  What this means for NAT:
    → In IPv6, EVERY device can have its own UNIQUE global IP address
    → No need to share addresses via NAT
    → Every smartphone, laptop, IoT sensor, car, fridge → its own public IPv6
    → Private address ranges still exist in IPv6 (link-local, ULA)
      but NAT is NOT required for connectivity

  WHY NAT IS STILL WIDELY USED TODAY:
    → IPv4 is still dominant — the internet is not fully IPv6 yet
    → Transition is ongoing — most ISPs support both (dual stack)
    → NAT provides a form of security (hides internal topology)
    → Many legacy systems, routers, and services still only speak IPv4
    → Full IPv6 adoption may take many more years

  FOR THIS COURSE:
    → NAT = IPv4 context
    → IPv6 does not require NAT (though it can optionally use it)
    → Exam questions on NAT always assume IPv4 unless stated otherwise
```

---

## 8. Security Implications of NAT

### 8.1 NAT as an Accidental Firewall

```
NAT PROVIDES A FORM OF SECURITY (by accident):

  Because NAT hides internal private IPs behind one public IP:

  → The internet CANNOT directly initiate connections to internal hosts.
    Example: An attacker on the internet CANNOT connect to 10.10.0.2
    because 10.10.0.2 is not routable — it doesn't exist on the internet.
    The attacker only knows 125.12.31.7 (the public IP).

  → Without a port forwarding entry in the NAT router:
    Inbound connections from the internet are DROPPED.
    NAT has no translation table entry for unsolicited inbound traffic.
    → Packets arrive, NAT doesn't know which internal host to send to → DROPS them.

  This gives NAT an incidental firewall-like property:
    → Internal hosts are INVISIBLE to the internet
    → Only responses to outbound connections are allowed back in
    → Unsolicited inbound connections are blocked by default

  IMPORTANT CAVEAT:
    NAT is NOT a true firewall — it provides NO encryption,
    NO packet inspection, NO access control lists.
    It just happens to block unsolicited inbound traffic as a side effect.
    A real firewall is still needed for proper security.
```

---

### 8.2 NAT Weaknesses and Bypass Techniques

```
WEAKNESSES OF NAT FROM A SECURITY PERSPECTIVE:

  1. NAT does NOT inspect or filter content:
     → Once a connection is established outbound (A→ internet)
     → Reply traffic flows back freely
     → Malware on an internal machine can phone home:
         10.10.0.2 connects to attacker's C2 server
         → NAT creates entry
         → Attacker's C2 sends data back — NAT lets it through
         → Internal host is now communicating with the attacker
     → NAT cannot distinguish legitimate replies from C2 traffic

  2. NAT tables can be manipulated:
     → An attacker who compromises a router can add port forwarding rules
     → Once port forwarding is set up: internet can reach internal hosts
     → Common in home router attacks (default admin credentials)

  3. NAT traversal techniques (used by VoIP, P2P, gaming):
     These are legitimate — but attackers also use them:
     → STUN (Session Traversal Utilities for NAT)
     → TURN (Traversal Using Relays around NAT)
     → ICE (Interactive Connectivity Establishment)
     → UPnP (Universal Plug and Play) — routers auto-create port forwarding
       → UPnP is a major security risk:
           Malware can call UPnP API to open port forwarding automatically
           → Punches a hole through NAT without user knowledge

  4. ICMP and UDP can bypass NAT easier than TCP:
     → TCP has explicit connection state (SYN, ACK)
     → UDP is connectionless — NAT uses timeouts to track sessions
     → Short UDP timeout can leave sessions dangling → security gap
```

---

### 8.3 NAT Traversal in Penetration Testing

```
NAT AND PENETRATION TESTING — REAL SCENARIOS:

  SCENARIO: You have a reverse shell payload on Metasploitable2.
  Metasploitable2 is behind NAT: private IP 192.168.56.102.
  Your Kali/Parrot machine is also behind NAT on a different network.

  Problem:
    → You want Metasploitable2 to connect back to your Parrot machine
    → But your Parrot machine's public IP has NAT in front of it
    → The reverse shell connection is: Meta → Your Parrot's public IP → ???
    → NAT doesn't know which internal host (your Parrot) to forward it to

  Solutions in penetration testing:

  Option A — Port Forwarding on your router:
    → Set up port forwarding: external_IP:4444 → 192.168.1.xx:4444 (your Parrot)
    → Now NAT knows to send inbound connections on port 4444 to your machine
    → Reverse shell connects to your public IP → NAT forwards to Parrot

  Option B — Use a VPS (cloud server) as relay:
    → Rent a server with a static public IP (no NAT)
    → Reverse shell connects to VPS → you SSH to VPS → pivot to shell
    → Metasploit's handler runs on the VPS

  Option C — NAT-to-NAT traversal (same local network):
    → In your lab (VirtualBox): both machines are on the same host-only/NAT network
    → NAT is handled by VirtualBox itself
    → Private IP to private IP works directly within the lab
    → No real NAT traversal needed in local lab scenarios

  DETECTING NAT IN RECONNAISSANCE:
    → Seeing RFC 1918 addresses in TTL/ICMP error messages → target is behind NAT
    → Running traceroute: private IP hops visible → NAT device identified
    → Port scanning a NAT device: closed ports vs filtered ports behaviour differs
```

---

## 9. 🧪 Practical Labs

### Lab 1 — Observe NAT Translation in Your Own Network

```bash
# PURPOSE: See NAT in action — observe how your private IP becomes a public IP
# No special tools needed — uses standard commands

# ── Step 1: Find your private IP ─────────────────────────────────────────────

# Linux / Parrot OS:
ip addr show
# or
ifconfig
# Look for inet 192.168.x.x or 10.x.x.x → this is your PRIVATE IP

# More specific:
ip addr show | grep "inet " | grep -v "127.0.0"
# Output example: inet 192.168.1.105/24 brd 192.168.1.255 scope global

# ── Step 2: Find your public IP (what the internet sees after NAT) ────────────

# Method 1: curl to a public IP echo service
curl https://api.ipify.org
# Output: 125.12.31.7  (your ISP-assigned public IP)

# Method 2: verbose — also shows HTTP headers
curl -v https://api.ipify.org 2>&1 | grep -E "IP:|< HTTP|^[0-9]"

# Method 3: using dig
dig +short myip.opendns.com @resolver1.opendns.com

# ── Step 3: Compare and confirm NAT is happening ──────────────────────────────

echo "=== NAT CONFIRMATION ==="
echo ""
echo "Private IP (before NAT — only visible on local network):"
ip addr show | grep "inet " | grep -v "127.0" | awk '{print $2}'

echo ""
echo "Public IP (after NAT — what the internet sees):"
curl -s https://api.ipify.org

echo ""
echo "These are DIFFERENT → NAT is translating between them."
echo "The internet only knows the public IP."
echo "Your private IP is completely invisible outside your network."

# ── Step 4: Find your default gateway (the NAT device) ───────────────────────

ip route show default
# Output: default via 192.168.1.1 dev wlan0
# 192.168.1.1 = your router = the NAT device

# ── Step 5: Trace the path — see NAT happen at hop 1 ─────────────────────────

traceroute google.com
# First hop (192.168.1.1) = your router (NAT device)
# After the router: your private IP disappears, public IP takes over
# Traceroute to internet goes out under your PUBLIC IP from this point on
```

---

### Lab 2 — Inspect NAT with Wireshark (Before and After Translation)

```bash
# PURPOSE: Capture packets on both sides of NAT to see the IP change
# SETUP: Parrot OS as attacker, Metasploitable2 as target (VirtualBox lab)

# In VirtualBox — two interfaces:
#   vboxnet0: Host-only (192.168.56.0/24) — PRIVATE side (before NAT)
#   enp0s3:   NAT adapter — PUBLIC side (after VirtualBox NAT)

# ── Step 1: Capture traffic on the private (internal) side ───────────────────

# Open Wireshark on the internal interface:
sudo wireshark &
# Select interface: vboxnet0
# Filter: ip.addr == 192.168.56.101  (your Parrot's private IP)

# Generate some traffic:
curl http://example.com

# Observe in Wireshark:
# Source IP = 192.168.56.101  (private IP — BEFORE NAT)
# Destination IP = 93.184.216.34  (example.com)

# ── Step 2: Capture on the external (NAT) side ───────────────────────────────

# The NAT adapter interface:
ip addr show  # find the interface with 10.0.2.x (VirtualBox NAT range)
# Usually: enp0s3 with 10.0.2.15 (VirtualBox gives this to VMs in NAT mode)

sudo tcpdump -i enp0s3 -n "host 93.184.216.34" -v
# This captures traffic AFTER VirtualBox's built-in NAT

# Generate traffic again:
curl http://example.com

# Expected observation:
# On vboxnet0 (internal): Source = 192.168.56.101 (private)
# On enp0s3   (external): Source = 10.0.2.15 (VirtualBox internal NAT IP)
# → VirtualBox is performing NAT between its internal network and your host

# ── Step 3: View NAT connection tracking on Linux ────────────────────────────

# Linux tracks NAT sessions in the conntrack table:
sudo apt install conntrack -y

# View active NAT connections:
sudo conntrack -L
# Output shows:
# tcp  6  431999  ESTABLISHED
#   src=192.168.56.101 dst=93.184.216.34  sport=54321 dport=80
#   src=93.184.216.34  dst=10.0.2.15      sport=80    dport=54321
#                                              ↑ translated IP

# This is the NAT translation table in action:
# Private side:  192.168.56.101:54321 ←→ 93.184.216.34:80
# Public side:   10.0.2.15:54321      ←→ 93.184.216.34:80

# View only ESTABLISHED connections:
sudo conntrack -L | grep ESTABLISHED

# View NAT statistics:
sudo conntrack -S
```

---

### Lab 3 — Configure NAT on Linux (iptables — Masquerade)

```bash
# PURPOSE: Manually set up NAT on Parrot OS / Linux using iptables
# SCENARIO: Make Parrot OS act as a NAT router for Metasploitable2
# This is how Linux routers, firewalls, and cloud VMs handle NAT

# ── Setup ─────────────────────────────────────────────────────────────────────
# Parrot OS:       vboxnet0 = 192.168.56.101 (internal)
#                  enp0s3   = 10.0.2.15      (external / internet-facing)
# Metasploitable2: 192.168.56.102

# ── Step 1: Enable IP forwarding ─────────────────────────────────────────────

# Temporary (resets on reboot):
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Permanent (survives reboot):
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

# Verify:
cat /proc/sys/net/ipv4/ip_forward   # Should show: 1

# ── Step 2: Set up NAT with iptables MASQUERADE ──────────────────────────────

# MASQUERADE = dynamic NAT (automatically uses the outgoing interface's IP)
# This is PAT/NAT Overload — the most common form

# The NAT rule (applied to the POSTROUTING chain — outbound packets):
sudo iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE

# Explanation:
# -t nat         → apply to the nat table
# -A POSTROUTING → after routing decision is made (packet is leaving)
# -o enp0s3      → on the outgoing internet-facing interface
# -j MASQUERADE  → perform NAT (replace source IP with interface IP)

# Allow forwarding between interfaces:
sudo iptables -A FORWARD -i vboxnet0 -o enp0s3 -j ACCEPT  # internal → external
sudo iptables -A FORWARD -i enp0s3 -o vboxnet0 \
  -m state --state RELATED,ESTABLISHED -j ACCEPT           # replies back in

# ── Step 3: Verify NAT rules are in place ────────────────────────────────────

sudo iptables -t nat -L -n -v
# Should show the MASQUERADE rule in POSTROUTING chain

sudo iptables -L FORWARD -n -v
# Should show the two FORWARD rules

# ── Step 4: Test — from Metasploitable2, traffic should now go via Parrot NAT ─

# On Metasploitable2 — set default gateway to Parrot's IP:
# sudo route add default gw 192.168.56.101
# Then: curl http://example.com
# Traffic flows: Metasploitable2 → Parrot (NAT) → Internet

# On Parrot — watch NAT sessions:
sudo conntrack -L

# ── Step 5: Add a STATIC NAT rule (port forwarding) ─────────────────────────

# Port forwarding = Static NAT = allow internet to reach internal host
# Example: forward all inbound TCP port 8080 to Metasploitable2's port 80

sudo iptables -t nat -A PREROUTING \
  -i enp0s3 -p tcp --dport 8080 \
  -j DNAT --to-destination 192.168.56.102:80

# Explanation:
# -A PREROUTING  → before routing decision (inbound packet)
# --dport 8080   → match packets arriving on port 8080
# -j DNAT        → Destination NAT (change destination IP/port)
# --to-destination 192.168.56.102:80  → forward to Meta's port 80

# Now: external_IP:8080 → Metasploitable2:80

# ── Step 6: Save and clean up ─────────────────────────────────────────────────

# Save iptables rules:
sudo iptables-save > /etc/iptables/rules.v4

# Flush all NAT rules when done with lab:
sudo iptables -t nat -F
sudo iptables -F FORWARD
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```

---

### Lab 4 — NAT and Your MERN/PERN Web Project

```bash
# PURPOSE: Understand how NAT affects your web application development
# and how to expose your local dev server through NAT for testing

# ── Scenario 1: Running your app locally — behind NAT ────────────────────────

# Start your MERN/PERN app:
cd ~/your-mern-project
npm start
# App running on: http://localhost:3000
# OR: http://192.168.56.101:3000 (accessible on local network)

# Problem: Your friend/client on the internet wants to test your app.
# They CANNOT connect to:
#   http://192.168.56.101:3000  ← private IP, not routable on internet
#   http://localhost:3000        ← only your machine
# They CAN only reach your public IP: 125.12.31.7
# But NAT doesn't know to forward :3000 traffic to your machine by default.

# ── Solution A: Expose via ngrok (NAT traversal tunnel) ──────────────────────

# Install ngrok:
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-v3-stable-linux-amd64.tgz
tar -xzf ngrok-v3-stable-linux-amd64.tgz
sudo mv ngrok /usr/local/bin/

# Sign up at ngrok.com → get your auth token → configure:
ngrok config add-authtoken YOUR_TOKEN_HERE

# Expose your app to the internet through ngrok's NAT tunnel:
ngrok http 3000

# Output:
# Forwarding  https://abc123.ngrok.io  →  http://localhost:3000
# Now ANYONE on the internet can reach your app at https://abc123.ngrok.io
# ngrok creates a tunnel through your NAT automatically

# ── Solution B: Port forwarding on your router ────────────────────────────────

# Log into your router admin panel (usually 192.168.1.1 or 192.168.0.1)
# Find: Port Forwarding / Virtual Servers / NAT settings
# Add rule:
#   External Port:  3000
#   Internal IP:    192.168.1.xx  (your machine's private IP)
#   Internal Port:  3000
#   Protocol:       TCP
# Save

# Now: your_public_ip:3000 → your machine:3000
# Friend accesses: http://125.12.31.7:3000

# ── Scenario 2: Security testing your own app through NAT ────────────────────

# Running nmap against your app from OUTSIDE your NAT:
# (From a VPS or external server — never from inside your own network)
# nmap -sV your_public_ip -p 3000
# If port forwarding NOT set: nmap shows port as filtered (NAT is blocking)
# If port forwarding SET: nmap can scan port 3000 → reaches your dev server

# This is exactly what attackers do to find exposed dev servers!

# ── Security lesson: What NOT to expose through NAT ──────────────────────────

cat << 'EOF'
PORTS YOU SHOULD NEVER PORT-FORWARD IN PRODUCTION:

  3000  → Node.js/React dev server (no auth, no SSL)
  5432  → PostgreSQL (database — direct internet access = CRITICAL RISK)
  27017 → MongoDB (often no auth by default = CRITICAL RISK)
  22    → SSH (if using password auth = brute force target)
  3306  → MySQL (never expose directly)
  6379  → Redis (no auth by default in older versions)
  8080  → Alternative HTTP (often dev tools with debug endpoints)

SAFER ALTERNATIVES:
  → Use a reverse proxy (Nginx) in front of your app
  → Only expose ports 80 (HTTP) and 443 (HTTPS)
  → Use Nginx to route /api to your Node backend (port 3000 stays internal)
  → SSH: use key-based auth only, disable password auth
  → Database: NEVER expose to internet, use internal network only
  → VPN: access dev machine securely without port forwarding
EOF
```

---

### Lab 5 — Port Forwarding Through NAT

```bash
# PURPOSE: Understand and test port forwarding — Static NAT in practice
# SCENARIO: Make Metasploitable2's web server (port 80) accessible
#           from outside your VirtualBox NAT

# ── Step 1: Verify Metasploitable2's web server is running ───────────────────

# From Parrot OS (internal network):
curl http://192.168.56.102/
# Should return Metasploitable2's default web page HTML

# From "outside" (simulated by using Parrot's external interface):
curl http://10.0.2.15:8080/
# Should FAIL — no port forwarding rule yet

# ── Step 2: Set up port forwarding (Static NAT) ──────────────────────────────

sudo iptables -t nat -A PREROUTING \
  -p tcp --dport 8080 \
  -j DNAT --to-destination 192.168.56.102:80

sudo iptables -A FORWARD -p tcp -d 192.168.56.102 --dport 80 -j ACCEPT

echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# ── Step 3: Test port forwarding is working ───────────────────────────────────

# From Parrot (accessing via the forwarded port):
curl http://10.0.2.15:8080/
# Should NOW return Metasploitable2's web page

# Verify in conntrack:
sudo conntrack -L | grep 8080
# Should show the NAT translation entry:
# tcp ... src=10.0.2.15 dst=10.0.2.15 sport=XXXXX dport=8080
#          src=192.168.56.102 dst=10.0.2.15 sport=80 dport=XXXXX [UNREPLIED]

# ── Step 4: Scan through the NAT to understand attacker perspective ───────────

# From Parrot's external IP — simulating external attacker:
nmap -sV 10.0.2.15 -p 8080

# Expected output:
# 8080/tcp open  http  Apache httpd 2.2.8
# Attacker sees Apache — doesn't see that this is actually port 80 on Meta!
# NAT hides the real internal architecture

# ── Step 5: Cleanup ───────────────────────────────────────────────────────────

sudo iptables -t nat -F
sudo iptables -F FORWARD
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```

---

## 10. Solved Examples

### Example 1 — Basic NAT Translation

**Question:** Host A (private IP: 10.10.0.5) wants to access a website at 200.100.50.25. The NAT router's public IP is 203.15.20.1. Describe what happens to the source IP address at each stage.

```
Answer:

  Stage 1: Host A generates a packet.
    Source IP:      10.10.0.5    (Host A's private IP)
    Destination IP: 200.100.50.25 (target web server)

  Stage 2: Packet arrives at the NAT router.
    NAT router checks: Is the source IP a private address?
    → YES (10.10.0.5 is in the 10.0.0.0/8 private range)
    → Must translate before forwarding to the internet

  Stage 3: NAT translates the source IP.
    NAT REPLACES source IP: 10.10.0.5 → 203.15.20.1
    NAT records in translation table:
      10.10.0.5 : [private port] → 203.15.20.1 : [public port]
    → 203.15.20.1 → 200.100.50.25

  Stage 4: Translated packet goes out on the internet.
    Source IP:      203.15.20.1   (public IP — visible to internet)
    Destination IP: 200.100.50.25 (unchanged — still the web server)

  Stage 5: Web server receives packet and sends reply.
    Source IP:      200.100.50.25 (web server)
    Destination IP: 203.15.20.1   (the public IP it saw as the sender)

  Stage 6: Reply arrives at NAT router.
    NAT checks translation table for destination 203.15.20.1:[port]
    → Finds: this belongs to 10.10.0.5
    → Replaces destination IP: 203.15.20.1 → 10.10.0.5

  Stage 7: Final packet delivered to Host A.
    Source IP:      200.100.50.25 (web server)
    Destination IP: 10.10.0.5     (Host A's private IP — delivered correctly)

  Summary:
    Outbound: 10.10.0.5 → [NAT] → 203.15.20.1  (private to public)
    Inbound:  203.15.20.1 → [NAT] → 10.10.0.5  (public to private)
```

---

### Example 2 — PAT with Multiple Hosts

**Question:** Three hosts (10.0.0.2, 10.0.0.3, 10.0.0.4) all access the same server (50.50.50.50) simultaneously. The public IP is 80.1.1.1. How does NAT differentiate the return traffic for each host?

```
Answer:

  Without port numbers, all three hosts would have the same entry:
    10.0.0.2 → 50.50.50.50 (via 80.1.1.1)
    10.0.0.3 → 50.50.50.50 (via 80.1.1.1)
    10.0.0.4 → 50.50.50.50 (via 80.1.1.1)
  NAT cannot tell which reply belongs to which host → ambiguous!

  PAT (Port Address Translation) solves this:
  Each connection gets a UNIQUE source port number.

  PAT Translation Table:
  ┌──────────────┬──────────────┬──────────────┬──────────────┐
  │ Private IP   │ Private Port │ Public IP    │ Public Port  │
  ├──────────────┼──────────────┼──────────────┼──────────────┤
  │ 10.0.0.2     │ 50200        │ 80.1.1.1     │ 50200        │
  │ 10.0.0.3     │ 50201        │ 80.1.1.1     │ 50201        │
  │ 10.0.0.4     │ 50202        │ 80.1.1.1     │ 50202        │
  └──────────────┴──────────────┴──────────────┴──────────────┘

  All three use public IP 80.1.1.1 but each has a DIFFERENT public port.

  When server 50.50.50.50 replies:
    Reply to 80.1.1.1:50200 → NAT looks up :50200 → sends to 10.0.0.2
    Reply to 80.1.1.1:50201 → NAT looks up :50201 → sends to 10.0.0.3
    Reply to 80.1.1.1:50202 → NAT looks up :50202 → sends to 10.0.0.4

  Each host receives only its own reply — correct delivery guaranteed.
  ONE public IP serves all three simultaneously.
```

---

### Example 3 — Why Private IPs Cannot Reach the Internet

**Question:** Why can't a packet with source IP 192.168.1.5 travel across the internet to reach a server at 8.8.8.8?

```
Answer:

  1. 192.168.1.5 is in the RFC 1918 private address range (192.168.0.0/16).
     Private IP addresses are NON-ROUTABLE on the public internet.

  2. Internet routers are configured to DROP packets
     with private source or destination IP addresses.
     Reason: Private IPs are not globally unique.
     → Two organisations can both have 192.168.1.5 internally.
     → A router on the internet cannot know which organisation to send a reply to.
     → Therefore: private IPs cannot be used on the internet at all.

  3. If a packet with source = 192.168.1.5 were to somehow reach
     an internet router, that router would DISCARD it (RFC 1918 filtering).

  4. NAT MUST translate the source IP to a globally unique PUBLIC IP
     before the packet can be forwarded out to the internet.
     Only after NAT translates 192.168.1.5 → 125.12.31.7 (example public IP)
     can the packet travel across the internet to 8.8.8.8.

  This is the fundamental purpose of NAT.
```

---

### Example 4 — NAT Table Ambiguity (Why Ports are Needed)

**Question:** Two internal hosts (10.0.0.2 and 10.0.0.3) both send packets to 172.217.0.1 (Google). The NAT table only stores IP addresses (no ports). What problem arises and how is it solved?

```
Answer:

  PROBLEM — IP-only NAT table is ambiguous:

  NAT Table (IP only):
  ┌──────────────────┬────────────────────┐
  │ Private IP       │ Destination IP     │
  ├──────────────────┼────────────────────┤
  │ 10.0.0.2         │ 172.217.0.1        │
  │ 10.0.0.3         │ 172.217.0.1        │
  └──────────────────┴────────────────────┘

  When Google replies to the public IP (80.1.1.1):
  → The reply destination is 80.1.1.1 (public IP)
  → NAT looks up: who was talking to 172.217.0.1?
  → Table says: BOTH 10.0.0.2 AND 10.0.0.3!
  → NAT cannot decide which private host gets the reply → AMBIGUOUS!

  SOLUTION — Add port numbers (PAT):

  NAT Table with ports:
  ┌──────────────┬──────────────┬──────────────┬──────────────┐
  │ Private IP   │ Private Port │ Public IP    │ Public Port  │
  ├──────────────┼──────────────┼──────────────┼──────────────┤
  │ 10.0.0.2     │ 43500        │ 80.1.1.1     │ 43500        │
  │ 10.0.0.3     │ 44200        │ 80.1.1.1     │ 44200        │
  └──────────────┴──────────────┴──────────────┴──────────────┘

  Google replies to 80.1.1.1:43500 → NAT finds exactly → sends to 10.0.0.2
  Google replies to 80.1.1.1:44200 → NAT finds exactly → sends to 10.0.0.3

  Port numbers make each NAT table entry UNIQUE even when multiple hosts
  access the same destination IP.

  This is why real-world NAT is almost always PAT (Port Address Translation).
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════╗
║           NAT (Network Address Translation) — EXAM CHEAT SHEET           ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHAT IS NAT?                                                              ║
║  → Translates Private IP ↔ Public IP at the router/gateway              ║
║  → Layer: Network Layer (Layer 3) — implemented at the router            ║
║  → Outbound: Private IP → Public IP (leaving the network)               ║
║  → Inbound:  Public IP → Private IP (replies returning)                 ║
║  → Conserves IPv4 public address space                                   ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHY NAT IS NEEDED                                                         ║
║  → IPv4 = 32-bit = only 2³² ≈ 4.3 billion addresses                    ║
║  → Not enough for every device to have a unique public IP               ║
║  → Private IPs are NON-ROUTABLE on the internet                         ║
║  → NAT lets MANY private devices share ONE public IP                    ║
╠══════════════════════════════════════════════════════════════════════════╣
║  PRIVATE IP RANGES (RFC 1918) — memorise these:                           ║
║  Class A:  10.0.0.0    – 10.255.255.255    (2²⁴ addresses)              ║
║  Class B:  172.16.0.0  – 172.31.255.255    (2²⁰ addresses)              ║
║  Class C:  192.168.0.0 – 192.168.255.255   (2¹⁶ addresses)              ║
║  → These are reusable within different organisations (no conflict)       ║
║  → NOT routable on the internet → must be NAT'd before going out        ║
╠══════════════════════════════════════════════════════════════════════════╣
║  THREE TYPES OF NAT:                                                       ║
║  Static NAT:    1 private IP ←→ 1 public IP (fixed, permanent)         ║
║                 Use: servers that must be reachable at fixed public IP   ║
║  Dynamic NAT:   private IPs share a POOL of public IPs                  ║
║                 IP assigned dynamically from pool per session            ║
║  PAT/Overload:  MANY private IPs share ONE public IP via PORT NUMBERS   ║
║                 MOST COMMON — this is what your home router does         ║
╠══════════════════════════════════════════════════════════════════════════╣
║  HOW PAT WORKS (the most important type):                                  ║
║  → Each outbound connection gets a unique SOURCE PORT NUMBER             ║
║  → NAT table entry: Private IP:Port → Public IP:Port                   ║
║  → ONE public IP supports ~64,000 simultaneous connections              ║
║  → Port numbers (16-bit) make each session uniquely identifiable        ║
║  → Reply comes in → NAT matches the port → routes to correct private IP ║
╠══════════════════════════════════════════════════════════════════════════╣
║  NAT TRANSLATION PROCESS:                                                  ║
║  Outbound:                                                                 ║
║    1. Internal host sends packet (src = private IP)                      ║
║    2. Packet arrives at NAT router                                        ║
║    3. NAT replaces src IP with public IP (+ assigns port for PAT)        ║
║    4. NAT records in translation table                                   ║
║    5. Packet sent to internet with public IP as source                  ║
║  Inbound:                                                                  ║
║    6. Reply arrives at public IP                                          ║
║    7. NAT looks up translation table by destination port                ║
║    8. NAT replaces destination IP with private IP                       ║
║    9. Packet delivered to correct internal host                          ║
╠══════════════════════════════════════════════════════════════════════════╣
║  NAT TRANSLATION TABLE FORMAT:                                             ║
║  Private IP | Private Port | Public IP  | Public Port | Destination      ║
║  ─────────────────────────────────────────────────────────────────────   ║
║  10.10.0.2  |  52341       | 125.12.31.7 |  52341     | 25.25.25.10     ║
║  10.10.0.3  |  48123       | 125.12.31.7 |  48123     | 31.13.72.36     ║
╠══════════════════════════════════════════════════════════════════════════╣
║  NAT AND IPv6:                                                             ║
║  → IPv6 = 128-bit = 2¹²⁸ addresses (astronomically large)              ║
║  → Every device can have a globally unique public IPv6 address           ║
║  → NAT is NOT required in IPv6 (enough addresses for everything)        ║
║  → NAT may eventually become obsolete as IPv6 adoption grows            ║
║  → For now: IPv4 + NAT is still dominant worldwide                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SECURITY ASPECTS OF NAT:                                                  ║
║  → Hides internal network topology from the internet                    ║
║  → Acts as accidental firewall: unsolicited inbound connections blocked  ║
║  → NOT a true firewall: no content inspection, no ACLs, no encryption   ║
║  → UPnP danger: malware can auto-create port forwarding rules            ║
║  → C2 traffic bypasses NAT: malware initiates outbound → NAT allows it  ║
║  → Port forwarding = Static NAT = allows internet to reach internal host ║
╠══════════════════════════════════════════════════════════════════════════╣
║  EXAM TRAPS:                                                               ║
║  → NAT translates IP addresses, NOT port numbers alone (PAT does both)  ║
║  → Private IPs are NOT routable on internet — they must be NAT'd        ║
║  → Same private IP CAN exist in multiple organisations — no conflict    ║
║  → NAT is a ROUTER function (Layer 3) — NOT done by switches (Layer 2)  ║
║  → Without NAT: private IP packets would be DROPPED by internet routers ║
║  → NAT works STATEFULLY: tracks sessions in translation table           ║
║  → Static NAT does NOT conserve public IPs (still 1-to-1)              ║
║  → Only PAT (NAT Overload) allows MANY private IPs → ONE public IP     ║
║  → IPv6 does NOT need NAT — enough addresses exist without it           ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LINUX COMMANDS FOR NAT:                                                   ║
║  Enable IP forwarding: echo 1 > /proc/sys/net/ipv4/ip_forward           ║
║  PAT/Masquerade:       iptables -t nat -A POSTROUTING -o eth0 -j MASQ.. ║
║  Port forward:         iptables -t nat -A PREROUTING -p tcp --dport X   ║
║                          -j DNAT --to-destination PRIVATE_IP:PORT        ║
║  View NAT table:       sudo conntrack -L                                  ║
║  Find your public IP:  curl https://api.ipify.org                        ║
║  Find your private IP: ip addr show                                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS:                                                             ║
║  NAT = "Nobody Actually sees your True address (from outside)"           ║
║  Private IP = hotel room number (101 exists in every hotel worldwide)   ║
║  Public IP  = hotel's street address (unique worldwide)                  ║
║  NAT router = hotel reception desk (routes mail to right room)           ║
║  PAT port   = reference number on each letter (tells which room to reply)║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] **DHCP — Dynamic Host Configuration Protocol** (automatic IP assignment — works with NAT)
- [ ] **DNS — Domain Name System** (how domain names resolve to IPs that NAT then handles)
- [ ] **IPv6 Addressing** (why NAT becomes obsolete — 2¹²⁸ addresses)
- [ ] **Subnetting and CIDR** (understanding the private ranges NAT uses)
- [ ] **Routing Protocols (RIP, OSPF, BGP)** (how routers decide where to forward packets)
- [ ] **Firewall vs NAT** (why NAT is not a true security control)
- [ ] **Port Forwarding / DMZ** (advanced NAT configurations for servers)
- [ ] **UPnP Security** (how malware bypasses NAT via Universal Plug and Play)
- [ ] **VPN and NAT Traversal** (how VPNs work through NAT — STUN, TURN, ICE)
- [ ] **CGNAT (Carrier-Grade NAT)** (ISPs doing NAT at scale — NAT behind NAT)

---

_Notes compiled from: Networking Course — NAT (Network Address Translation) — Lecture 61 (Gate Smashers)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student_
