# 📡 CSMA — Carrier Sense Multiple Access

### " "Networking Course — Lecture 34

> **Source:** Gate Smashers — Multiple Access Protocols (CSMA)
> " "
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects
> **Builds on:** Lecture 32 (Pure ALOHA) & Lecture 33 (Slotted ALOHA) — CSMA is the next evolutionary step after ALOHA

---

## 📑 Table of Contents

1. [What is CSMA?](#1-what-is-csma)
2. [Where CSMA Fits vs ALOHA](#2-where-csma-fits-vs-aloha)
3. [The Core Rule: Sense Before You Send](#3-the-core-rule-sense-before-you-send)
4. [Critical Clarification: Local Sensing, Not Global Sensing](#4-critical-clarification-local-sensing-not-global-sensing)
5. [The Three Persistence Strategies](#5-the-three-persistence-strategies)
6. [Timeline / Behavior Model for Each Strategy](#6-timeline--behavior-model-for-each-strategy)
7. [Vulnerable Time — CSMA vs ALOHA](#7-vulnerable-time--csma-vs-aloha)
8. [Side-by-Side: All Three Persistence Types](#8-side-by-side-all-three-persistence-types)
9. [Side-by-Side: ALOHA vs CSMA](#9-side-by-side-aloha-vs-csma)
10. [🔴 Security Considerations & Modern Relevance](#10--security-considerations--modern-relevance)
11. [🧪 Practical Labs — Your Setup](#11--practical-labs--your-setup)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. What is CSMA?

- **CSMA (Carrier Sense Multiple Access)** is the **next protocol after ALOHA** in the family of **Random/Contention-Based Access Protocols** at the **MAC sub-layer** of the **Data Link Layer**.
- It is one of the **most heavily tested protocols** — both theory questions and numericals appear frequently in university and competitive exams.

```
 ┌───────────────────────┐
 │   Network Layer        │  (Layer 3)
 ├───────────────────────┤
 │ ► Data Link Layer  ◄   │  (Layer 2) — MAC sub-layer: ALOHA → CSMA → CSMA/CD → CSMA/CA
 ├───────────────────────┤
 │   Physical Layer       │  (Layer 1)
 └───────────────────────┘
```

### Key implication

CSMA doesn't remove contention entirely — it adds a **"listen before you talk"** discipline on top of the shared-medium idea, which drastically cuts down (but does not fully eliminate) collisions compared to ALOHA.

---

## 2. Where CSMA Fits vs ALOHA

| Protocol          | Core Behavior                                                                                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------- |
| **Pure ALOHA**    | Transmit **anytime**, no check on the channel at all                                                                             |
| **Slotted ALOHA** | Transmit **anytime**, but only at fixed slot boundaries — still **no channel check**                                             |
| **CSMA**          | **Sense the channel first.** Only proceed to transmit based on what is sensed (idle/busy) and which persistence strategy is used |

> **Exam one-liner:** _"ALOHA protocols never check the channel before transmitting; CSMA's entire innovation is requiring a carrier-sense step before transmission."_

---

## 3. The Core Rule: Sense Before You Send

**Multiple Access** → the channel/medium is **shared** among multiple nodes.

**Carrier Sense** → before transmitting, a node's transmitter must **check the shared medium** to see if a signal is already present (channel busy) or not (channel idle).

```
Node wants to transmit
        │
        ▼
 ┌─────────────────┐
 │ Sense the medium │
 └────────┬────────┘
          │
   ┌──────┴──────┐
   ▼             ▼
 IDLE           BUSY
   │             │
   ▼             ▼
Transmit    Hold data — behavior now depends
(strategy-    on persistence strategy
 dependent)   (Section 5)
```

- If the channel is **busy** and the node transmits anyway → **collision** → wasted bandwidth → **throughput drops**.
- This is exactly why the carrier-sense step exists: to **avoid blindly colliding** the way ALOHA does.

---

## 4. Critical Clarification: Local Sensing, Not Global Sensing

This is a very common **exam trap / conceptual confusion**.

**Misconception:** "Carrier sense" means checking the **entire length** of the shared medium/channel for activity.

**Reality:** A node only senses the channel **at its own point of connection** — never the whole wire end-to-end.

### Worked example from the lecture

```
Medium length: 500 meters (half a kilometer), 5 nodes attached

  Node A        Node B      Node C      Node D      Node E
    │              │           │           │           │
────┴──────────────┴───────────┴───────────┴───────────┴──── (500 m wire)

A only senses the medium AT ITS OWN POINT — not across all 500 m.
```

- If some other node has already started transmitting, but the signal **hasn't yet physically reached A's point** on the wire (because of **propagation delay**), then **A has no way of detecting that a transmission is already underway**.
- **Analogy used in the lecture:** _You only check whether a vehicle is passing directly in front of your house — not the entire street. You only find out about a vehicle once it reaches your point, not when it left someone else's driveway on the other end of the road._

### Why this matters

> **This is precisely why CSMA still suffers from collisions** despite the carrier-sense mechanism: sensing is inherently **local** and bounded by **propagation delay**. Two nodes far apart on the medium can both sense "idle" and transmit at nearly the same time, only to collide once their signals actually meet somewhere on the medium.

---

## 5. The Three Persistence Strategies

CSMA has **3 major variations**, based on what a node does **after** sensing the channel — especially regarding _when exactly_ it acts on an idle/busy result:

| Strategy                          | Idle Channel Response                            | Busy Channel Response                                                           | Value Meaning                                               |
| --------------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| **1-Persistent**                  | Transmit **immediately**                         | Sense **continuously**; transmit the **instant** it turns idle                  | Probability of transmitting on idle = **1**                 |
| **0-Persistent (Non-Persistent)** | Transmit **immediately**                         | Wait a **random backoff time**, then re-check (does **not** continuously sense) | Effectively probability of immediate action on busy = **0** |
| **P-Persistent**                  | Transmit with **probability P** (not guaranteed) | Sense **continuously** (like 1-Persistent)                                      | Probability = tunable value **P**, where **0 < P < 1**      |

---

### 5.1 1-Persistent CSMA

- **Idle →** transmit immediately.
- **Busy →** keep sensing **continuously**, consuming resources the whole time, and the **instant** the channel goes idle, transmit right away.
- **Real-world use:** the general concept behind **Ethernet**.

**Worst-case problem:**

- Say nodes **A** and **D** are already communicating (channel busy), and three more nodes — **N1, N2, N3** — all have data queued and are all sensing "busy" at the same time.
- The moment the A–D transmission ends and the channel goes idle, **all three sense "idle" at the same instant** and, per the 1-Persistent rule, **all transmit immediately**.
- **Result: high collision probability** in this multi-waiting-node scenario.

---

### 5.2 0-Persistent (Non-Persistent) CSMA

- **Idle →** transmit immediately (same as 1-Persistent in this case).
- **Busy →** do **NOT** continuously sense. Instead, pick a **random backoff time**, wait it out fully, and only then attempt to sense/transmit again.
- **Important nuance:** the wait is **not adaptive** — even if the channel becomes free earlier, the node still waits out its full randomly chosen duration before trying again.

**Worked example:**

- Same setup: A–D busy, N1/N2/N3 waiting.
- Each picks a **different random time** — e.g., N1 waits 5 minutes, N2 waits 15, N3 waits 30.
- N1 acts first (after 5 min), while N2 and N3 are still waiting → **staggered attempts → fewer simultaneous collisions** than 1-Persistent.

**Trade-off (downside):**

- If a node picks a long random wait (e.g., 1 hour) but the channel actually frees up within seconds, it still **delays unnecessarily** — wasting idle channel time and initiation delay.

---

### 5.3 P-Persistent CSMA

- **Busy →** sense **continuously** (like 1-Persistent).
- **Idle →** does **not** transmit with certainty. Instead, transmits with **probability P** (a value between 0 and 1, e.g., 0.5 or 0.1).
  - `P = 1` → collapses to 1-Persistent behavior.
  - `P → 0` → tends toward Non-Persistent-like caution.
- **Real-world use:** **Wi-Fi** generally follows this approach.
- **Result:** because not every waiting node necessarily fires the instant the channel is idle, the chance that **multiple nodes transmit at the exact same moment is reduced**, improving throughput and lowering collisions relative to the two extremes.

---

## 6. Timeline / Behavior Model for Each Strategy

```
1-PERSISTENT
Busy ────────────────┐ Idle
                      ▼
     N1 ──────────────┼──► transmits immediately
     N2 ──────────────┼──► transmits immediately   } ALL FIRE AT ONCE
     N3 ──────────────┼──► transmits immediately   } → COLLISION LIKELY


0-PERSISTENT (NON-PERSISTENT)
Busy ────────────────┐ Idle (but nodes don't know exactly when)
                      │
     N1 ── waits 5m ──┼────► transmits (first)
     N2 ── waits 15m ─┼───────────► transmits (later, staggered)
     N3 ── waits 30m ─┼─────────────────────► transmits (last)
     → Fewer simultaneous collisions, but possible wasted idle time


P-PERSISTENT
Busy ────────────────┐ Idle
                      ▼
     N1 ──────────────┼──► transmits with probability P
     N2 ──────────────┼──► transmits with probability P
     N3 ──────────────┼──► transmits with probability P
     → Not all fire at once (probabilistic) → balanced collision/throughput
```

---

## 7. Vulnerable Time — CSMA vs ALOHA

While the lecture (Part 34) doesn't derive a closed-form efficiency formula for CSMA the way Pure/Slotted ALOHA got one, the **conceptual vulnerable-time comparison** is important for exams:

| Protocol          | What determines Vulnerable Time                                                                                                                                                                                                       |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Pure ALOHA**    | `VT = 2 × Transmission Time (TT)` — no channel check at all                                                                                                                                                                           |
| **Slotted ALOHA** | `VT = 1 × TT` — slot boundaries remove the "started just before me" case                                                                                                                                                              |
| **CSMA**          | Vulnerable time is tied to **Propagation Delay (Tp)**, not transmission time — because the only real risk window is the time it takes for a "channel busy" signal to physically reach another node before it also starts transmitting |

> **Conceptual takeaway:** Since **Propagation Delay is normally much smaller than Transmission Time** (`Tp << TT`) in practical wired LANs, CSMA's effective collision window is **much smaller** than either ALOHA variant — which is _why_ CSMA achieves noticeably **higher real-world efficiency** than Pure or Slotted ALOHA, even though a single unified closed-form efficiency number (like ALOHA's 18.4%/36.8%) isn't typically taught at this stage. (Formal CSMA throughput analysis introduces a parameter `a = Tp / Tt` and gets substantially more complex — usually covered only in advanced/graduate treatments, not at this exam level.)

---

## 8. Side-by-Side: All Three Persistence Types

| Persistence Type | On Idle                     | On Busy                                        | Collision Risk                                           | Efficiency/Delay Trade-off                   | Example Use |
| ---------------- | --------------------------- | ---------------------------------------------- | -------------------------------------------------------- | -------------------------------------------- | ----------- |
| **1-Persistent** | Transmit immediately        | Sense continuously; fire the instant it's idle | **Highest** — multiple waiting nodes fire simultaneously | Fast reaction, but wasteful under contention | Ethernet    |
| **0-Persistent** | Transmit immediately        | Random backoff wait (non-adaptive), then retry | **Lower** — staggered by random waits                    | Can waste idle channel time (delayed start)  | —           |
| **P-Persistent** | Transmit with probability P | Sense continuously                             | **Lowest/tunable** via P                                 | Best throughput/collision balance            | Wi-Fi       |

---

## 9. Side-by-Side: ALOHA vs CSMA

| Property                           | Pure ALOHA                   | Slotted ALOHA                                 | CSMA                                                                                            |
| ---------------------------------- | ---------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| **Checks channel before sending?** | ❌ No                        | ❌ No                                         | ✅ Yes                                                                                          |
| **Timing structure**               | None                         | Fixed slots                                   | None required (event-driven, sensing-based)                                                     |
| **Vulnerable Time**                | 2 × TT                       | 1 × TT                                        | ~ Propagation Delay (Tp), much smaller than TT                                                  |
| **Max theoretical efficiency**     | 18.4%                        | 36.8%                                         | Typically higher in practice (no single fixed % at this level)                                  |
| **Main collision cause**           | Any overlapping transmission | Multiple nodes starting at same slot boundary | Propagation delay masking an already-busy channel, or multiple nodes reacting to "idle" at once |
| **Requires synchronization?**      | No                           | Yes (slot boundaries)                         | No (but does require carrier-sensing hardware)                                                  |

---

## 10. 🔴 Security Considerations & Modern Relevance

CSMA's "listen before you talk" model introduces its own distinct security-relevant angles, separate from the ALOHA-style pure-collision issues:

| CSMA Concept                                                                   | Related Security Topic               | Notes                                                                                                                                                                                                                            |
| ------------------------------------------------------------------------------ | ------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Carrier sensing is local, bounded by propagation delay**                     | **Hidden node problem**              | Two nodes that can't "hear" each other (but both can reach a common receiver) may both sense idle and transmit simultaneously — a well-known wireless reliability issue with real DoS implications in contested RF environments  |
| **1-Persistent nodes sense continuously and fire the instant channel is idle** | **Selfish/greedy node behavior**     | A misconfigured node ignoring proper backoff and always firing the instant it senses idle can **starve other nodes** of fair channel access — a fairness/availability concern in shared-medium networks                          |
| **Reliance on "sensing idle" as the trigger to transmit**                      | **Jamming / channel-busy spoofing**  | An adversary who can make the medium _appear_ continuously busy (e.g., RF jamming) can force all well-behaved CSMA nodes to indefinitely defer transmission — a denial-of-service concept relevant to wireless network hardening |
| **Random backoff timers (Non-Persistent / P-Persistent)**                      | **Predictable backoff exploitation** | If a backoff/random-number source is weak or predictable, a node could time transmissions to systematically win channel access over others                                                                                       |
| **Wi-Fi's use of P-Persistent-style access**                                   | **802.11 MAC-layer attacks**         | Connects to real attacks studied in wireless security coursework — e.g., deauthentication attacks and other MAC-layer DoS techniques that exploit the same "fair channel access" assumptions CSMA relies on                      |

> **Key takeaway:** CSMA improves efficiency by requiring nodes to _cooperate_ (sense before sending), but any protocol whose fairness depends on **nodes behaving honestly** creates an opportunity for **non-cooperative behavior** to exploit that trust — this is the conceptual seed for many real MAC-layer wireless attacks you'll study later (CSMA/CD, CSMA/CA, 802.11 deauth, jamming, etc.).

---

## 11. 🧪 Practical Labs — Your Setup

### Lab 1 — Simulate Collision Rates for Each Persistence Strategy

```bash
# Discrete-event simulation comparing 1-Persistent, Non-Persistent, and
# P-Persistent collision behavior under varying numbers of waiting nodes
python3 - <<'EOF'
import random

def simulate_1_persistent(num_waiting_nodes, trials=100000):
    # All waiting nodes fire the instant the channel goes idle
    collisions = sum(1 for _ in range(trials) if num_waiting_nodes > 1)
    return collisions / trials

def simulate_non_persistent(num_waiting_nodes, trials=100000, max_wait=30):
    collisions = 0
    for _ in range(trials):
        waits = [random.uniform(0, max_wait) for _ in range(num_waiting_nodes)]
        waits.sort()
        tolerance = 0.05
        if len(waits) > 1 and (waits[1] - waits[0]) < tolerance:
            collisions += 1
    return collisions / trials

def simulate_p_persistent(num_waiting_nodes, p=0.3, trials=100000):
    collisions = 0
    for _ in range(trials):
        firers = sum(1 for _ in range(num_waiting_nodes) if random.random() < p)
        if firers > 1:
            collisions += 1
    return collisions / trials

for n in [2, 3, 5, 10]:
    print(f"--- {n} waiting nodes ---")
    print(f"1-Persistent collision rate:    {simulate_1_persistent(n)*100:.2f}%")
    print(f"Non-Persistent collision rate:  {simulate_non_persistent(n)*100:.2f}%")
    print(f"P-Persistent collision rate:    {simulate_p_persistent(n)*100:.2f}%")
EOF
```

### Lab 2 — Visualize Propagation Delay vs Transmission Time (the `a` Parameter)

```bash
# CSMA's real-world efficiency depends heavily on a = Tp / Tt.
# Plot how a smaller "a" (propagation delay much less than transmission
# time) reduces the effective collision window.
python3 - <<'EOF'
import numpy as np
import matplotlib.pyplot as plt

Tt = 1.0  # normalized transmission time
Tp_values = np.linspace(0.001, 1.0, 200)
a_values = Tp_values / Tt

plt.plot(a_values, a_values, label="Vulnerable window ratio (a = Tp/Tt)")
plt.xlabel("a = Propagation Delay / Transmission Time")
plt.ylabel("Relative vulnerable window")
plt.title("Why small 'a' makes CSMA efficient")
plt.legend()
plt.grid(True)
plt.savefig("csma_a_parameter.png")
print("Smaller 'a' (Tp << Tt) => much smaller effective vulnerable window than ALOHA's fixed 1x/2x TT")
EOF
```

### Lab 3 — Observe Real Carrier-Sense Behavior on Your Own Network

```bash
# Passive observation only — capture on your own lab AP/interface.
# CSMA/CA (used by Wi-Fi) can be observed via channel utilization and
# retry rates, which hint at contention/collisions in real traffic.
sudo airmon-ng start wlan0
sudo airodump-ng wlan0mon
# Watch the "#/s" (data rate) and station retry behavior on your own
# test AP — high retry counts under load are a real-world sign of
# CSMA/CA contention, conceptually mirroring the collision scenarios
# discussed in Section 5
```

### Lab 4 — Test Carrier-Sensing Concept with a Local Lock-File Simulation

```bash
# Simulate multiple "nodes" (processes) sharing one resource (a lock file)
# to mimic carrier sensing behavior — safe, local-only, no network involved.
python3 - <<'EOF'
import time, os, random

LOCK_FILE = "/tmp/csma_channel.lock"

def sense_and_transmit(node_id, persistence="1-persistent", p=0.3):
    while True:
        if not os.path.exists(LOCK_FILE):
            if persistence == "0-persistent":
                time.sleep(random.uniform(0, 0.05))  # random backoff before retry
                if os.path.exists(LOCK_FILE):
                    continue
            if persistence == "p-persistent" and random.random() > p:
                continue  # chose not to transmit this round
            with open(LOCK_FILE, "w") as f:
                f.write(str(node_id))
            time.sleep(0.01)
            os.remove(LOCK_FILE)
            print(f"Node {node_id} transmitted successfully ({persistence})")
            return
        # else: busy, loop and sense again (mirrors continuous sensing)

sense_and_transmit(1, persistence="1-persistent")
EOF
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                    CSMA — EXAM CHEAT SHEET                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  PROTOCOL TYPE                                                         ║
║  ─────────────────────────────────────────────────────────           ║
║  Contention-based Random Access Protocol (MAC sub-layer)              ║
║  Comes directly after ALOHA (Pure/Slotted) in the protocol family     ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE INNOVATION vs ALOHA                                              ║
║  ─────────────────────────────────────────────────────────           ║
║  Sense the channel BEFORE transmitting (ALOHA never does this)        ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY CLARIFICATION (common trap)                                       ║
║  ─────────────────────────────────────────────────────────           ║
║  Carrier sense = LOCAL sensing at the node's own connection point,    ║
║  NOT sensing across the entire medium. Limited by propagation delay.  ║
╠══════════════════════════════════════════════════════════════════════╣
║  THREE PERSISTENCE STRATEGIES                                          ║
║  ─────────────────────────────────────────────────────────           ║
║  1-Persistent:   idle→transmit now; busy→sense continuously,          ║
║                  fire instantly on idle. HIGH collision risk.         ║
║                  Used in: Ethernet                                    ║
║                                                                        ║
║  0-Persistent:   idle→transmit now; busy→random backoff wait,         ║
║                  non-adaptive. LOWER collision, but wastes idle time. ║
║                                                                        ║
║  P-Persistent:   idle→transmit with probability P; busy→sense         ║
║                  continuously. BEST balance. Used in: Wi-Fi           ║
╠══════════════════════════════════════════════════════════════════════╣
║  VULNERABLE TIME COMPARISON                                            ║
║  ─────────────────────────────────────────────────────────           ║
║  Pure ALOHA:     VT = 2 × TT                                          ║
║  Slotted ALOHA:  VT = 1 × TT                                          ║
║  CSMA:           VT ≈ Propagation Delay (Tp), Tp << TT in practice    ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "What does CSMA add over ALOHA?"        → Carrier sensing step       ║
║  "Does sensing cover whole medium?"      → No — only local point,     ║
║                                              limited by prop. delay    ║
║  "Which strategy has highest collision?" → 1-Persistent               ║
║  "Which strategy has lowest collision?"  → P-Persistent (tunable)     ║
║  "Which is used in Ethernet?"            → 1-Persistent               ║
║  "Which is used in Wi-Fi?"               → P-Persistent               ║
║  "Range of P value?"                     → 0 < P < 1                 ║
║  "Why can collisions still occur?"       → Propagation delay means    ║
║                                              a busy signal may not     ║
║                                              have reached a sensing    ║
║                                              node yet                  ║
║  "Next protocols after CSMA?"            → CSMA/CD, CSMA/CA           ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 35)

- [ ] **CSMA/CD** (Collision Detection) — how a node detects an in-progress collision and aborts/backs off (basis of classic Ethernet)
- [ ] **CSMA/CA** (Collision Avoidance) — proactive avoidance strategies used in wireless (802.11) networks, including RTS/CTS handshakes

---

_Notes compiled from: Networking Course Lecture 34 — CSMA (Multiple Access Protocols)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
