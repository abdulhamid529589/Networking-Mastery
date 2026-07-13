# ⏱️ Stop-and-Wait Protocol — Flow & Error Control

### Cybersecurity Student Notes | Networking Course — Lecture 22

> **Source:** Gate Smashers — Stop and Wait Protocol
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Core Concept](#1-core-concept)
2. [What is an Acknowledgment (ACK)?](#2-what-is-an-acknowledgment-ack)
3. [Why Do We Need Sequence Numbers?](#3-why-do-we-need-sequence-numbers)
4. [Timeout Timer](#4-timeout-timer)
5. [Window Size — The Most Important Point](#5-window-size--the-most-important-point)
6. [Normal Operation — Diagram](#6-normal-operation--diagram)
7. [Case 1 — Frame Lost in Transit](#7-case-1--frame-lost-in-transit)
8. [Case 2 — Acknowledgment Lost in Transit](#8-case-2--acknowledgment-lost-in-transit)
9. [Golden Rule: ACK is Always for the NEXT Expected Frame](#9-golden-rule-ack-is-always-for-the-next-expected-frame)
10. [Summary Table — All Scenarios](#10-summary-table--all-scenarios)
11. [🔴 Security Considerations & Modern Relevance](#11--security-considerations--modern-relevance)
12. [🧪 Practical Labs — Your Setup](#12--practical-labs--your-setup)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. Core Concept

**Stop-and-Wait** is one of the flow/error control protocols used at the **Data Link Layer** (introduced in Part 21).

> **The rule is simple: Send ONE frame. Then STOP. WAIT peacefully until you get the acknowledgment for it. Only then send the next frame.**

```
[Sender] ──frame──► [Receiver]
[Sender]  ⏸ STOP & WAIT ⏸
[Sender] ◄──ACK────  [Receiver]
[Sender] ──next frame──► [Receiver]
    ... repeat ...
```

- The sender does **not** send the next frame until it has received confirmation (ACK) that the **current** frame arrived successfully.
- This is the simplest possible flow-control scheme — one frame "in flight" at a time.

---

## 2. What is an Acknowledgment (ACK)?

> **Acknowledgment = a signal from the receiver confirming that a frame was successfully received.**

Real-world analogy: an **online transaction** — at the end, you get a message saying whether the transaction was **successful or failed**. That confirmation is what lets you know the money/data actually arrived.

```
Frame arrives successfully  →  Receiver SENDS an ACK
Frame does NOT arrive       →  Receiver has nothing to acknowledge
                                (no ACK is sent — receiver doesn't
                                even know a frame was meant for it)
```

- ACK is only sent when data has **actually and successfully** reached the receiver.
- If the frame never arrives, there's nothing for the receiver to acknowledge — silence is the natural result of a lost frame.

---

## 3. Why Do We Need Sequence Numbers?

### The Problem Without Sequence Numbers

```
[Sender] ──frame──► [Receiver]  ✓ received
[Sender] ◄──ACK────  [Receiver]  (ACK sent...)
[Sender]  ✗          ← ACK is LOST in transit, never arrives!

Sender doesn't know the frame was actually received.
Sender only knows: "I haven't gotten an ACK yet."
```

Without any way to number/identify frames, if an ACK gets lost, the sender has **no way to tell** whether:

- (a) the original frame never arrived at all, or
- (b) the frame arrived fine, but the ACK confirming it got lost on the way back.

Either way, the sender will **resend** the frame after waiting. But when the receiver gets this resent frame, **how does it know if this is a brand-new frame or a duplicate of one it already accepted?**

> **This is exactly why sequence numbers exist** — to let the receiver distinguish a genuinely new frame from a duplicate retransmission caused by a lost ACK.

---

## 4. Timeout Timer

If the sender doesn't receive an ACK, it doesn't wait forever — it waits for a specific duration called the **timeout timer**, after which it assumes the frame (or the ACK) was lost and retransmits.

```
Timeout Timer ≈ 2 × Round-Trip Time (RTT)

Round-Trip Time (RTT) = time for data to go to receiver
                          + time for ACK to come back

Timeout = double the RTT, to allow extra margin for
          unexpected delay in the path
```

- **Round-Trip Time (RTT):** the time taken for a frame to reach the receiver **and** for the corresponding ACK to return to the sender.
- **Timeout Timer:** deliberately set to **~2× RTT**, building in a safety margin for unexpected network delay.
- If the timeout timer expires and **still no ACK has arrived**, the sender **resends the same frame**.

---

## 5. Window Size — The Most Important Point

> ⚠️ **This is explicitly flagged as the highest-yield exam point in this topic.**

```
Sender's Window Size   = 1   (can only have ONE unacknowledged frame outstanding)
Receiver's Window Size = 1   (can only accept ONE frame before needing to ACK it)
```

- **Sender window size** = how many frames the sender can transmit before needing an ACK → here, exactly **1**.
- **Receiver window size** = how many frames the receiver can accept before it must acknowledge → here, exactly **1**.
- Neither side is allowed to exceed this — the sender cannot send more than one frame at a time, and the receiver cannot accept more than one at a time.

### Deriving the Sequence Numbers Needed

```
Total movement = Sender window size + Receiver window size
                = 1 + 1
                = 2

→ We need exactly 2 sequence numbers: 0 and 1
```

This is why **Stop-and-Wait uses only sequence numbers 0 and 1** (alternating) — no more are needed, because at most one frame is ever "in flight" unacknowledged at any given time.

---

## 6. Normal Operation — Diagram

```
Sender                                          Receiver
  │                                                 │
  │──────────── Frame 0 ─────────────────────────► │  (accepted)
  │                                                 │  expecting frame 1 next
  │ ◄─────────── ACK 1 ──────────────────────────── │  ("I'm now expecting 1")
  │                                                 │
  │──────────── Frame 1 ─────────────────────────► │  (accepted)
  │                                                 │  expecting frame 0 next
  │ ◄─────────── ACK 0 ──────────────────────────── │  ("I'm now expecting 0")
  │                                                 │
  │──────────── Frame 0 ─────────────────────────► │  ... cycle repeats ...
```

- Frame numbers **alternate**: 0 → 1 → 0 → 1 → ...
- After accepting frame `0`, the receiver's ACK says **"1"** — because it is now expecting frame `1` next (**not** re-confirming the `0` it just got).

---

## 7. Case 1 — Frame Lost in Transit

```
Sender                                          Receiver
  │                                                 │
  │──────────── Frame 1 ────────╳ (LOST)            │  (never arrives)
  │                                                 │  (nothing received → no ACK sent)
  │  ⏳ waiting... timeout timer running             │
  │  ⏰ TIMEOUT EXPIRES — no ACK received             │
  │                                                 │
  │──────────── Frame 1 (RESENT) ─────────────────► │  (accepted this time)
  │                                                 │  expecting frame 0 next
  │ ◄─────────── ACK 0 ──────────────────────────── │
```

- Frame `1` is lost → receiver gets **nothing** → sends **no ACK**.
- Sender's timeout timer expires with no ACK received → sender **resends frame 1**.
- This time it arrives → receiver accepts it → sends ACK `0` (expecting `0` next) → normal flow resumes.

---

## 8. Case 2 — Acknowledgment Lost in Transit

This is the trickier case — the frame **did** arrive, but the **ACK confirming it gets lost** on the way back.

```
Sender                                          Receiver
  │                                                 │
  │──────────── Frame 0 ─────────────────────────► │  (accepted successfully)
  │                                                 │  expecting frame 1 next
  │ ◄─────────── ACK 1 ────────────╳ (LOST)          │  (ACK sent, but never arrives)
  │                                                 │
  │  ⏳ waiting... timeout timer running             │
  │  ⏰ TIMEOUT EXPIRES — sender assumes frame lost   │
  │                                                 │
  │──────────── Frame 0 (RESENT — duplicate!) ────► │  Receiver is expecting
  │                                                 │  frame "1", but gets "0" again
  │                                                 │  → Receiver DISCARDS this
  │                                                 │    duplicate frame
  │                                                 │  → Receiver correctly infers:
  │                                                 │    "my earlier ACK must have
  │                                                 │     gotten lost" — so it
  │                                                 │    RESENDS the ACK
  │ ◄─────────── ACK 1 (resent) ──────────────────── │
  │                                                 │
  │  Now sender gets ACK 1 → proceeds normally      │
  │──────────── Frame 1 ─────────────────────────► │
```

### Why This Works Correctly

- The receiver already accepted frame `0` and is now expecting frame `1`.
- When frame `0` arrives **again** (a duplicate), the receiver recognizes — thanks to the **sequence number** — that this is not the new frame it's expecting.
- It **discards the duplicate data** (to avoid processing the same frame twice) but **resends the ACK**, because the most likely explanation is that its previous ACK got lost, not that the sender is confused.
- This is precisely why **sequence numbers are essential**: without them, the receiver would have no way to tell this resent frame `0` apart from a legitimate new frame.

---

## 9. Golden Rule: ACK is Always for the NEXT Expected Frame

> **We never acknowledge the frame we just received by its own number. We acknowledge by naming the frame we expect NEXT.**

```
Received Frame 0  →  Send ACK "1"   (i.e., "I now expect frame 1")
Received Frame 1  →  Send ACK "0"   (i.e., "I now expect frame 0")
```

This convention is what allows the sender to know unambiguously: _"my last frame was accepted, AND here's specifically what to send next."_

---

## 10. Summary Table — All Scenarios

| Scenario                      | What Sender Sees                 | What Sender Does                     | What Receiver Does                                                        |
| ----------------------------- | -------------------------------- | ------------------------------------ | ------------------------------------------------------------------------- |
| **Normal operation**          | ACK arrives before timeout       | Sends next frame in sequence         | Accepts frame, sends ACK for next expected frame                          |
| **Frame lost**                | No ACK arrives → timeout expires | Resends the same frame               | Never received anything → sends nothing                                   |
| **ACK lost (frame was fine)** | No ACK arrives → timeout expires | Resends the same frame (a duplicate) | Recognizes duplicate via sequence number → discards data, resends the ACK |

---

## 11. 🔴 Security Considerations & Modern Relevance

Stop-and-Wait is a simplified teaching model, but the exact concepts it introduces — **sequence numbers, acknowledgments, and timeout-based retransmission** — are the direct conceptual ancestors of mechanisms used in **TCP** (which uses sliding-window flow control, a generalization of this idea). Understanding Stop-and-Wait gives real insight into several well-known network security topics:

| Stop-and-Wait Concept                  | Real-World Security Relevance                                                                                                                                                                                                                                                                                                                                                                                                            |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Sequence numbers**                   | TCP sequence numbers exist for exactly this reason (detecting duplicates/reordering) — but they are also the target of classic **TCP sequence number prediction / session hijacking** attacks, where an attacker who can guess or observe sequence numbers can inject or hijack a stream                                                                                                                                                 |
| **Acknowledgment (ACK)**               | **ACK spoofing** — an off-path or on-path attacker forging ACKs is a building block of some TCP hijacking and injection techniques; this is why modern TCP stacks use randomized initial sequence numbers as a defense                                                                                                                                                                                                                   |
| **Timeout / retransmission logic**     | Deliberately **dropping or delaying ACKs** (e.g., via a man-in-the-middle or a jamming attacker on wireless) forces repeated retransmission — a resource-exhaustion / availability concern, especially on constrained or high-latency links (satellite, IoT)                                                                                                                                                                             |
| **Discarding duplicate frames**        | This duplicate-detection logic is the same underlying principle used to defend against **replay attacks** in security protocols generally — an attacker resending a captured, valid-looking message should be recognized and rejected rather than reprocessed                                                                                                                                                                            |
| **Low throughput (1 frame at a time)** | Because Stop-and-Wait only allows one unacknowledged frame, its throughput is very poor on high-latency links (bandwidth-delay product problem) — this is a real reason **actual networks use sliding-window protocols (Go-Back-N / Selective Repeat / TCP windows)** instead of pure Stop-and-Wait, both for performance AND because a single-frame window is trivially easy to stall entirely with just one dropped ACK per round trip |

> **Key takeaway for a security student:** Any protocol relying on sequence numbers and acknowledgments for reliability is implicitly relying on those values being **trustworthy**. Stop-and-Wait (and TCP after it) assumes a cooperative or at-worst lossy network — not a maliciously adversarial one. That gap between "designed for reliability" and "designed for security" is exactly why TCP needed additional protections (randomized ISNs, TCP timestamps, and ultimately TLS) layered on top of it over time.

---

## 12. 🧪 Practical Labs — Your Setup

### Lab 1 — Observe Real Sequence Numbers & ACKs (TCP, the modern descendant of this idea)

```bash
# Capture a TCP handshake + data exchange to Metasploitable2 and inspect
# sequence numbers and ACK numbers directly — same underlying concept,
# generalized into a sliding window instead of Stop-and-Wait's window of 1

sudo tcpdump -i eth0 host 192.168.56.101 and tcp -S -v
# The -S flag shows ABSOLUTE sequence numbers instead of relative ones
```

### Lab 2 — Watch Retransmissions Happen (Simulate Loss)

```bash
# Introduce artificial packet loss on your own lab interface to observe
# timeout-driven retransmission behavior firsthand (own lab only)

sudo tc qdisc add dev eth0 root netem loss 20%
# Now generate some traffic to Metasploitable2 and capture it
curl http://192.168.56.101/ --max-time 10 -v
sudo tcpdump -i eth0 host 192.168.56.101 -v

# Remove the induced loss when done
sudo tc qdisc del dev eth0 root netem
```

### Lab 3 — Measure Round-Trip Time (RTT) Directly

```bash
# RTT is the exact quantity Stop-and-Wait's timeout timer is based on
ping -c 10 192.168.56.101
# Note the min/avg/max/mdev RTT values in the summary line —
# a real timeout timer would be set to roughly 2x this average
```

### Lab 4 — Conceptually Observe "ACK Spoofing" Risk (Read-Only Demonstration)

```bash
# On YOUR OWN isolated lab traffic only: capture a live TCP stream and
# examine how easily sequence/ack numbers could be read by anyone with
# visibility into the traffic (this is exactly why encryption at higher
# layers, e.g. TLS, protects data even if L3/L4 metadata is visible)

sudo tcpdump -i eth0 host 192.168.56.101 and tcp -A -v | head -50
# Observe: sequence numbers and ACK numbers are sent in CLEAR TEXT
# at the TCP header level — they are not secret or cryptographically
# protected, which is why relying on them alone is not "security"
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║             STOP-AND-WAIT PROTOCOL — EXAM CHEAT SHEET                ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE RULE                                                             ║
║  ─────────────────────────────────────────────────────────           ║
║  Send ONE frame → STOP → WAIT for ACK → only then send next frame    ║
╠══════════════════════════════════════════════════════════════════════╣
║  WINDOW SIZE (★ MOST IMPORTANT EXAM POINT ★)                          ║
║  ─────────────────────────────────────────────────────────           ║
║  Sender window size    = 1                                            ║
║  Receiver window size  = 1                                            ║
║  Total sequence numbers needed = sender window + receiver window = 2 ║
║  → Sequence numbers used: 0 and 1 (alternating)                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  TIMEOUT TIMER                                                         ║
║  ─────────────────────────────────────────────────────────           ║
║  Timeout ≈ 2 × RTT (Round-Trip Time)                                 ║
║  RTT = time to send frame + time to receive its ACK                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  GOLDEN RULE — ACK NUMBERING                                          ║
║  ─────────────────────────────────────────────────────────           ║
║  ACK always names the NEXT EXPECTED frame — never the frame          ║
║  just received.                                                       ║
║    Received frame 0 → send ACK "1"                                   ║
║    Received frame 1 → send ACK "0"                                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY SCENARIOS                                                         ║
║  ─────────────────────────────────────────────────────────           ║
║  Frame lost      → no ACK sent → sender times out → resends frame    ║
║  ACK lost        → sender times out → resends SAME frame (duplicate) ║
║                     → receiver detects duplicate via sequence number ║
║                     → discards duplicate DATA, but RESENDS the ACK   ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY SEQUENCE NUMBERS EXIST                                            ║
║  ─────────────────────────────────────────────────────────           ║
║  To let the receiver distinguish a genuinely NEW frame from a        ║
║  DUPLICATE retransmission caused by a lost ACK.                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 23)

- [ ] Go-Back-N ARQ — sliding window, cumulative ACKs, retransmission from error point onward
- [ ] Selective Repeat ARQ — individual frame retransmission, receiver buffering
- [ ] Stop-and-Wait vs Go-Back-N vs Selective Repeat — full comparison table
- [ ] Efficiency/throughput calculations for each protocol

---

_Notes compiled from: Networking Course Lecture 22 — Stop-and-Wait Protocol_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
