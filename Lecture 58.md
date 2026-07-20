# 🌐 Count to Infinity Problem — Distance Vector Routing

### " "Networking Course — Lecture 58

> **Source:** Gate Smashers — Count to Infinity Problem in Distance Vector Routing
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Distance Vector Routing?](#1-what-is-distance-vector-routing)
2. [Network Setup for This Example](#2-network-setup-for-this-example)
3. [Normal Operation — Before Link Failure](#3-normal-operation--before-link-failure)
4. [The Trigger — Link Failure](#4-the-trigger--link-failure)
5. [Count to Infinity Begins — Round by Round](#5-count-to-infinity-begins--round-by-round)
6. [Why Does This Happen? Root Cause](#6-why-does-this-happen-root-cause)
7. [Solutions to Count to Infinity](#7-solutions-to-count-to-infinity)
8. [All Solutions Side-by-Side](#8-all-solutions-side-by-side)
9. [🔴 Security Context](#9--security-context)
10. [🧪 Practical Labs](#10--practical-labs)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. What is Distance Vector Routing?

### Quick Recap (prerequisite)

```
Distance Vector Routing is a dynamic routing protocol where:
  → Each router maintains a TABLE (vector) of distances
    (costs) to reach every known destination
  → Routers share ONLY their distance table with
    DIRECT NEIGHBOURS (not with the whole network)
  → Based on neighbour's table, each router UPDATES
    its own table using Bellman-Ford algorithm

Key property:
  Routers send ONLY cost values — NOT the path taken.
  "I can get you to X with cost 3" — not "I go via Y then Z"

This is the EXACT REASON count to infinity happens.

Examples of Distance Vector protocols:
  RIP  (Routing Information Protocol) — uses hop count
  IGRP (Interior Gateway Routing Protocol) — Cisco legacy
  EIGRP (Enhanced IGRP) — uses feasibility condition fix
```

### Bellman-Ford Update Rule

```
For each destination D:
  new_cost(D) = min over all neighbours N of:
                [cost(me → N) + cost(N → D)]

In plain English:
  "My best cost to D = cheapest (cost to reach neighbour + neighbour's cost to D)"
```

---

## 2. Network Setup for This Example

### Topology

```
[Internet]
    │
    │ cost = 1
    │
   [B]
   / \
  /   \
 / 1   \ 1
/       \
[A]─────[C]
      1
```

### Link Costs (all links = cost 1)

| Link         | Cost |
| ------------ | ---- |
| B → Internet | 1    |
| A → B        | 1    |
| A → C        | 1    |

### Initial Distance Tables (before any exchange)

```
Each node knows only its DIRECT connections initially.
Nodes don't know anything else → cost = ∞ for unknown destinations.

B's table:
  Destination: Internet → Cost: 1   (directly connected)

A's table:
  Destination: Internet → Cost: ∞   (not directly connected)

C's table:
  Destination: Internet → Cost: ∞   (not directly connected)
```

---

## 3. Normal Operation — Before Link Failure

### Round 1 — First Distance Vector Exchange

```
Action: All neighbours share their distance tables simultaneously.

B sends to A: "I can reach Internet with cost 1"
B sends to C: "I can reach Internet with cost 1"

A receives B's update:
  cost(A → Internet) = cost(A→B) + cost(B→Internet)
                     = 1 + 1 = 2
  A updates: Internet → cost 2 (via B)

C receives B's update:
  C to B is not direct. C hears nothing useful yet.
  C still has: Internet → cost ∞

After Round 1:
  B: Internet = 1
  A: Internet = 2  ← updated!
  C: Internet = ∞  ← no change yet
```

### Round 2 — Second Exchange

```
A sends to C: "I can reach Internet with cost 2"
C receives A's update:
  cost(C → Internet) = cost(C→A) + cost(A→Internet)
                     = 1 + 2 = 3
  C updates: Internet → cost 3 (via A → B → Internet)

After Round 2:
  B: Internet = 1
  A: Internet = 2
  C: Internet = 3  ← converged!

System has CONVERGED. All costs are stable.
No more updates will change anything.
→ This is NORMAL operation — distance vector working correctly.
```

### Stable State Summary

```
      [Internet]
           │
     cost=1│
           │
          [B] → Internet = 1
         /   \
    1   /     \  1
       /       \
      [A]─────[C]
          1

A → Internet = 2  (A→B→Internet)
C → Internet = 3  (C→A→B→Internet)
```

---

## 4. The Trigger — Link Failure

### What Happens

```
The link from B to Internet BREAKS.

          [Internet]
               │
          ✗ BROKEN ✗
               │
              [B]
             /   \
        1   /     \  1
           /       \
          [A]─────[C]
               1
```

### B's Immediate Response

```
B detects link failure via:
  → Hello/keepalive message stops arriving from Internet
  → Physical link goes down

B IMMEDIATELY updates its own table:
  B: Internet → cost = ∞  (I am no longer connected!)

But B has NOT yet told A or C about this.
A and C still think Internet = 2 and 3 respectively.
→ This is where the problem begins.
```

### State After Link Break (before next exchange)

```
B: Internet = ∞   (B knows — just updated)
A: Internet = 2   (A still thinks old value — stale!)
C: Internet = 3   (C still thinks old value — stale!)
```

---

## 5. Count to Infinity Begins — Round by Round

### The Critical Mistake — Why It Goes Wrong

```
IMPORTANT: Distance Vector sends ONLY the cost value.
           It does NOT say "I reach Internet via B" or "via A".
           It only says "I can reach Internet with cost X".

This is the root cause of count to infinity.
```

### Round 1 After Link Break — The "Counting" Starts

```
All nodes exchange tables simultaneously:

B sends to A: "Internet = ∞"    (B knows it's broken)
C sends to A: "Internet = 3"    (C still has stale value!)
B sends to C: "Internet = ∞"    (B knows it's broken)
A sends to C: "Internet = 2"    (A still has stale value!)
A sends to B: "Internet = 2"    (A still has stale value!)
C sends to B: "Internet = 3"    (C still has stale value!)

─────────────────────────────────────────────────────
A receives from B: ∞
A receives from C: 3 (C says it can reach in cost 3)

A thinks: "B can't help me (∞), but C says cost 3!
           My cost to C = 1, so: 1 + 3 = 4"
A updates: Internet → cost 4  ← WRONG! (C is going via A!)

─────────────────────────────────────────────────────
B receives from A: 2
B receives from C: 3

B thinks: "Maybe A can take me! A says cost 2.
           My cost to A = 1, so: 1 + 2 = 3"
B updates: Internet → cost 3  ← WRONG! (A is going via B = loop!)

─────────────────────────────────────────────────────
C receives from B: ∞
C receives from A: 2

C thinks: "B can't help (∞), but A says cost 2!
           My cost to A = 1, so: 1 + 2 = 3"
C keeps:  Internet → cost 3 (same — no change yet)

After Round 1 after break:
  B: Internet = 3   ← was ∞, now 3 (wrong!)
  A: Internet = 4   ← was 2, now 4 (wrong!)
  C: Internet = 3   ← same (still wrong)
```

### Round 2 After Break — Keeps Growing

```
A sends to B and C: "Internet = 4"
B sends to A and C: "Internet = 3"
C sends to A and B: "Internet = 3"

─────────────────────────────────────────────────────
A receives B=3, C=3:
  min(1+3, 1+3) = 4 → no change (stays 4)

B receives A=4, C=3:
  min(1+4, 1+3) = min(5, 4) = 4
  B updates: Internet → 4

C receives A=4, B=3:
  min(1+4, 1+3) = min(5, 4) = 4
  C updates: Internet → 4

After Round 2 after break:
  B: Internet = 4
  A: Internet = 4
  C: Internet = 4
```

### Round 3 After Break — Still Growing

```
All send cost=4 to each other:
  Each node computes: 1 + 4 = 5
  All update to 5.

After Round 3: All = 5
After Round 4: All = 6
After Round 5: All = 7
...
After Round N: All = N + 3
...
→ Costs KEEP INCREASING without bound → approaching INFINITY
```

### Count to Infinity — Full Progression Table

| Round     | B (Internet) | A (Internet) | C (Internet) | Event                       |
| --------- | ------------ | ------------ | ------------ | --------------------------- |
| Initial   | 1            | ∞            | ∞            | System starts               |
| 1         | 1            | 2            | ∞            | Normal convergence          |
| 2         | 1            | 2            | 3            | Normal convergence          |
| **BREAK** | **∞**        | 2            | 3            | **B–Internet link fails**   |
| B+1       | 3            | 4            | 3            | Loop forms, counting begins |
| B+2       | 4            | 4            | 4            | All stuck at 4              |
| B+3       | 5            | 5            | 5            | All increment to 5          |
| B+4       | 6            | 6            | 6            | All increment to 6          |
| ...       | ...          | ...          | ...          | Keeps counting up           |
| ∞         | ∞            | ∞            | ∞            | Eventually reaches infinity |

### Visual — The Routing Loop

```
WHY is this happening? A ROUTING LOOP formed:

A thinks: "I'll send Internet traffic to C (cost 4)"
C thinks: "I'll send Internet traffic to A (cost 3 or 4)"

A → C → A → C → A → C → ∞   (packets bounce forever!)

And B thinks: "I'll send Internet traffic to A or C"
So B → A → C → A → C → ... (also bouncing)

The loop is:
  A ←→ C ←→ A ←→ C  (round-trip infinity loop)
  B → A or C → loop
```

---

## 6. Why Does This Happen? Root Cause

### The Fundamental Problem

```
Distance Vector ONLY shares COST — not the PATH.

When C tells A: "I can reach Internet with cost 3"
C does NOT say: "I reach Internet via A → B → Internet"

So A doesn't know that C's route goes THROUGH A itself!
A thinks C has an independent path to Internet.
A routes via C, C routes via A → ROUTING LOOP!

If C had said: "I reach Internet via A with cost 3"
Then A would know: "C is depending on ME — if I'm down, C is too"
A would not update via C → NO loop → NO count to infinity.

This is exactly why the SOLUTION is to share path information
or use tricks that prevent using the same path you heard from.
```

### Analogy

> Imagine A asks C: "Can you get me to the airport?"
> C says: "Yes, I can get you there in 30 minutes!"
> But C's plan is to ride with A to a bus stop first.
> A doesn't know C's route depends on A.
> So A sits in C's car, C drives to pick up A… infinite loop.
>
> If C had said "I'm going with you first then taking a taxi"
> A would have said "Wait, that doesn't help me at all!"

---

## 7. Solutions to Count to Infinity

### Solution 1 — Define Infinity as Small Number

```
Problem: True infinity takes too long to reach.
Fix:     Define infinity = 16 (used in RIP protocol).

If cost reaches 16 → treat as unreachable immediately.
Network converges faster (max 15 hops in RIP).

Limitation:
  Network diameter limited to 15 hops maximum.
  Not scalable for large networks.
  RIP uses this as its fix — that's why RIP max hops = 15.
```

### Solution 2 — Split Horizon

```
Rule: Do NOT advertise a route back to the neighbour
      from whom you LEARNED it.

Example:
  A learned: Internet = 2 (via B)
  → A should NOT tell B: "Internet = 2"
    (B knows already — B told A!)
  → A should NOT tell C if C was the source either.

Effect on our problem:
  C learned Internet = 3 from A.
  C will NOT advertise Internet back to A.
  → A never gets the false "cost 3 via C" update.
  → A correctly sees B=∞ and updates to ∞.
  → No loop forms!

Limitation:
  Only works for simple loops between 2 nodes.
  Does NOT prevent loops involving 3+ nodes.
  (A → C → B → A type loops still possible)
```

### Solution 3 — Split Horizon with Poison Reverse

```
Enhancement of Split Horizon:

Rule: Instead of NOT advertising a route back to its source,
      advertise it with cost = INFINITY (poison it).

Example:
  A learned Internet via B.
  A sends to B: "Internet = ∞" (poisoned!)

This explicitly tells B: "Don't route via me for Internet"
More aggressive than plain split horizon.

Effect:
  Faster convergence after failure.
  B immediately knows A is not a valid path.
  Prevents 2-node loops more decisively.

Limitation:
  Still cannot prevent all multi-hop loops.
  Increases advertisement size (more entries sent as ∞).
```

### Solution 4 — Triggered Updates (Route Poisoning)

```
Normal Distance Vector: Send updates only at FIXED INTERVALS
                        (e.g., every 30 seconds in RIP)

Problem: During 30-second wait, stale routes persist → loops form.

Fix — Triggered Updates:
  When a route changes (especially when a link goes down),
  send an UPDATE IMMEDIATELY — don't wait for the timer.

Effect on our problem:
  B detects link break.
  B IMMEDIATELY sends to A and C: "Internet = ∞"
  A and C receive this BEFORE sending their stale values.
  → No false updates propagate.

Combined with:
  Route Poisoning: Set failing route to ∞ and broadcast immediately.
  Holddown Timer:  After receiving poison, ignore better-looking updates
                   for a hold-down period (prevents premature acceptance
                   of stale routes still circulating).
```

### Solution 5 — Path Vector Routing

```
Ultimate fix: Send FULL PATH, not just cost.

Protocol: BGP (Border Gateway Protocol) uses this approach.

Example:
  C advertises: "I reach Internet via path: C→A→B→Internet, cost 3"
  A receives this: "Wait! My own ID (A) is in C's path!
                    That means C depends on me. I won't use C."
  → Loop never forms!

This is how BGP prevents count to infinity completely.
BGP is used between different Autonomous Systems on the internet.

Why not used everywhere?
  Path information = much more data to send.
  Fine for inter-AS (BGP) but overhead for intra-AS routing.
  OSPF uses a different approach (link-state) to avoid this entirely.
```

### Solution 6 — Switch to Link-State Routing (OSPF, IS-IS)

```
REAL solution: Don't use Distance Vector at all for large networks.

Link-State Routing (OSPF):
  Each router knows the COMPLETE TOPOLOGY of the network.
  Runs Dijkstra's shortest path algorithm locally.
  No routing loops possible — full picture always available.

Cost: More memory, more CPU, more bandwidth for topology flooding.
Benefit: No count to infinity, faster convergence, loop-free.

Used in: Enterprise networks, ISP backbones.
RIP (Distance Vector) is used only in small/simple networks.
```

---

## 8. All Solutions Side-by-Side

| Solution          | How It Works                   | Fixes All Loops?           | Used In     |
| ----------------- | ------------------------------ | -------------------------- | ----------- |
| Infinity = 16     | Stop counting at 16            | No (just limits damage)    | RIP         |
| Split Horizon     | Don't advertise back to source | 2-node loops only          | RIP, EIGRP  |
| Poison Reverse    | Advertise as ∞ back to source  | 2-node loops only          | RIP         |
| Triggered Updates | Send immediately on change     | Reduces but not eliminates | RIP         |
| Holddown Timer    | Ignore better routes briefly   | Reduces false convergence  | RIP         |
| Path Vector       | Send full path, detect loops   | Yes — all loops            | BGP         |
| Link-State        | Complete topology map          | Yes — completely           | OSPF, IS-IS |

---

## 9. 🔴 Security Context

### Count to Infinity as an Attack Vector

```
Count to Infinity is not just a design flaw — it can be
DELIBERATELY TRIGGERED by an attacker on a network.

If an attacker can:
  1. Connect a rogue router to a RIP-enabled network
  2. Advertise false routes with manipulated cost values

Then they can:
  → Trigger count to infinity artificially
  → Cause network-wide routing instability
  → Drop all traffic to targeted destinations (DoS)
  → Redirect traffic through attacker's router (MITM)
```

### Attack 1 — RIP Route Poisoning Attack

```
RIP (Routing Information Protocol) uses Distance Vector.
RIP has no authentication by default (RIPv1).

Attack:
  Rogue router connects to LAN.
  Sends RIP update: "I can reach 0.0.0.0/0 with cost 1"
  (claims to be the cheapest default gateway)

  All routers update: default route → via attacker
  All internet traffic now goes through attacker's router
  Attacker performs MITM on entire network traffic

Trigger count to infinity variation:
  Rogue router repeatedly changes cost for critical routes
  Causes all routers to keep recomputing → CPU exhaustion → DoS

Defense:
  RIPv2 with MD5 authentication
  Passive interfaces on untrusted segments
  Route filtering (only accept routes from known neighbors)
```

### Attack 2 — Routing Loop as DoS

```
Attacker deliberately creates a routing loop:
  Router A thinks: "Send X traffic to B"
  Router B thinks: "Send X traffic to A"

Packets bounce A → B → A → B until TTL = 0

Effect:
  Each packet consumes bandwidth on BOTH links repeatedly
  Large traffic flood → link saturation → legitimate traffic dropped
  TTL expiry generates ICMP messages → ICMP flood
  → Effective DoS without needing high-speed attack

Detection:
  Wireshark: packets with rapidly decreasing TTL
  Router logs: high ICMP TTL-exceeded messages
  NetFlow: same packet patterns looping on links
```

### Attack 3 — BGP Hijacking (Real-World Count to Infinity Analog)

```
BGP uses Path Vector (fixes count to infinity).
But BGP has its own version of the problem: BGP hijacking.

What happened (Pakistan Telecom, 2008):
  Pakistan Telecom accidentally advertised YouTube's IP prefix
  with a shorter AS path (appeared cheaper)
  → All BGP routers worldwide updated routes to YouTube
     to go through Pakistan
  → YouTube went down globally for 2 hours

This is essentially count to infinity's cousin:
  Bad route advertised → network-wide adoption of wrong route
  Traffic goes to wrong destination (blackhole or MITM)

Modern defense: RPKI (Resource Public Key Infrastructure)
  Cryptographically signs which AS owns which IP prefix
  Routers verify before accepting BGP updates
```

### Detecting Routing Instability in Your Lab

```bash
# Watch for rapid route changes (sign of count to infinity or attack)

# Monitor routing table changes in real time
watch -n 1 ip route

# Check if routes are flapping (rapidly appearing/disappearing)
# Install quagga/frr for a real routing daemon
sudo apt install frr -y

# View RIP neighbors and database (if RIP configured)
sudo vtysh -c "show ip rip"
sudo vtysh -c "show ip rip database"

# Check for routing loops using traceroute
# If you see the same hops repeating → LOOP DETECTED
traceroute 8.8.8.8
# Normal: each hop is different
# Loop: hop 5 = hop 7 = hop 9 (same IP appearing again)

# Wireshark filters for routing protocol traffic:
# RIP:  udp.port == 520
# OSPF: ip.proto == 89
# BGP:  tcp.port == 179
```

### Defense — Securing RIP Against Attacks

```bash
# RIPv2 with MD5 authentication (prevents rogue router injection)
# Configure in /etc/frr/ripd.conf:

# interface eth0
#  ip rip authentication mode md5
#  ip rip authentication string YourSecretKey

# Alternatively, use passive interfaces to prevent RIP on untrusted ports
# interface eth0
#  ip rip passive   ← listens but doesn't send RIP updates

# Filter routes — only accept routes from known neighbors
# ip prefix-list ALLOWED permit 192.168.56.0/24
# ip prefix-list ALLOWED deny any

# Better: switch from RIP to OSPF with authentication
# OSPF has no count to infinity problem (link-state)
# OSPF with MD5: much more secure than RIP
```

---

## 10. 🧪 Practical Labs

### Lab 1 — Count to Infinity Simulator (Python)

```python
# Save as count_to_infinity.py — run: python3 count_to_infinity.py
# Simulates the exact network from the lecture

INF = float('inf')

class Router:
    def __init__(self, name):
        self.name = name
        self.table = {}        # {destination: cost}
        self.neighbours = {}   # {router_name: link_cost}
        self.received = {}     # last received from each neighbour

    def update(self, neighbour_name: str, neighbour_table: dict):
        """Receive neighbour's distance table and update own table"""
        link_cost = self.neighbours.get(neighbour_name, INF)
        changed = False
        for dest, neigh_cost in neighbour_table.items():
            if neigh_cost == INF:
                new_cost = INF
            else:
                new_cost = link_cost + neigh_cost
            current = self.table.get(dest, INF)
            if new_cost < current:
                self.table[dest] = new_cost
                changed = True
        return changed

    def __repr__(self):
        t = ', '.join(f"{d}:{c if c!=INF else '∞'}" for d, c in self.table.items())
        return f"Router {self.name}: [{t}]"

def run_simulation():
    # Create routers from lecture
    B = Router('B')
    A = Router('A')
    C = Router('C')

    # Initial costs (directly connected)
    B.table    = {'Internet': 1}
    A.table    = {'Internet': INF}
    C.table    = {'Internet': INF}

    # Neighbour connections
    B.neighbours = {'A': 1, 'C': 1}
    A.neighbours = {'B': 1, 'C': 1}
    C.neighbours = {'A': 1, 'B': 1}

    routers = {'A': A, 'B': B, 'C': C}

    print("="*60)
    print("PHASE 1: Normal Convergence")
    print("="*60)

    for round_num in range(1, 4):
        # Snapshot current tables
        snapshots = {name: dict(r.table) for name, r in routers.items()}
        # Exchange with all neighbours
        for name, router in routers.items():
            for neigh_name, _ in router.neighbours.items():
                router.update(neigh_name, snapshots[neigh_name])
        print(f"\nRound {round_num}:")
        for r in [B, A, C]:
            cost = r.table.get('Internet', INF)
            print(f"  {r.name}: Internet = {'∞' if cost==INF else cost}")

    print("\n" + "="*60)
    print("LINK FAILURE: B-Internet link breaks!")
    print("="*60)
    B.table['Internet'] = INF
    print(f"  B immediately sets Internet = ∞")
    print(f"  A still thinks Internet = {A.table.get('Internet',INF)}")
    print(f"  C still thinks Internet = {C.table.get('Internet',INF)}")

    print("\n" + "="*60)
    print("PHASE 2: Count to Infinity begins")
    print("="*60)

    for round_num in range(1, 12):
        snapshots = {name: dict(r.table) for name, r in routers.items()}
        for name, router in routers.items():
            for neigh_name in router.neighbours:
                router.update(neigh_name, snapshots[neigh_name])

        b_cost = B.table.get('Internet', INF)
        a_cost = A.table.get('Internet', INF)
        c_cost = C.table.get('Internet', INF)
        print(f"Round {round_num}: B={b_cost if b_cost!=INF else '∞':>4}  "
              f"A={a_cost if a_cost!=INF else '∞':>4}  "
              f"C={c_cost if c_cost!=INF else '∞':>4}  "
              f"{'← counting up!' if round_num > 1 else '← loop starts'}")

    print("\nCosts keep increasing → COUNT TO INFINITY")
    print("In RIP, this stops at 16 (infinity = 16)")

run_simulation()
```

### Lab 2 — Split Horizon Fix Simulator

```python
# Save as split_horizon_fix.py
# Shows how split horizon prevents count to infinity

INF = float('inf')

def simulate_with_split_horizon():
    """
    Split Horizon Rule:
    Do NOT advertise a route back to the neighbour you learned it from.
    Track: learned_from[destination] = neighbour_name
    """

    # Initial state after normal convergence
    tables       = {'B': {'Internet': 1},
                    'A': {'Internet': 2},
                    'C': {'Internet': 3}}
    learned_from = {'B': {'Internet': None},   # B is directly connected
                    'A': {'Internet': 'B'},     # A learned via B
                    'C': {'Internet': 'A'}}     # C learned via A

    neighbours   = {'B': ['A', 'C'],
                    'A': ['B', 'C'],
                    'C': ['A', 'B']}
    link_cost    = 1  # all links = 1

    print("="*60)
    print("SPLIT HORIZON FIX — After B-Internet link breaks")
    print("="*60)

    # B's link breaks
    tables['B']['Internet'] = INF
    print("B sets Internet = ∞")
    print()

    for round_num in range(1, 5):
        snapshots = {n: dict(t) for n, t in tables.items()}
        print(f"Round {round_num} exchanges (with Split Horizon):")

        for sender, neigh_list in neighbours.items():
            for receiver in neigh_list:
                dest = 'Internet'
                # Split Horizon: if receiver is who we learned dest from, skip!
                if learned_from[sender].get(dest) == receiver:
                    print(f"  {sender}→{receiver}: SUPPRESSED (learned Internet from {receiver})")
                    continue

                sent_cost = snapshots[sender].get(dest, INF)
                print(f"  {sender}→{receiver}: Internet = {'∞' if sent_cost==INF else sent_cost}")

                # Receiver updates
                new_cost = INF if sent_cost == INF else link_cost + sent_cost
                current  = tables[receiver].get(dest, INF)
                if new_cost < current:
                    tables[receiver][dest] = new_cost
                    learned_from[receiver][dest] = sender

        print(f"  After round {round_num}: ", end="")
        for n in ['B', 'A', 'C']:
            c = tables[n].get('Internet', INF)
            print(f"{n}={'∞' if c==INF else c}  ", end="")
        print()
        print()

    print("Result: All nodes correctly reach ∞ — no count to infinity!")
    print("Split Horizon PREVENTED the loop.")

simulate_with_split_horizon()
```

### Lab 3 — Trace TTL Decrements to Detect Loops

```bash
# In a real network, routing loops cause TTL to expire
# Traceroute reveals loops when same IP appears multiple times

# Normal traceroute (each hop is unique):
traceroute 8.8.8.8

# If a loop exists, you'll see something like:
#  1. 192.168.56.1   (your gateway)
#  2. 10.0.0.1       (ISP router)
#  3. 10.0.0.2
#  4. 10.0.0.1       ← same as hop 2! LOOP DETECTED
#  5. 10.0.0.2       ← same as hop 3! LOOP CONFIRMED
#  ... repeats until TTL expires
# Then: * * * * (TTL expired, no more hops)

# Detect TTL expiry messages (signs of a loop):
sudo tcpdump -i eth0 'icmp[icmptype] = icmp-timxceed' -n
# ICMP type 11 = Time Exceeded = TTL hit 0 = likely loop

# Monitor routing table changes (loop = frequent changes):
watch -n 1 'ip route show | grep -v "scope link"'

# Check for duplicate routes appearing:
ip route show table all | sort | uniq -d
```

### Lab 4 — RIP Configuration and Monitoring with FRR

```bash
# Install Free Range Routing (FRR) — modern replacement for Quagga
sudo apt install frr frr-pythontools -y

# Enable RIP daemon
sudo sed -i 's/ripd=no/ripd=yes/' /etc/frr/daemons

# Start FRR
sudo systemctl start frr
sudo systemctl enable frr

# Enter FRR shell
sudo vtysh

# Inside vtysh — configure RIP:
# configure terminal
# router rip
#   version 2
#   network 192.168.56.0/24
#   neighbor 192.168.56.101
# !
# exit

# Monitor RIP in real time:
# show ip rip                     → routing table
# show ip rip status              → RIP status and timers
# show ip rip database            → full RIP database
# debug rip events                → live RIP events
# debug rip packet                → all RIP packets sent/received

# Observe count to infinity in real-time:
# 1. Set up two FRR instances (or use GNS3/EVE-NG)
# 2. Create a loop artificially
# 3. Watch 'show ip rip database' increment costs

echo "FRR installed. Use 'sudo vtysh' to enter routing shell."
echo "Key commands: show ip rip, debug rip events"
```

### Lab 5 — Visualise Routing Table and Detect Issues

```python
# Save as routing_monitor.py
# Monitor routing table and flag anomalies

import subprocess
import time
import re

def get_routing_table():
    """Get current Linux routing table"""
    result = subprocess.run(['ip', 'route', 'show'],
                          capture_output=True, text=True)
    return result.stdout

def parse_routes(table_str: str) -> dict:
    """Parse routing table into dict"""
    routes = {}
    for line in table_str.strip().split('\n'):
        if not line:
            continue
        # Extract destination and gateway
        match = re.match(r'(\S+)\s+(?:via\s+(\S+)\s+)?dev\s+(\S+)', line)
        if match:
            dest = match.group(1)
            gw   = match.group(2) or 'direct'
            dev  = match.group(3)
            routes[dest] = {'gateway': gw, 'dev': dev, 'raw': line}
    return routes

def monitor_routing_table(interval=5, duration=60):
    """Monitor routing table for changes (sign of instability)"""
    print(f"Monitoring routing table every {interval}s for {duration}s")
    print("Changes = potential routing instability / attack")
    print("="*55)

    prev_routes = {}
    changes = 0
    start = time.time()

    while time.time() - start < duration:
        current = get_routing_table()
        curr_routes = parse_routes(current)

        # Detect additions
        for dest in curr_routes:
            if dest not in prev_routes:
                print(f"[+] NEW ROUTE:     {dest} via {curr_routes[dest]['gateway']}")
                changes += 1

        # Detect removals
        for dest in prev_routes:
            if dest not in curr_routes:
                print(f"[-] ROUTE REMOVED: {dest} (was via {prev_routes[dest]['gateway']})")
                changes += 1

        # Detect gateway changes (route flap)
        for dest in curr_routes:
            if dest in prev_routes:
                if curr_routes[dest]['gateway'] != prev_routes[dest]['gateway']:
                    print(f"[~] ROUTE CHANGED: {dest}")
                    print(f"    Before: {prev_routes[dest]['gateway']}")
                    print(f"    After:  {curr_routes[dest]['gateway']}")
                    print(f"    ⚠️  Route flapping detected!")
                    changes += 1

        prev_routes = curr_routes
        if changes == 0:
            print(f"[{int(time.time()-start):>3}s] Stable — {len(curr_routes)} routes, 0 changes")

        time.sleep(interval)

    print(f"\nTotal changes detected: {changes}")
    if changes > 5:
        print("⚠️  HIGH INSTABILITY — possible routing loop or attack!")
    else:
        print("✓ Network appears stable")

monitor_routing_table(interval=3, duration=30)
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║     COUNT TO INFINITY — DISTANCE VECTOR ROUTING EXAM CHEAT SHEET   ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHAT IS IT?                                                         ║
║  A problem in Distance Vector Routing where costs keep              ║
║  increasing indefinitely after a link failure, due to              ║
║  routing loops caused by incomplete route information.              ║
╠══════════════════════════════════════════════════════════════════════╣
║  ROOT CAUSE                                                          ║
║  Distance Vector sends ONLY COST — not the PATH                    ║
║  Router doesn't know if neighbour's route goes through ITSELF       ║
║  → Loop forms → costs increment every round → ∞                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  LECTURE EXAMPLE (all link costs = 1)                               ║
║  Normal state:  B→Internet=1, A→Internet=2, C→Internet=3          ║
║  B–Internet breaks:                                                 ║
║    B sets Internet=∞                                               ║
║    C tells A: "Internet=3" (stale — C goes via A via B!)          ║
║    A thinks: 1+3=4 (via C) → updates to 4 WRONG!                  ║
║    B thinks: 1+4=5 → updates to 5 WRONG!                          ║
║    Everyone keeps incrementing → ∞                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  ROUND-BY-ROUND AFTER BREAK                                         ║
║  Break:   B=∞,  A=2,  C=3                                         ║
║  Round 1: B=3,  A=4,  C=3                                         ║
║  Round 2: B=4,  A=4,  C=4                                         ║
║  Round 3: B=5,  A=5,  C=5  ← counting up!                        ║
║  Round N: all = N+3 → eventually ∞                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  SOLUTIONS                                                           ║
║  1. Infinity = 16        → RIP stops counting at 16               ║
║  2. Split Horizon        → don't advertise back to source         ║
║  3. Poison Reverse       → advertise as ∞ back to source          ║
║  4. Triggered Updates    → send immediately on link change         ║
║  5. Path Vector          → send full path (BGP) — complete fix    ║
║  6. Link-State (OSPF)    → full topology map — no loop possible   ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY RULES TO REMEMBER                                              ║
║  Distance Vector → shares COST only, NOT path                     ║
║  Link-State      → shares full TOPOLOGY (no count to ∞)           ║
║  Path Vector     → shares full PATH (BGP — no count to ∞)        ║
║  RIP uses Distance Vector → suffers from count to infinity        ║
║  RIP fix: max hops = 15, infinity = 16                            ║
║  OSPF uses Link-State → no count to infinity problem              ║
╠══════════════════════════════════════════════════════════════════════╣
║  SPLIT HORIZON — ONE LINE                                           ║
║  "Never advertise a route back to the neighbour you got it from"   ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  Count to ∞ can be triggered by rogue router (RIP attack)         ║
║  RIPv1 has NO authentication → anyone can inject routes           ║
║  RIPv2 uses MD5 authentication → use this always                  ║
║  BGP hijacking = real-world cousin of count to infinity            ║
║  Defense: RPKI for BGP, MD5 auth for RIP, OSPF for internal nets  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Split Horizon — Deep dive with numerical examples
- [ ] Poison Reverse — How it differs from split horizon
- [ ] RIP Protocol — Complete configuration and timer values
- [ ] OSPF — Link-state routing, Dijkstra, no count to infinity
- [ ] BGP — Path vector, AS paths, how the internet routes between ISPs

---

_Notes compiled from: Networking Course Lecture 58 — Count to Infinity Problem_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
