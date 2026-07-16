# 🪟 Go-Back-N ARQ — Sliding Window Protocol

### Cybersecurity Student Notes | Networking Course — Lecture 23

> **Source:** Gate Smashers — Go-Back-N ARQ
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Go-Back-N vs Stop-and-Wait — Why "Sliding Window"?](#1-go-back-n-vs-stop-and-wait--why-sliding-window)
2. [Window Size Formulas](#2-window-size-formulas)
3. [Understanding "m" — Sequence Number Bits](#3-understanding-m--sequence-number-bits)
4. [Worked Example — m = 2](#4-worked-example--m--2)
5. [How the Sliding Window Moves](#5-how-the-sliding-window-moves)
6. [Why Window Size Must Be STRICTLY LESS THAN 2^m](#6-why-window-size-must-be-strictly-less-than-2m)
   - [6.1 Correct Case — Window Size = 2^m − 1](#61-correct-case--window-size--2m--1)
   - [6.2 Broken Case — Window Size = 2^m (Equal)](#62-broken-case--window-size--2m-equal)
7. [Cumulative Acknowledgment](#7-cumulative-acknowledgment)
8. [Out-of-Order Packets Are REJECTED](#8-out-of-order-packets-are-rejected)
9. [Summary Table](#9-summary-table)
10. [🔴 Security Considerations & Modern Relevance](#10--security-considerations--modern-relevance)
11. [🧪 Practical Labs — Your Setup](#11--practical-labs--your-setup)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. Go-Back-N vs Stop-and-Wait — Why "Sliding Window"?

> **Go-Back-N is called a Sliding Window Protocol.**

- **Stop-and-Wait (Part 22)** was **NOT** a sliding window protocol — it only had a window of size **1**. Send one frame, wait, repeat.
- **Go-Back-N** allows a **bigger window** — the sender can transmit **multiple frames before needing an acknowledgment**, and this window **slides forward** as frames get acknowledged.

```
Stop-and-Wait:   [0] ──► wait for ACK ──► [1] ──► wait for ACK ──► [0]...
                  (window size = 1, one frame in flight at a time)

Go-Back-N:       [0][1][2] ──► sent together, window SLIDES forward
                  as acknowledgments come in, allowing new frames in
                  (window size > 1, multiple frames in flight)
```

---

## 2. Window Size Formulas

> ⚠️ **Most important formulas in this topic — note these down exactly.**

```
Sender's Window Size    = 2^m − 1
Receiver's Window Size  = 1
```

- **Sender window size = 2^m − 1** (this is the number of frames the sender can transmit before it must stop and wait for acknowledgments).
- **Receiver window size = 1** — the receiver can still only accept **one specific, correctly-ordered frame at a time** (this remains identical to Stop-and-Wait's receiver behavior).

---

## 3. Understanding "m" — Sequence Number Bits

> **m = the number of bits used to represent the sequence number.**

This is a common point of confusion — `m` is **not** the count of sequence numbers itself, it's the **number of bits** used to represent them.

```
If m = 2 bits, how many DISTINCT sequence numbers can 2 bits represent?

  2 bits → 00, 01, 10, 11  → 4 possible values → sequence numbers: 0,1,2,3

General rule:
  Total sequence numbers = 2^m

  m = 2 → 2^2 = 4 sequence numbers   (0,1,2,3)
  m = 3 → 2^3 = 8 sequence numbers   (0,1,2,3,4,5,6,7)
  m = 4 → 2^4 = 16 sequence numbers  (0 through 15)
```

| Bits (m) | Total Sequence Numbers (2^m) | Range  |
| -------- | ---------------------------- | ------ |
| 2        | 4                            | 0 – 3  |
| 3        | 8                            | 0 – 7  |
| 4        | 16                           | 0 – 15 |

---

## 4. Worked Example — m = 2

With **m = 2**, total sequence numbers = **4** (values 0, 1, 2, 3), and they **repeat/synchronize cyclically**:

```
Sequence numbers cycle:   0 1 2 3 | 0 1 2 3 | 0 1 2 3 | ...
                          (synchronized on both sender and receiver side)
```

### Sender Window Size for m = 2

```
Sender window size = 2^m − 1 = 2² − 1 = 4 − 1 = 3
```

```
Sequence timeline:   [0][1][2] 3
                      └──┬───┘
                    window size = 3
                  (sender can send 0, 1, 2 together)
```

- The sender's window covers **3 frames** at a time (e.g., frames `0, 1, 2`).
- The **receiver**, however, still has window size **1** — it accepts frames **one at a time, strictly in order** (accepts 0, then 1, then 2 — one by one, even though the sender fired them off together).

---

## 5. How the Sliding Window Moves

```
Step 1:  Window covers [0][1][2]   3
         Sender transmits 0, 1, 2 (the entire window)

Step 2:  Frame 0 gets acknowledged → window SLIDES forward by one
         Window now covers  0  [1][2][3]
         → Frame 3 now enters the window and can be sent

Step 3:  As each frame gets acknowledged, the window keeps sliding,
         always admitting the next new frame number into the window.
```

> **Key idea:** The window doesn't just sit still — it **slides** forward one position for every frame that gets successfully acknowledged, continuously admitting new frames as old ones are confirmed.

---

## 6. Why Window Size Must Be STRICTLY LESS THAN 2^m

> **Rule: Window size must be less than 2^m — NEVER equal to 2^m.**
> **Correct formula: Window size = 2^m − 1**

This is explained by comparing a **correct** scenario against a **broken** one.

### 6.1 Correct Case — Window Size = 2^m − 1

Using m = 2 → window size = 3 (frames 0, 1, 2 sent together):

```
Sender sends:  0, 1, 2   (all 3 ACKs get LOST on the way back)

Receiver side (still processes correctly even without ACKs arriving back):
  Receives 0 → accepts → now expecting 1
  Receives 1 → accepts → now expecting 2
  Receives 2 → accepts → now expecting 3
  Receiver's "next expected" pointer (Rn) is now at 3

  Receiver keeps sending ACK "3", but that ACK keeps getting lost too.

Sender side:
  No ACK arrives before timeout → timeout timer expires
  Sender assumes frames were lost → RESENDS 0, 1, 2 again

Receiver:
  Is expecting frame 3 — but frame 0 arrives instead
  → Receiver correctly DISCARDS it (it already accepted this 0 before)
  → No ambiguity — the receiver knows this is old/duplicate data
```

✅ **No confusion occurs** — because the receiver's expected pointer (3) is clearly different from the resent frame (0), so there's no risk of accidentally re-accepting old data as new.

---

### 6.2 Broken Case — Window Size = 2^m (Equal)

Now suppose (incorrectly) the window size were made **equal to 2^m = 4** (i.e., window covers 0, 1, 2, **and** 3 — the full cycle):

```
Sender sends:  0, 1, 2, 3   (the ENTIRE sequence space, all 4 numbers)

Receiver:
  Receives 0 → accepts → expecting 1
  Receives 1 → accepts → expecting 2
  Receives 2 → accepts → expecting 3
  Receives 3 → accepts → expecting... 0   (wrapped back around!)

  Now the receiver's "next expected" pointer is back at 0 —
  the SAME number the sequence started with.

Suppose ALL acknowledgments got lost on the way back:

  Sender times out → assumes nothing was received → RESENDS 0 (again)

Receiver:
  Is expecting 0 (because it wrapped around) — and 0 arrives!
  → Receiver ACCEPTS it, thinking it's a brand-new frame
  → ⚠️ BUT THIS IS THE OLD FRAME 0, being mistaken for a new one!
```

❌ **This is a real, unrecoverable ambiguity.** Because the sequence number space fully wrapped around, the receiver cannot tell the difference between "a genuinely new frame 0" and "the old frame 0 being resent after a lost ACK."

> **This is exactly why window size must be 2^m − 1, not 2^m.** Reducing the window by just 1 guarantees the resent (duplicate) frame's number can **never** coincide with the receiver's current expected number.

---

## 7. Cumulative Acknowledgment

Go-Back-N uses **cumulative acknowledgments** — an ACK number doesn't just confirm one frame, it confirms **everything up to that point**.

```
Example (m = 3, sequence numbers 0–7, window size = 2³ − 1 = 7):

Sender sends: 0, 1, 2, 3, 4, 5... (up to window limit)

Receiver:
  Receives 0 → accepts → sends ACK "1"  (means: "0 received, send 1 next")
  Receives 1 → accepts → sends ACK "2"
  Receives 2 → accepts → sends ACK "3" ✗ (this ACK gets LOST in transit)
  Receives 3 → accepts → sends ACK "4" ✓ (this one arrives successfully)

Sender receives ACK "4":
  → Sender interprets this as: "frames 0, 1, 2, AND 3 have ALL been
     successfully received" — even though the individual ACK for
     frame 2 never arrived!
```

> **Cumulative ACK meaning:** _"ACK N"_ means _"I have correctly received everything up through frame N−1, please send frame N next."_ A single ACK arriving late can implicitly confirm several earlier frames at once — this is what makes Go-Back-N resilient to occasional lost ACKs (as long as a _later_ ACK gets through).

---

## 8. Out-of-Order Packets Are REJECTED

This is described as a **major limitation** of Go-Back-N (and the key difference from **Selective Repeat**, covered next):

```
Sender sends: 0, 1, 2, 3   (all transmitted together, in the window)

Receiver:
  Receives 0 → accepts → sends ACK "1"
  Receiver is now waiting specifically for frame 1

  Frame 1 gets LOST in transit
  Frame 2 arrives — but receiver is waiting for 1, NOT 2
     → Receiver DISCARDS frame 2 (even though it arrived intact!)
  Frame 3 arrives — same story
     → Receiver DISCARDS frame 3 too (even though it also arrived intact!)

Result: Frames 2 and 3 are thrown away, DESPITE having arrived
        successfully, simply because they arrived "out of order"
        relative to what the receiver's single-frame window expects.

Sender:
  Eventually times out waiting for ACK of frame 1
  → Must RESEND frame 1, AND frame 2, AND frame 3 — even though
     2 and 3 had already reached the receiver once and were thrown away!
```

> **This wasteful "resend everything from the lost point onward" behavior is literally where the protocol gets its name: Go-Back-N** — when something goes wrong, the sender has to go back and resend N frames, including ones that actually arrived fine the first time.

### Why This Happens

- **Receiver window size = 1** — it can only hold/accept **exactly** the one frame it's currently expecting, in strict sequential order.
- Anything arriving out of that exact order is discarded, no exceptions — the receiver's window doesn't move until it gets the specific frame it's waiting for.

> **Contrast (preview):** In **Selective Repeat** (next topic), the receiver has a **larger** window and CAN buffer out-of-order packets instead of discarding them — meaning only the actually-lost frame needs to be resent, not everything after it. This is Go-Back-N's core efficiency trade-off vs. Selective Repeat.

---

## 9. Summary Table

| Feature                 | Go-Back-N                                                                                     |
| ----------------------- | --------------------------------------------------------------------------------------------- |
| Protocol type           | Sliding Window                                                                                |
| Sender window size      | 2^m − 1                                                                                       |
| Receiver window size    | 1                                                                                             |
| Sequence numbers needed | 2^m (m = number of bits used for sequence numbering)                                          |
| Window size vs 2^m      | Must be **strictly less than** 2^m (never equal)                                              |
| Acknowledgment type     | **Cumulative** (one ACK can confirm multiple prior frames)                                    |
| Out-of-order packets    | **Rejected/discarded** — receiver accepts only in strict sequence                             |
| Recovery after loss     | Resends the lost frame **and every frame after it**, even ones that already arrived correctly |

---

## 10. 🔴 Security Considerations & Modern Relevance

Go-Back-N's sliding window is the direct conceptual ancestor of **TCP's sliding window** mechanism (real TCP implementations behave more like Selective Repeat with SACK today, but the foundational sliding-window idea — and several of its rough edges — trace straight back here).

| Go-Back-N Concept                        | Real-World Security Relevance                                                                                                                                                                                                                                                                                                                                                                                                      |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Sequence number wraparound**           | The "window size must be < 2^m" problem is a small-scale version of the real **TCP sequence number wraparound** issue — TCP's 32-bit sequence space can wrap on very high-throughput or long-lived connections, historically relevant to attacks that rely on guessing/colliding sequence numbers                                                                                                                                  |
| **Cumulative acknowledgment**            | TCP's cumulative ACK model means a single spoofed or replayed ACK can implicitly "confirm" a large amount of data — this is part of why TCP session hijacking (forging ACKs/sequence numbers to inject or truncate a stream) has historically been a real attack class, mitigated today mainly by randomized initial sequence numbers and, more importantly, by encryption (TLS) making injected data cryptographically detectable |
| **Discarding valid out-of-order data**   | This is a genuine **efficiency/availability weakness** — an attacker who can selectively drop a _single_ frame (e.g., via targeted jamming or a malicious on-path node) forces the sender to retransmit far more data than necessary, amounting to a **traffic amplification / bandwidth-waste DoS** technique against Go-Back-N-style links                                                                                       |
| **Window size manipulation**             | Historically, attacks against TCP have manipulated advertised window sizes (e.g., the "shrinking window" / persistent small-window techniques used in some resource-exhaustion attacks like Sockstress) to force a server into holding excessive state or stalling — conceptually related to how window size directly controls how much unacknowledged data can be "in flight"                                                     |
| **Receiver only accepts exact sequence** | Understanding this rigid in-order requirement helps explain why protocols built on Go-Back-N-style logic are vulnerable to simple **selective packet loss/injection** by an on-path attacker — dropping just one frame in the middle of a window can force disproportionate retransmission, which is a useful mental model when reasoning about resilience of custom or legacy protocols you might assess                          |

> **Key takeaway for a security student:** Go-Back-N optimizes for **simplicity of implementation**, not for resilience against an adversary who can selectively interfere with specific frames. Its "discard everything out of order" behavior is efficient to implement but creates an amplification effect that a targeted attacker (rather than random loss) could exploit far more effectively than the random packet loss the protocol was designed to tolerate.

---

## 11. 🧪 Practical Labs — Your Setup

### Lab 1 — Observe TCP's Sliding Window in Action

```bash
# TCP is the real-world descendant of this sliding-window idea.
# Capture and inspect the advertised window size field live.
sudo tcpdump -i eth0 host 192.168.56.101 and tcp -v
# Look at the "win" field in each packet — this is the modern,
# dynamically-adjusted equivalent of Go-Back-N's fixed window size
```

### Lab 2 — Simulate Selective Frame Loss (Out-of-Order Effect)

```bash
# Use netem to induce loss and correlate it, simulating the scenario
# where a single dropped frame forces retransmission of everything after it

sudo tc qdisc add dev eth0 root netem loss 10% 25%
# "loss 10% 25%" = 10% loss probability, with 25% correlation
# (correlated loss more closely resembles a targeted/burst loss pattern)

curl http://192.168.56.101/ --max-time 15 -v
sudo tcpdump -i eth0 host 192.168.56.101 -v

# Clean up
sudo tc qdisc del dev eth0 root netem
```

### Lab 3 — Watch Cumulative ACK Behavior in a Real TCP Capture

```bash
# Capture a larger transfer and look at how ACK numbers jump forward,
# cumulatively confirming multiple segments even if individual ACKs
# were delayed or coalesced
sudo tcpdump -i eth0 host 192.168.56.101 and tcp -S -v -c 100
```

### Lab 4 — Measure Retransmission Overhead Under Loss

```bash
# Compare bytes transferred vs bytes actually needed, under induced loss,
# to see the real bandwidth cost of Go-Back-N-style "resend everything
# after the lost frame" behavior

sudo tc qdisc add dev eth0 root netem loss 5%
time curl http://192.168.56.101/ -o /dev/null --max-time 20
sudo tc qdisc del dev eth0 root netem
# Compare the time/throughput against a run with no induced loss
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                 GO-BACK-N ARQ — EXAM CHEAT SHEET                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  TYPE                                                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  Sliding Window Protocol (unlike Stop-and-Wait, window size > 1)     ║
╠══════════════════════════════════════════════════════════════════════╣
║  WINDOW SIZE FORMULAS (★ CRITICAL ★)                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  Sender window size    = 2^m − 1                                      ║
║  Receiver window size  = 1                                            ║
║  m = number of BITS used to represent the sequence number            ║
║  Total sequence numbers = 2^m                                         ║
║                                                                        ║
║  m=2 → seq 0–3,  sender window = 3                                    ║
║  m=3 → seq 0–7,  sender window = 7                                    ║
║  m=4 → seq 0–15, sender window = 15                                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY WINDOW SIZE ≠ 2^m                                                ║
║  ─────────────────────────────────────────────────────────           ║
║  If window size = 2^m (equal, full cycle used):                      ║
║    → sequence numbers can WRAP AROUND completely                     ║
║    → a resent OLD frame can collide with the receiver's CURRENT      ║
║      expected number → frame gets wrongly accepted as new (BUG)      ║
║  → Therefore window size MUST be 2^m − 1 (strictly less)             ║
╠══════════════════════════════════════════════════════════════════════╣
║  CUMULATIVE ACKNOWLEDGMENT                                             ║
║  ─────────────────────────────────────────────────────────           ║
║  "ACK N" means: all frames UP TO N−1 have been received correctly    ║
║  A later ACK can implicitly confirm several earlier lost ACKs        ║
╠══════════════════════════════════════════════════════════════════════╣
║  OUT-OF-ORDER PACKETS                                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  Go-Back-N receiver window size = 1 → STRICT in-order only           ║
║  Any frame arriving out of order is DISCARDED, even if it arrived    ║
║  correctly — sender must resend it AND everything after it           ║
║  (This inefficiency is fixed in Selective Repeat — next topic)       ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 24)

- [ ] Selective Repeat ARQ — individual retransmission, receiver buffering of out-of-order frames
- [ ] Selective Repeat window size formula (2^(m−1)) and why it differs from Go-Back-N's
- [ ] Full comparison: Stop-and-Wait vs Go-Back-N vs Selective Repeat
- [ ] Efficiency/throughput calculations for sliding window protocols

---

_Notes compiled from: Networking Course Lecture 23 — Go-Back-N ARQ_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
