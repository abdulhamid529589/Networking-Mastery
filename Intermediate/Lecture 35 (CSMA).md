# 🔀 CSMA/CD — Carrier Sense Multiple Access / Collision Detection

### " "Networking Course — Lecture 32

> **Source:** Gate Smashers — CSMA/CD (Collision Detection Deep Dive)
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is CSMA/CD?](#1-what-is-csmacd)
2. [Why No Acknowledgment System?](#2-why-no-acknowledgment-system)
3. [Key Terminology](#3-key-terminology)
4. [How a Station Detects Its Own Collision](#4-how-a-station-detects-its-own-collision)
5. [Scenario Walkthrough — Synchronized Start](#5-scenario-walkthrough--synchronized-start)
6. [Worst-Case Scenario Derivation](#6-worst-case-scenario-derivation)
7. [The Core Formula: Tt ≥ 2 × Tp](#7-the-core-formula-tt--2--tp)
8. [Efficiency of CSMA/CD](#8-efficiency-of-csmacd)
9. [Quick Reference Rules](#9-quick-reference-rules)
10. [Worked Numerical Examples](#10-worked-numerical-examples)
11. [🔴 Security Context](#11--security-context)
12. [🧪 Practical Labs](#12--practical-labs)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. What is CSMA/CD?

**CSMA/CD (Carrier Sense Multiple Access with Collision Detection)** is a modification of plain CSMA. The modification is the **"CD"** part — a transmitting station can **detect its own collision while it is still transmitting**, instead of only sensing the channel before it starts.

```
CSMA:     Listen before you talk (sense, then transmit)
CSMA/CD:  Listen before you talk  +  keep listening WHILE you talk
          → if a collision is detected mid-transmission, STOP immediately
```

> **Classic use case:** traditional shared-medium (hub-based) Ethernet LANs. This is the mechanism that made early Ethernet networks usable on a shared coaxial/hub medium.

---

## 2. Why No Acknowledgment System?

A common student question: _"Why not just use acknowledgments (ACKs) to know if data collided, like other protocols do?"_

```
Reasoning against ACKs in CSMA/CD:

  On a shared medium, EVERY transmission (data AND ack) competes for
  the same channel. Data-vs-data collisions already happen; adding an
  ACK for every frame means the channel now carries twice the traffic
  (data + ack), which increases collision frequency significantly.

  → CSMA/CD deliberately has NO acknowledgment system.
  → Instead, the sender detects collision directly, in real time,
    by listening to the channel while it transmits.
```

This is the foundational reason the entire "Collision Detection" mechanism exists — it's the substitute for acknowledgments.

---

## 3. Key Terminology

| Term                          | Meaning                                                                                                                        |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| **Transmission Time (Tt)**    | Time to put an entire frame onto the channel = Frame length / Bandwidth                                                        |
| **Propagation Time (Tp)**     | Time for a bit to physically travel from sender to the farthest receiver = Distance / Velocity                                 |
| **Propagation Delay**         | Same as Tp — used interchangeably in the lecture                                                                               |
| **Collision Signal**          | The corrupted/garbled signal produced when two transmissions overlap on the medium                                             |
| **Jam Signal**                | (Standard CSMA/CD addition, not detailed in this lecture) A short signal sent to ensure all stations know a collision occurred |
| **a (in efficiency formula)** | Ratio a = Tp / Tt (propagation time divided by transmission time)                                                              |

---

## 4. How a Station Detects Its Own Collision

**Core rule (the entire funda of Collision Detection):**

```
A station can ONLY know its own data collided if:

  It is STILL TRANSMITTING at the exact moment the collision signal
  arrives back at it.

If transmission has already finished before the collision signal
returns, the station CANNOT tell whether it was its own data that
collided, or some completely unrelated collision between other
stations. It has no way to distinguish the two cases.
```

**Physical reasoning:**

- When Station A transmits, its signal **propagates** outward to every other station on the shared medium (B, C, D, …).
- Every station checks the destination address in the frame header. If the address doesn't match them, they discard it; only the intended destination accepts it. (This is standard LAN broadcast-then-filter behavior.)
- If another station (say D) starts transmitting **before A's signal has fully cleared the medium**, the two signals physically overlap at some point on the wire, producing a garbled, invalid signal — the "collision."
- That corrupted signal then propagates back toward A. **Only if A is still actively transmitting when that corrupted signal reaches it** can A conclude "my data collided."

---

## 5. Scenario Walkthrough — Synchronized Start

**Setup:**

```
Two stations: A and B
Propagation delay between them: 1 hour (deliberately exaggerated for teaching clarity)
Both sense the channel at exactly 12:00 PM and find it idle
Both immediately begin transmitting at 12:00 PM
```

**What happens:**

```
12:00 PM — A starts transmitting. B starts transmitting (each unaware of the other,
           since neither's signal has reached the other yet — CSMA can only sense
           "locally," not across the whole network instantly).

12:30 PM — Exactly halfway between A and B (since distance and propagation speed
           are identical in both directions), the two signals COLLIDE.
           A useless, corrupted signal now begins propagating outward in both
           directions from the collision point.

1:00 PM  — The collision signal reaches BOTH A and B (each 30 minutes after the
           collision, since the collision occurred at the midpoint).
           Since both A and B are STILL transmitting at 1:00 PM (their planned
           transmission runs at least that long in this scenario), both stations
           successfully detect that their data collided.
```

**Key insight extracted from this scenario:**

```
Transmission Time should be GREATER than Propagation Delay.

If either station had stopped transmitting before the collision
signal returned to it, that station would never have known its data
collided.
```

This synchronized-start scenario is the **simple case**. The **worst case** (used for deriving the actual formula used in exams) is different and more subtle.

---

## 6. Worst-Case Scenario Derivation

**Setup (same propagation delay, but staggered start — the worst case):**

```
A and B, propagation delay = 1 hour, as before.

12:00 PM — A senses the channel, finds it idle, and immediately starts
           transmitting. Transmission time = 1 hour (so A finishes at 1:00 PM
           if uninterrupted).

12:59:59 PM — (i.e., just before A's first bit arrives at B) — B senses the
              channel. Because A's signal has NOT YET physically reached B
              (it takes a full hour to arrive), B also senses "idle" and
              immediately starts transmitting.

              The instant B transmits, its first bit collides with A's
              signal arriving at that same location (B's location).
              B notices this collision INSTANTLY (it happens right at B's
              own position), and B stops immediately.

              But we care about the WORST CASE for A, not B.

~2:00 PM — The collision (which occurred essentially at B's location, at
           just before 1:00 PM) now has to propagate all the way back to A
           — another full hour of propagation delay.
           A only learns about the collision when this signal reaches it,
           approximately at 2:00 PM.
```

**The critical condition:** A can only detect this collision **if A is still transmitting when the collision signal returns**, at ~2:00 PM. Since A started at 12:00 PM, that means:

```
A must still be transmitting a full 2 hours after starting
(1 hour for its own signal to reach B, plus 1 more hour for the
collision signal to propagate back to A)

→ Transmission Time (Tt) must be ≥ 2 × Propagation Delay (Tp)
```

> This is the **worst case** because B started transmitting at the very last possible instant before A's signal arrived — maximizing the delay before the collision is even created, and therefore maximizing the total round-trip time before A can find out.

---

## 7. The Core Formula: Tt ≥ 2 × Tp

```
┌─────────────────────────────────────────────────────────────┐
│   Tt ≥ 2 × Tp                                                │
│                                                                │
│   Expanded (since Tt = Frame Length / Bandwidth):            │
│                                                                │
│   Frame Length / Bandwidth  ≥  2 × Tp                        │
│                                                                │
│   ⇒  Frame Length  ≥  2 × Tp × Bandwidth                      │
└─────────────────────────────────────────────────────────────┘
```

**Why this matters:** This formula tells you the **minimum frame length** a network must use, given its propagation delay and bandwidth, in order for CSMA/CD's collision detection to actually work at all. If frames are shorter than this minimum, a sender could finish transmitting _before_ a worst-case collision signal gets back to it — meaning it would never know its data collided.

> **Both formulas are important and commonly tested:**
>
> 1. `Tt ≥ 2 × Tp`
> 2. `Frame Length ≥ 2 × Tp × Bandwidth`
>
> Numericals frequently give you some of these values and ask you to check whether a given frame length/bandwidth/propagation-delay combination is valid (i.e., whether collision detection is guaranteed to work).

---

## 8. Efficiency of CSMA/CD

```
Efficiency = 1 / (1 + 6.44a)

Where:
  a = Tp / Tt   (propagation time divided by transmission time)
```

> The lecture notes that the full derivation of this formula is lengthy and will be covered in a later video — for now, **memorize the formula itself** and the definition of `a`, since numericals will directly hand you values to plug in.

> **Note for study purposes:** Different textbooks present slightly different constants in the CSMA/CD efficiency formula depending on the assumptions made about persistence strategy and traffic model (some references use `1/(1+5a)` as an approximation, others derive `1/(1+6.44a)` under different assumptions). For this course, use **`1/(1+6.44a)`** as given, and always check which version your specific textbook/instructor expects in an exam.

---

## 9. Quick Reference Rules

```
┌─────────────────────────────────────────────────────────────────┐
│  RULE 1: No acknowledgment in CSMA/CD                            │
│    Reason: ACKs would double channel traffic and increase        │
│    collisions — collision detection replaces the need for ACKs   │
│                                                                    │
│  RULE 2: A station can only detect ITS OWN collision              │
│    ...if it is STILL transmitting when the collision signal       │
│    returns to it. If transmission has ended, it cannot tell.      │
│                                                                    │
│  RULE 3: Worst case = staggered start                             │
│    One station starts just before the other's signal arrives —   │
│    maximizes the round-trip time before collision is discovered  │
│                                                                    │
│  RULE 4: The core inequality                                      │
│    Tt ≥ 2 × Tp   ⇔   Frame Length ≥ 2 × Tp × Bandwidth            │
│                                                                    │
│  RULE 5: Efficiency                                                │
│    Efficiency = 1 / (1 + 6.44a),  where a = Tp / Tt               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 10. Worked Numerical Examples

### Example 1 — Checking If a Frame Length Is Valid

```
Given:
  Bandwidth = 10 Mbps
  Propagation delay (Tp) = 25.6 µs
  Proposed frame length = 512 bits

Check: Frame Length ≥ 2 × Tp × Bandwidth ?

  2 × Tp × Bandwidth = 2 × 25.6×10^-6 × 10×10^6
                     = 2 × 256
                     = 512 bits

  Proposed frame length (512 bits) = minimum required (512 bits)
  → Exactly meets the minimum. Valid (this matches real Ethernet's
    64-byte / 512-bit minimum frame size).
```

### Example 2 — Finding Minimum Frame Length

```
Given:
  Bandwidth = 100 Mbps
  Distance = 2500 m
  Propagation speed = 2×10^8 m/s

Step 1: Tp = Distance / Speed = 2500 / 2×10^8 = 12.5 µs

Step 2: Minimum Frame Length = 2 × Tp × Bandwidth
                              = 2 × 12.5×10^-6 × 100×10^6
                              = 2 × 1250
                              = 2500 bits

→ Any frame shorter than 2500 bits would risk finishing transmission
  before a worst-case collision signal could return — collision
  detection would NOT be guaranteed to work.
```

### Example 3 — Efficiency Calculation

```
Given:
  Tp = 20 µs
  Tt = 100 µs

a = Tp / Tt = 20 / 100 = 0.2

Efficiency = 1 / (1 + 6.44 × 0.2)
           = 1 / (1 + 1.288)
           = 1 / 2.288
           ≈ 0.437  →  43.7%
```

### Example 4 — Checking Tt ≥ 2×Tp Directly

```
Given:
  Tp = 15 µs
  Frame length = 1000 bits
  Bandwidth = 20 Mbps

Tt = Frame length / Bandwidth = 1000 / 20×10^6 = 50 µs

Check: Tt ≥ 2 × Tp ?
       50 µs ≥ 2 × 15 µs = 30 µs ?
       50 ≥ 30 → TRUE

→ This frame length is valid; collision detection is guaranteed
  to work for this network configuration.
```

---

## 11. 🔴 Security Context

### CSMA/CD's Lack of Acknowledgment Is a Deliberate Efficiency Trade-off, Not a Security Feature

```
CSMA/CD assumes every station on the shared medium plays fair:
senses honestly, backs off honestly, and only uses collision
detection to recover from GENUINE accidental collisions. None of
this was designed with a hostile station in mind.
```

### 11.1 Why "No Acknowledgment" Also Means "No Built-in Tamper Evidence"

```
Because there's no acknowledgment layer at the MAC level, CSMA/CD
provides zero assurance that a frame actually reached its intended
destination uncorrupted by anything other than an honest collision.
A malicious station with access to the shared medium could:
  - Deliberately transmit at the worst-case moment (mirroring the
    scenario in Section 6) to reliably force collisions against a
    specific victim's frames — a targeted, timing-based
    Denial-of-Service technique exploiting the very physics this
    lecture describes.
  - Continuously transmit garbage to keep triggering the collision
    detection logic in every other station, effectively jamming the
    shared medium without ever needing to "win" carrier sense first.
```

### 11.2 The Minimum-Frame-Size Rule Has a Security-Adjacent Side Effect

```
The Tt ≥ 2×Tp rule is why Ethernet enforces a 64-byte minimum frame
size (padding shorter frames up to that length). This is purely an
engineering requirement for collision detection to work correctly —
but it's worth knowing for security work too: tools that craft raw
frames (e.g., Scapy) must respect this minimum, or switches/NICs may
silently pad, drop, or reject undersized frames, which can affect the
reliability of crafted-packet testing on your own lab network.
```

### 11.3 What Modern Networks Do Instead

```
Modern switched, full-duplex Ethernet eliminates the shared-medium
collision problem entirely (each link is effectively point-to-point
between a host and a switch port), which is why:
  - CSMA/CD is largely irrelevant on modern wired LANs (it's disabled/
    unused in full-duplex mode).
  - Real integrity/security assurance instead comes from upper-layer
    mechanisms: TCP checksums + retransmission, TLS integrity checks,
    and — where tamper resistance actually matters — HMAC/digital
    signatures (same conclusion as the CRC and Hamming Code security
    sections in earlier lecture notes).
```

| Mechanism                         | Purpose                                     | Defends against a deliberate attacker on the shared medium?  |
| --------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| CSMA/CD collision detection       | Recover from accidental collisions          | ❌ No — assumes honest participants                          |
| Ethernet minimum frame size (64B) | Guarantees collision detection window works | ❌ No — an engineering constraint, not a security control    |
| Switched/full-duplex Ethernet     | Removes the shared-medium collision problem | ✅ Partially (removes this specific attack surface entirely) |
| 802.1X port authentication        | Controls who can even join the medium       | ✅ Yes                                                       |
| TLS / HMAC at higher layers       | Confidentiality & tamper-evident integrity  | ✅ Yes                                                       |

---

## 12. 🧪 Practical Labs

### Lab 1 — Simulate the Worst-Case Collision Timing in Python

```python
# Save as csma_cd_worst_case.py — run: python3 csma_cd_worst_case.py
# Reproduces the "A starts at 12:00, B starts at 12:59:59" scenario
# from the lecture, using simple time units instead of literal hours.

def worst_case_detection_time(propagation_delay: float) -> float:
    """
    Returns the time (relative to A's transmission start) at which A
    will detect a worst-case collision, given a propagation delay Tp.
    Per the lecture: this is approximately 2 * Tp.
    """
    return 2 * propagation_delay

def is_frame_long_enough(frame_bits: int, bandwidth_bps: float, tp_seconds: float) -> dict:
    tt = frame_bits / bandwidth_bps
    required_tt = 2 * tp_seconds
    return {
        "Tt (transmission time, s)": tt,
        "Required Tt (2*Tp, s)": required_tt,
        "Detection guaranteed?": tt >= required_tt
    }

# Reproduce the lecture's own example: Tp = 25.6 microseconds, 10 Mbps, 512-bit frame
result = is_frame_long_enough(frame_bits=512, bandwidth_bps=10e6, tp_seconds=25.6e-6)
for k, v in result.items():
    print(f"{k}: {v}")

print()
print(f"Worst-case detection delay for Tp=25.6µs → {worst_case_detection_time(25.6e-6)*1e6:.2f} µs")
```

### Lab 2 — CSMA/CD Efficiency Calculator

```python
# Save as csma_cd_efficiency.py — run: python3 csma_cd_efficiency.py

def csma_cd_efficiency(tp: float, tt: float) -> float:
    a = tp / tt
    return 1 / (1 + 6.44 * a)

test_cases = [
    (20e-6, 100e-6),   # Tp=20µs, Tt=100µs
    (25.6e-6, 51.2e-6),
    (10e-6, 500e-6),
]

for tp, tt in test_cases:
    eff = csma_cd_efficiency(tp, tt)
    a = tp / tt
    print(f"Tp={tp*1e6:.1f}µs, Tt={tt*1e6:.1f}µs, a={a:.3f} → Efficiency = {eff*100:.2f}%")
```

### Lab 3 — Visualize Collision Timing on a Shared Medium (Text-based Timeline)

```python
# Save as collision_timeline.py — run: python3 collision_timeline.py
# A simple text-based visualization of the staggered-start worst case,
# useful for building intuition before an exam.

def print_timeline(tp_units=60):  # tp_units = propagation delay, in arbitrary "minutes"
    print(f"Propagation delay (A <-> B): {tp_units} minutes\n")
    print(f"t=0                  : A senses idle, starts transmitting")
    print(f"t={tp_units - 1:<3}               : B senses idle (A's signal not arrived yet), starts transmitting")
    print(f"t={tp_units:<3}               : Collision occurs near B's location")
    print(f"t={tp_units * 2 - 1:<3}             : Collision signal reaches B (near-instant from B's perspective)")
    print(f"t={tp_units * 2:<3}             : Collision signal reaches A — A finally detects the collision")
    print(f"\n→ A must still be transmitting at t={tp_units*2} to detect this.")
    print(f"→ Minimum required transmission time: 2 x Tp = {tp_units * 2} minutes")

print_timeline()
```

### Lab 4 — Observe Ethernet's 64-byte Minimum Frame Padding in Wireshark

```bash
# On your own lab network (Parrot OS <-> Metasploitable2), send a very
# small payload and observe that Ethernet still pads it to the minimum
# frame size — a direct, observable consequence of the Tt >= 2*Tp rule.

# Send a tiny ICMP echo with minimal payload
ping -c 3 -s 0 192.168.56.101   # -s 0 = smallest possible ICMP payload

# In Wireshark, open the capture and inspect the frame:
#   Frame length on the wire should still show >= 64 bytes total,
#   even though the IP+ICMP payload itself is much smaller — the NIC
#   pads the frame to Ethernet's minimum.
# Look at: Frame → "Frame Length" vs the sum of Ethernet+IP+ICMP header
# sizes, to see the padding bytes explicitly.
```

### Lab 5 — Craft a Minimum-Size Frame with Scapy (Educational, Own Lab Only)

```bash
sudo python3 - << 'EOF'
from scapy.all import *

# Demonstrates that crafted frames below Ethernet's minimum will be
# auto-padded by the OS/NIC — directly ties back to the Tt >= 2*Tp
# minimum-frame-length requirement discussed in this lecture.

tiny_payload = b"hi"
frame = Ether(dst="ff:ff:ff:ff:ff:ff") / Raw(load=tiny_payload)

print(f"Constructed frame length before padding: {len(frame)} bytes")
print("Sending on your own lab interface will typically result in the")
print("kernel/NIC padding this up to the 64-byte Ethernet minimum before")
print("it actually goes out on the wire.")

# To actually send on your own lab network:
# sendp(frame, iface="eth0")
EOF
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              CSMA/CD — EXAM CHEAT SHEET                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHAT IS CSMA/CD?                                                     ║
║  CSMA + ability to detect a collision WHILE still transmitting       ║
║  Used in classic shared-medium (hub-based) Ethernet LANs             ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY NO ACKNOWLEDGMENT?                                               ║
║  ACKs would double channel traffic → more collisions, lower          ║
║  efficiency. Collision detection replaces the need for ACKs.         ║
╠══════════════════════════════════════════════════════════════════════╣
║  HOW A STATION DETECTS ITS OWN COLLISION                              ║
║  ONLY if it is STILL TRANSMITTING at the moment the collision        ║
║  signal returns to it. If transmission already ended, it cannot      ║
║  tell whether the collision involved its own data or not.            ║
╠══════════════════════════════════════════════════════════════════════╣
║  WORST CASE SCENARIO                                                  ║
║  Station B starts transmitting at the LAST possible instant before   ║
║  A's signal arrives → maximizes total round-trip time before A       ║
║  can learn about the collision (≈ 2 × Tp after A started)            ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE FORMULAS (memorize both)                                        ║
║  Tt ≥ 2 × Tp                                                          ║
║  Frame Length ≥ 2 × Tp × Bandwidth                                    ║
║  (Real Ethernet: 64-byte / 512-bit minimum frame size)               ║
╠══════════════════════════════════════════════════════════════════════╣
║  EFFICIENCY FORMULA                                                   ║
║  Efficiency = 1 / (1 + 6.44a),  where a = Tp / Tt                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  NUMERICAL SKILLS TO PRACTICE                                         ║
║  - Given Tp, Bandwidth → find minimum frame length                    ║
║  - Given frame length, Bandwidth, Tp → check if Tt ≥ 2Tp holds        ║
║  - Given Tp and Tt → compute efficiency                               ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                       ║
║  CSMA/CD assumes cooperative, honest stations — provides zero        ║
║  tamper-evidence and no acknowledgment-based delivery assurance      ║
║  A hostile station could deliberately time transmissions to force    ║
║  collisions against a target, or flood the medium with garbage       ║
║  Modern switched/full-duplex Ethernet removes this shared-medium     ║
║  attack surface entirely; real integrity guarantees come from        ║
║  TCP + TLS + HMAC/signatures at higher layers, not CSMA/CD itself    ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] CSMA/CD — full numerical problem set (multiple worked GATE-style questions)
- [ ] Derivation of the 6.44a efficiency formula in detail
- [ ] CSMA/CA — deep dive (IFS, contention window, RTS/CTS, hidden node problem)
- [ ] Token Ring — Delayed Token Reinsertion vs Early Token Release deep dive
- [ ] Network Layer — IP Addressing, Subnetting

---

_Notes compiled from: Networking Course Lecture 32 — CSMA/CD (Collision Detection Deep Dive)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
