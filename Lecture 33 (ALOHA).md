# 📡 Slotted ALOHA — Random Access Protocol

### Cybersecurity Student Notes | Networking Course — Lecture 33

> **Source:** Gate Smashers — Multiple Access Protocols (Slotted ALOHA)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects
> **Builds on:** Lecture 32 — Pure ALOHA (read that first, this lecture is a direct comparison)

---

## 📑 Table of Contents

1. [What is Slotted ALOHA?](#1-what-is-slotted-aloha)
2. [The Two Rule Changes vs Pure ALOHA](#2-the-two-rule-changes-vs-pure-aloha)
3. [Timeline / Slot Model](#3-timeline--slot-model)
4. [Why Collisions Can Still Happen](#4-why-collisions-can-still-happen)
5. [The "12:00 Lecture" Analogy](#5-the-1200-lecture-analogy)
6. [Vulnerable Time — Concept & Derivation](#6-vulnerable-time--concept--derivation)
7. [Efficiency of Slotted ALOHA](#7-efficiency-of-slotted-aloha)
8. [Side-by-Side: Pure ALOHA vs Slotted ALOHA](#8-side-by-side-pure-aloha-vs-slotted-aloha)
9. [Worked Example (100 Kbps Channel)](#9-worked-example-100-kbps-channel)
10. [🔴 Security Considerations & Modern Relevance](#10--security-considerations--modern-relevance)
11. [🧪 Practical Labs — Your Setup](#11--practical-labs--your-setup)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. What is Slotted ALOHA?

- **Slotted ALOHA** is an **improved version of Pure ALOHA** — still a **Random Access Protocol** at the **Access Control (MAC sub-layer)** of the **Data Link Layer**.
- It keeps the same basic idea (stations transmit whenever they have data), but adds **structure to the timeline** to reduce the chance of collision.

```
 ┌───────────────────────┐
 │   Network Layer        │  (Layer 3)
 ├───────────────────────┤
 │ ► Data Link Layer  ◄   │  (Layer 2) — Access Control sub-function lives here
 ├───────────────────────┤
 │   Physical Layer       │  (Layer 1)
 └───────────────────────┘
```

### Key implication

Slotted ALOHA doesn't eliminate collisions — it **reduces the window during which they can happen**, which directly improves maximum efficiency (as shown in Section 7).

---

## 2. The Two Rule Changes vs Pure ALOHA

Slotted ALOHA introduces **exactly two new conditions** on top of Pure ALOHA:

| #     | Rule                                                       | Explanation                                                                                                                                 |
| ----- | ---------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| **1** | **Time is divided into slots**                             | The timeline is chopped into fixed-size slots, where **slot size = Transmission Time (TT)**                                                 |
| **2** | **Transmission can only start at the beginning of a slot** | A station is **not allowed** to start transmitting mid-slot, near the end, or at any arbitrary moment — only exactly when a new slot begins |

> Transmission Time (TT) is still assumed **fixed** for everyone, calculated the same way as in Pure ALOHA:
>
> ```
> TT = Message Size / Bandwidth
> ```
>
> (Message size and bandwidth are both assumed constant across stations, so TT stays fixed — this assumption is what keeps the slot-size math clean.)

---

## 3. Timeline / Slot Model

```
Pure ALOHA (no slots — transmit anytime):

──┬────┬──┬───────┬────┬───────► time
  A       B          C
  (each station can start at literally any point)


Slotted ALOHA (fixed slots, start only at slot boundary):

┌────────┬────────┬────────┬────────┬────────┐
│ Slot 1 │ Slot 2 │ Slot 3 │ Slot 4 │ Slot 5 │   (each slot = 1 × TT)
└────────┴────────┴────────┴────────┴────────┘
   ▲A starts        ▲C starts
   here only           here only
   (slot boundary)     (slot boundary)
```

- A station wanting to transmit must **wait for the next slot boundary** before it can begin — it cannot "jump in" partway through a slot.

---

## 4. Why Collisions Can Still Happen

Even with slot boundaries enforced, **collisions are still possible** — just for a different reason than in Pure ALOHA:

- Since **everyone can only start at a slot boundary**, no station can start _between_ two other stations' transmissions anymore.
- **BUT**: it's entirely possible that **2, 3, or more stations all decide to start at the exact same slot boundary**.
- If multiple stations begin transmitting **at the same slot start**, their signals still collide.

```
Slot boundary
     │
     ▼
   ┌─┴──────────────┐
   │  A starts here │
   │  B starts here │   ← COLLISION (both begin at same slot boundary)
   │  C starts here │
   └────────────────┘
```

---

## 5. The "12:00 Lecture" Analogy

The lecture uses a simple real-world analogy to make the slot-boundary rule intuitive:

> Imagine a lecture scheduled strictly from **12:00 to 1:00**. The rule is: **you may only join exactly at 12:00** — not at 12:05, 12:10, 12:20, 12:45, or even 12:59.

- If **Student A** joins exactly at 12:00, that's fine — no one else can join partway through (12:05 onward is not allowed).
- But it's entirely possible that **2 or 4 students all arrive exactly at 12:00** — they'd all "listen" (transmit) at the same instant. **That's the collision case.**

> This maps directly onto Slotted ALOHA: only the **start-of-slot** moment is a valid transmission opportunity, so collisions only happen when **multiple stations pick the exact same slot start** — never from someone starting "in between."

---

## 6. Vulnerable Time — Concept & Derivation

### 6.1 Why Vulnerable Time Shrinks

In **Pure ALOHA**, a station's transmission was at risk from:

- Any other station starting **during** its own transmission window, **AND**
- Any other station that had started **just before** it (whose last bit could still overlap).

This gave: `VT (Pure ALOHA) = 2 × TT`

In **Slotted ALOHA**, because transmissions can **only** begin at a slot boundary:

- There is **no possibility** of a transmission starting "just before" and bleeding into the current slot — that scenario is structurally disallowed by the rules.
- The **only** remaining risk is: another station starting **at the very same slot boundary** as you.

### 6.2 Final Vulnerable Time Formula

```
Vulnerable Time (VT) = 1 × Transmission Time (TT)
```

Compare directly:

```
Pure ALOHA:     VT = 2 × TT
Slotted ALOHA:  VT = 1 × TT      ← HALVED
```

### 6.3 Interpretation

- The **vulnerable time is cut in half** compared to Pure ALOHA, purely because of the slot-boundary restriction.
- This single change is the reason Slotted ALOHA's maximum efficiency turns out to be **double** that of Pure ALOHA (Section 7).

---

## 7. Efficiency of Slotted ALOHA

### 7.1 Efficiency Formula

```
η = G × e^(-G)
```

Where:

- **G** = number of stations that want to transmit within a given slot (same definition as Pure ALOHA)
- The exponent is **−G** (not −2G) because **Vulnerable Time = 1 × TT** here, instead of 2 × TT.

### 7.2 Deriving Maximum Efficiency

**Step 1 — Differentiate (product rule):**

```
η = G × e^(-G)

dη/dG = e^(-G) × 1 + G × (-1) × e^(-G)
      = e^(-G) (1 - G)
```

**Step 2 — Set derivative to zero:**

```
e^(-G) (1 - G) = 0

Since e^(-G) is never zero:
1 - G = 0
G = 1
```

### 7.3 Meaning of G = 1

- Maximum efficiency occurs when, **on average, exactly 1 station** attempts transmission per slot.
- This matches intuition directly: **if only one station transmits per slot, it's collision-free.**
- Compare with Pure ALOHA, where max efficiency needed **G = 1/2** (half a station "on average," a less intuitive/idealized value) — Slotted ALOHA's G = 1 is a cleaner, more directly meaningful result.

### 7.4 Calculating Maximum Efficiency Value

```
η_max = 1 × e^(-1)
      = 1/e
```

Using **e ≈ 2.71**:

```
η_max = 1 / 2.71
      ≈ 0.368  →  36.8%
```

### 7.5 Interpretation of Result

- **Maximum efficiency of Slotted ALOHA ≈ 36.8%**
- This is **exactly double** Pure ALOHA's 18.4% maximum efficiency.
- The improvement comes entirely from **halving the Vulnerable Time** via the slot-boundary rule.

---

## 8. Side-by-Side: Pure ALOHA vs Slotted ALOHA

| Property                    | Pure ALOHA                                                      | Slotted ALOHA                                                             |
| --------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------- |
| **Transmission start rule** | Anytime                                                         | Only at the start of a slot                                               |
| **Time structure**          | No slots — continuous timeline                                  | Divided into fixed slots (slot size = TT)                                 |
| **Vulnerable Time**         | 2 × TT                                                          | 1 × TT                                                                    |
| **Efficiency formula**      | η = G × e^(−2G)                                                 | η = G × e^(−G)                                                            |
| **G at maximum efficiency** | 1/2                                                             | 1                                                                         |
| **Maximum efficiency**      | 18.4%                                                           | 36.8%                                                                     |
| **Collision cause**         | Any overlapping transmission, from before or during your window | Only when 2+ stations start at the exact same slot boundary               |
| **Complexity**              | Simpler (no synchronization needed)                             | Requires **synchronization** — all stations must agree on slot boundaries |

> **Exam one-liner:** _"Slotted ALOHA doubles the maximum efficiency of Pure ALOHA (36.8% vs 18.4%) by restricting transmissions to slot boundaries, which halves the Vulnerable Time (1×TT instead of 2×TT)."_

---

## 9. Worked Example (100 Kbps Channel)

Given a channel with **Bandwidth = 100 Kbps**:

```
Pure ALOHA:
  Max efficiency = 18.4%
  Usable throughput = 18.4% × 100 Kbps = 18.4 Kbps successfully transmitted
  (remaining ~81.6 Kbps worth of attempts wasted in collisions)

Slotted ALOHA:
  Max efficiency = 36.8%
  Usable throughput = 36.8% × 100 Kbps = 36.8 Kbps successfully transmitted
  (remaining ~63.2 Kbps worth of attempts wasted in collisions)
```

**Conclusion:** Slotted ALOHA delivers **exactly 2× the usable throughput** of Pure ALOHA under identical bandwidth conditions, at the cost of requiring synchronized slot timing across all stations.

---

## 10. 🔴 Security Considerations & Modern Relevance

Slotted ALOHA is a good bridge concept between "pure chaos" (Pure ALOHA) and today's more structured/synchronized shared-medium protocols. A few security-relevant angles:

| Slotted ALOHA Concept                                     | Related Security Topic                   | Notes                                                                                                                                                                                                                                                |
| --------------------------------------------------------- | ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Requires synchronized slot boundaries**                 | **Timing/clock synchronization attacks** | Any protocol that depends on all nodes agreeing on a shared clock/slot boundary opens a class of attacks where an adversary manipulates timing references (e.g., spoofing time-sync signals) to desynchronize legitimate nodes and induce collisions |
| **Collisions only at exact slot-start**                   | **Deliberate collision timing**          | A malicious node that learns the slot boundary could deliberately transmit at the start of every slot to maximize forced collisions against a target — a more "precision" version of the jamming concept discussed for Pure ALOHA                    |
| **Still fundamentally contention-based**                  | **Fairness / starvation**                | Even with slots, a greedy or malfunctioning station can transmit in every slot, statistically causing repeated collisions for others — a fairness/DoS concern also relevant to modern slotted/TDMA-like wireless schemes                             |
| **Relies on all stations trusting the shared slot clock** | **Rogue synchronization source**         | If slot timing comes from a broadcast reference (a beacon, for example), spoofing that reference is conceptually similar to attacks on beacon frames in real Wi-Fi (802.11) networks                                                                 |

> **Key takeaway:** Introducing _any_ synchronization requirement (like slot boundaries) improves efficiency, but it also introduces a **new trust dependency** — the shared timing reference — which becomes a new attack surface. This same trade-off (efficiency gain vs new synchronization-based attack surface) recurs in modern TDMA-based and beacon-based wireless protocols.

---

## 11. 🧪 Practical Labs — Your Setup

### Lab 1 — Simulate & Compare Both Efficiency Curves

```bash
# Plot both efficiency curves together to visually confirm the
# G=1/2 vs G=1 peak locations and the 18.4% vs 36.8% max values
python3 - <<'EOF'
import numpy as np
import matplotlib.pyplot as plt

G = np.linspace(0.01, 3, 300)
pure = G * np.exp(-2 * G)
slotted = G * np.exp(-G)

plt.plot(G, pure * 100, label="Pure ALOHA (max at G=0.5)")
plt.plot(G, slotted * 100, label="Slotted ALOHA (max at G=1)")
plt.xlabel("G (offered load)")
plt.ylabel("Efficiency (%)")
plt.title("Pure ALOHA vs Slotted ALOHA Efficiency")
plt.legend()
plt.grid(True)
plt.savefig("aloha_comparison.png")
print("Pure ALOHA max:", max(pure) * 100, "%")
print("Slotted ALOHA max:", max(slotted) * 100, "%")
EOF
```

### Lab 2 — Discrete-Event Simulation of Slot-Based Collisions

```bash
# Simulate N stations randomly choosing whether to transmit each slot,
# and measure how often exactly 1 station transmits (success) vs 0 or 2+ (idle/collision)
python3 - <<'EOF'
import random

def simulate_slotted_aloha(num_stations=10, num_slots=100000, p_transmit=0.1):
    success = idle = collision = 0
    for _ in range(num_slots):
        transmitters = sum(1 for _ in range(num_stations) if random.random() < p_transmit)
        if transmitters == 0:
            idle += 1
        elif transmitters == 1:
            success += 1
        else:
            collision += 1
    total = num_slots
    print(f"Success rate:   {success/total*100:.2f}%")
    print(f"Idle rate:      {idle/total*100:.2f}%")
    print(f"Collision rate: {collision/total*100:.2f}%")

simulate_slotted_aloha()
EOF
```

### Lab 3 — Observe TDMA-Like Synchronization in Real Protocols

```bash
# TDMA-based systems (cellular, some IoT protocols) rely on the same
# "synchronized slot" principle as Slotted ALOHA. Observe how your own
# devices handle beacon/timing frames in Wi-Fi (which uses CSMA/CA, not
# slots, but broadcasts a similar shared timing reference via beacons):
sudo airmon-ng start wlan0
sudo airodump-ng wlan0mon
# Look at "Beacons" column — this is the timing reference all
# stations on that AP synchronize against, conceptually similar to
# the slot-boundary reference in Slotted ALOHA
```

### Lab 4 — Verify the Efficiency Doubling Empirically

```bash
# Run many trials at different station counts to numerically confirm
# that Slotted ALOHA's peak efficiency is consistently ~2x Pure ALOHA's
python3 - <<'EOF'
import numpy as np

def pure_aloha_efficiency(G):
    return G * np.exp(-2 * G)

def slotted_aloha_efficiency(G):
    return G * np.exp(-G)

Gs = np.linspace(0.01, 3, 1000)
pure_max = max(pure_aloha_efficiency(Gs))
slotted_max = max(slotted_aloha_efficiency(Gs))

print(f"Pure ALOHA max efficiency:    {pure_max*100:.2f}%")
print(f"Slotted ALOHA max efficiency: {slotted_max*100:.2f}%")
print(f"Ratio (Slotted / Pure):       {slotted_max/pure_max:.2f}x")
EOF
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                SLOTTED ALOHA — EXAM CHEAT SHEET                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  PROTOCOL TYPE                                                         ║
║  ─────────────────────────────────────────────────────────           ║
║  Random Access Protocol (Access Control, Data Link Layer)            ║
║  Improved version of Pure ALOHA                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  TWO NEW RULES vs PURE ALOHA                                           ║
║  ─────────────────────────────────────────────────────────           ║
║  1. Timeline divided into slots (slot size = Transmission Time)      ║
║  2. Transmission can start ONLY at the beginning of a slot           ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FORMULAS                                                          ║
║  ─────────────────────────────────────────────────────────           ║
║  Vulnerable Time (VT)    = 1 × TT           (Pure ALOHA: 2 × TT)     ║
║  Efficiency (η)          = G × e^(−G)       (Pure ALOHA: G×e^(−2G)) ║
║  G at Max Efficiency     = 1                (Pure ALOHA: 1/2)       ║
║  Maximum Efficiency      ≈ 36.8%            (Pure ALOHA: 18.4%)     ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY VT = 1×TT (not 2×TT)?                                             ║
║  ─────────────────────────────────────────────────────────           ║
║  Transmissions can ONLY start at slot boundaries → no "started       ║
║  just before me" overlap scenario is possible anymore. Only risk:    ║
║  another station starting at the SAME slot boundary.                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY EFFICIENCY DOUBLES                                                ║
║  ─────────────────────────────────────────────────────────           ║
║  Halving Vulnerable Time (2×TT → 1×TT) directly doubles the           ║
║  maximum achievable efficiency (18.4% → 36.8%)                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "Slot size equals"                    → Transmission Time            ║
║  "When can transmission start?"        → Only at start of a slot     ║
║  "Formula for Vulnerable Time"         → 1 × Transmission Time       ║
║  "Formula for Efficiency"              → G × e^(−G)                  ║
║  "G value for max efficiency"          → 1                           ║
║  "Max efficiency value"                → 36.8%                       ║
║  "How does it compare to Pure ALOHA?"  → Exactly 2× the efficiency  ║
║  "What causes collision here?"         → 2+ stations starting at     ║
║                                            the same slot boundary     ║
║  "Extra requirement vs Pure ALOHA?"    → Synchronization (all        ║
║                                            stations must agree on     ║
║                                            slot boundaries)           ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 34)

- [ ] CSMA (Carrier Sense Multiple Access) — 1-persistent, non-persistent, p-persistent variants
- [ ] CSMA/CD — collision detection and Ethernet's backoff behavior
- [ ] CSMA/CA — collision avoidance, used in wireless (802.11) networks

---

_Notes compiled from: Networking Course Lecture 33 — Slotted ALOHA (Multiple Access Protocols)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
