# 🔗 Digital-to-Digital Encoding — Manchester & Differential Manchester

### Cybersecurity Student Notes | Networking Course — Lecture 07

> **Source:** Gate Smashers — Manchester & Differential Manchester Encoding
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Topic Area:** Physical Layer — Line Coding / Digital-to-Digital Encoding

---

## 📑 Table of Contents

1. [Where This Fits — Physical Layer Encoding](#1-where-this-fits--physical-layer-encoding)
2. [Manchester Encoding](#2-manchester-encoding)
   - [2.1 The Two Conventions](#21-the-two-conventions)
   - [2.2 Encoding Rule (Dr. Thomas Convention)](#22-encoding-rule-dr-thomas-convention)
   - [2.3 Worked Example 1 — `1 0 1 0 0`](#23-worked-example-1--1-0-1-0-0)
   - [2.4 Worked Example 2 — `0 0 1 1 1`](#24-worked-example-2--0-0-1-1-1)
   - [2.5 IEEE 802.3 Convention](#25-ieee-8023-convention)
3. [Differential Manchester Encoding](#3-differential-manchester-encoding)
   - [3.1 Core Rule](#31-core-rule)
   - [3.2 Worked Example 1 — `1 0 1 1 0 0`](#32-worked-example-1--1-0-1-1-0-0)
   - [3.3 Worked Example 2 — `0 1 1 1 0`](#33-worked-example-2--0-1-1-1-0)
4. [Manchester vs Differential Manchester — Comparison](#4-manchester-vs-differential-manchester--comparison)
5. [Why This Matters for Cybersecurity](#5-why-this-matters-for-cybersecurity)
6. [Exam Cheat Sheet](#6-exam-cheat-sheet)

---

## 1. Where This Fits — Physical Layer Encoding

Data from the upper layers arrives at the **physical layer** as digital data (a stream of 1s and 0s). The physical layer must **encode** this data into a signal suitable for transmission over the medium. Four general categories exist:

| Encoding Type      | Meaning                                        |
| ------------------ | ---------------------------------------------- |
| Digital-to-Digital | Digital data → digital signal (our topic here) |
| Analog-to-Digital  | Analog data → digital signal                   |
| Digital-to-Analog  | Digital data → analog signal                   |
| Analog-to-Analog   | Analog data → analog signal                    |

**Manchester** and **Differential Manchester** encoding are both **digital-to-digital encoding schemes** — a very common exam topic because the rules are simple but easy to mix up under time pressure.

---

## 2. Manchester Encoding

### 2.1 The Two Conventions

There are **two different conventions** used to represent bits in Manchester encoding, and exam questions may specify which one to use:

| Convention                | Notes                                                                                                                           |
| ------------------------- | ------------------------------------------------------------------------------------------------------------------------------- |
| **Dr. Thomas Convention** | One common textbook convention.                                                                                                 |
| **IEEE 802.3 Convention** | The **default** convention if the exam question does not specify one. It is the **exact reverse** of the Dr. Thomas convention. |

> ⚠️ **Exam tip:** If the question does not mention which convention to use, assume **IEEE 802.3** by default.

---

### 2.2 Encoding Rule (Dr. Thomas Convention)

The core idea of Manchester encoding is that **every bit has a transition (edge) in the middle of the bit interval** — this transition itself represents the bit value.

| Bit | Representation (Dr. Thomas)                                                            |
| --- | -------------------------------------------------------------------------------------- |
| `1` | Signal transitions **high → low** (drawn as a "Z" shape going down through the middle) |
| `0` | Signal transitions **low → high** (the reverse "Z" shape)                              |

**Key memory trick:** Think of `1` as a **"Z"** shape and `0` as the **"reverse Z"** (mirror image) shape. Whichever level you end on for one bit, that becomes your starting level for connecting into the next bit's shape.

---

### 2.3 Worked Example 1 — `1 0 1 0 0`

```
Bit:      1        0        1        0        0
Signal:   ┐        ┌        ┐        ┌        ┌
          └─►      └►       └─►      └►       └►
        (high→low)(low→high)(high→low)(low→high)(low→high)
```

Step-by-step (Dr. Thomas convention):

1. **1** → draw the "Z" (transition from high level down to low level).
2. **0** → draw the "reverse Z" (transition from low level up to high level). Connect continuously from where bit 1 ended.
3. **1** → "Z" again — connect from the previous ending point.
4. **0** → "reverse Z" again.
5. **0** → "reverse Z" again, connected to the previous segment.

> **Golden rule:** Always **connect** each bit's shape to where the previous bit's shape ended — the waveform must be continuous, never left "floating."

---

### 2.4 Worked Example 2 — `0 0 1 1 1`

Step-by-step:

1. **0** → start with "reverse Z."
2. **0** → another "reverse Z," connected to where the first one ended.
3. **1** → "Z," connected onward.
4. **1** → "Z" again, connected.
5. **1** → "Z" again, connected.

This demonstrates that **repeating the same bit** still requires drawing the shape again for **each** bit (Manchester always transitions in the middle of _every_ bit, regardless of repetition) — connected end-to-end with the prior segment.

---

### 2.5 IEEE 802.3 Convention

The **IEEE 802.3** convention is simply the **mirror image** of Dr. Thomas:

| Bit | Representation (IEEE 802.3)                     |
| --- | ----------------------------------------------- |
| `1` | Signal transitions **low → high** ("reverse Z") |
| `0` | Signal transitions **high → low** ("Z")         |

**Memory trick for IEEE:** Think of it as "Z is zero" — i.e., the plain "Z" shape represents `0` (opposite of the Dr. Thomas mapping, where "Z" represents `1`).

> Since IEEE 802.3 is the **default/standard** used in real Ethernet networks and most modern exam questions (unless a question explicitly says "Dr. Thomas convention"), memorize this one first.

---

## 3. Differential Manchester Encoding

### 3.1 Core Rule

Differential Manchester works differently from standard Manchester — the transition **in the middle** of every bit interval still exists (for clocking purposes), but the **bit value** is determined by whether there is an **edge (transition) at the start** of the bit interval, not the mid-bit transition itself.

| Bit | Rule                                                                                                              |
| --- | ----------------------------------------------------------------------------------------------------------------- |
| `0` | **Edge at the start** of the bit interval — the signal level must change compared to the end of the previous bit. |
| `1` | **No edge at the start** — the signal **continues** at the same level the previous bit ended on.                  |

**Key points:**

- The waveform can **never be left empty/flat without a transition** at a bit boundary if the bit is `0` — you _must_ draw an edge.
- For `1`, you simply **continue** the signal level with no transition at the boundary (though there is still a mid-bit transition, consistent with all Manchester-family codes, for clock synchronization).
- At the **very first bit**, since there's no "previous" signal level, you may **start with either level** (high or low) — both are valid starting choices.

---

### 3.2 Worked Example 1 — `1 0 1 1 0 0`

Step-by-step:

1. **1** → No edge at start. Since this is the first bit, choose either starting level (say, start low).
2. **0** → Edge required at start. Since bit 1 ended on a particular level, you **must transition** to the opposite level to create the edge — pick the direction that is logically forced by where the signal currently sits.
3. **1** → No edge — continue at the same level bit 2 ended on.
4. **1** → No edge — continue further at the same level.
5. **0** → Edge required — transition to the opposite level from where bit 4 ended.
6. **0** → Edge required again — transition to the opposite level from where bit 5 ended.

> **Important:** For `0`, you don't get to freely choose which direction the edge goes — it is **determined by the current signal level** (you must move to the opposite level to create a valid edge). For `1`, there is no choice needed at all — just continue.

---

### 3.3 Worked Example 2 — `0 1 1 1 0`

Step-by-step:

1. **0** → First bit: draw an edge (choose a starting direction, e.g., transition downward).
2. **1** → No edge — continue at the same level.
3. **1** → No edge — continue.
4. **1** → No edge — continue.
5. **0** → Edge required — transition to the opposite level from where bit 4 ended, then hold flat for the rest of the interval.

This example highlights that a **run of consecutive 1s** produces a flat continuation with no boundary transitions (only the standard mid-bit transition), while every `0` forces a boundary-edge change.

---

## 4. Manchester vs Differential Manchester — Comparison

| Feature                                    | Manchester                                                                             | Differential Manchester                                                                                                                   |
| ------------------------------------------ | -------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| **Bit value determined by**                | Direction of the **mid-bit** transition                                                | **Presence/absence of an edge at the start** of the bit interval                                                                          |
| **Self-clocking**                          | Yes — mid-bit transition every bit                                                     | Yes — mid-bit transition every bit                                                                                                        |
| **Sensitive to signal polarity/direction** | Yes — depends on convention (Thomas vs IEEE)                                           | No — depends only on presence/absence of an edge, not absolute polarity                                                                   |
| **Noise/inversion resilience**             | Lower — inverting the whole signal changes bit interpretation under a fixed convention | Higher — differential coding is more resistant to signal polarity inversion, since it relies on relative transitions, not absolute levels |
| **First bit**                              | Determined strictly by convention rule                                                 | Starting level can be freely chosen                                                                                                       |
| **Used in**                                | Classic Ethernet (10 Mbps, IEEE 802.3)                                                 | Token Ring, FDDI                                                                                                                          |

---

## 5. Why This Matters for Cybersecurity

While this topic is primarily theoretical/exam-focused, understanding line coding at the physical layer has practical relevance for a cybersecurity student:

- **Protocol/traffic analysis fundamentals:** Line coding sits below the frames and packets you analyze with tools like Wireshark/tcpdump — understanding it builds intuition for how "1s and 0s" actually become an electrical or optical signal on the wire.
- **Physical-layer attacks & forensics:** Techniques like **TEMPEST** (electromagnetic eavesdropping) and cable-tapping attacks rely on understanding how bits are encoded as signal transitions — self-clocking codes like Manchester are relevant background for signal-level interception concepts (studied at a conceptual level, not implemented on real hardware without proper authorization and equipment).
- **Historical relevance to legacy networks:** Bus and Ring topologies (covered in Part 6) historically used Manchester-family encodings (classic Ethernet used Manchester; Token Ring/FDDI used Differential Manchester) — so this topic connects directly back to the topology material.

---

## 6. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║       MANCHESTER & DIFFERENTIAL MANCHESTER — EXAM CHEAT SHEET        ║
╠══════════════════════════════════════════════════════════════════════╣
║  MANCHESTER ENCODING                                                  ║
║  ─────────────────────────────────────────────────────────          ║
║  Dr. Thomas Convention:                                               ║
║      1 → high-to-low transition ("Z")                                ║
║      0 → low-to-high transition ("reverse Z")                        ║
║                                                                        ║
║  IEEE 802.3 Convention (DEFAULT if not specified):                    ║
║      1 → low-to-high transition ("reverse Z")                        ║
║      0 → high-to-low transition ("Z")                                ║
║                                                                        ║
║  Rule: IEEE 802.3 = exact REVERSE of Dr. Thomas convention           ║
║  Rule: Every waveform segment must CONNECT to the previous one       ║
╠══════════════════════════════════════════════════════════════════════╣
║  DIFFERENTIAL MANCHESTER ENCODING                                     ║
║  ─────────────────────────────────────────────────────          ║
║      0 → EDGE at the start of the bit interval (mandatory transition)║
║      1 → NO edge at start — continue previous signal level            ║
║                                                                        ║
║  Rule: First bit — starting level can be freely chosen               ║
║  Rule: For 0, direction of the edge is forced by current signal level║
║  Rule: More noise/inversion resistant than plain Manchester           ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK DEFAULT TO REMEMBER                                             ║
║  ─────────────────────────────────────────────────────          ║
║  If exam doesn't mention a convention → use IEEE 802.3                ║
║  "Z is zero" → memory aid for IEEE 802.3 convention                   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Other line coding schemes (NRZ, NRZ-I, RZ, Bipolar/AMI)
- [ ] Analog-to-Digital encoding (PCM, Delta Modulation)
- [ ] Digital-to-Analog encoding (ASK, FSK, PSK, QAM)

---

_Notes compiled from: Networking Course Lecture 07 — Manchester & Differential Manchester Encoding_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
