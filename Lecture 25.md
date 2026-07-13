# ⚡ Flow Control Protocols — Full Comparison & Efficiency Formulas

### Cybersecurity Student Notes | Networking Course — Lecture 25 (Last-Minute Revision)

> **Source:** Gate Smashers — Flow Control Protocols: Stop & Wait vs Go-Back-N vs Selective Repeat (Quick Revision)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Why This Video Matters](#1-why-this-video-matters)
2. [The Efficiency Formula — Master Concept](#2-the-efficiency-formula--master-concept)
3. [Stop & Wait — Quick Revision](#3-stop--wait--quick-revision)
4. [Go-Back-N — Quick Revision](#4-go-back-n--quick-revision)
5. [Selective Repeat — Quick Revision](#5-selective-repeat--quick-revision)
6. [Understanding "k" — Bits vs Window Size (Common Trap)](#6-understanding-k--bits-vs-window-size-common-trap)
7. [Master Comparison Table](#7-master-comparison-table)
8. [Worked Numerical Example (All 3 Protocols)](#8-worked-numerical-example-all-3-protocols)
9. [🔴 Security Considerations & Modern Relevance](#9--security-considerations--modern-relevance)
10. [🧪 Practical Labs — Your Setup](#10--practical-labs--your-setup)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. Why This Video Matters

This lecture is a **pure revision/formula-recap** of the three flow control protocols covered in Parts 22–24 (Stop & Wait, Go-Back-N, Selective Repeat). Both **numerical** and **theoretical** questions on these three protocols are described as **frequently asked** in competitive exams — this is meant to be read right before an exam as a final refresher.

---

## 2. The Efficiency Formula — Master Concept

> **This single formula underlies ALL efficiency calculations for all three protocols.**

```
Efficiency (η) = Useful Time / Total Time
              = Transmission Time (TT) / (Transmission Time + Round Trip Time)
              = TT / (TT + RTT)

Where:
  RTT (Round Trip Time) = 2 × Propagation Delay (Tp)

Dividing through by TT (taking TT common), this simplifies to the
standard exam formula:

  Efficiency (η) = 1 / (1 + 2x)

  where   x = Tp / TT   (Propagation Delay ÷ Transmission Time)
```

### Intuition

- **"Useful time"** = the time actually spent transmitting real data.
- **"Total time"** = useful time **plus** all the time spent waiting (propagation there, propagation back).
- The **less time we spend "waiting" relative to "sending,"** the higher the efficiency.

> This base formula `1/(1+2x)` is literally the **Stop & Wait** efficiency — because Stop & Wait sends exactly **one** frame in that "total time" window. Go-Back-N and Selective Repeat both **scale this same formula up** by their respective window sizes (see below).

---

## 3. Stop & Wait — Quick Revision

| Property                       | Value                                                                         |
| ------------------------------ | ----------------------------------------------------------------------------- |
| **Core behavior**              | Send 1 frame → wait for ACK → only then send next frame                       |
| **Sender window size**         | **1**                                                                         |
| **Receiver window size**       | **1**                                                                         |
| **Available sequence numbers** | Sender window + Receiver window = 1 + 1 = **2**                               |
| **Efficiency**                 | `1 / (1 + 2x)` where x = Tp / TT                                              |
| **Retransmission on loss**     | **1 packet only** (the single frame that was sent)                            |
| **Out-of-order acceptance**    | N/A (only ever 1 frame outstanding)                                           |
| **Implementation complexity**  | **Simplest** — no searching/sorting logic needed                              |
| **Efficiency ranking**         | **WORST** of the three (by default, if asked "least efficient" → Stop & Wait) |

> **Why is efficiency poor?** Because only **1 frame** is transmitted within the entire round-trip time window — a huge amount of "total time" is spent simply waiting, not sending.

---

## 4. Go-Back-N — Quick Revision

| Property                      | Value                                                                                                                    |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **Core behavior**             | Sliding window — sends an **entire window** of multiple frames at once                                                   |
| **Sender window size**        | `2^k − 1` (where k = number of bits used to represent the window)                                                        |
| **Receiver window size**      | **1**                                                                                                                    |
| **Out-of-order acceptance**   | **NO** — receiver window is 1, so it strictly accepts only the exact expected frame, in order                            |
| **Efficiency**                | `(2^k − 1) × [1 / (1 + 2x)]` — i.e., Stop & Wait's efficiency, **scaled up by the window size**                          |
| **Acknowledgment type**       | **Cumulative** — ACK always confirms the _next expected_ frame, implicitly confirming everything before it               |
| **Retransmission on loss**    | Potentially the **ENTIRE window** (up to `2^k − 1` frames) — even frames that arrived correctly get discarded and resent |
| **Retransmission ranking**    | **HIGHEST** of the three protocols                                                                                       |
| **Implementation complexity** | Moderate                                                                                                                 |

### Why Retransmission Is So High

```
Sent: [1][2][3][4]   (all within the window)

Frame 1 is lost. Receiver's window = 1, so it is strictly waiting for frame 1.

  Frame 2 arrives → REJECTED (receiver only wants 1)
  Frame 3 arrives → REJECTED
  Frame 4 arrives → REJECTED

Even though 2, 3, and 4 arrived successfully, they are all discarded
because the receiver's single-frame window insists on strict order.

→ Sender must resend the ENTIRE window again: 1, 2, 3, AND 4.
```

### Why Cumulative ACK Reduces Network Traffic

> Analogy from the lecture: sending **100 packets** from Chandigarh to Delhi.
>
> - **Bad approach:** send 1 packet → wait for ACK → send next packet → wait for ACK ... (100 round trips of overhead)
> - **Better approach:** send all 100 packets, then get **1 cumulative acknowledgment** confirming all of them at once.
> - This significantly **reduces overall network traffic/overhead**.

---

## 5. Selective Repeat — Quick Revision

| Property                      | Value                                                                                                                                               |
| ----------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Core behavior**             | Sliding window — sends multiple frames, but receiver ALSO has a large window                                                                        |
| **Sender window size**        | `2^(k−1)` ⚠️ note: exponent placement differs from Go-Back-N                                                                                        |
| **Receiver window size**      | `2^(k−1)` — **equal to sender window** (this is the key structural difference from Go-Back-N)                                                       |
| **Out-of-order acceptance**   | **YES** — receiver has "empty space" for multiple frames, so it can accept and buffer frames that arrive out of order                               |
| **Efficiency**                | `2^(k−1) × [1 / (1 + 2x)]` — scaled by its (smaller, but still multi-frame) window size                                                             |
| **Acknowledgment type**       | Uses **BOTH** cumulative acknowledgment **and** independent (individual) acknowledgment, **plus** Negative Acknowledgment (NAK)                     |
| **Retransmission on loss**    | **Only the single lost packet** — equal to Stop & Wait's retransmission count (i.e., minimal/lowest)                                                |
| **Retransmission ranking**    | **LOWEST** of the three (tied with Stop & Wait, both = 1 packet retransmitted)                                                                      |
| **Implementation complexity** | **HIGHEST/most difficult** — requires **searching** (to locate the specific lost packet) and **sorting** (to reorder buffered out-of-order packets) |

### Why Out-of-Order Acceptance Reduces Retransmission

```
Sent: [1][2][3][4]

Frame 1 is lost. But receiver's window has EMPTY SPACE for 2, 3, 4.

  Frame 2 arrives → ACCEPTED (buffered, waiting for the gap to fill)
  Frame 3 arrives → ACCEPTED (buffered)
  Frame 4 arrives → ACCEPTED (buffered)

Only frame 1 is actually missing → only frame 1 needs to be resent.
```

### Negative Acknowledgment (NAK) — Time-Saving Benefit

```
WITHOUT NAK:
  Frame arrives corrupted → receiver stays silent → sender must wait
  the FULL timeout timer duration before assuming loss and resending

WITH NAK:
  Frame arrives corrupted → receiver immediately sends NAK,
  proactively telling the sender: "there's a problem with this
  packet, resend it now"
  → Sender does NOT have to wait for the full timeout timer to expire
  → Time is saved
```

---

## 6. Understanding "k" — Bits vs Window Size (Common Trap)

> ⚠️ **This is explicitly flagged as a frequent point of confusion in exams.**

```
If a question says "window size is represented by k bits"...

  This does NOT mean the window size equals k directly!

  k = number of BITS used to represent the window/sequence numbers
  N (total available sequence numbers) = 2^k
```

### Example: k = 3

```
3 bits can represent: 000, 001, 010, 011, 100, 101, 110, 111
                     = values 0 through 7
                     = 8 total distinct values

  N (total sequence numbers) = 2^3 = 8

  Go-Back-N sender window size    = 2^k − 1 = 2^3 − 1 = 7
  Selective Repeat window size    = 2^(k−1) = 2^(3−1) = 2^2 = 4
```

> **Shortcut:** If you're given the window size **N** directly (not the bit count), you can also just write Go-Back-N's sender window as **N − 1** — it's the same value either way. The formula `2^k − 1` is just the "long way" of expressing it when you're given bits instead of the raw window size.

---

## 7. Master Comparison Table

| Parameter                          | Stop & Wait                  | Go-Back-N                       | Selective Repeat                                |
| ---------------------------------- | ---------------------------- | ------------------------------- | ----------------------------------------------- |
| **Sender window size**             | 1                            | `2^k − 1`                       | `2^(k−1)`                                       |
| **Receiver window size**           | 1                            | 1                               | `2^(k−1)` (equal to sender)                     |
| **Available sequence numbers**     | 2 (sender + receiver window) | `2^k`                           | `2^k`                                           |
| **Efficiency formula**             | `1/(1+2x)`                   | `(2^k − 1) × 1/(1+2x)`          | `2^(k−1) × 1/(1+2x)`                            |
| **Efficiency ranking**             | Worst                        | Better                          | Best (or comparable to Go-Back-N, depends on k) |
| **Out-of-order packets accepted?** | N/A                          | **No**                          | **Yes**                                         |
| **Acknowledgment type**            | Simple ACK (next expected)   | Cumulative                      | Cumulative **+** Independent **+** NAK          |
| **Retransmission on error**        | 1 packet                     | Up to entire window (`2^k − 1`) | 1 packet only (lowest, tied w/ Stop&Wait)       |
| **Retransmission ranking**         | Low                          | **Highest**                     | **Lowest**                                      |
| **Implementation complexity**      | **Simplest**                 | Moderate                        | **Most difficult** (needs searching + sorting)  |

---

## 8. Worked Numerical Example (All 3 Protocols)

**Given:** k = 3 bits, Transmission Time (TT) = 1 ms, Propagation Delay (Tp) = 2 ms.

```
Step 1 — Calculate x:
  x = Tp / TT = 2 / 1 = 2

Step 2 — Base efficiency (Stop & Wait formula):
  η(Stop&Wait) = 1 / (1 + 2x) = 1 / (1 + 2×2) = 1 / 5 = 0.20  (20%)

Step 3 — Total sequence numbers:
  N = 2^k = 2^3 = 8

Step 4 — Go-Back-N:
  Sender window = 2^k − 1 = 8 − 1 = 7
  η(Go-Back-N) = 7 × 0.20 = 1.40  → capped/interpreted as up to 100%
  (if the computed value exceeds 1, efficiency is effectively 100%,
   meaning the link's bandwidth-delay product is fully utilized)

Step 5 — Selective Repeat:
  Sender/Receiver window = 2^(k−1) = 2^(3−1) = 2^2 = 4
  η(Selective Repeat) = 4 × 0.20 = 0.80  (80%)
```

> **Note:** If a computed efficiency value exceeds `1` (100%), it means the window is **large enough that the link is never idle** — the sender always has more data ready to send within one RTT, so the link's capacity becomes the limiting factor rather than waiting time. In such cases, the practical efficiency is capped at 100%.

---

## 9. 🔴 Security Considerations & Modern Relevance

This lecture is a pure efficiency/formula recap, but the comparison itself highlights a security principle worth internalizing:

| Comparison Point                                 | Security Relevance                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Efficiency vs Complexity trade-off**           | Selective Repeat is the most efficient but also the **most complex to implement** (searching + sorting logic) — this mirrors a general security principle: the most performant protocol design is often also the one with the **largest attack surface**, because more logic means more edge cases (as seen concretely in Part 24's CVE-2019-11477 "SACK Panic" example) |
| **Go-Back-N's high retransmission cost**         | A protocol that discards valid data just to preserve simplicity (Go-Back-N) is more vulnerable to **amplification-style waste** if an attacker can selectively interfere with just one frame per window — forcing far more retransmission than the actual amount of data lost                                                                                            |
| **Window size directly controls "blast radius"** | In all three protocols, window size determines how much data is "at risk" per round — a larger window (Go-Back-N, Selective Repeat) means a single well-timed interference event (dropped frame, corrupted frame) affects proportionally more in-flight data than in Stop & Wait                                                                                         |
| **Sequence number exhaustion (2^k space)**       | Every protocol here depends on a **finite modular sequence number space** — as seen in Parts 23–24, getting the window-size-vs-sequence-space math wrong (window = 2^k instead of 2^k − 1, etc.) creates real ambiguity/duplicate-acceptance bugs. This is the same general bug class behind real historical **TCP sequence number attacks**                             |

> **Key takeaway for a security student:** When evaluating any custom or legacy protocol during an assessment, this comparison table is a useful mental checklist — ask "what's the window size, how is loss handled, and does out-of-order data get buffered or discarded?" These three questions alone tell you a lot about both a protocol's **performance characteristics** and its **potential resource-exhaustion / amplification weaknesses**.

---

## 10. 🧪 Practical Labs — Your Setup

### Lab 1 — Measure Real TT and Tp on Your Lab Link

```bash
# Approximate propagation delay via ping RTT (RTT ≈ 2 × Tp for a
# roughly symmetric link with negligible transmission time on a LAN)
ping -c 10 192.168.56.101
# Take the average RTT, divide by 2 to estimate Tp for your own "x" calculation
```

### Lab 2 — Compare Throughput Under Different Simulated Window Behaviors

```bash
# Simulate a "Stop & Wait"-like constrained link (very high delay per
# exchange, e.g., forcing single small transfers with waits)
sudo tc qdisc add dev eth0 root netem delay 100ms
time curl http://192.168.56.101/ -o /dev/null --max-time 20
sudo tc qdisc del dev eth0 root netem

# Now compare against normal TCP behavior on the same link (TCP uses
# a much larger sliding window, closer to Go-Back-N/Selective Repeat
# in spirit)
time curl http://192.168.56.101/ -o /dev/null --max-time 20
```

### Lab 3 — Calculate Efficiency From Your Own Measurements

```bash
# Use your measured RTT (Lab 1) and a known transfer size to
# manually compute an approximate real-world "x" and efficiency,
# applying the exact formulas from Section 2 of this document
# TT ≈ (file size in bits) / (link bandwidth in bits/sec)
# x = Tp / TT
# η = 1 / (1 + 2x)
```

### Lab 4 — Observe TCP Window Scaling (Real-World Analogue)

```bash
# Modern TCP's window is dynamically much larger than any of these
# fixed academic examples — inspect it directly
cat /proc/sys/net/ipv4/tcp_window_scaling
sudo tcpdump -i eth0 host 192.168.56.101 and tcp -v | grep -i "win "
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║        FLOW CONTROL PROTOCOLS — MASTER EXAM CHEAT SHEET              ║
╠══════════════════════════════════════════════════════════════════════╣
║  UNIVERSAL EFFICIENCY FORMULA                                          ║
║  ─────────────────────────────────────────────────────────           ║
║  η = TT / (TT + RTT) = 1 / (1 + 2x)                                   ║
║  x = Tp / TT   (Propagation Delay ÷ Transmission Time)                ║
║  RTT = 2 × Tp                                                          ║
╠══════════════════════════════════════════════════════════════════════╣
║  WINDOW SIZE FORMULAS                                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  Stop & Wait:       sender=1        receiver=1                        ║
║  Go-Back-N:         sender=2^k−1    receiver=1                        ║
║  Selective Repeat:  sender=2^(k−1)  receiver=2^(k−1)  (equal!)        ║
║                                                                        ║
║  k = number of BITS used to represent sequence number                 ║
║  Total sequence numbers available = 2^k                               ║
╠══════════════════════════════════════════════════════════════════════╣
║  EFFICIENCY FORMULAS (scale base formula by window size)              ║
║  ─────────────────────────────────────────────────────────           ║
║  Stop & Wait:       η = 1/(1+2x)                                      ║
║  Go-Back-N:         η = (2^k − 1) × 1/(1+2x)                          ║
║  Selective Repeat:  η = 2^(k−1) × 1/(1+2x)                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  RANKINGS (memorize these directly)                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  Efficiency:        Stop&Wait < Go-Back-N ≈/< Selective Repeat        ║
║                      (Stop & Wait = WORST by default)                 ║
║  Retransmission:    Go-Back-N (HIGHEST) > Stop&Wait = Selective Repeat (LOWEST, both =1 packet) ║
║  Implementation:    Stop&Wait (simplest) < Go-Back-N < Selective Repeat (hardest — search+sort) ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "Least efficient protocol"           → Stop & Wait                  ║
║  "Highest retransmission"             → Go-Back-N                    ║
║  "Lowest retransmission"              → Selective Repeat (=Stop&Wait)║
║  "Accepts out-of-order packets"       → Selective Repeat ONLY        ║
║  "Uses cumulative ACK"                → Go-Back-N (and Selective Repeat too) ║
║  "Uses NAK"                            → Selective Repeat            ║
║  "Hardest to implement (search+sort)" → Selective Repeat             ║
║  "Sequence numbers = sender+receiver window" → Stop & Wait (2 total) ║
║  "Sequence numbers = 2^k"             → Go-Back-N / Selective Repeat  ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 26)

- [ ] CRC (Cyclic Redundancy Check) — step-by-step calculation method
- [ ] Checksum — calculation and verification process
- [ ] CSMA/CD — detailed working and Ethernet collision handling
- [ ] Hamming Code — error detection and correction

---

_Notes compiled from: Networking Course Lecture 25 — Flow Control Protocols (Last-Minute Revision)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
