# CSMA/CA — Carrier Sense Multiple Access with Collision Avoidance

### Computer Networks | Data Link Layer | Wireless LAN

---

## Table of Contents

1. [Overview & Where It's Used](#1-overview--where-its-used)
2. [Why CA Instead of CD?](#2-why-ca-instead-of-cd)
3. [CSMA/CD vs CSMA/CA — Side-by-Side](#3-csmacd-vs-csmaca--side-by-side)
4. [Key Terminology](#4-key-terminology)
5. [CSMA/CA Full Flowchart & Algorithm](#5-csmaca-full-flowchart--algorithm)
6. [RTS/CTS Mechanism](#6-rtscts-mechanism)
7. [Exponential Backoff (Contention Window)](#7-exponential-backoff-contention-window)
8. [Worked Examples](#8-worked-examples)
9. [Hidden Node Problem](#9-hidden-node-problem)
10. [IEEE 802.11 Frame Exchange](#10-ieee-80211-frame-exchange)
11. [Exam Questions & Answers](#11-exam-questions--answers)
12. [Quick Summary Cheatsheet](#12-quick-summary-cheatsheet)

---

## 1. Overview & Where It's Used

| Property       | Value                                                      |
| -------------- | ---------------------------------------------------------- |
| Full Name      | Carrier Sense Multiple Access with Collision **Avoidance** |
| Standard       | IEEE 802.11 (Wi-Fi)                                        |
| Layer          | Data Link Layer (MAC sublayer)                             |
| Medium         | **Wireless LAN**                                           |
| Problem Solved | Multiple stations sharing one wireless channel             |

**Core idea:** In a shared wireless medium, multiple stations want to transmit simultaneously. CSMA/CA coordinates this access so collisions are _avoided before they happen_, rather than detected after.

---

## 2. Why CA Instead of CD?

### The Problem with Collision Detection in Wireless

In **wired networks (Ethernet)**, Collision Detection works like this:

```
Station S transmits signal with energy E1
↓
Another signal arrives with energy E1
↓
Total energy at S = 2×E1
↓
S detects: "My energy doubled → COLLISION!"
```

This works because signals travel over wire with **minimal energy loss**.

In **wireless networks**, signals lose energy as they travel through air (path loss, attenuation):

```
Station A transmits with energy E1
↓
Signal travels through air → energy drops to 0.3×E1 by the time it reaches station B
↓
If B is also transmitting E1, total energy B detects = E1 + 0.3×E1 = 1.3×E1
↓
B cannot tell this is a collision — it's not double E1!
```

**Conclusion:** Energy-based collision detection is **unreliable in wireless** because signals weaken with distance. So instead of detecting collisions after they happen, we _avoid_ them beforehand.

### Real-World Analogy

> In wired: You're on a phone call. If someone else picks up the same landline, you both hear noise — you detect the collision immediately.
>
> In wireless: You're shouting across a field. You can't hear if someone far away is also shouting — you can't detect the collision. So instead, you raise your hand first (RTS), wait for a signal (CTS), and only then speak.

---

## 3. CSMA/CD vs CSMA/CA — Side-by-Side

| Property               | CSMA/CD                        | CSMA/CA                                     |
| ---------------------- | ------------------------------ | ------------------------------------------- |
| Full Name              | Collision **Detection**        | Collision **Avoidance**                     |
| Used In                | Wired LAN (Ethernet)           | Wireless LAN (Wi-Fi / IEEE 802.11)          |
| Standard               | IEEE 802.3                     | IEEE 802.11                                 |
| When collision happens | Detects during transmission    | Avoids before transmission                  |
| Mechanism              | Jam signal + backoff           | IFS + Contention Window + RTS/CTS           |
| Energy                 | Signal strength remains stable | Signal weakens with distance                |
| Efficiency             | Higher (fewer waits)           | Lower (many deliberate waits)               |
| Acknowledgements       | Not required for detection     | **Required** (only way to confirm delivery) |

---

## 4. Key Terminology

### IFS — Inter Frame Space

A mandatory **waiting period** before a station can attempt transmission. Even if the channel is sensed idle, stations don't transmit immediately — they wait IFS time first.

**Why?** Another station may have already started transmission just moments ago, and the signal hasn't reached you yet (propagation delay).

**Types of IFS:**

| Type | Name         | Priority                | Used For                          |
| ---- | ------------ | ----------------------- | --------------------------------- |
| SIFS | Short IFS    | Highest (shortest wait) | ACK, CTS responses                |
| DIFS | DCF IFS      | Normal                  | New data frame transmission       |
| EIFS | Extended IFS | Lowest                  | After receiving a corrupted frame |

**DIFS = DCF Inter Frame Space**, where **DCF = Distributed Coordination Function** — the decentralized coordination method used in 802.11.

---

### DCF — Distributed Coordination Function

The standard access method in 802.11. **Distributed** means there's no central controller telling each station when to transmit — each station decides independently using the CSMA/CA rules.

---

### Contention Window (CW)

The range of random backoff values a station chooses from before transmitting.

```
Initial window: [0, 2^(K+1) - 1]
After K collisions:
  K=0 → [0, 1]     (choose 0 or 1)
  K=1 → [0, 3]     (choose 0, 1, 2, or 3)
  K=2 → [0, 7]
  K=3 → [0, 15]
  K=n → [0, 2^(n+1) - 1]
```

This is called **Binary Exponential Backoff (BEB)** — the window doubles with each collision, reducing the probability that two stations pick the same value.

---

### Slot Time

A fixed unit of time used to measure backoff. Generally equal to the **round trip propagation time** of the channel.

```
Backoff Time (TB) = R × Slot Time
where R = randomly chosen value from contention window
```

---

### K — Attempt Counter

Tracks how many transmission attempts have been made. Starts at 0, increments by 1 after each failed attempt.

```
If K < K_max → retry
If K ≥ K_max → ABORT transmission
(K_max is typically 10 in 802.11)
```

---

### RTS — Ready To Send

A short control frame sent by the transmitter to **reserve the channel** before sending actual data. Contains the expected duration of the upcoming data transmission.

---

### CTS — Clear To Send

A short control frame sent by the **receiver** (usually the access point) in response to RTS, confirming the channel is clear.

All other stations that hear the CTS update their **NAV (Network Allocation Vector)** and stay silent for the duration.

---

### NAV — Network Allocation Vector

A virtual timer maintained by every station. When a station hears an RTS or CTS, it reads the duration field and sets its NAV to that value. During this time it stays silent — this is called **virtual carrier sensing**.

---

### ACK — Acknowledgement

Since CSMA/CA cannot detect collisions during transmission, the **only way to know if a frame was successfully received is to wait for an ACK**. If ACK doesn't arrive before timeout → assume collision → retry.

---

## 5. CSMA/CA Full Flowchart & Algorithm

```
START
  │
  ▼
K = 0  (reset attempt counter)
  │
  ▼
┌─────────────────┐
│  Sense channel  │◄────────────────────────┐
└────────┬────────┘                         │
         │                                  │
    Idle?                                   │
    │YES                │NO                 │
    ▼                   │                   │
Wait DIFS               └──────────────────►┘
    │                        (sense again)
    ▼
Choose random R from [0, 2^(K+1) - 1]
(Contention Window)
    │
    ▼
Start countdown timer = R × Slot Time
(Decrement only when channel is idle)
    │
    ▼
Timer reaches 0?
    │YES
    ▼
Send RTS (Ready To Send)
    │
    ▼
Wait SIFS (Short IFS)
    │
    ▼
CTS received before timeout?
    │YES                │NO
    ▼                   ▼
Wait SIFS           Collision suspected
    │               → increment K
    ▼               → if K < K_max: go to Sense
Send DATA           → if K ≥ K_max: ABORT
    │
    ▼
Wait SIFS
    │
    ▼
ACK received before timeout?
    │YES               │NO
    ▼                  ▼
SUCCESS            COLLISION
                   K = K + 1
                   if K < K_max → go to Sense
                   if K ≥ K_max → ABORT

END
```

### Step-by-Step Explanation

**Step 1: Sense the channel**

The station listens to the medium. This is the "CS" part — Carrier Sense.

**Step 2: Wait DIFS (if channel is idle)**

Even if the channel is idle, don't transmit yet. Wait DIFS time. This prevents two stations that sense simultaneously from both starting at exactly the same instant.

**Step 3: Contention Window — choose random R**

Each station picks a random number R from the current window [0, 2^(K+1) - 1]. This randomization is the key to _avoiding_ collisions — if two stations pick different values, they transmit at different times.

**Step 4: Count down R slots**

The countdown pauses whenever the channel is busy (someone else is transmitting). It only decrements when the channel is idle. This is called **backoff freezing**.

**Step 5: Send RTS**

When the timer hits 0, send a short RTS frame to the access point/receiver.

**Step 6: Wait for CTS**

The receiver processes the RTS, waits SIFS, then sends CTS. All other stations that hear either the RTS or CTS update their NAV and go silent.

**Step 7: Transmit data**

After receiving CTS, wait SIFS, then transmit the actual data frame.

**Step 8: Wait for ACK**

Receiver sends ACK after SIFS. If ACK arrives → success. If not → collision assumed → increment K → retry.

---

## 6. RTS/CTS Mechanism

### Why RTS/CTS?

The RTS/CTS handshake solves the **Hidden Node Problem** (covered in Section 9) and provides **virtual channel reservation**.

### Frame Sequence Timeline

```
Time →

Sender:   │──RTS──│  SIFS  │          │  SIFS  │────DATA────│  SIFS  │
                            ↑
Receiver:                   │──CTS──│  SIFS  │              │──ACK──│

Other nodes that hear CTS:  NAV ←────────────────────────────────────┘
(they stay silent for the duration specified in CTS)
```

### Timing Breakdown

```
1. Sender senses idle channel
2. Sender waits DIFS
3. Sender runs contention window countdown
4. Sender transmits RTS
5. Wait SIFS
6. Receiver sends CTS
7. Wait SIFS
8. Sender transmits DATA
9. Wait SIFS
10. Receiver sends ACK
11. If ACK received → SUCCESS
12. If no ACK → COLLISION → retry with K+1
```

### Duration Field in RTS/CTS

Both RTS and CTS contain a **duration field** telling all stations how long the upcoming exchange will take. Stations set their NAV accordingly and do not attempt to transmit during this time.

```
RTS duration = SIFS + CTS + SIFS + DATA + SIFS + ACK
CTS duration = SIFS + DATA + SIFS + ACK
```

---

## 7. Exponential Backoff (Contention Window)

### How the Window Grows

```
K = 0 (first attempt):   Window = [0, 2^1 - 1] = [0, 1]     → 2 choices
K = 1 (after 1 fail):    Window = [0, 2^2 - 1] = [0, 3]     → 4 choices
K = 2 (after 2 fails):   Window = [0, 2^3 - 1] = [0, 7]     → 8 choices
K = 3 (after 3 fails):   Window = [0, 2^4 - 1] = [0, 15]    → 16 choices
K = n:                   Window = [0, 2^(n+1) - 1]          → 2^(n+1) choices
```

Note: Some textbooks use [0, 2^K - 1] as the formula. The Gate Smashers version uses [0, 2^(K+1) - 1] starting from K=0. Check your syllabus for the exact formula — the concept is the same.

### Why Exponential?

If two stations collide and both choose from a small range (say [0, 1]), there's a 50% chance they pick the same number and collide again. By doubling the range after each collision, the probability of two stations picking the same value drops significantly, reducing repeated collisions.

### Backoff Time Calculation

```
TB = R × Slot Time

where:
  R = randomly chosen from contention window
  Slot Time = typically equal to round trip propagation time

Example:
  Slot Time = 9 µs (typical for 802.11g)
  K = 1, R chosen = 3
  TB = 3 × 9 = 27 µs
```

---

## 8. Worked Examples

### Example 1: Successful First Transmission

```
Setup:
  Station A wants to transmit
  K = 0

Step 1: A senses channel → IDLE
Step 2: A waits DIFS = 50 µs
Step 3: Window [0, 1] → A randomly picks R = 0
Step 4: Countdown = 0 → immediately proceed
Step 5: A sends RTS
Step 6: Access Point receives RTS, waits SIFS, sends CTS
        All other stations hear CTS → set NAV
Step 7: A receives CTS, waits SIFS, sends DATA
Step 8: Access Point receives data, sends ACK after SIFS
Step 9: A receives ACK → SUCCESS ✓
```

---

### Example 2: Collision Occurs

```
Setup:
  Stations A and B both want to transmit
  Both sense channel → IDLE at the same time
  Both wait DIFS
  Both choose R from [0, 1]
  A picks R = 0, B picks R = 0 (same!)

Step 1: Both count down to 0 simultaneously
Step 2: Both send RTS at the same time → COLLISION
        Neither receives CTS before timeout
Step 3: Both: K = K + 1 = 1
Step 4: Both now choose R from [0, 3]
        A picks R = 1, B picks R = 3
Step 5: A counts down faster → transmits first
        B's countdown pauses (channel busy)
Step 6: A completes RTS → CTS → DATA → ACK → SUCCESS
Step 7: B's NAV expires → B resumes countdown → transmits next
```

---

### Example 3: K Limit Reached

```
Station C has attempted 10 times (K = 10)
K_max = 10
K ≥ K_max → ABORT

Frame is dropped. Upper layer protocols
(TCP) will handle retransmission.
```

---

### Example 4: Calculating Backoff Time

```
Given:
  Slot Time = 20 µs
  K = 2 (third attempt)
  R is randomly chosen as 5

Window = [0, 2^3 - 1] = [0, 7]
R = 5 (valid, within window)
TB = R × Slot Time = 5 × 20 = 100 µs

Station waits 100 µs before attempting transmission.
```

---

## 9. Hidden Node Problem

### What is it?

In wireless networks, two stations may be out of range of each other but both in range of a common access point.

```
        [Station A] ←──────────────────────────────→ [Access Point] ←── [Station B]

A cannot hear B, B cannot hear A.
Both can communicate with the Access Point.
```

If A is transmitting to the AP, B cannot sense the channel as busy (it can't hear A). So B may start transmitting simultaneously → collision at the Access Point.

### How RTS/CTS Solves It

```
A sends RTS to AP
AP sends CTS — both A and B can hear the CTS
B reads the CTS duration field → sets NAV → stays silent
A transmits data without collision
```

The CTS from the AP reaches B even though A's transmission cannot. This effectively reserves the channel for A even from B's perspective.

---

## 10. IEEE 802.11 Frame Exchange

### Complete Timing Diagram

```
Sender      │ DIFS │ Backoff │ RTS │ SIFS │       │ SIFS │ DATA │ SIFS │      │
AP          │      │         │     │      │  CTS  │      │      │      │  ACK │
Others      │      │         │     │ NAV ←──────────────────────────────────── │
            ────────────────────────────────────────────────────────────────────→ time
```

### Why So Many Waits?

Each deliberate delay serves a purpose:

| Wait              | Duration | Purpose                                   |
| ----------------- | -------- | ----------------------------------------- |
| DIFS              | ~50 µs   | Ensure channel clear before starting      |
| Backoff           | Variable | Randomize access among competing stations |
| SIFS (after RTS)  | ~10 µs   | Give receiver time to prepare CTS         |
| SIFS (after CTS)  | ~10 µs   | Give sender time to prepare data          |
| SIFS (after DATA) | ~10 µs   | Give receiver time to prepare ACK         |

**SIFS < DIFS** — This ensures ACK/CTS responses get priority over new transmissions trying to start (which must wait DIFS).

---

## 11. Exam Questions & Answers

**Q1: Why is CSMA/CD not used in wireless networks?**

In wireless, signals lose energy as they travel (path loss). A transmitting station cannot detect a collision by comparing transmitted vs received energy because the received collision signal is much weaker than double the transmitted energy. CSMA/CA avoids collisions before they happen rather than detecting them after.

---

**Q2: What is the purpose of the IFS (Inter Frame Space) in CSMA/CA?**

IFS is a mandatory waiting period that prevents stations from immediately transmitting even when the channel appears idle. It accounts for propagation delay — another station may have started transmitting just before, and the signal hasn't arrived yet. DIFS is used before new transmissions; SIFS (shorter) is used for high-priority responses like ACK and CTS.

---

**Q3: What happens if a station doesn't receive an ACK?**

If ACK is not received before the timeout:

1. Collision is assumed
2. K is incremented by 1
3. If K < K_max (e.g., 10): station retries with a larger contention window
4. If K ≥ K_max: frame is aborted (dropped)

---

**Q4: Explain the Contention Window and Binary Exponential Backoff.**

The contention window is the range [0, 2^(K+1) - 1] from which a station randomly picks a backoff value R. After each collision, K increments by 1, doubling the window size. This exponential growth reduces the probability of repeated collisions between the same stations, as more choices mean less chance of two stations picking the same value.

---

**Q5: What is the difference between physical carrier sensing and virtual carrier sensing?**

- **Physical carrier sensing:** The station listens to the medium to detect ongoing transmissions (actual radio energy).
- **Virtual carrier sensing:** The station reads the duration field in RTS/CTS frames and sets its NAV accordingly, staying silent for that duration even if it doesn't directly hear the transmission.

---

**Q6: Why does CSMA/CA have lower efficiency than CSMA/CD?**

CSMA/CA introduces multiple deliberate delays (DIFS, backoff, SIFS, RTS/CTS handshake) to avoid collisions. These overhead intervals reduce the proportion of time actually used for data transmission. CSMA/CD doesn't need these pre-transmission waits — it just detects and recovers from collisions.

---

**Q7: What is DCF? What does it stand for?**

DCF = Distributed Coordination Function. It is the fundamental access mechanism in IEEE 802.11 (Wi-Fi). "Distributed" means there is no central controller — each station independently follows the CSMA/CA rules to decide when to transmit. DIFS stands for DCF Inter Frame Space.

---

**Q8: A station fails to receive CTS after sending RTS. What does it do?**

It assumes a collision occurred (either RTS collided with another RTS, or the channel was busy at the receiver). It increments K by 1, selects a new random R from the larger contention window, waits backoff time, then retries the entire process from sensing the channel.

---

**Q9: Calculate the backoff time if Slot Time = 9 µs, K = 2, and R = 4.**

```
Window at K=2: [0, 2^3 - 1] = [0, 7]
R = 4 (valid)
TB = R × Slot Time = 4 × 9 = 36 µs
```

---

**Q10: Solve the Hidden Node Problem using RTS/CTS.**

Stations A and B cannot hear each other but both connect to Access Point (AP). Without RTS/CTS, A's transmission is invisible to B, causing collisions at the AP.

With RTS/CTS: When A sends RTS, AP replies with CTS. B hears the CTS and reads its duration field, setting its NAV and staying silent. A then transmits safely without collision even though B cannot hear A directly.

---

## 12. Quick Summary Cheatsheet

```
CSMA/CA at a Glance
═══════════════════════════════════════════════════════════

WHERE: Wireless LAN (IEEE 802.11 / Wi-Fi)
WHY:   Collision Detection doesn't work in wireless (energy loss)
HOW:   Avoid collisions through deliberate waiting and handshaking

KEY STEPS:
  1. Sense channel (CS = Carrier Sense)
  2. If idle → wait DIFS (DCF Inter Frame Space)
  3. Choose random R from contention window [0, 2^(K+1) - 1]
  4. Count down R slots (pause if channel goes busy)
  5. Send RTS (Ready To Send)
  6. Wait SIFS → receive CTS (Clear To Send)
  7. Wait SIFS → transmit DATA
  8. Wait SIFS → receive ACK
  9. No ACK → K++ → retry (up to K_max times then abort)

CONTENTION WINDOW GROWTH:
  K=0: [0, 1]   → 2 options
  K=1: [0, 3]   → 4 options
  K=2: [0, 7]   → 8 options
  K=3: [0, 15]  → 16 options
  (doubles each collision)

TIMING PRIORITIES (shortest to longest wait):
  SIFS < DIFS < EIFS
  (ACK/CTS use SIFS — highest priority response)

BACKOFF TIME:
  TB = R × Slot Time
  Slot Time ≈ Round Trip Propagation Time

RTS/CTS PURPOSE:
  → Solves Hidden Node Problem
  → Virtual channel reservation via NAV
  → All stations hearing CTS stay silent for its duration

DCF = Distributed Coordination Function
      (decentralized, each station decides independently)

CSMA/CD vs CSMA/CA:
  CD → Wired (IEEE 802.3) → Detect after collision
  CA → Wireless (IEEE 802.11) → Avoid before collision
```

---

_Topic: CSMA/CA | Subject: Computer Networks | Data Link Layer — MAC Sublayer_
