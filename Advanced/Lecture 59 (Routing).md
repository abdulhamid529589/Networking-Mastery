# 🌐 Link State Routing

### " "Networking Course — Network Layer Routing

> **Source:** Gate Smashers — Link State Routing
> " " · CCNA
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Link State Routing?](#1-what-is-link-state-routing)
2. [Why Link State? — Problems with Distance Vector](#2-why-link-state--problems-with-distance-vector)
3. [The Network Topology — Example Graph](#3-the-network-topology--example-graph)
4. [Step 1 — Building the Link State Table](#4-step-1--building-the-link-state-table)
   - [4.1 What is a Link State Table?](#41-what-is-a-link-state-table)
   - [4.2 Hello Message — How Neighbours are Discovered](#42-hello-message--how-neighbours-are-discovered)
   - [4.3 Link State Packet (LSP) Structure](#43-link-state-packet-lsp-structure)
   - [4.4 Link State Tables for All Routers](#44-link-state-tables-for-all-routers)
5. [Step 2 — Flooding](#5-step-2--flooding)
   - [5.1 What is Flooding?](#51-what-is-flooding)
   - [5.2 Why Flooding (Not Just Neighbour Sharing)?](#52-why-flooding-not-just-neighbour-sharing)
   - [5.3 Flooding Problems — Sequence Number and TTL](#53-flooding-problems--sequence-number-and-ttl)
6. [Step 3 — Building the Global Database](#6-step-3--building-the-global-database)
7. [Step 4 — Dijkstra's Algorithm (SSSP)](#7-step-4--dijkstras-algorithm-sssp)
   - [7.1 What is Dijkstra's Algorithm?](#71-what-is-dijkstras-algorithm)
   - [7.2 Running Dijkstra from R1 — Full Walkthrough](#72-running-dijkstra-from-r1--full-walkthrough)
   - [7.3 Dijkstra Iteration Table](#73-dijkstra-iteration-table)
8. [Step 5 — Building the Routing Table](#8-step-5--building-the-routing-table)
9. [Link State vs Distance Vector — Full Comparison](#9-link-state-vs-distance-vector--full-comparison)
10. [Security Analysis — Attacking Link State Routing](#10-security-analysis--attacking-link-state-routing)
    - [10.1 LSA Spoofing / Fake LSP Injection](#101-lsa-spoofing--fake-lsp-injection)
    - [10.2 Database Overflow Attack](#102-database-overflow-attack)
    - [10.3 Routing Table Poisoning via Flooding Abuse](#103-routing-table-poisoning-via-flooding-abuse)
    - [10.4 Reconnaissance via OSPF/IS-IS Sniffing](#104-reconnaissance-via-ospfis-is-sniffing)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Solved Examples](#12-solved-examples)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. What is Link State Routing?

Link State Routing is one of the two fundamental categories of **dynamic routing algorithms** used by routers to build and maintain their routing tables automatically.

```
ROUTING ALGORITHM CATEGORIES:

  Dynamic Routing Algorithms
  │
  ├── Distance Vector Routing
  │     → Share distance (cost) vectors with NEIGHBOURS only
  │     → Routers know cost to destination but NOT the full topology
  │     → Example protocols: RIP (Routing Information Protocol)
  │
  └── Link State Routing  ◄──── THIS VIDEO
        → Share link state tables with EVERYONE (flooding)
        → Every router builds a COMPLETE MAP of the network
        → Example protocols: OSPF, IS-IS

Full name:    Link State Routing
Abbreviation: LS Routing
Real protocols that use it:
  OSPF  — Open Shortest Path First (most common interior gateway protocol)
  IS-IS — Intermediate System to Intermediate System (used in ISP backbone networks)
Layer: Network Layer (Layer 3)
```

```
The core idea in one sentence:
  "Every router learns the COMPLETE topology of the network,
   then independently calculates the shortest path to every destination
   using Dijkstra's algorithm."

Two key words:
  LINK  → the connection (edge) between two routers
  STATE → whether that link is UP or DOWN and its cost/distance

Link State = knowing the STATUS (up/down + cost) of every link in the network.
```

> **Why this matters for security:** OSPF (which uses Link State Routing) is one of the most widely deployed routing protocols in enterprise and ISP networks. Understanding how it works is essential for both network engineering and attacking/defending network infrastructure.

---

## 2. Why Link State? — Problems with Distance Vector

Link State Routing was specifically designed to fix the problems that exist in Distance Vector Routing:

```
Distance Vector Problems that Link State Solves:

  Problem 1 — COUNT TO INFINITY:
    In Distance Vector, when a link fails, routers keep incrementing
    the distance indefinitely ("counting to infinity") before they
    figure out the route is broken.
    Link State: No count to infinity. Every router has the full map.
    A link failure → LSP is immediately flooded → everyone updates their map.

  Problem 2 — SLOW CONVERGENCE:
    Distance Vector converges slowly — updates hop one router at a time.
    Link State: Flooding ensures updates reach ALL routers simultaneously.
    Convergence is MUCH faster.

  Problem 3 — ROUTING LOOPS:
    Because Distance Vector routers only know distances (not full paths),
    they can form routing loops (A→B→A→B forever).
    Link State: Full topology knowledge + Dijkstra → no loops possible
    because the algorithm computes the actual shortest path tree.

  Problem 4 — PARTIAL KNOWLEDGE:
    Distance Vector routers only know their NEIGHBOURS.
    They trust neighbour-reported distances blindly.
    Link State: Every router has GLOBAL KNOWLEDGE — the complete network map.
    No blind trust — every router independently verifies the shortest path.
```

```
The key philosophical difference:

  Distance Vector:
    "Tell your neighbours HOW FAR things are from you."
    → Routers share DISTANCES
    → Each router only knows its own view (local knowledge)

  Link State:
    "Tell EVERYONE about YOUR DIRECT LINKS."
    → Routers share LINK INFORMATION (who I'm connected to, at what cost)
    → Every router builds a COMPLETE MAP (global knowledge)
    → Then everyone independently calculates shortest paths
```

---

## 3. The Network Topology — Example Graph

Throughout this entire topic, we work with the following 6-router network:

```
Network Topology:

           6
    R1 ─────────── R2
    │  \          / │
  3 │   \        /  │ 2
    │    \      /   │
    R3    \    /    │
    │  2   \  / 7   │
  9 │       \/      │ 7
    │       /\      │
    R5 ────    ──── R4
    │    1          │
  4 │               │ 8
    │               │
    R6 ─────────────┘

Cleaner representation with all edges and costs:

  Router Pair    Cost/Distance
  ───────────────────────────
  R1 ↔ R2           6
  R1 ↔ R3           3
  R2 ↔ R3           2
  R2 ↔ R4           7
  R3 ↔ R5           9
  R4 ↔ R5           1
  R4 ↔ R6           8
  R5 ↔ R6           4

Total routers:     6 (R1, R2, R3, R4, R5, R6)
Total links:       8
```

```
Adjacency list (who is directly connected to whom):

  R1: R2 (cost 6),  R3 (cost 3)
  R2: R1 (cost 6),  R3 (cost 2),  R4 (cost 7)
  R3: R1 (cost 3),  R2 (cost 2),  R5 (cost 9)
  R4: R2 (cost 7),  R5 (cost 1),  R6 (cost 8)
  R5: R3 (cost 9),  R4 (cost 1),  R6 (cost 4)
  R6: R4 (cost 8),  R5 (cost 4)
```

---

## 4. Step 1 — Building the Link State Table

### 4.1 What is a Link State Table?

```
LINK STATE TABLE (also called: Link State Advertisement / LSA):

  This is NOT the routing table.
  This is the FIRST thing a router builds — before the routing table.

  It contains ONLY what the router can see directly:
    → Which routers is it DIRECTLY connected to? (neighbours)
    → What is the COST/DISTANCE to each of those neighbours?
    → Is each link UP or DOWN? (the "state" of the link)

  At this stage, each router has LOCAL knowledge only.
  R1 knows only about R2 and R3 — it knows NOTHING about R4, R5, R6 yet.

  The routing table (built in Step 4) will have paths to ALL routers.
  The link state table is just the local starting point.
```

---

### 4.2 Hello Message — How Neighbours are Discovered

```
HELLO MESSAGE PROCESS:

  Step 1: Router powers on (or a link comes up)
  Step 2: Router sends HELLO message out of EVERY interface
  Step 3: Neighbour routers receive the HELLO and respond with their own HELLO
  Step 4: From these HELLO exchanges, each router learns:
            → Who is directly connected to it (neighbour identity)
            → What is the cost of that direct link (link metric)
            → Whether the link is currently UP (if a HELLO is received)
            → Whether the link is DOWN (if HELLO responses stop — dead timer expires)

  R1 sends HELLO:
    → Out interface towards R2: R2 responds → R1 learns: "I have a link to R2, cost=6"
    → Out interface towards R3: R3 responds → R1 learns: "I have a link to R3, cost=3"
    → R1 has NO other interfaces → discovery complete for R1

  This is exactly the same as how OSPF's "Hello" protocol works in real deployments.
  The concept maps directly to production networking.
```

---

### 4.3 Link State Packet (LSP) Structure

```
LINK STATE PACKET (LSP) — what gets created and flooded:

  A Link State Packet is NOT just "neighbour + distance".
  It contains EXTRA FIELDS for reliability and loop prevention:

  ┌──────────────────────────────────────────────────────────────────┐
  │                    LINK STATE PACKET (LSP)                        │
  ├──────────────────────────────────────────────────────────────────┤
  │  Router ID          │ Which router created this LSP              │
  │                     │ (e.g., R1)                                 │
  ├──────────────────────────────────────────────────────────────────┤
  │  Sequence Number    │ 32-bit counter                             │
  │  (32 bits)          │ Incremented each time the LSP is updated   │
  │                     │ Receivers use this to detect:              │
  │                     │   → Duplicate LSPs (discard if same seq)  │
  │                     │   → Old LSPs (discard if lower seq)       │
  │                     │   → New LSPs (process if higher seq)      │
  ├──────────────────────────────────────────────────────────────────┤
  │  TTL                │ Time To Live                               │
  │  (Time to Live)     │ Decremented at each router (like IP TTL)  │
  │                     │ When TTL reaches 0: packet is DISCARDED   │
  │                     │ Purpose: prevent infinite loops in flooding│
  ├──────────────────────────────────────────────────────────────────┤
  │  Neighbour List     │ List of (Neighbour_Router_ID, Link_Cost)  │
  │                     │ → (R2, 6)                                  │
  │                     │ → (R3, 3)                                  │
  │                     │ (This is the actual topology information)  │
  └──────────────────────────────────────────────────────────────────┘

  Why Sequence Number?
    Flooding sends the same LSP out of MULTIPLE interfaces.
    A router may RECEIVE the same LSP from multiple paths.
    Sequence number lets the receiver detect: "I already processed this LSP"
    and simply discard the duplicate instead of processing it again.

  Why TTL?
    Flooding + Loops = LSP circulates forever (infinite flooding loop).
    TTL prevents this: each router decrements TTL when forwarding.
    When TTL = 0: router does NOT forward → packet dies naturally.

  Bandwidth implication:
    More fields in the LSP = LARGER packet = MORE bandwidth consumed.
    This is a known trade-off: Link State uses MORE bandwidth than Distance Vector
    because it floods larger, more information-rich packets everywhere.
```

---

### 4.4 Link State Tables for All Routers

Each router builds its own LSP independently from its HELLO exchanges:

```
R1's Link State Table (what R1 discovers about itself):
  ┌──────────────────┬───────────────────┬──────────┐
  │  My Router ID    │  Neighbour        │  Cost    │
  ├──────────────────┼───────────────────┼──────────┤
  │  R1              │  R2               │  6       │
  │  R1              │  R3               │  3       │
  └──────────────────┴───────────────────┴──────────┘
  R1 knows ONLY about R2 and R3 at this stage.
  R4, R5, R6 are completely unknown to R1 right now.

R2's Link State Table:
  ┌──────────────────┬───────────────────┬──────────┐
  │  My Router ID    │  Neighbour        │  Cost    │
  ├──────────────────┼───────────────────┼──────────┤
  │  R2              │  R1               │  6       │
  │  R2              │  R3               │  2       │
  │  R2              │  R4               │  7       │
  └──────────────────┴───────────────────┴──────────┘

R3's Link State Table:
  ┌──────────────────┬───────────────────┬──────────┐
  │  My Router ID    │  Neighbour        │  Cost    │
  ├──────────────────┼───────────────────┼──────────┤
  │  R3              │  R1               │  3       │
  │  R3              │  R2               │  2       │
  │  R3              │  R5               │  9       │
  └──────────────────┴───────────────────┴──────────┘

R4's Link State Table:
  ┌──────────────────┬───────────────────┬──────────┐
  │  My Router ID    │  Neighbour        │  Cost    │
  ├──────────────────┼───────────────────┼──────────┤
  │  R4              │  R2               │  7       │
  │  R4              │  R5               │  1       │
  │  R4              │  R6               │  8       │
  └──────────────────┴───────────────────┴──────────┘

R5's Link State Table:
  ┌──────────────────┬───────────────────┬──────────┐
  │  My Router ID    │  Neighbour        │  Cost    │
  ├──────────────────┼───────────────────┼──────────┤
  │  R5              │  R3               │  9       │
  │  R5              │  R4               │  1       │
  │  R5              │  R6               │  4       │
  └──────────────────┴───────────────────┴──────────┘

R6's Link State Table:
  ┌──────────────────┬───────────────────┬──────────┐
  │  My Router ID    │  Neighbour        │  Cost    │
  ├──────────────────┼───────────────────┼──────────┤
  │  R6              │  R4               │  8       │
  │  R6              │  R5               │  4       │
  └──────────────────┴───────────────────┴──────────┘
```

```
At the end of Step 1:
  ✅ Every router knows its OWN neighbours and costs
  ❌ No router knows anything BEYOND its direct connections
  ❌ R1 still doesn't know R4, R5, R6 exist
  → Step 2 (Flooding) fixes this
```

---

## 5. Step 2 — Flooding

### 5.1 What is Flooding?

```
FLOODING:
  Every router takes its OWN Link State Packet (LSP)
  and sends it out of ALL its interfaces — not just to neighbours.

  The receiver then forwards the LSP out of ALL its own interfaces
  (except the one it received it on).

  This continues until EVERY router in the network has received
  EVERY other router's LSP.

  Result: R1 receives LSPs from R2, R3, R4, R5, and R6.
          R2 receives LSPs from R1, R3, R4, R5, and R6.
          ... and so on for every router.

Contrast with Distance Vector:
  Distance Vector: R1 shares ONLY with R2 and R3 (direct neighbours)
  Link State:      R1's LSP eventually reaches R4, R5, R6 via flooding

Visual example — R1's LSP flooding:

  R1 sends LSP to R2 and R3 (its direct neighbours)
       ↓                  ↓
  R2 receives R1's LSP → R2 forwards to R3, R4 (its OTHER neighbours)
  R3 receives R1's LSP → R3 forwards to R2, R5 (its OTHER neighbours)
       ↓                        ↓
  R4 receives R1's LSP → R4 forwards to R5, R6
  R5 receives R1's LSP → R5 forwards to R4, R6
       ↓
  R6 receives R1's LSP

  Eventually: ALL 6 routers have R1's LSP.
  Simultaneously: R2, R3, R4, R5, R6 are doing the SAME flooding of their OWN LSPs.
```

---

### 5.2 Why Flooding (Not Just Neighbour Sharing)?

```
RELIABILITY — the core reason for flooding:

  If R1 only sent its LSP to R2 and R3, and the R1-R2 link failed
  during transmission, then R2 wouldn't receive R1's LSP.
  If R2 doesn't have R1's LSP, R2 can't know R1's connections.

  With FLOODING:
    R1 sends its LSP out BOTH the R1-R2 AND R1-R3 links.
    R3 also floods R1's LSP onwards (via R3-R2, R3-R5).
    Even if R1-R2 fails: R2 gets R1's LSP through R3!
    Multiple redundant paths = reliable delivery.

  This is the TRADE-OFF:
    ✅ High reliability — packets reach everywhere via multiple paths
    ✅ Fast convergence — all routers update simultaneously
    ❌ High bandwidth consumption — same LSP delivered multiple times via multiple paths
    ❌ More congestion — network is flooded with LSPs during convergence
    ❌ More processing — every router must handle duplicate LSPs (check seq number)
```

---

### 5.3 Flooding Problems — Sequence Number and TTL

```
PROBLEM 1 — DUPLICATE LSPs:

  Flooding sends the same LSP through multiple paths.
  A router may receive the SAME LSP 2, 3, or more times.
  Without a mechanism to detect this: the router would process and re-flood
  the same LSP endlessly → infinite flooding loop.

  SOLUTION: SEQUENCE NUMBER (32-bit counter)
    → When router R1 creates an LSP, it assigns Sequence Number = 1
    → Every router that receives R1's LSP records: "Seen R1's LSP with seq=1"
    → If the SAME LSP arrives again (same Router ID, same seq):
        → "Already processed this!" → DISCARD immediately → do NOT re-flood
    → When R1 updates its LSP (link changed): it increments seq to 2
        → Routers see seq=2 > seq=1 → NEW information → process and re-flood

  32-bit Sequence Number:
    → Range: 0 to 4,294,967,295 (about 4.3 billion)
    → High enough that wraparound is not a practical concern

PROBLEM 2 — INFINITE LOOP (Flooding Loop):

  In rare cases, routing loops in the NETWORK GRAPH itself can cause
  an LSP to loop forever: R1 → R2 → R3 → R2 → R3 → R2 → ...
  Sequence numbers alone might not stop this if both copies are "new".

  SOLUTION: TTL (Time To Live)
    → LSP starts with a TTL value (e.g., TTL = 7)
    → EVERY router that forwards the LSP DECREMENTS TTL by 1
    → When TTL reaches 0: router DROPS the LSP — does NOT forward
    → Even if the LSP is stuck in a loop, it will die after TTL hops

  TTL in Link State = same concept as TTL in IP headers
    → IP TTL: how many router hops a PACKET can survive
    → LSP TTL: how many router hops an LSP can survive during flooding

PROBLEM 3 — OUT-OF-ORDER ARRIVAL:

  LSPs can arrive out of order if they take different paths.
  A router might receive R1's seq=3 BEFORE seq=2.
  → The router should accept seq=3 (it's newer)
  → If seq=2 arrives later: DISCARD it (older information)
  → Always keep the HIGHEST sequence number seen so far
```

---

## 6. Step 3 — Building the Global Database

After flooding, every router has received LSPs from EVERY other router. Now each router builds its **Link State Database (LSDB)**:

```
LINK STATE DATABASE (LSDB) at R1 — after receiving all flooded LSPs:

  R1 now knows the COMPLETE network topology:

  ┌──────────────────────────────────────────────────────────┐
  │             R1's LINK STATE DATABASE                      │
  ├───────────────┬──────────────────────────────────────────┤
  │  Source       │  Connections Learned                      │
  ├───────────────┼──────────────────────────────────────────┤
  │  R1 (self)    │  R2 (cost 6),  R3 (cost 3)               │
  ├───────────────┼──────────────────────────────────────────┤
  │  R2 (LSP rcv) │  R1 (cost 6),  R3 (cost 2),  R4 (cost 7)│
  ├───────────────┼──────────────────────────────────────────┤
  │  R3 (LSP rcv) │  R1 (cost 3),  R2 (cost 2),  R5 (cost 9)│
  ├───────────────┼──────────────────────────────────────────┤
  │  R4 (LSP rcv) │  R2 (cost 7),  R5 (cost 1),  R6 (cost 8)│
  ├───────────────┼──────────────────────────────────────────┤
  │  R5 (LSP rcv) │  R3 (cost 9),  R4 (cost 1),  R6 (cost 4)│
  ├───────────────┼──────────────────────────────────────────┤
  │  R6 (LSP rcv) │  R4 (cost 8),  R5 (cost 4)               │
  └───────────────┴──────────────────────────────────────────┘

  This database is called: GLOBAL KNOWLEDGE
  R1 now has a COMPLETE MAP of the entire network.

  IMPORTANT: Every other router (R2, R3, R4, R5, R6) builds THE SAME database.
  Because they ALL receive ALL LSPs via flooding.
  Every router ends up with an IDENTICAL copy of the LSDB.

This is fundamentally different from Distance Vector:
  Distance Vector: each router has ONLY its own view (local knowledge)
  Link State:      every router has the COMPLETE topology (global knowledge)
```

---

## 7. Step 4 — Dijkstra's Algorithm (SSSP)

### 7.1 What is Dijkstra's Algorithm?

```
DIJKSTRA'S ALGORITHM:
  Full name: Dijkstra's Shortest Path Algorithm
  Type:      Single Source Shortest Path (SSSP)
  Input:     A weighted graph (the LSDB — the complete network topology)
             A source node (the router running the algorithm)
  Output:    Shortest path cost from the source to EVERY other node

  Why "Single Source"?
    The algorithm calculates shortest paths FROM ONE specific router
    to ALL others.
    R1 runs Dijkstra with R1 as source → gets shortest paths from R1 to all.
    R2 runs the SAME algorithm with R2 as source → gets shortest paths from R2 to all.
    Every router runs it independently, each as their own source.

  Why does Link State use Dijkstra and Distance Vector doesn't?
    Dijkstra REQUIRES a complete map of the network to work.
    Distance Vector routers don't have a complete map — only local knowledge.
    Link State routers DO have a complete map (the LSDB) — so Dijkstra is possible.

Algorithm steps:
  1. Start: source node has distance 0, all others have distance ∞ (unknown)
  2. Pick the UNVISITED node with the MINIMUM current distance
  3. Mark it as VISITED (finalised — its shortest distance is now known)
  4. Update distances of all its UNVISITED neighbours:
       new_distance = current_node_distance + link_cost_to_neighbour
       if new_distance < known_distance: UPDATE
  5. Repeat from step 2 until all nodes are visited

  Once a node is VISITED (marked finalised): its distance NEVER changes.
  This is the key property of Dijkstra — greedy, always picks the minimum.
```

---

### 7.2 Running Dijkstra from R1 — Full Walkthrough

```
SOURCE: R1
GOAL:   Find shortest distance from R1 to R2, R3, R4, R5, R6

INITIAL STATE:
  R1 = 0       (source — we start here)
  R2 = ∞       (unknown)
  R3 = ∞       (unknown)
  R4 = ∞       (unknown)
  R5 = ∞       (unknown)
  R6 = ∞       (unknown)
  Visited set: {}

────────────────────────────────────────────────────────
ITERATION 1: Pick minimum distance unvisited node = R1 (cost 0)
────────────────────────────────────────────────────────
  Mark R1 as VISITED. Visited set: {R1}

  Update R1's neighbours:
    R2: dist = 0 + 6 = 6   → 6 < ∞  → UPDATE  R2 = 6   (via R1)
    R3: dist = 0 + 3 = 3   → 3 < ∞  → UPDATE  R3 = 3   (via R1)

  Current distances:
    R1=0✅  R2=6  R3=3  R4=∞  R5=∞  R6=∞

────────────────────────────────────────────────────────
ITERATION 2: Pick minimum distance unvisited node = R3 (cost 3)
────────────────────────────────────────────────────────
  Mark R3 as VISITED. Visited set: {R1, R3}

  Update R3's unvisited neighbours:
    R2: new dist via R3 = 3 + 2 = 5  → 5 < 6  → UPDATE  R2 = 5   (via R3)
    R5: new dist via R3 = 3 + 9 = 12 → 12 < ∞ → UPDATE  R5 = 12  (via R3)
    (R1 is already visited — skip)

  Current distances:
    R1=0✅  R2=5  R3=3✅  R4=∞  R5=12  R6=∞

  Key observation: R1→R2 directly costs 6.
                   R1→R3→R2 costs only 3+2 = 5.
                   Going THROUGH R3 to R2 is CHEAPER!
                   Dijkstra correctly found this.

────────────────────────────────────────────────────────
ITERATION 3: Pick minimum distance unvisited node = R2 (cost 5)
────────────────────────────────────────────────────────
  Mark R2 as VISITED. Visited set: {R1, R3, R2}

  Update R2's unvisited neighbours:
    R4: new dist via R2 = 5 + 7 = 12 → 12 < ∞ → UPDATE  R4 = 12  (via R3→R2)
    (R1 and R3 already visited — skip)

  Current distances:
    R1=0✅  R2=5✅  R3=3✅  R4=12  R5=12  R6=∞

────────────────────────────────────────────────────────
ITERATION 4: Pick minimum distance unvisited node = R4 or R5 (both cost 12)
             Choose R4 (or R5 — tie-breaking by label order)
────────────────────────────────────────────────────────
  Mark R4 as VISITED. Visited set: {R1, R3, R2, R4}

  Update R4's unvisited neighbours:
    R5: new dist via R4 = 12 + 1 = 13 → 13 > 12 → NO UPDATE (R5 stays at 12)
    R6: new dist via R4 = 12 + 8 = 20 → 20 < ∞  → UPDATE  R6 = 20  (via R3→R2→R4)
    (R2 already visited — skip)

  Current distances:
    R1=0✅  R2=5✅  R3=3✅  R4=12✅  R5=12  R6=20

────────────────────────────────────────────────────────
ITERATION 5: Pick minimum distance unvisited node = R5 (cost 12)
────────────────────────────────────────────────────────
  Mark R5 as VISITED. Visited set: {R1, R3, R2, R4, R5}

  Update R5's unvisited neighbours:
    R6: new dist via R5 = 12 + 4 = 16 → 16 < 20 → UPDATE  R6 = 16  (via R3→R5)
    (R3 and R4 already visited — skip)

  Current distances:
    R1=0✅  R2=5✅  R3=3✅  R4=12✅  R5=12✅  R6=16

────────────────────────────────────────────────────────
ITERATION 6: Pick minimum distance unvisited node = R6 (cost 16)
────────────────────────────────────────────────────────
  Mark R6 as VISITED. Visited set: {R1, R3, R2, R4, R5, R6}

  No unvisited neighbours remain.

  ALGORITHM COMPLETE.

FINAL SHORTEST DISTANCES FROM R1:
  R1 → R1: 0
  R1 → R3: 3   (direct link)
  R1 → R2: 5   (via R3)
  R1 → R4: 12  (via R3 → R2)
  R1 → R5: 12  (via R3)
  R1 → R6: 16  (via R3 → R5)
```

---

### 7.3 Dijkstra Iteration Table

```
DIJKSTRA ITERATION TABLE (Standard Exam Format):

  Format: D(node) = shortest known distance, predecessor shown in path

  Iter │ Visited  │  D(R2)   │  D(R3)   │  D(R4)   │  D(R5)   │  D(R6)
  ─────┼──────────┼──────────┼──────────┼──────────┼──────────┼──────────
  Init │  {R1}    │  6,R1    │  3,R1    │   ∞      │   ∞      │   ∞
   1   │  {R1,R3} │  5,R3    │  3,R1✅  │   ∞      │  12,R3   │   ∞
   2   │+{R2}     │  5,R3✅  │  (done)  │  12,R2   │  12,R3   │   ∞
   3   │+{R4}     │  (done)  │  (done)  │  12,R2✅ │  12,R3   │  20,R4
   4   │+{R5}     │  (done)  │  (done)  │  (done)  │  12,R3✅ │  16,R5
   5   │+{R6}     │  (done)  │  (done)  │  (done)  │  (done)  │  16,R5✅

  ✅ = node finalised (visited) in that iteration
  Format of cells: cost, predecessor (which router was the previous hop in shortest path)

FINAL RESULTS:
  Destination │ Shortest Cost │ Predecessor │ Full Path from R1
  ────────────┼───────────────┼─────────────┼─────────────────────
  R2          │ 5             │ R3          │ R1 → R3 → R2
  R3          │ 3             │ R1          │ R1 → R3
  R4          │ 12            │ R2          │ R1 → R3 → R2 → R4
  R5          │ 12            │ R3          │ R1 → R3 → R5
  R6          │ 16            │ R5          │ R1 → R3 → R5 → R6
```

---

## 8. Step 5 — Building the Routing Table

From Dijkstra's output, R1 now builds its **routing table**:

```
R1's ROUTING TABLE (final output):

  ┌─────────────┬──────────────┬────────────────────────┬──────────────────────────┐
  │ Destination │ Shortest Cost│ Next Hop (via)         │ Full Path                │
  ├─────────────┼──────────────┼────────────────────────┼──────────────────────────┤
  │ R1          │ 0            │ R1 (self)              │ R1                       │
  │ R2          │ 5            │ R3                     │ R1 → R3 → R2             │
  │ R3          │ 3            │ R3 (direct)            │ R1 → R3                  │
  │ R4          │ 12           │ R3                     │ R1 → R3 → R2 → R4       │
  │ R5          │ 12           │ R3                     │ R1 → R3 → R5             │
  │ R6          │ 16           │ R3                     │ R1 → R3 → R5 → R6       │
  └─────────────┴──────────────┴────────────────────────┴──────────────────────────┘

  "Next Hop" = the immediate next router R1 should forward to.
  Even though R4 is reached via R3→R2→R4, R1's next hop is just R3.
  R1 doesn't need to know the FULL PATH — it only needs the NEXT HOP.
  Each router along the path makes its own forwarding decision.

Key insight: R1→R2 direct link (cost 6) is NOT used for routing to R2!
  R1 routes to R2 via R3 (cost 5) because Dijkstra found a shorter path.
  The routing table does NOT always use the direct link — it uses the SHORTEST path.
```

```
How this routing table is USED:

  When R1 receives a packet destined for R6:
    1. R1 looks up R6 in routing table
    2. Next hop = R3
    3. R1 forwards the packet to R3 (via the R1-R3 link)
    4. R3 receives it, looks up R6 in ITS OWN routing table
    5. R3 sends to R5
    6. R5 sends to R6
    7. Packet arrives at R6

  R1 does NOT need to know the full path R1→R3→R5→R6.
  It only needs to know: "for R6, send to R3".
  R3 handles the rest.
```

---

## 9. Link State vs Distance Vector — Full Comparison

| Property                     | Distance Vector                                 | Link State                                      |
| ---------------------------- | ----------------------------------------------- | ----------------------------------------------- |
| **Information shared**       | Distance vectors (cost to destinations)         | Link State Packets (direct connections + costs) |
| **Shared with**              | NEIGHBOURS ONLY                                 | EVERYONE (via flooding)                         |
| **Knowledge at each router** | Local (partial view of network)                 | Global (complete topology map)                  |
| **Algorithm used**           | Bellman-Ford (distributed)                      | Dijkstra's (run locally at each router)         |
| **Convergence speed**        | Slow (hop-by-hop update propagation)            | Fast (flooding reaches all routers quickly)     |
| **Routing loops**            | Possible (count to infinity problem)            | Not possible (full topology → no loops)         |
| **Bandwidth usage**          | Lower (small distance vectors, neighbours only) | Higher (large LSPs, flooding everywhere)        |
| **CPU/Memory usage**         | Lower (simple calculation)                      | Higher (Dijkstra + LSDB storage)                |
| **Scalability**              | Better for small networks                       | Better for large networks (faster convergence)  |
| **Protocols**                | RIP (Routing Information Protocol)              | OSPF, IS-IS                                     |
| **Count to infinity**        | YES — major problem                             | NO — solved by global knowledge                 |
| **Loop prevention**          | Slow (split horizon, route poisoning)           | Built-in (Dijkstra produces loop-free tree)     |
| **TTL / Sequence Numbers**   | Not needed                                      | REQUIRED (for flooding control)                 |

```
Which is better?
  For small networks (< 15 routers): Distance Vector (RIP) is simpler, fine
  For large enterprise/ISP networks: Link State (OSPF/IS-IS) — faster convergence,
  no count-to-infinity, scales better despite higher resource usage
```

---

## 10. Security Analysis — Attacking Link State Routing

> **Note:** All techniques are for educational use only. Test only on your own lab (Parrot OS, Metasploitable2, your own MERN/PERN projects, GNS3/EVE-NG simulations).

OSPF (which uses Link State Routing) is widely deployed in real enterprise networks. Understanding its attacks is essential for both penetration testers and network defenders.

### 10.1 LSA Spoofing / Fake LSP Injection

```
WHAT IT IS:
  An attacker injects a FAKE Link State Advertisement (LSA) into the network.
  The fake LSA claims to be from a legitimate router (e.g., claims to be R3's LSA).
  If accepted, it poisons the LSDB of every router in the network.
  Result: all routers calculate incorrect shortest paths → traffic misdirected.

HOW IT WORKS (conceptual):
  1. Attacker gains access to a device on the OSPF network (or compromises a router)
  2. Attacker crafts a fake LSA:
       "I am R3. My links are: R1 (cost 1), ATTACKER_ROUTER (cost 1)"
       (This adds a fake low-cost link to the attacker's controlled router)
  3. Attacker injects this fake LSA into the network
  4. All routers receive it via flooding, update their LSDB
  5. Dijkstra runs → routes through attacker's router calculated as shortest path
  6. Traffic now flows through the attacker → Man-in-the-Middle achieved

WHY SEQUENCE NUMBERS DON'T ALWAYS HELP:
  If the attacker uses a HIGHER sequence number than the real R3's current LSA:
  → All routers accept the fake LSA (higher seq = newer information)
  → Real R3 will eventually re-flood its own LSA with an even higher seq
  → This causes an ongoing "LSA war" (attacker and real router keep incrementing)
  → Can cause network instability even if the attacker doesn't "win"

DETECTION:
  → Unexplained route changes in routing tables
  → OSPF log entries showing sequence number conflicts
  → Network monitoring: compare expected LSDB with actual LSDB

DEFENCE:
  → OSPF authentication (MD5 or SHA-based HMAC on all OSPF packets)
  → Only authenticated routers can inject LSAs
  → Cryptographic sequence: attacker can't forge a valid HMAC without the key
```

```bash
# Lab simulation: Observe OSPF LSA flooding on your GNS3/EVE-NG lab

# If you have a router running OSPF (e.g., a Cisco router in GNS3):
# On the Cisco router:
# show ip ospf database          → displays the full LSDB
# show ip ospf neighbor          → shows neighbour relationships
# debug ip ospf events           → watch LSA flooding in real time

# On Parrot OS with a real OSPF network in your lab:
# Capture OSPF traffic (Protocol 89):
sudo tcpdump -i eth0 -n "proto 89" -v -w /tmp/ospf_capture.pcap

# Open in Wireshark:
wireshark /tmp/ospf_capture.pcap &
# Filter: ospf
# You'll see:
#   OSPF Hello packets (neighbour discovery)
#   OSPF DBD (Database Description — initial LSDB exchange)
#   OSPF LSR (Link State Request)
#   OSPF LSU (Link State Update — contains the actual LSAs)
#   OSPF LSAck (acknowledgement)
```

---

### 10.2 Database Overflow Attack

```
WHAT IT IS:
  Attacker floods the network with massive numbers of FAKE LSAs.
  Each LSA is different (different sequence numbers, different "routers").
  Target: overflow the LSDB memory of legitimate routers.

HOW IT WORKS:
  Normal LSDB size: proportional to network size (manageable)
  Under attack: attacker generates thousands of fake LSAs rapidly
  Router memory fills up → OSPF process crashes → routing breaks → DoS

IMPACT:
  → Router CPU spikes (Dijkstra runs repeatedly on huge "network")
  → Router memory exhausted (LSDB too large)
  → OSPF process crashes → router stops routing → network outage

DEFENCE:
  → OSPF authentication prevents injection of fake LSAs
  → OSPF LSA rate limiting (max-lsa command in Cisco IOS)
  → Monitoring: alert on sudden LSDB size increases
```

---

### 10.3 Routing Table Poisoning via Flooding Abuse

```
WHAT IT IS:
  If an attacker controls a legitimate router (or has compromised one),
  they can inject real but malicious LSAs — advertising fake low-cost routes
  to attract traffic through the attacker's controlled path.

SCENARIO — Traffic Hijacking:
  Normal shortest path R1 → R3 → R5 → R6 (cost 16)
  Attacker compromises R3 and changes its LSA:
    "R3 is now connected to EVIL_ROUTER (cost 1)"
    "EVIL_ROUTER is connected to R6 (cost 1)"
  New "shortest path": R1 → R3 → EVIL_ROUTER → R6 (cost 5)
  All traffic for R6 now flows through EVIL_ROUTER → full MitM

REAL-WORLD EQUIVALENT:
  BGP hijacking (at the internet scale) works on a similar principle.
  Attackers advertise routes with lower cost → attract traffic.
  Famous example: YouTube BGP hijack by Pakistan Telecom (2008).

DEFENCE:
  → OSPF authentication (HMAC) on all LSAs
  → OSPF passive interfaces (don't run OSPF on interfaces not needed)
  → Route filtering (only accept LSAs from known, trusted sources)
  → Network monitoring with route change alerting
```

---

### 10.4 Reconnaissance via OSPF/IS-IS Sniffing

```
WHAT IT IS:
  OSPF flooding broadcasts the COMPLETE network topology to every router.
  If an attacker can sniff OSPF packets: they get the FULL NETWORK MAP for free.

WHAT AN ATTACKER LEARNS FROM OSPF CAPTURE:
  → Every router's IP address in the network
  → Every link in the network (who is connected to whom)
  → Link costs (can infer network capacity/importance)
  → Router IDs → often match management IP addresses
  → Area structure (OSPF areas reveal network design)
  → Which routers are ABRs (Area Border Routers) — high-value targets

This is essentially getting the network diagram handed to you.
An attacker who can capture OSPF traffic needs ZERO additional scanning.

CAPTURE OSPF TRAFFIC (on your own lab only):
```

```bash
# Capture OSPF LSU packets (contain the full LSA database):
sudo tcpdump -i eth0 -n "proto 89" -w /tmp/ospf_recon.pcap -v

# In Wireshark:
# Filter: ospf.msg.lsupdate    → shows Link State Updates (LSAs)
# Expand: OSPF → LSA → Router LSA → Link information
# You'll see every router's connections and costs laid out perfectly

# Extract router IDs and connections with tshark:
tshark -r /tmp/ospf_recon.pcap \
  -T fields \
  -e ospf.advrouter \
  -e ospf.link.id \
  -e ospf.link.metric \
  -Y "ospf.msg.lsupdate" \
  -q 2>/dev/null | sort | uniq
# Output: complete topology table extracted from passive sniffing

# Defence: Use OSPF authentication so packets are harder to parse
# Even with HMAC auth: the topology info is still flooded (authenticated, not encrypted)
# For true confidentiality at routing layer: use IPsec over OSPF links
# (combine IPsec Transport Mode with OSPF — packets are encrypted before being flooded)
```

---

## 11. 🧪 Practical Labs

### Lab 1 — Simulate Link State Routing with Python (Dijkstra)

```python
#!/usr/bin/env python3
# Save as: link_state_dijkstra.py
# Run:     python3 link_state_dijkstra.py
# PURPOSE: Simulate the complete Link State Routing process in code

import heapq
from collections import defaultdict

# ─── Network Topology from the lecture ───────────────────────────────────────

# Adjacency list: graph[router] = [(neighbour, cost), ...]
graph = {
    'R1': [('R2', 6), ('R3', 3)],
    'R2': [('R1', 6), ('R3', 2), ('R4', 7)],
    'R3': [('R1', 3), ('R2', 2), ('R5', 9)],
    'R4': [('R2', 7), ('R5', 1), ('R6', 8)],
    'R5': [('R3', 9), ('R4', 1), ('R6', 4)],
    'R6': [('R4', 8), ('R5', 4)],
}

routers = list(graph.keys())

# ─── Step 1: Build Link State Tables ─────────────────────────────────────────

print("="*65)
print("STEP 1: LINK STATE TABLES (what each router knows initially)")
print("="*65)

for router in routers:
    print(f"\n  {router}'s Link State Table:")
    print(f"  {'Neighbour':<12} {'Cost':<6}")
    print(f"  {'─'*9:<12} {'─'*4:<6}")
    for neighbour, cost in graph[router]:
        print(f"  {neighbour:<12} {cost:<6}")

# ─── Step 2: Flooding Simulation ──────────────────────────────────────────────

print("\n" + "="*65)
print("STEP 2: FLOODING — sharing LSPs with entire network")
print("="*65)

# Simulate flooding: after flooding, every router has the COMPLETE graph
# (In reality this happens via the flooding mechanism described in notes)
print("\n  After flooding:")
print("  Every router now has an identical copy of the complete network.")
print("  GLOBAL KNOWLEDGE achieved at every node.")
print(f"\n  All 6 routers now know about {len(routers)} routers and {sum(len(v) for v in graph.values())//2} links")

# ─── Step 3: Dijkstra's Algorithm ─────────────────────────────────────────────

def dijkstra(graph, source):
    """
    Dijkstra's Shortest Path Algorithm
    Returns: (distances dict, predecessors dict)
    """
    # Initialize all distances to infinity
    distances = {node: float('inf') for node in graph}
    distances[source] = 0
    predecessors = {node: None for node in graph}

    # Priority queue: (cost, node)
    pq = [(0, source)]
    visited = set()

    while pq:
        current_cost, current_node = heapq.heappop(pq)

        # Skip if already visited
        if current_node in visited:
            continue
        visited.add(current_node)

        # Update neighbours
        for neighbour, link_cost in graph[current_node]:
            if neighbour not in visited:
                new_cost = current_cost + link_cost
                if new_cost < distances[neighbour]:
                    distances[neighbour] = new_cost
                    predecessors[neighbour] = current_node
                    heapq.heappush(pq, (new_cost, neighbour))

    return distances, predecessors

def get_path(predecessors, source, dest):
    """Reconstruct the full path from source to destination"""
    path = []
    current = dest
    while current is not None:
        path.append(current)
        current = predecessors[current]
    return ' → '.join(reversed(path))

print("\n" + "="*65)
print("STEP 3 & 4: DIJKSTRA'S ALGORITHM + ROUTING TABLE CONSTRUCTION")
print("="*65)

# Run Dijkstra from each router (each builds its own routing table)
for source in routers:
    distances, predecessors = dijkstra(graph, source)

    print(f"\n{'─'*65}")
    print(f"  ROUTING TABLE for {source} (source = {source})")
    print(f"{'─'*65}")
    print(f"  {'Dest':<8} {'Cost':<8} {'Next Hop':<12} {'Full Path'}")
    print(f"  {'─'*4:<8} {'─'*4:<8} {'─'*8:<12} {'─'*20}")

    for dest in routers:
        if dest == source:
            print(f"  {dest:<8} {'0':<8} {'(self)':<12} {source}")
        else:
            cost = distances[dest]
            path = get_path(predecessors, source, dest)
            # Next hop = first router after source in the path
            path_nodes = path.split(' → ')
            next_hop = path_nodes[1] if len(path_nodes) > 1 else '(direct)'
            print(f"  {dest:<8} {cost:<8} {next_hop:<12} {path}")

# ─── Step 4: Verify R1's routing table matches the lecture ────────────────────

print("\n" + "="*65)
print("VERIFICATION: R1's routing table (should match lecture)")
print("="*65)
distances_r1, pred_r1 = dijkstra(graph, 'R1')
expected = {'R2': 5, 'R3': 3, 'R4': 12, 'R5': 12, 'R6': 16}
all_correct = True
for dest, expected_cost in expected.items():
    actual = distances_r1[dest]
    status = "✅" if actual == expected_cost else "❌ MISMATCH"
    print(f"  R1 → {dest}: Expected={expected_cost}, Got={actual} {status}")
    if actual != expected_cost:
        all_correct = False

print(f"\n  All values match lecture: {'✅ YES' if all_correct else '❌ NO'}")

# ─── Bonus: Link State Packet Simulation ──────────────────────────────────────

print("\n" + "="*65)
print("BONUS: LINK STATE PACKET (LSP) SIMULATION")
print("="*65)

class LinkStatePacket:
    def __init__(self, router_id, seq_num, ttl, neighbours):
        self.router_id  = router_id
        self.seq_num    = seq_num
        self.ttl        = ttl
        self.neighbours = neighbours  # [(neighbour, cost), ...]

    def __repr__(self):
        return (f"LSP(from={self.router_id}, seq={self.seq_num}, "
                f"ttl={self.ttl}, neighbours={self.neighbours})")

# Simulate R1 creating and flooding its LSP
r1_lsp = LinkStatePacket('R1', seq_num=1, ttl=7, neighbours=[('R2', 6), ('R3', 3)])
print(f"\n  R1 creates its LSP: {r1_lsp}")

# Simulate flooding hop by hop
print(f"\n  Flooding simulation:")
current_ttl = r1_lsp.ttl
for hop in range(1, 7):
    current_ttl -= 1
    status = "FORWARDED" if current_ttl > 0 else "DROPPED (TTL=0)"
    print(f"    Hop {hop}: TTL decremented to {current_ttl} → {status}")
    if current_ttl == 0:
        break

# Simulate duplicate detection with sequence numbers
print(f"\n  Sequence number duplicate detection:")
received_lsps = {}  # router_id → highest seq seen

def receive_lsp(lsp):
    rid = lsp.router_id
    if rid not in received_lsps:
        received_lsps[rid] = lsp.seq_num
        return f"ACCEPTED (first LSP from {rid})"
    elif lsp.seq_num > received_lsps[rid]:
        received_lsps[rid] = lsp.seq_num
        return f"ACCEPTED (new seq={lsp.seq_num} > old={received_lsps[rid]-1})"
    else:
        return f"DISCARDED (duplicate or old: seq={lsp.seq_num} ≤ seen={received_lsps[rid]})"

lsp1 = LinkStatePacket('R1', seq_num=1, ttl=7, neighbours=[('R2', 6), ('R3', 3)])
lsp1_dup = LinkStatePacket('R1', seq_num=1, ttl=6, neighbours=[('R2', 6), ('R3', 3)])
lsp2 = LinkStatePacket('R1', seq_num=2, ttl=7, neighbours=[('R2', 6), ('R3', 3), ('R4', 1)])

print(f"    Receive seq=1: {receive_lsp(lsp1)}")
print(f"    Receive seq=1 again (duplicate): {receive_lsp(lsp1_dup)}")
print(f"    Receive seq=2 (updated LSP): {receive_lsp(lsp2)}")
```

---

### Lab 2 — Capture and Analyse Real OSPF Traffic

```bash
# PURPOSE: Observe Link State Routing (OSPF) in action on a real network
# REQUIREMENT: A lab with at least two OSPF-enabled routers
#              Options: GNS3, EVE-NG, real Cisco/Juniper equipment, or FRRouting on Linux

# ── Option A: FRRouting (free, runs on Linux, including Parrot OS) ────────────

# Install FRRouting:
sudo apt update && sudo apt install frr -y

# Enable OSPF daemon:
sudo sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons
sudo systemctl restart frr

# Configure OSPF on a loopback network (safe, no external network needed):
sudo vtysh << 'EOF'
configure terminal
router ospf
  ospf router-id 1.1.1.1
  network 192.168.56.0/24 area 0
  network 10.0.0.0/8 area 0
exit
interface lo
  ip address 1.1.1.1/32
exit
EOF

# View OSPF status:
sudo vtysh -c "show ip ospf"
sudo vtysh -c "show ip ospf database"        # The LSDB — global knowledge!
sudo vtysh -c "show ip ospf neighbor"        # Neighbour table
sudo vtysh -c "show ip route ospf"           # Routes learned via OSPF

# ── Option B: Capture OSPF on a network with existing OSPF routers ───────────

# OSPF uses Protocol number 89 (not TCP/UDP — direct IP protocol)
# OSPF multicast addresses:
#   224.0.0.5 — AllSPFRouters (all OSPF routers listen here)
#   224.0.0.6 — AllDRouters (Designated Routers only)

# Capture OSPF Hello packets (neighbour discovery):
sudo tcpdump -i eth0 -n "proto 89 and host 224.0.0.5" -v

# Capture OSPF Link State Updates (LSAs — the actual topology data):
sudo tcpdump -i eth0 -n "proto 89" -v -w /tmp/ospf_full.pcap

# Analyse in Wireshark:
wireshark /tmp/ospf_full.pcap &

# Wireshark OSPF filters:
#   ospf                     → all OSPF packets
#   ospf.msg == 1            → Hello packets (neighbour discovery)
#   ospf.msg == 2            → DBD (Database Description — initial sync)
#   ospf.msg == 4            → LSU (Link State Update — actual LSAs!)
#   ospf.lsa                 → packets containing LSAs
#   ospf.advrouter == X.X.X.X → LSAs from a specific router

# Extract topology from captured OSPF:
tshark -r /tmp/ospf_full.pcap \
  -Y "ospf.msg == 4" \
  -T fields \
  -e ospf.advrouter \
  -e ospf.link.id \
  -e ospf.link.data \
  -e ospf.link.metric \
  -q | sort | uniq
# This gives you: advertising router, link ID, link data, metric
# = complete topology from passive capture alone
```

---

### Lab 3 — Link State Security: Test OSPF Authentication

```bash
# PURPOSE: Configure OSPF MD5 authentication and verify it blocks unauthenticated LSAs
# This demonstrates the defence against LSA spoofing attacks

# ── Configure OSPF with MD5 authentication in FRRouting ──────────────────────

sudo vtysh << 'EOF'
configure terminal
interface eth0
  ip ospf authentication message-digest
  ip ospf message-digest-key 1 md5 MySecretOSPFKey123!
exit
router ospf
  area 0 authentication message-digest
exit
EOF

# Verify authentication is configured:
sudo vtysh -c "show ip ospf interface eth0"
# Look for: "Cryptographic authentication" in the output

# ── Capture authenticated vs unauthenticated OSPF ─────────────────────────────

# Capture OSPF traffic:
sudo tcpdump -i eth0 -n "proto 89" -v -w /tmp/ospf_auth.pcap

# In Wireshark:
# ospf.auth.type == 2    → MD5 authenticated OSPF packets
# You'll see: "Auth Crypt: MD5 (2)" and the HMAC digest field
# Without the key: an attacker cannot forge the HMAC → LSA injection blocked

# ── Demonstrate what happens without authentication (on separate test VM) ─────

# On a second machine WITHOUT OSPF auth configured:
# Try to form OSPF neighbour relationship with the authenticated router

# On second machine:
sudo vtysh << 'EOF'
configure terminal
router ospf
  ospf router-id 2.2.2.2
  network 192.168.56.0/24 area 0
exit
EOF

# Without matching auth key: neighbour relationship WILL NOT FORM
sudo vtysh -c "show ip ospf neighbor"
# Expected: no neighbours (authentication mismatch = rejected)

# Check logs for authentication failure:
sudo tail -f /var/log/frr/ospfd.log | grep -i "auth"
# You'll see: "OSPF packet from [IP] discarded: auth type mismatch"
# This is exactly what stops LSA spoofing in production networks

# Conclusion:
# Without OSPF auth: attacker on the network can inject any LSA → routing manipulation
# With OSPF auth (MD5/SHA):  attacker cannot forge valid LSA → topology protected
```

---

### Lab 4 — Visualise Dijkstra Step by Step

```python
#!/usr/bin/env python3
# Save as: dijkstra_visualiser.py
# Run:     python3 dijkstra_visualiser.py
# PURPOSE: Print Dijkstra's algorithm step by step in exam table format

import heapq

graph = {
    'R1': [('R2', 6), ('R3', 3)],
    'R2': [('R1', 6), ('R3', 2), ('R4', 7)],
    'R3': [('R1', 3), ('R2', 2), ('R5', 9)],
    'R4': [('R2', 7), ('R5', 1), ('R6', 8)],
    'R5': [('R3', 9), ('R4', 1), ('R6', 4)],
    'R6': [('R4', 8), ('R5', 4)],
}

def dijkstra_verbose(graph, source):
    routers = sorted(graph.keys())
    dests = [r for r in routers if r != source]

    distances = {node: float('inf') for node in graph}
    distances[source] = 0
    predecessors = {node: None for node in graph}
    visited = set()
    pq = [(0, source)]
    iteration = 0

    # Print header
    header = f"  {'Iter':<6} {'Visited':<20} " + "  ".join(f"D({r})" for r in dests)
    print(header)
    print("  " + "─" * (len(header) - 2))

    def print_row(iter_num, visited_set):
        visited_str = "{" + ",".join(sorted(visited_set)) + "}"
        row = f"  {iter_num:<6} {visited_str:<20} "
        for r in dests:
            if distances[r] == float('inf'):
                cell = "∞"
            else:
                cell = f"{distances[r]}"
                if predecessors[r]:
                    cell += f",{predecessors[r]}"
                if r in visited_set:
                    cell += "✅"
            row += f"{cell:<8}  "
        print(row)

    print_row(iteration, visited)

    while pq:
        current_cost, current_node = heapq.heappop(pq)
        if current_node in visited:
            continue
        visited.add(current_node)
        iteration += 1

        for neighbour, link_cost in graph[current_node]:
            if neighbour not in visited:
                new_cost = current_cost + link_cost
                if new_cost < distances[neighbour]:
                    distances[neighbour] = new_cost
                    predecessors[neighbour] = current_node
                    heapq.heappush(pq, (new_cost, neighbour))

        print_row(iteration, visited)

    return distances, predecessors

source = 'R1'
print(f"\nDijkstra's Algorithm — Source: {source}")
print("─" * 70)
distances, predecessors = dijkstra_verbose(graph, source)

print(f"\nFinal Routing Table for {source}:")
print(f"  {'Dest':<8} {'Cost':<8} {'Path'}")
print("  " + "─" * 40)
for dest in sorted(graph.keys()):
    if dest == source:
        print(f"  {dest:<8} {'0':<8} {source}")
    else:
        path = []
        cur = dest
        while cur:
            path.append(cur)
            cur = predecessors[cur]
        path.reverse()
        print(f"  {dest:<8} {distances[dest]:<8} {' → '.join(path)}")
```

---

## 12. Solved Examples

### Example 1 — Identify the Algorithm

**Question:** In a network, each router has complete knowledge of the entire network topology and independently computes shortest paths. Which routing algorithm is being used? Name one protocol that implements this.

```
Answer:
  Algorithm type: Link State Routing

  Characteristics that identify it:
    → "Complete knowledge of entire network topology" = Global Knowledge
      This is achieved via LSP flooding — every router receives every other
      router's Link State Packet.
    → "Independently computes shortest paths" = Dijkstra's algorithm run
      locally at each router, using the LSDB (global knowledge) as input.
    → Both of these characteristics are exclusive to Link State Routing.
      Distance Vector uses partial knowledge and Bellman-Ford (distributed).

  Protocol example: OSPF (Open Shortest Path First)
  Also valid:       IS-IS (Intermediate System to Intermediate System)

  Key terms to use in exam answers:
    → LSP (Link State Packet) — what is flooded
    → LSDB (Link State Database) — the global topology map at each router
    → Flooding — mechanism to share LSPs with all routers (not just neighbours)
    → Dijkstra's algorithm — used to compute shortest paths from LSDB
```

---

### Example 2 — Why Sequence Number and TTL?

**Question:** Link State Routing uses a Sequence Number and TTL field in its Link State Packets. Explain why each field is needed with one example for each.

```
Answer:

  SEQUENCE NUMBER (32-bit counter):

  Why needed: Flooding sends the same LSP through multiple paths.
  A router may receive the SAME LSP multiple times from different directions.
  Without sequence numbers: the router would process and re-flood duplicates
  endlessly, causing an infinite flooding storm.

  How it works:
    Router R3's LSP has seq=47. R1 receives it twice:
      → Once from R2 (via R1-R2-R3 path)
      → Once directly from R3 (via R1-R3 path)
    First arrival: R1 records "R3's LSP seq=47 received" → processes → floods onward
    Second arrival: R1 checks "Already have R3's LSP with seq=47" → DISCARDS
    Re-flooding is stopped → flooding storm prevented.

    When R3's topology changes: R3 creates new LSP with seq=48.
    All routers see 48 > 47 → NEW information → accept and re-flood.

  TTL (Time To Live):

  Why needed: Sequence numbers catch duplicates but NOT routing loops.
  If a loop exists in the network graph (R2→R3→R2→R3→...), an LSP could
  cycle through it forever even with sequence numbers (if it somehow escapes
  the duplicate detection — e.g., different paths with slightly different timing).
  TTL provides a hard limit: the packet MUST die after a maximum number of hops.

  How it works:
    R1 creates LSP with TTL=7.
    R2 receives it → decrements TTL to 6 → forwards
    R3 receives it → decrements TTL to 5 → forwards
    ... after 7 hops total → TTL = 0 → next router DROPS it, does not forward.
    Even if the packet is caught in a loop: it will die after 7 hops maximum.
```

---

### Example 3 — 5-Mark: Full Dijkstra Walkthrough

**Question:** Given the network in the lecture with 6 routers R1–R6, run Dijkstra's algorithm from source R3. Build the complete routing table for R3.

```
Network edges:
  R1-R2=6, R1-R3=3, R2-R3=2, R2-R4=7, R3-R5=9, R4-R5=1, R4-R6=8, R5-R6=4

SOURCE: R3
Initial: R3=0, R1=∞, R2=∞, R4=∞, R5=∞, R6=∞

Iteration 1: Visit R3 (cost 0). Mark visited: {R3}
  Update neighbours:
    R1: 0+3=3   < ∞   → R1=3  (via R3)
    R2: 0+2=2   < ∞   → R2=2  (via R3)
    R5: 0+9=9   < ∞   → R5=9  (via R3)
  Distances: R1=3  R2=2  R4=∞  R5=9  R6=∞

Iteration 2: Visit R2 (minimum unvisited = 2). Mark visited: {R3, R2}
  Update R2's unvisited neighbours:
    R1: 2+6=8   > 3   → NO UPDATE (R1 stays at 3 via R3)
    R4: 2+7=9   < ∞   → R4=9  (via R3→R2)
  Distances: R1=3  R2=2✅  R4=9  R5=9  R6=∞

Iteration 3: Visit R1 (minimum unvisited = 3). Mark visited: {R3, R2, R1}
  R1's unvisited neighbours: R2 (visited), R3 (visited) → no updates
  Distances: R1=3✅  R2=2✅  R4=9  R5=9  R6=∞

Iteration 4: Visit R4 or R5 (both = 9). Choose R4.
  Mark visited: {R3, R2, R1, R4}
  Update R4's unvisited neighbours:
    R5: 9+1=10  > 9   → NO UPDATE (R5 stays at 9 via R3)
    R6: 9+8=17  < ∞   → R6=17 (via R3→R2→R4)
  Distances: R1=3✅  R2=2✅  R4=9✅  R5=9  R6=17

Iteration 5: Visit R5 (cost 9). Mark visited: {R3,R2,R1,R4,R5}
  Update R5's unvisited neighbours:
    R6: 9+4=13  < 17  → UPDATE R6=13 (via R3→R5)
  Distances: R1=3✅  R2=2✅  R4=9✅  R5=9✅  R6=13

Iteration 6: Visit R6 (cost 13). Mark visited: all.
  No unvisited neighbours. DONE.

R3's ROUTING TABLE:
  Destination │ Cost │ Next Hop │ Full Path
  ────────────┼──────┼──────────┼──────────────────────
  R1          │  3   │ R1       │ R3 → R1
  R2          │  2   │ R2       │ R3 → R2
  R3          │  0   │ (self)   │ R3
  R4          │  9   │ R2       │ R3 → R2 → R4
  R5          │  9   │ R5       │ R3 → R5
  R6          │  13  │ R5       │ R3 → R5 → R6

Note: R3 reaches R4 via R2 (cost 9) — NOT via R5→R4 (cost 9+1=10).
      R3 reaches R6 via R5 (cost 13) — NOT via R2→R4→R6 (cost 2+7+8=17).
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════╗
║           LINK STATE ROUTING — EXAM CHEAT SHEET                          ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHAT IS LINK STATE ROUTING?                                              ║
║  → Dynamic routing algorithm — routers auto-build routing tables         ║
║  → Every router gets GLOBAL KNOWLEDGE (complete network map)             ║
║  → Achieved through: LSP FLOODING                                         ║
║  → Shortest paths computed with: DIJKSTRA'S ALGORITHM                   ║
║  → Protocols: OSPF (most common), IS-IS                                  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  THE 4 STEPS (MEMORISE THIS ORDER)                                        ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  STEP 1: Each router sends HELLO messages                                 ║
║          → Discovers neighbours + link costs                              ║
║          → Builds own LINK STATE TABLE (local knowledge)                 ║
║                                                                            ║
║  STEP 2: FLOODING                                                          ║
║          → Each router floods its LSP to EVERYONE (not just neighbours)  ║
║          → Every router receives every other router's LSP                 ║
║                                                                            ║
║  STEP 3: BUILD LSDB (Link State Database)                                ║
║          → From received LSPs, each router builds a COMPLETE MAP         ║
║          → All routers end up with IDENTICAL copies of the LSDB          ║
║          → This is called GLOBAL KNOWLEDGE                               ║
║                                                                            ║
║  STEP 4: DIJKSTRA'S ALGORITHM                                             ║
║          → Each router runs Dijkstra on its LSDB (with itself as source) ║
║          → Computes shortest path to ALL other routers                   ║
║          → Builds the ROUTING TABLE from the results                     ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LINK STATE PACKET (LSP) — WHAT IT CONTAINS                               ║
║  → Router ID:        which router generated this LSP                     ║
║  → Sequence Number:  32-bit counter (prevent duplicate processing)       ║
║  → TTL:             prevents infinite loop during flooding               ║
║  → Neighbour List:  (neighbour, cost) pairs — the actual topology info   ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHY SEQUENCE NUMBER? WHY TTL?                                             ║
║  Seq Number: Flooding sends same LSP via multiple paths → duplicates     ║
║              Seq number detects and discards duplicates                  ║
║              Higher seq = newer LSP → accept                             ║
║              Same or lower seq = duplicate/old → discard                 ║
║                                                                            ║
║  TTL:        Network loops can trap LSPs forever                         ║
║              Each router decrements TTL by 1 when forwarding            ║
║              TTL = 0 → DISCARD (never forward again)                     ║
║              Hard limit on how far an LSP can travel                     ║
╠══════════════════════════════════════════════════════════════════════════╣
║  FLOODING vs NEIGHBOUR SHARING                                             ║
║  Distance Vector: share with NEIGHBOURS ONLY (local reach)              ║
║  Link State:      FLOOD to EVERYONE (global reach)                       ║
║  Flooding trade-off:                                                      ║
║    ✅ Reliable (multiple paths = no single point of failure)             ║
║    ✅ Fast convergence (all routers update simultaneously)               ║
║    ❌ More bandwidth (same LSP delivered multiple times)                 ║
║    ❌ More processing (duplicate detection needed)                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  DIJKSTRA'S ALGORITHM — QUICK RULES                                        ║
║  1. Start: source=0, all others=∞                                        ║
║  2. Pick MINIMUM cost UNVISITED node → mark VISITED (finalised)          ║
║  3. Update all UNVISITED neighbours: if (current + link_cost) < known    ║
║     distance → UPDATE distance and predecessor                           ║
║  4. Repeat until ALL nodes visited                                        ║
║  5. Visited node's distance NEVER CHANGES again                          ║
║  6. Result = routing table (destination, cost, next hop, full path)      ║
╠══════════════════════════════════════════════════════════════════════════╣
║  LINK STATE vs DISTANCE VECTOR (exam favourite)                            ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  Property          Distance Vector      Link State                        ║
║  Info shared       Distance vectors     Link state packets (LSPs)         ║
║  Shared with       Neighbours ONLY      EVERYONE (flooding)               ║
║  Knowledge         Local (partial)      Global (complete map)             ║
║  Algorithm         Bellman-Ford         Dijkstra's                        ║
║  Convergence       SLOW                 FAST                              ║
║  Count to inf.     YES (big problem)    NO (solved)                       ║
║  Bandwidth         Lower                Higher (flooding)                 ║
║  CPU/Memory        Lower                Higher (LSDB + Dijkstra)          ║
║  Protocol          RIP                  OSPF, IS-IS                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  RESULTS FOR EXAMPLE NETWORK (memorise for quick verification)            ║
║  Source R1: R2=5(via R3), R3=3, R4=12(via R3→R2), R5=12(via R3), R6=16  ║
║  Key: R1→R2 direct=6 but R1→R3→R2=5 → Dijkstra picks the INDIRECT path  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SECURITY EXAM POINTS                                                      ║
║  → LSA spoofing: fake LSP injection → route hijacking                   ║
║  → OSPF authentication (MD5/SHA) prevents fake LSA injection            ║
║  → Flooding makes full network topology visible to any eavesdropper     ║
║  → No encryption in OSPF by default → passive recon possible            ║
║  → Combine IPsec + OSPF for link-level encryption                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  EXAM TRAPS                                                                ║
║  → Link State packets are NOT routing tables — they are local LSPs      ║
║  → Flooding sends to EVERYONE, not just neighbours                      ║
║  → Dijkstra may NOT use the direct link if another path is shorter      ║
║    (R1→R2 direct=6, but routing uses R1→R3→R2=5)                        ║
║  → Every router builds its OWN routing table — routing tables differ    ║
║    but every router's LSDB is identical (same global knowledge)         ║
║  → Both Distance Vector AND Link State are DYNAMIC routing              ║
║    (vs static routing where admin manually configures all routes)       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS                                                              ║
║  Link State = "Everyone LINKS their STATE to the whole network"          ║
║  LSDB = "Link State Data Base" — the map every router has               ║
║  Flooding = "flood the whole network with YOUR link info"               ║
║  Dijkstra = "Dig shortest paths out of the LSDB"                        ║
║  Seq Number = "Sequence stops duplicates"                               ║
║  TTL = "Time To Leave (the network) — kicks loops out"                  ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] **OSPF — Deep Dive** (areas, DR/BDR election, LSA types 1–7, OSPF states)
- [ ] **IS-IS Routing** (used by ISPs — differences from OSPF)
- [ ] **Distance Vector vs Link State** — problem sets and numerical comparisons
- [ ] **BGP (Border Gateway Protocol)** — path vector routing, internet-scale routing
- [ ] **OSPF Security** — MD5/SHA authentication, OSPF over IPsec
- [ ] **GNS3 Lab** — full OSPF simulation with 6+ routers, watching convergence
- [ ] **Route Redistribution** — connecting OSPF and other routing domains
- [ ] **Routing Loop Prevention** — split horizon, route poisoning, hold-down timers (Distance Vector)

---

_Notes compiled from: Networking Course — Link State Routing (Gate Smashers)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student_
