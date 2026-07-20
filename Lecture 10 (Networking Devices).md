# 🔗 Networking Devices — Repeater

### " "Networking Course — Lecture 10

> **Source:** Gate Smashers — Networking Devices: Repeater
> **Exam Relevance:** GATE (lower priority) · UGC NET · PSU Exams · University Exams
> **Topic Area:** Physical Layer — Signal Regeneration Device

---

## 📑 Table of Contents

1. [What is a Repeater?](#1-what-is-a-repeater)
2. [Why Do We Use a Repeater?](#2-why-do-we-use-a-repeater)
3. [Repeater vs Amplifier — Not the Same Thing](#3-repeater-vs-amplifier--not-the-same-thing)
4. [Effect on Distance Coverage](#4-effect-on-distance-coverage)
5. [Physical Structure — Two-Port Device](#5-physical-structure--two-port-device)
6. [Does a Repeater Forward?](#6-does-a-repeater-forward)
7. [Does a Repeater Filter?](#7-does-a-repeater-filter)
8. [Collision Domain of a Repeater](#8-collision-domain-of-a-repeater)
9. [Summary Table](#9-summary-table)
10. [Why This Matters for Cybersecurity](#10-why-this-matters-for-cybersecurity)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. What is a Repeater?

A **repeater** operates purely at the **Physical Layer (Layer 1)** of the OSI model — it is **purely hardware**.

**Why physical layer, not data link layer?**

| Layer                         | What it can check                            |
| ----------------------------- | -------------------------------------------- |
| Data Link Layer               | MAC address                                  |
| Network Layer                 | IP address                                   |
| **Physical Layer (Repeater)** | **Nothing** — no addressing awareness at all |

Since a repeater has **no software**, it has no role or capability at the data link layer (cannot check MAC addresses) or above.

---

## 2. Why Do We Use a Repeater?

**Scenario (using the `10Base2` cable standard from Part 9):**

```
10Base2 = 10 Mbps, Baseband, 200m distance before attenuation

[Device] ════════════ 200m ════════════ [Attenuation — signal too weak]
```

- A LAN is built using `10Base2` cable — 200 meters, after which **attenuation** occurs (signal strength drops, becomes corrupted/lost, unusable).
- To extend the network, another `10Base2` segment is added — but it too is limited to 200m.
- **Solution: insert a repeater** between the two segments.

**What does a repeater do?** It takes a signal whose strength has weakened and **regenerates** it back to its original strength.

---

## 3. Repeater vs Amplifier — Not the Same Thing

A very common exam confusion: _"Isn't a repeater just an amplifier?"_ — **No.**

| Device        | Behavior                                                                                                                                                                                                         |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Amplifier** | Takes a signal (original strength `x`, now weakened) and **increases** it — possibly to `2x`, `4x`, or beyond. It boosts the signal _and_ any noise along with it, without restoring a specific reference value. |
| **Repeater**  | Takes the weakened signal and **regenerates it bit by bit back to its exact original strength/voltage (`x`)** — not amplified beyond that, but precisely restored to the original value.                         |

> **Exam tip:** Repeater = **regenerates to original strength**. Amplifier = **boosts strength (often beyond original, and boosts noise too)**. These are frequently tested as a "same or different" trick question — the answer is **different**.

---

## 4. Effect on Distance Coverage

```
Segment 1 (200m) ══[Repeater — regenerates signal to original strength]══ Segment 2 (200m)

Total effective distance = 400m (2 × 200m)
```

- By regenerating the signal at the point where it would otherwise experience attenuation, a repeater effectively **increases the total distance a signal can travel**.
- With one repeater and two `10Base2` segments: **200m + 200m = 400m** total coverage.

---

## 5. Physical Structure — Two-Port Device

- A repeater is a **two-port device**: Port 1 and Port 2.
- Each port connects to a **different wire/cable segment**.
- It is purely a **physical layer hardware device**.

```
[Cable Segment A] ──►[Port 1: Repeater :Port 2]──► [Cable Segment B]
```

---

## 6. Does a Repeater Forward?

**Yes — a repeater always forwards.**

```
A ──(message)──► [Repeater] ──(forwarded)──► B
```

- When a signal reaches the repeater from one side, it **forwards** it onward through the other port onto the connected link.
- This is unconditional — the repeater has no logic to decide otherwise.

---

## 7. Does a Repeater Filter?

**No — a repeater cannot filter.**

**What does "filter" mean here?** Stopping a signal from moving further if it doesn't need to (e.g., because the destination is already within the same local segment/LAN).

```
Segment 1: [A]────[B]────[Repeater]────[Segment 2]

If A sends a message to B (both on Segment 1, same link):
Does the repeater still forward the signal onward to Segment 2?
YES — even though B is already reachable without crossing the repeater.
```

- The repeater has **no software** to recognize that the destination is already local and that forwarding is unnecessary.
- **Devices with software** (e.g., **routers**, **bridges** — covered in later parts) _do_ have this filtering capability, because they can inspect addressing information.
- A repeater, being pure hardware, **always forwards, never filters.**

---

## 8. Collision Domain of a Repeater

**Maximum possible collisions through a repeater = `n`**, where `n` = the **total number of devices connected on both sides** of the repeater.

**Why is the collision domain not reduced by the repeater?**

- A repeater has **no buffer/memory** to store and forward packets (this "store-and-forward" behavior belongs to devices like switches/bridges, covered later).
- Any signal arriving from either port is immediately forwarded — if signals arrive from both sides at the same time, they **collide**, just as if all devices were on one single shared cable.
- Since the repeater doesn't isolate or manage traffic in any way, it **does not create separate collision domains** — the entire combined network (both connected segments) remains **one collision domain**.

---

## 9. Summary Table

| Property              | Repeater                                                                                          |
| --------------------- | ------------------------------------------------------------------------------------------------- |
| **OSI Layer**         | Physical Layer (Layer 1)                                                                          |
| **Hardware/Software** | Pure hardware                                                                                     |
| **Ports**             | 2 (two-port device)                                                                               |
| **Forwarding**        | Always forwards                                                                                   |
| **Filtering**         | Cannot filter (no MAC/IP awareness)                                                               |
| **Collision domain**  | Single combined domain — max collisions = `n` (all devices on both sides)                         |
| **Function**          | Regenerates a weakened signal to its original strength, extending effective transmission distance |
| **Not the same as**   | Amplifier (which boosts signal, potentially beyond original strength, including noise)            |

---

## 10. Why This Matters for Cybersecurity

- **No isolation = broad exposure:** Since a repeater doesn't create separate collision or broadcast domains, and cannot filter traffic at all, any traffic-sniffing risk that exists on one segment effectively extends across the entire repeated network — reinforcing the same lesson from Part 6 (Bus topology) and Part 9 (collision domains): shared, unfiltered media is inherently harder to secure at Layer 1.
- **Layer 1 devices as "dumb" relay points:** From a security architecture perspective, repeaters (like hubs) are a good example of _why_ modern networks rely on Layer 2/3 devices with filtering intelligence (switches, routers, firewalls) rather than layer 1 signal-relay devices for any kind of segmentation or access control.
- **Physical-layer troubleshooting vs security controls:** Understanding that a repeater's only job is signal regeneration (not security) helps clarify why physical-layer devices are never an appropriate place to enforce access control — that responsibility belongs to higher-layer devices covered in upcoming parts (bridge, switch, router, firewall).

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                    REPEATER — EXAM CHEAT SHEET                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI LAYER:  Physical Layer (Layer 1) — Pure Hardware                ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE FUNCTION                                                          ║
║  ─────────────────────────────────────────────────────────          ║
║      Regenerates a WEAKENED signal back to its ORIGINAL strength     ║
║      → increases the DISTANCE a signal can travel                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  REPEATER vs AMPLIFIER                                                 ║
║  ─────────────────────────────────────────────────────          ║
║      Repeater  → restores signal to ORIGINAL strength (x → x)        ║
║      Amplifier → boosts signal, possibly BEYOND original (x → 2x/4x),║
║                   also amplifies noise                                ║
║      They are NOT the same device/purpose                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FACTS                                                              ║
║  ─────────────────────────────────────────────────────          ║
║      Ports              → 2 (two-port device)                         ║
║      Forwarding         → YES, always forwards                       ║
║      Filtering          → NO, cannot filter (no MAC/IP awareness)    ║
║      Collision domain   → n (all devices on both connected sides)    ║
║      Buffer/memory      → NONE (no store-and-forward capability)     ║
╠══════════════════════════════════════════════════════════════════════╣
║  NOTE (per instructor): Don't over-study sub-types (analog/digital/  ║
║  optical repeaters) — not needed for GATE; basic points above are    ║
║  sufficient for UGC NET / PSU exams.                                  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Hub — active vs passive, why it's considered a multi-port repeater
- [ ] Bridge — filtering capability, how it differs from a repeater
- [ ] Switch — detailed comparison with hub and bridge

---

_Notes compiled from: Networking Course Lecture 10 — Networking Devices: Repeater_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
