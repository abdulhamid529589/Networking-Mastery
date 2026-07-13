# 📡 Medium/Multiple Access Control (MAC) Protocols

### Cybersecurity Student Notes | Networking Course — Lecture 31

> **Source:** Gate Smashers — Various Medium Access Control Protocols
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Where MAC Fits in the OSI Model](#1-where-mac-fits-in-the-osi-model)
2. [Why We Need Medium Access Control](#2-why-we-need-medium-access-control)
3. [The Three Categories of MAC Protocols](#3-the-three-categories-of-mac-protocols)
4. [Category 1 — Random Access Protocols](#4-category-1--random-access-protocols)
   - [4.1 ALOHA (Pure & Slotted)](#41-aloha-pure--slotted)
   - [4.2 CSMA (Carrier Sense Multiple Access)](#42-csma-carrier-sense-multiple-access)
   - [4.3 CSMA/CD (Collision Detection)](#43-csmacd-collision-detection)
   - [4.4 CSMA/CA (Collision Avoidance)](#44-csmaca-collision-avoidance)
5. [Category 2 — Controlled Access Protocols](#5-category-2--controlled-access-protocols)
   - [5.1 Polling](#51-polling)
   - [5.2 Token Passing](#52-token-passing)
6. [Category 3 — Channelization Protocols](#6-category-3--channelization-protocols)
   - [6.1 FDMA](#61-fdma)
   - [6.2 TDMA](#62-tdma)
   - [6.3 CDMA](#63-cdma-bonus-completeness)
7. [Formula Reference Sheet](#7-formula-reference-sheet)
8. [Quick Reference Rules](#8-quick-reference-rules)
9. [Worked Numerical Examples](#9-worked-numerical-examples)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. Where MAC Fits in the OSI Model

```
OSI Model (bottom-up):
  Layer 1 — Physical Layer
  Layer 2 — Data Link Layer
              ├── LLC  (Logical Link Control)  — framing, error control, flow control
              └── MAC  (Medium Access Control) — WHO gets to use the shared medium, and WHEN
  Layer 3 — Network Layer
  ...
```

- **LLC (Logical Link Control):** handles frame synchronization, error detection/correction (CRC, Hamming), and flow control.
- **MAC (Medium Access Control):** decides **which station is allowed to transmit** on a shared medium at any given moment, to avoid collisions.

> This lecture is entirely about the **MAC sublayer** — not LLC.

---

## 2. Why We Need Medium Access Control

The type of physical link between stations determines whether MAC protocols are even needed:

| Link Type                                    | Example Topology                        | Is MAC needed?                                               |
| -------------------------------------------- | --------------------------------------- | ------------------------------------------------------------ |
| **Point-to-point** (dedicated link per pair) | Full mesh topology                      | ❌ No — each pair has its own private link, nothing to share |
| **Shared / broadcast medium**                | Bus topology, wireless LAN, shared coax | ✅ Yes — multiple stations compete for the same channel      |

**The core problem:** If two or more stations transmit on a shared medium **at the same time**, their signals **collide**, and the data from all colliding transmissions is corrupted/lost.

**MAC protocols exist to answer two questions:**

1. **Who** gets to transmit right now?
2. **How do we prevent / handle collisions** when they happen?

---

## 3. The Three Categories of MAC Protocols

```
Medium Access Control Protocols
│
├── 1. Random Access Protocols
│      No fixed turn, no priority, any station can transmit any time
│      → ALOHA, CSMA, CSMA/CD, CSMA/CA
│
├── 2. Controlled Access Protocols
│      A controlling mechanism decides whose turn it is
│      → Polling, Token Passing (Token Ring)
│
└── 3. Channelization Protocols
       The shared medium is divided into fixed channels/slots/codes
       → FDMA, TDMA, CDMA
```

| Category              | Key Idea                                            | Collision Possible?             |
| --------------------- | --------------------------------------------------- | ------------------------------- |
| **Random Access**     | No priority, no schedule — first come, first served | ✅ Yes, must be handled         |
| **Controlled Access** | A poll or a token decides who transmits next        | ❌ No (by design)               |
| **Channelization**    | Medium is pre-divided into separate channels/slots  | ❌ No (each gets its own slice) |

> **Exam priority (as emphasized in the lecture):** Random Access → **CSMA and CSMA/CD are the most important** topics for GATE/UGC-NET, followed closely by ALOHA. Then Token Ring (Controlled Access) — especially **Delayed Token Reinsertion** and **Early Token Release**, which have appeared in GATE before.

---

## 4. Category 1 — Random Access Protocols

**Defining characteristics** (as stated in the lecture):

- No priority among stations — all stations are equal.
- No fixed turn/schedule — a station transmits whenever its data is ready.
- Any amount of data can be sent at any time.

Because of this freedom, collisions are always a risk, and each protocol in this family is really just a different strategy for **reducing the chance of collision** or **reacting to collision once it happens**.

### 4.1 ALOHA (Pure & Slotted)

ALOHA is the earliest and simplest random access protocol (developed at the University of Hawaii).

#### Pure ALOHA

```
Rule: A station transmits WHENEVER it has data — no waiting, no listening first.

Consequence: Highest collision risk of all protocols in this family,
because there's no attempt to sense the channel before sending.

Vulnerable Time = 2 × Tt   (Tt = time to transmit one frame)

Why 2×Tt? A frame sent at time t0 collides with:
  - Any frame started between (t0 − Tt) and t0   → collides with the tail of it
  - Any frame started between t0 and (t0 + Tt)    → collides with the start of it
  → total vulnerable window = 2 frame-times
```

#### Slotted ALOHA

```
Rule: Time is divided into discrete slots, each exactly Tt long.
      A station may ONLY start transmitting at the beginning of a slot.

Consequence: Cuts the vulnerable window in half, because a
             transmission can now only ever fully overlap with
             ONE other slot's transmission, not a sliding window.

Vulnerable Time = Tt   (just one frame-time)
```

**Throughput formulas** (G = offered load, i.e. average number of transmission attempts per frame-time):

```
Pure ALOHA:
  S = G × e^(−2G)
  Maximum throughput  ≈ 18.4%   (occurs at G = 0.5)

Slotted ALOHA:
  S = G × e^(−G)
  Maximum throughput  ≈ 36.8%   (occurs at G = 1)
```

> Slotted ALOHA doubles the maximum throughput of Pure ALOHA simply by synchronizing transmissions to slot boundaries.

---

### 4.2 CSMA (Carrier Sense Multiple Access)

```
Rule: "Listen before you talk."
      A station first SENSES the medium. If it's busy, it waits.
      If it's idle, it transmits.

Improvement over ALOHA: Reduces (but does NOT eliminate) collisions,
because two stations can still both sense "idle" at nearly the same
moment (due to propagation delay) and transmit simultaneously.
```

**CSMA Persistence Strategies** (what to do if the channel is sensed busy):

| Strategy           | Behavior                                                                                                                                   |
| ------------------ | ------------------------------------------------------------------------------------------------------------------------------------------ |
| **1-Persistent**   | Keep sensing continuously; transmit immediately once idle (greedy — higher collision chance)                                               |
| **Non-Persistent** | Wait a **random** amount of time, then sense again (reduces collision chance, but adds delay)                                              |
| **p-Persistent**   | If idle, transmit with probability `p`; with probability `(1−p)` defer to the next slot (used in slotted channels; balances the above two) |

---

### 4.3 CSMA/CD (Collision Detection)

```
Rule: Same as CSMA (listen before talk), PLUS:
      While transmitting, the station keeps LISTENING.
      If it detects a collision → STOP transmitting immediately,
      send a short jam signal, then wait a random backoff time
      before retrying.

Key improvement: Doesn't waste time transmitting a full frame that's
already doomed to collide — aborts early, saving bandwidth.

Classic use case: Traditional (shared-medium/hub-based) Ethernet.
Modern switched, full-duplex Ethernet does NOT need CSMA/CD, since
each link is effectively point-to-point.
```

**Efficiency formula:**

```
Efficiency = 1 / (1 + 5×(Tp / Tt))

Where:
  Tp = propagation time (time for signal to travel end-to-end)
  Tt = transmission time (time to put the whole frame on the wire)

As Tp/Tt → 0 (very fast transmission relative to propagation),
efficiency → 1 (100%).

As Tp/Tt grows (long cable / large network relative to frame size),
efficiency drops sharply.
```

**Minimum frame size rule (important for GATE numericals):**

```
Tt ≥ 2 × Tp

This ensures that the sender is STILL transmitting the frame when a
worst-case collision signal could arrive back — otherwise the sender
would have already finished sending and would have no way to detect
the collision at all.

Minimum frame length = 2 × Tp × Bandwidth
```

---

### 4.4 CSMA/CA (Collision Avoidance)

```
Rule: Since detecting collisions is often IMPOSSIBLE on wireless media
      (a station can't reliably listen while transmitting on the same
      frequency, and signal strength drops off with distance —
      the "hidden node problem"), CSMA/CA focuses on AVOIDING
      collisions altogether rather than detecting them.

Mechanisms used:
  1. Interframe Space (IFS) — wait a fixed gap before sensing again
  2. Contention Window — random backoff timer even before first attempt
  3. Acknowledgment (ACK) — receiver sends ACK on success; no ACK
     within a timeout implies collision/loss, triggering retransmission
  4. RTS/CTS handshake (optional) — Request To Send / Clear To Send,
     used to reserve the medium and combat the hidden node problem

Classic use case: Wi-Fi (IEEE 802.11)
```

---

## 5. Category 2 — Controlled Access Protocols

**Defining characteristic:** Some form of **controlling authority** decides whose turn it is to transmit — collisions are structurally prevented rather than detected/avoided.

### 5.1 Polling

```
Rule: A central controller (e.g., a primary/master station) asks each
      station in turn, "Do you have data to send?"
      Only the station that "wins" that round of polling may transmit;
      all others must wait.

Analogy (from the lecture): Like an actual poll/vote among A, B, C, D —
whoever the controller polls and confirms has data gets to go first.

Drawback: The controller itself can become a single point of failure
or a bottleneck (polling overhead if many stations have no data to send).
```

### 5.2 Token Passing

```
Rule: A special control frame called a TOKEN continuously circulates
      around the network (commonly a ring topology — "Token Ring").
      A station may transmit ONLY while it is holding the token.
      Once done, it releases the token to the next station in sequence.

Key exam topics (GATE has asked about these specifically):
  - Delayed Token Reinsertion: the sending station waits for its own
    frame to circle all the way back around before releasing the
    token — safer, but wastes ring time.
  - Early Token Release (ETR): the sending station releases the token
    immediately after finishing transmission, WITHOUT waiting for its
    frame to come back — improves ring utilization/throughput,
    especially on rings with high propagation delay relative to
    frame transmission time.
```

---

## 6. Category 3 — Channelization Protocols

**Defining characteristic:** The shared medium's total capacity (bandwidth, time, or code space) is **divided in advance** among the stations, so no contention/collision is possible by design.

### 6.1 FDMA (Frequency Division Multiple Access)

```
Rule: The available frequency band is split into fixed, non-overlapping
      sub-bands (channels). Each station is permanently assigned one
      sub-band and may transmit continuously within it.

Analogy: Like separate radio stations, each on its own frequency —
no two stations ever interfere because they're on different frequencies.
```

### 6.2 TDMA (Time Division Multiple Access)

```
Rule: All stations share the SAME frequency, but time is divided into
      fixed slots/frames. Each station may only transmit during its
      assigned time slot; it must stay silent otherwise.

Analogy: Like a shared meeting room where each person gets an exact
5-minute speaking slot in a fixed rotation — no overlap possible if
everyone respects their slot.

Requires: Tight time synchronization among all stations.
```

### 6.3 CDMA (bonus — completeness)

```
Rule: Every station transmits at the SAME time, on the SAME frequency,
      but each is assigned a unique orthogonal "code." The receiver
      uses that code to extract only the intended signal from the
      combined transmission (used in older cellular networks — the
      "CDMA" in 3G "CDMA2000" networks).

Not covered directly in this lecture, but commonly grouped alongside
FDMA/TDMA in the Channelization category for exams.
```

---

## 7. Formula Reference Sheet

```
┌───────────────────────────────────────────────────────────────────┐
│  ALOHA                                                             │
│    Pure ALOHA   vulnerable time = 2 × Tt                          │
│    Slotted ALOHA vulnerable time = Tt                              │
│    Pure ALOHA    throughput S = G·e^(−2G)   →  max ≈ 18.4% at G=0.5│
│    Slotted ALOHA throughput S = G·e^(−G)    →  max ≈ 36.8% at G=1  │
├───────────────────────────────────────────────────────────────────┤
│  CSMA/CD                                                            │
│    Efficiency = 1 / (1 + 5·(Tp/Tt))                                │
│    Minimum frame transmission time: Tt ≥ 2 × Tp                    │
│    Minimum frame length = 2 × Tp × Bandwidth                       │
├───────────────────────────────────────────────────────────────────┤
│  General definitions                                               │
│    Tt (transmission time) = Frame size / Bandwidth                 │
│    Tp (propagation time)  = Distance / Propagation speed           │
└───────────────────────────────────────────────────────────────────┘
```

---

## 8. Quick Reference Rules

```
┌─────────────────────────────────────────────────────────────────┐
│  RULE 1: Identify whether MAC is even relevant                  │
│    Shared/broadcast medium → MAC needed                         │
│    Dedicated point-to-point links → MAC NOT needed               │
│                                                                   │
│  RULE 2: Random Access family (memorize this order of maturity) │
│    ALOHA → CSMA → CSMA/CD → CSMA/CA                              │
│    Each step reduces collision risk further than the last       │
│                                                                   │
│  RULE 3: CD vs CA — pick based on medium                        │
│    Wired, shared, low propagation delay → CSMA/CD                │
│    Wireless, hidden-node-prone → CSMA/CA                         │
│                                                                   │
│  RULE 4: Controlled Access = no collisions by design             │
│    Polling → central controller decides turn                    │
│    Token Passing → possession of token decides turn              │
│                                                                   │
│  RULE 5: Channelization = pre-divided resource, no contention    │
│    FDMA → split frequency   |   TDMA → split time                │
│    CDMA → split by orthogonal code, same time & frequency        │
└─────────────────────────────────────────────────────────────────┘
```

---

## 9. Worked Numerical Examples

> The lecture itself is conceptual/overview-level (numericals are covered in follow-up lectures per protocol), but these are the standard GATE-style numericals you should practice using the formulas above.

### Example 1 — Pure ALOHA Throughput

```
Given: G = 0.5 (offered load)
S = G × e^(−2G) = 0.5 × e^(−1) = 0.5 × 0.3679 = 0.1839
→ Throughput ≈ 18.4%  (this is the theoretical maximum for Pure ALOHA)
```

### Example 2 — Slotted ALOHA Throughput

```
Given: G = 1
S = G × e^(−G) = 1 × e^(−1) = 0.3679
→ Throughput ≈ 36.8%  (theoretical maximum for Slotted ALOHA)
```

### Example 3 — CSMA/CD Efficiency

```
Given:
  Bandwidth = 10 Mbps
  Frame size = 1000 bits  → Tt = 1000 / 10×10^6 = 100 µs
  Distance = 2000 m, Propagation speed = 2×10^8 m/s
  Tp = 2000 / 2×10^8 = 10 µs

Efficiency = 1 / (1 + 5×(Tp/Tt))
           = 1 / (1 + 5×(10/100))
           = 1 / (1 + 0.5)
           = 1 / 1.5
           ≈ 0.667  →  66.7%
```

### Example 4 — Minimum Frame Size for CSMA/CD

```
Given:
  Bandwidth = 10 Mbps
  Tp = 25.6 µs (round trip = 2×Tp = 51.2 µs — this is the standard
  Ethernet worst-case round-trip propagation delay figure)

Minimum Tt = 2 × Tp = 51.2 µs
Minimum frame length = Tt × Bandwidth = 51.2×10^−6 × 10×10^6 = 512 bits (64 bytes)

→ This matches the real-world Ethernet minimum frame size of 64 bytes!
```

---

## 10. 🔴 Security Context

### Shared-Medium MAC Protocols Were Never Designed With an Adversary in Mind

```
All Random Access and Controlled Access protocols assume that every
station on the medium is a COOPERATIVE, honest participant. None of
them were designed to resist a station that deliberately breaks the
rules. This has real security implications.
```

### 10.1 Jamming / Denial of Service at the MAC Layer

```
Because CSMA-family protocols rely on "listen before you talk," a
malicious station can simply:
  - Transmit continuously (ignoring carrier sense) to keep the medium
    permanently "busy," starving all honest stations — a MAC-layer
    Denial of Service.
  - On Wi-Fi (CSMA/CA), send bogus RTS/CTS frames to reserve the
    channel for long durations without ever actually transmitting
    useful data ("virtual carrier sense abuse" / NAV attacks).

This is conceptually similar to how a physical-layer jammer works,
but it can be done purely by misusing the protocol logic — no special
radio hardware required beyond a normal Wi-Fi adapter in monitor/
injection mode.
```

### 10.2 Token Theft / Token Ring Abuse

```
In Token Passing networks, a malicious or malfunctioning station
could refuse to release the token — freezing the entire ring, since
by design NO ONE else may transmit without holding the token. This is
a good illustration of why "controlled access" protocols also
concentrate risk: whoever controls the token controls the network.
```

### 10.3 Hidden Node Exploitation

```
The "hidden node problem" (two stations that can each reach an access
point but not each other) is normally just a reliability issue —
but on wireless networks it can be leveraged to increase collision
rates deliberately, degrading service for other legitimate clients
without needing to associate with the network as a fully trusted peer.
```

### 10.4 What Actually Defends the MAC Layer

| Mechanism                             | Purpose                                                | Defends Against MAC-layer abuse?                      |
| ------------------------------------- | ------------------------------------------------------ | ----------------------------------------------------- |
| CSMA/CD, CSMA/CA (as designed)        | Fairness among cooperative stations                    | ❌ No — assumes good faith                            |
| 802.11w (Management Frame Protection) | Protects Wi-Fi management/control frames from forgery  | ✅ Partially (mitigates some deauth/disassoc attacks) |
| Port security / 802.1X                | Authenticates devices before granting network access   | ✅ Yes (limits who can even join the shared medium)   |
| Wireless IDS/IPS                      | Detects abnormal jamming/flooding patterns at RF layer | ✅ Yes (detection, not prevention)                    |
| Physical layer spectrum monitoring    | Detects RF jamming outside protocol logic              | ✅ Yes                                                |

> **Rule:** MAC protocols solve the _engineering_ problem of fair, collision-free sharing among honest stations. They do **not** solve the _security_ problem of a dishonest station on the same medium — that requires authentication, monitoring, and physical/RF-layer defenses layered on top.

---

## 11. 🧪 Practical Labs

### Lab 1 — Simulate Pure vs Slotted ALOHA Throughput in Python

```python
# Save as aloha_simulation.py — run: python3 aloha_simulation.py
import random

def simulate_pure_aloha(num_stations=20, num_slots=100000, load=0.5):
    """Rough Monte Carlo simulation of Pure ALOHA collision behavior."""
    successful = 0
    attempts = 0
    for _ in range(num_slots):
        # Each station independently decides to transmit with prob = load/num_stations
        transmitting = [s for s in range(num_stations) if random.random() < load / num_stations]
        if len(transmitting) == 1:
            successful += 1
        attempts += len(transmitting)
    throughput = successful / num_slots
    return throughput

def simulate_slotted_aloha(num_stations=20, num_slots=100000, load=1.0):
    """Slotted ALOHA: transmissions only start at slot boundaries."""
    successful = 0
    for _ in range(num_slots):
        transmitting = [s for s in range(num_stations) if random.random() < load / num_stations]
        if len(transmitting) == 1:
            successful += 1
    throughput = successful / num_slots
    return throughput

pure_throughput = simulate_pure_aloha(load=0.5)
slotted_throughput = simulate_slotted_aloha(load=1.0)

print(f"Pure ALOHA simulated throughput:    {pure_throughput*100:.2f}%  (theory: 18.4%)")
print(f"Slotted ALOHA simulated throughput: {slotted_throughput*100:.2f}%  (theory: 36.8%)")
```

### Lab 2 — Calculate CSMA/CD Efficiency & Minimum Frame Size

```python
# Save as csma_cd_calc.py — run: python3 csma_cd_calc.py

def csma_cd_efficiency(tp_seconds: float, tt_seconds: float) -> float:
    return 1 / (1 + 5 * (tp_seconds / tt_seconds))

def min_frame_size_bits(tp_seconds: float, bandwidth_bps: float) -> float:
    tt_min = 2 * tp_seconds
    return tt_min * bandwidth_bps

# Example values (classic 10 Mbps Ethernet numbers)
bandwidth = 10e6      # 10 Mbps
frame_bits = 1000
tt = frame_bits / bandwidth

distance_m = 2000
prop_speed = 2e8      # 2x10^8 m/s (typical for copper)
tp = distance_m / prop_speed

eff = csma_cd_efficiency(tp, tt)
print(f"Transmission time (Tt): {tt*1e6:.2f} µs")
print(f"Propagation time (Tp):  {tp*1e6:.2f} µs")
print(f"CSMA/CD Efficiency:     {eff*100:.2f}%")

min_bits = min_frame_size_bits(tp, bandwidth)
print(f"Minimum frame size for this Tp/bandwidth: {min_bits:.0f} bits")

# Real Ethernet worst-case round-trip propagation figure
tp_ethernet = 25.6e-6
min_bits_ethernet = min_frame_size_bits(tp_ethernet, bandwidth)
print(f"\nClassic Ethernet check — Tp = 25.6 µs, BW = 10 Mbps")
print(f"Minimum frame size: {min_bits_ethernet:.0f} bits ({min_bits_ethernet/8:.0f} bytes)")
print("Real Ethernet minimum frame size is 512 bits / 64 bytes — matches!")
```

### Lab 3 — Observe Real Collisions & CSMA/CD Behavior with Wireshark/tcpdump

```bash
# On your own lab network (Parrot OS ↔ Metasploitable2), most modern
# switched networks are full-duplex and collision-free, so to actually
# OBSERVE collision-domain behavior you'd typically need a hub (rare
# today) or a network namespace / virtual bridge setup. This lab instead
# focuses on inspecting frame timing and MAC-layer fields you CAN see.

# Capture raw frames including MAC-layer info
sudo tcpdump -i eth0 -e -c 20 host 192.168.56.101
# -e shows the Ethernet (MAC) header for every frame: source/dest MAC,
# EtherType — useful for understanding what MAC-layer fields exist.

# In Wireshark:
#   Statistics → Capture File Properties  → check avg frame rate/timing
#   Right-click a frame → "Follow" → observe inter-frame timing
#   (In a real shared/hub network you could look for `eth.fcs_bad`
#   spikes correlating with high traffic bursts as a proxy for collisions)
```

### Lab 4 — Simulate a Token Ring's "Whoever Holds the Token Can Transmit" Rule

```python
# Save as token_ring_sim.py — run: python3 token_ring_sim.py
import time
import random

class TokenRingStation:
    def __init__(self, name):
        self.name = name
        self.has_data = random.choice([True, False])

def run_token_ring(stations, rounds=3, early_release=False):
    idx = 0
    for r in range(rounds):
        for _ in range(len(stations)):
            station = stations[idx % len(stations)]
            if station.has_data:
                print(f"[Round {r+1}] {station.name} holds token → TRANSMITTING")
                if early_release:
                    print(f"           (Early Token Release: token passed on immediately)")
                else:
                    print(f"           (Delayed Reinsertion: waiting for frame to circle back first)")
            else:
                print(f"[Round {r+1}] {station.name} holds token → no data, passes immediately")
            idx += 1

stations = [TokenRingStation(n) for n in ["A", "B", "C", "D"]]
print("=== Token Ring simulation (Early Token Release) ===")
run_token_ring(stations, rounds=1, early_release=True)
```

### Lab 5 — Demonstrate a MAC-Layer "Channel Hogging" Concept (Defensive Awareness Only)

```python
# Save as channel_fairness_demo.py — run: python3 channel_fairness_demo.py
# Purely a conceptual simulation to illustrate WHY fairness assumptions
# in CSMA matter for network defenders — no real transmission involved.

import random

def simulate_csma_fairness(num_honest=5, malicious_ignores_backoff=True, slots=20):
    honest_successes = 0
    malicious_successes = 0

    for _ in range(slots):
        honest_wants_to_send = [random.random() < 0.3 for _ in range(num_honest)]
        malicious_wants_to_send = True  # malicious station always tries

        contenders = sum(honest_wants_to_send) + 1

        if malicious_ignores_backoff:
            # Malicious station transmits regardless of channel state —
            # in a real network this causes it to "win" far more often
            malicious_successes += 1
        elif contenders == 1 and any(honest_wants_to_send):
            honest_successes += 1

    print(f"Honest stations' successful transmissions: {honest_successes}")
    print(f"Malicious (rule-breaking) station's successes: {malicious_successes}")
    print("\nTakeaway: CSMA's fairness entirely depends on every station")
    print("following carrier-sense + backoff rules honestly. A station")
    print("that ignores those rules can dominate the shared medium —")
    print("this is why authentication + RF monitoring matter, not just")
    print("the MAC protocol's built-in logic.")

simulate_csma_fairness()
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║        MEDIUM ACCESS CONTROL PROTOCOLS — EXAM CHEAT SHEET           ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHERE IT SITS                                                        ║
║  Data Link Layer → split into LLC (framing/error/flow control)       ║
║  and MAC (who transmits, when, on a SHARED medium)                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  3 CATEGORIES                                                         ║
║  1. Random Access    → ALOHA, CSMA, CSMA/CD, CSMA/CA                 ║
║  2. Controlled Access → Polling, Token Passing (Token Ring)          ║
║  3. Channelization    → FDMA, TDMA, CDMA                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  ★ HIGHEST EXAM PRIORITY: CSMA & CSMA/CD, then ALOHA, then Token Ring║
╠══════════════════════════════════════════════════════════════════════╣
║  ALOHA FORMULAS                                                       ║
║  Pure ALOHA:    vulnerable time = 2Tt | S = G·e^(−2G) | max ≈ 18.4%  ║
║  Slotted ALOHA: vulnerable time = Tt  | S = G·e^(−G)  | max ≈ 36.8%  ║
╠══════════════════════════════════════════════════════════════════════╣
║  CSMA/CD FORMULAS                                                     ║
║  Efficiency = 1 / (1 + 5·(Tp/Tt))                                    ║
║  Minimum Tt = 2 × Tp  →  Minimum frame length = 2×Tp×Bandwidth       ║
║  (Real Ethernet: 64 bytes minimum frame size)                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  CD vs CA                                                             ║
║  CD (Collision Detection) → wired, listens WHILE transmitting        ║
║  CA (Collision Avoidance) → wireless, can't reliably detect during   ║
║  transmission, so it uses IFS + random backoff + ACK + RTS/CTS       ║
╠══════════════════════════════════════════════════════════════════════╣
║  CONTROLLED ACCESS                                                    ║
║  Polling → central controller grants turns                           ║
║  Token Passing → possession of the token = permission to transmit    ║
║  GATE favorites: Delayed Token Reinsertion vs Early Token Release    ║
╠══════════════════════════════════════════════════════════════════════╣
║  CHANNELIZATION                                                       ║
║  FDMA → divides FREQUENCY   |   TDMA → divides TIME                  ║
║  CDMA → divides by unique CODE (same time & frequency for all)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                       ║
║  MAC protocols assume cooperative, honest stations — they are NOT    ║
║  designed to resist a deliberately rule-breaking station             ║
║  A station ignoring carrier-sense/backoff can dominate/DoS a shared  ║
║  medium; token-holding stations can freeze a Token Ring if malicious ║
║  Real defenses: 802.1X port authentication, 802.11w management      ║
║  frame protection, wireless IDS/IPS, RF spectrum monitoring          ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] ALOHA — detailed numericals (Pure vs Slotted, vulnerable time derivations)
- [ ] CSMA/CD — detailed numericals (efficiency, minimum frame size problems)
- [ ] Token Ring — Delayed Token Reinsertion vs Early Token Release deep dive
- [ ] Network Layer — IP Addressing, Subnetting
- [ ] Wireless LAN security — WEP/WPA/WPA2/WPA3 comparison

---

_Notes compiled from: Networking Course Lecture 31 — Medium/Multiple Access Control Protocols_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
