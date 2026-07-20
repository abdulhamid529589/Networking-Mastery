# 🎯 Selective Repeat ARQ — Sliding Window Protocol

### " "Networking Course — Lecture 24

> **Source:** Gate Smashers — Selective Repeat ARQ
> " "
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Selective Repeat — Where It Fits](#1-selective-repeat--where-it-fits)
2. [Window Size Formula](#2-window-size-formula)
3. [Worked Example — m = 2](#3-worked-example--m--2)
4. [Why Window Size Must Be EQUAL to 2^(m−1), Not Greater](#4-why-window-size-must-be-equal-to-2m1-not-greater)
   - [4.1 Correct Case — Window = 2^(m−1)](#41-correct-case--window--2m1)
   - [4.2 Broken Case — Window > 2^(m−1)](#42-broken-case--window--2m1)
5. [Out-of-Order Packets ARE Accepted](#5-out-of-order-packets-are-accepted)
6. [Negative Acknowledgment (NAK)](#6-negative-acknowledgment-nak)
7. [Full Worked Example — m = 3](#7-full-worked-example--m--3)
8. [Go-Back-N vs Selective Repeat — Full Comparison](#8-go-back-n-vs-selective-repeat--full-comparison)
9. [🔴 Security Considerations & Modern Relevance](#9--security-considerations--modern-relevance)
10. [🧪 Practical Labs — Your Setup](#10--practical-labs--your-setup)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. Selective Repeat — Where It Fits

**Selective Repeat** is the **next category of sliding window protocol**, after Stop-and-Wait (window = 1) and Go-Back-N (sender window = 2^m − 1, receiver window = 1).

```
Stop-and-Wait:     sender window = 1        receiver window = 1
Go-Back-N:         sender window = 2^m − 1  receiver window = 1
Selective Repeat:  sender window = 2^(m−1)  receiver window = 2^(m−1)
```

> **The defining difference:** in Selective Repeat, the **receiver's window is also bigger than 1** — both sender and receiver windows are **equal** in size. This is what allows out-of-order acceptance (covered in Section 5).

---

## 2. Window Size Formula

> ⚠️ **Note this exact formula — it's the most commonly tested point in this topic.**

```
Sender Window Size    = 2^(m−1)
Receiver Window Size  = 2^(m−1)

(Both are EQUAL — unlike Go-Back-N where only the sender window was large)
```

- `m` = number of bits used to represent the sequence number (same definition as in Go-Back-N).
- Compare carefully with Go-Back-N's formula, since the exponent placement is the easy part to mix up:

```
Go-Back-N sender window     = 2^m − 1     (the "−1" is OUTSIDE/below the exponent)
Selective Repeat window     = 2^(m−1)     (the "−1" is INSIDE/part of the exponent)
```

---

## 3. Worked Example — m = 2

```
m = 2 → total sequence numbers = 2^m = 2² = 4   → range: 0, 1, 2, 3

Sequence numbers cycle:  0 1 2 3 | 0 1 2 3 | 0 1 2 3 | ...
```

### Window Size Calculation

```
Sender window size    = 2^(m−1) = 2^(2−1) = 2^1 = 2
Receiver window size  = 2^(m−1) = 2^(2−1) = 2^1 = 2
```

```
Sequence timeline:   [0][1]  2   3
                      └─┬─┘
                  window size = 2
              (both sender AND receiver windows are size 2)
```

- The window covers sequence numbers **0 and 1** — both the sender and receiver operate with this same window size of **2**.
- Sender transmits `0` and `1` together → receiver accepts **both** `0` and `1`.

---

## 4. Why Window Size Must Be EQUAL to 2^(m−1), Not Greater

> **Rule: Window size must be exactly 2^(m−1) — it must NOT be greater than that.**
> **If window size is made larger, the same duplicate-acceptance ambiguity problem from Go-Back-N reappears.**

### 4.1 Correct Case — Window = 2^(m−1)

Using m = 2 → window size = 2 (correct):

```
Sender sends: 0, 1   (receiver accepts both — ACK for 0,1 gets LOST on the way back)

Sender's window is now expecting to work with 2, 3 next (window has moved on)
Sender times out (no ACK received) → resends 0

Receiver's window is now sitting on [2][3] — NOT on 0 or 1 anymore
Receiver sees 0 arrive and thinks: "I want 2 and 3, why is 0 arriving?"
→ Receiver correctly recognizes this as a duplicate/lost-ACK situation
→ Receiver simply RESENDS the acknowledgment (no ambiguity, no wrong acceptance)
```

✅ No confusion — because the receiver's current window (`2,3`) is clearly disjoint from the resent frame (`0`).

---

### 4.2 Broken Case — Window > 2^(m−1)

Now suppose (incorrectly) the window size were made **3** instead of the correct value of 2:

```
Sender sends: 0, 1, 2   (window size taken as 3 — INCORRECT for m=2)

Receiver accepts all of 0, 1, 2 → window moves forward
Receiver's window is now sitting on [3][0][1]   ← notice: 0 and 1 are back
                                                    in the window range again!

The acknowledgment for 0,1,2 gets LOST on the way back.

Sender times out → resends frame 0 (the old, already-accepted duplicate)

Receiver's window currently includes 0 (because it wrapped back around)
→ Receiver ACCEPTS this resent frame 0, thinking it's NEW
→ ⚠️ BUG: this is actually the OLD frame 0 — but the receiver has no way
   to tell the difference, because its window overlaps with the old
   sequence number again
```

❌ **This is the exact same class of bug seen in Go-Back-N's "window = 2^m" mistake** — if the window is too large, the sequence number space wraps around far enough that an old, already-accepted frame can land back inside the receiver's current window and get mistakenly re-accepted as new data.

> **This is why: Selective Repeat window size = 2^(m−1) exactly — never larger.** This has been a frequently repeated exam question point.

---

## 5. Out-of-Order Packets ARE Accepted

This is the **headline difference** from Go-Back-N.

```
Sender's window contains: 0, 1, 2  (all can be sent together)

In GO-BACK-N:
  Receiver window size = 1 → ONLY accepts the exact frame it's
  currently waiting for, strictly in order. Anything else = discarded.

In SELECTIVE REPEAT:
  Receiver window size = 2 (or more, per formula) → receiver's window
  covers MULTIPLE sequence numbers at once.
  If frame 0 arrives a little late, but frame 1 arrives FIRST —
  → Receiver does NOT reject frame 1.
  → Because frame 1 falls within the receiver's current window range,
    it is ACCEPTED and buffered, even though it arrived "out of order."
```

> **Because the receiver's window is bigger than 1, it can accept ANY frame that falls within its current window range — regardless of arrival order — instead of insisting on one specific frame first.**

This is precisely why Selective Repeat is more efficient than Go-Back-N: correctly-arrived frames are never wastefully discarded just because an earlier frame was delayed or lost.

---

## 6. Negative Acknowledgment (NAK)

Even though out-of-order frames are accepted and buffered, there's still a catch: **a frame cannot be handed up to the next (upper) layer until the sequence is fully complete.**

> Analogy used: a file transfer that's 90% complete cannot be opened/used — it has to be **100% complete** before it can be handed off. Same logic applies here: buffered out-of-order frames wait until the missing piece arrives before the complete, in-order data is passed upward.

### Walkthrough

```
Window contains: 1, 2, 3, 4  (m=3 example, window size = 4)

Frame 1 → LOST in transit
Frame 2 → arrives → ACCEPTED and buffered (even though 1 hasn't arrived)
Frame 3 → arrives → ACCEPTED and buffered

Receiver now has: [2] [3] buffered, but is STILL missing frame 1
                   (nothing can be passed to the upper layer yet —
                    the sequence has a gap)

Instead of just acknowledging what it DOES have, the receiver sends a:

  NAK 1   →  "Negative Acknowledgment for frame 1"

Meaning: "I have received 2 and 3 successfully, but frame 1 specifically
          is MISSING — please resend frame 1 specifically."
```

### Why NAK Instead of a Regular (Cumulative) ACK?

```
If the receiver instead sent a regular/cumulative ACK saying "ACK 4"
(as Go-Back-N-style cumulative ACKs would)...

  → Sender would incorrectly conclude that 0,1,2,3 were ALL received
    successfully, and that frame 4 is what's needed next.
  → The missing frame 1 would be silently LOST and never resent!

By sending NAK 1 specifically:

  → Sender knows EXACTLY which frame is missing (frame 1) and can
    resend JUST that one frame — nothing else needs to be retransmitted.
```

> **NAK is what makes "selective" retransmission possible** — only the specific missing frame is resent, unlike Go-Back-N where everything from the lost point onward had to be resent.

---

## 7. Full Worked Example — m = 3

```
m = 3 → total sequence numbers = 2³ = 8 → range: 0 to 7

Window size = 2^(m−1) = 2^(3−1) = 2² = 4
(both sender AND receiver window = 4 — this is HALF of Go-Back-N's
window of 6/7 for the same m value)
```

### Step-by-Step Flow

```
Step 1:  Window = [0][1][2][3]
         Sender transmits frame 0 → accepted by receiver
         Window slides → now covers [1][2][3][4]
         Receiver sends ACK "1"  (0 accepted, send next from window)

Step 2:  Sender sends frame 1 → LOST in transit ✗
         Sender ALSO sends frame 2 and frame 3 (still within window)
         → Frame 2 arrives → ACCEPTED (buffered)
         → Frame 3 arrives → ACCEPTED (buffered)

         Receiver now has 2 and 3 buffered, but frame 1 is still MISSING
         → Nothing can be passed to the upper layer yet (gap at position 1)

Step 3:  Receiver sends NAK 1  → "I'm missing frame 1 specifically"

Step 4:  Sender resends ONLY frame 1 (not 2 or 3 — they already arrived!)
         Frame 1 arrives → ACCEPTED
         → NOW the receiver has a complete, contiguous 0,1,2,3
         → Data can finally be passed up to the next layer

Step 5:  Window slides forward → now covers [4][5][6][7]
         Sender transmits 4, 5, 6, 7 → cycle continues
```

---

## 8. Go-Back-N vs Selective Repeat — Full Comparison

| Feature                           | Go-Back-N                                          | Selective Repeat                                                   |
| --------------------------------- | -------------------------------------------------- | ------------------------------------------------------------------ |
| Sender window size                | 2^m − 1                                            | 2^(m−1)                                                            |
| Receiver window size              | 1                                                  | 2^(m−1) _(same as sender — both equal)_                            |
| Out-of-order packets              | **Rejected/discarded**                             | **Accepted and buffered**                                          |
| Acknowledgment type               | Cumulative ACK                                     | Individual ACK + **NAK** for specifically missing frames           |
| Retransmission on loss            | Resends the lost frame **AND everything after it** | Resends **ONLY** the specific missing frame                        |
| Efficiency                        | Lower (wastes bandwidth resending good frames)     | Higher (minimal, targeted retransmission)                          |
| Receiver complexity/buffering     | Simple — no buffering needed                       | More complex — must buffer out-of-order frames until gap is filled |
| Window size relative to Go-Back-N | —                                                  | **Half** the size, for the same value of m                         |

---

## 9. 🔴 Security Considerations & Modern Relevance

Selective Repeat's design — buffering out-of-order data and using explicit gap-tracking (NAK-style) — closely mirrors how **modern TCP with SACK (Selective Acknowledgment)** actually works. This makes its security implications directly relevant:

| Selective Repeat Concept                     | Real-World Security Relevance                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| -------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Receiver buffers out-of-order data**       | This is architecturally identical to how TCP reassembly buffers work — and **buffering unverified out-of-order data consumes memory**. An attacker who deliberately sends many out-of-order segments (never completing the sequence) can exhaust reassembly buffer memory on a target — a known class of **resource-exhaustion DoS** against stateful reassembly logic (relevant to both TCP stacks and IDS/IPS systems that must reassemble streams to inspect them) |
| **TCP SACK (the real-world equivalent)**     | Linux's TCP SACK implementation had a serious real vulnerability: **CVE-2019-11477 ("SACK Panic")** — a maliciously crafted sequence of SACK segments could trigger an integer overflow/kernel panic on vulnerable kernels. This is a concrete historical example of exactly the kind of edge-case complexity that Selective-Repeat-style gap tracking introduces compared to simpler protocols like Go-Back-N                                                        |
| **Window size wraparound bug (Section 4.2)** | Same underlying class of issue as **TCP sequence number wraparound / prediction attacks** — protocols that rely on modular arithmetic over a fixed sequence space must be very careful about window bounds, or old data can be mistaken for new (a general class of bug relevant well beyond this specific protocol)                                                                                                                                                  |
| **NAK (explicit "I'm missing X")**           | Explicit signaling of exactly what's missing gives an on-path attacker more precise information to exploit — e.g., an attacker who can observe or spoof NAKs could manipulate retransmission behavior more surgically than with a simple cumulative-ACK protocol, potentially forcing selective, repeated retransmission of specific data (a targeted denial/degradation technique)                                                                                   |
| **Complexity trade-off**                     | Selective Repeat's efficiency comes at the cost of significantly more receiver-side state and logic — as a general security principle, **more stateful protocol logic tends to mean a larger attack surface** (more edge cases, more places for implementation bugs), which is exactly what SACK Panic demonstrated in practice                                                                                                                                       |

> **Key takeaway for a security student:** Selective Repeat is the more "production-realistic" of the ARQ protocols covered so far — its buffering and gap-tracking behavior is what real TCP stacks actually implement (via SACK). That realism also means its security lessons are the most directly transferable: buffering unverified data is a resource-exhaustion risk, and added protocol complexity (however efficient) tends to expand the attack surface, as CVE-2019-11477 concretely demonstrated in the Linux kernel.

---

## 10. 🧪 Practical Labs — Your Setup

### Lab 1 — Confirm SACK is Enabled/Observe It in a Real Capture

```bash
# Check if your system has TCP SACK enabled (it almost certainly does by default)
cat /proc/sys/net/ipv4/tcp_sack

# Capture traffic to Metasploitable2 and look for SACK options in the
# TCP header during a connection with induced loss (see Lab 2)
sudo tcpdump -i eth0 host 192.168.56.101 and tcp -v
```

### Lab 2 — Force Out-of-Order Delivery and Watch SACK Handle It

```bash
# Introduce reordering (not just loss) to simulate the out-of-order
# scenario Selective Repeat is designed to handle gracefully
sudo tc qdisc add dev eth0 root netem delay 10ms reorder 25% 50%

curl http://192.168.56.101/ --max-time 15 -v
sudo tcpdump -i eth0 host 192.168.56.101 and tcp -v

# Look for SACK blocks in the TCP options — this is the real-world,
# production equivalent of the NAK/selective-retransmission behavior
# described in this lecture

sudo tc qdisc del dev eth0 root netem
```

### Lab 3 — Research CVE-2019-11477 (SACK Panic) — Read-Only Research Exercise

```bash
# This is a research/reading exercise, not an exploitation lab.
# Look up the public CVE writeup and kernel patch notes to understand
# HOW a SACK-processing edge case became a real vulnerability —
# useful case study for "why buffering/gap-tracking logic needs careful
# bounds-checking," directly tying back to Section 4's window-size math.

# (Use your own research/reading — official references:)
#   - CVE-2019-11477 (NVD entry)
#   - Linux kernel commit history for tcp_input.c around June 2019
```

### Lab 4 — Compare Retransmission Volume: Loss-Only vs Loss+Reorder

```bash
# Pure loss (closer to Go-Back-N's worst case — resend a range)
sudo tc qdisc add dev eth0 root netem loss 8%
time curl http://192.168.56.101/ -o /dev/null --max-time 20
sudo tc qdisc del dev eth0 root netem

# Loss + reorder (closer to Selective Repeat's target scenario —
# SACK should let TCP recover more efficiently, retransmitting less)
sudo tc qdisc add dev eth0 root netem loss 8% reorder 20% 50%
time curl http://192.168.56.101/ -o /dev/null --max-time 20
sudo tc qdisc del dev eth0 root netem

# Compare total time/throughput between the two runs
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              SELECTIVE REPEAT ARQ — EXAM CHEAT SHEET                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  TYPE                                                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  Sliding Window Protocol — sender AND receiver windows are EQUAL     ║
╠══════════════════════════════════════════════════════════════════════╣
║  WINDOW SIZE FORMULA (★ CRITICAL — DON'T MIX UP WITH GO-BACK-N ★)     ║
║  ─────────────────────────────────────────────────────────           ║
║  Sender window size    = 2^(m−1)                                      ║
║  Receiver window size  = 2^(m−1)     (SAME as sender — both equal)   ║
║                                                                        ║
║  Compare:                                                              ║
║    Go-Back-N sender window  = 2^m − 1     (−1 OUTSIDE exponent)      ║
║    Selective Repeat window  = 2^(m−1)     (−1 INSIDE exponent)       ║
║                                                                        ║
║  m=2 → seq 0–3, window = 2                                            ║
║  m=3 → seq 0–7, window = 4                                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY WINDOW ≠ GREATER THAN 2^(m−1)                                     ║
║  ─────────────────────────────────────────────────────────           ║
║  If window size > 2^(m−1):                                            ║
║    → sequence space wraps far enough that an OLD resent frame can    ║
║      fall back inside the receiver's CURRENT window                  ║
║    → gets wrongly accepted as a NEW frame (duplicate bug)            ║
║  → Therefore window size MUST equal 2^(m−1) exactly                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  OUT-OF-ORDER PACKETS                                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  ACCEPTED and BUFFERED (receiver window > 1, unlike Go-Back-N)       ║
║  BUT: cannot be passed to upper layer until the sequence is complete ║
║  (no gaps) — like a file that must be 100% downloaded before opening ║
╠══════════════════════════════════════════════════════════════════════╣
║  NEGATIVE ACKNOWLEDGMENT (NAK)                                          ║
║  ─────────────────────────────────────────────────────────           ║
║  Explicitly names the ONE missing frame (e.g., "NAK 1")              ║
║  → sender resends ONLY that specific frame, nothing else             ║
║  → this is what makes retransmission "selective"                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "Out-of-order accepted?"              → YES (Selective Repeat only) ║
║  "Resends only the lost frame?"        → YES (Selective Repeat)      ║
║  "Resends lost frame + everything after"→ Go-Back-N                  ║
║  "Sender window = Receiver window?"     → YES, only in Selective Repeat (Go-Back-N receiver = 1) ║
║  "Uses NAK"                             → Selective Repeat            ║
║  "Uses cumulative ACK"                  → Go-Back-N                   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 25)

- [ ] Full comparison: Stop-and-Wait vs Go-Back-N vs Selective Repeat (efficiency/throughput formulas)
- [ ] CRC (Cyclic Redundancy Check) — step-by-step calculation method
- [ ] CSMA/CD — detailed working and Ethernet collision handling

---

_Notes compiled from: Networking Course Lecture 24 — Selective Repeat ARQ_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
