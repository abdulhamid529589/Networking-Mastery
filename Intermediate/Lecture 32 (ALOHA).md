# 📡 Pure ALOHA — Random Access Protocol

### " "Networking Course — Lecture 32

> **Source:** Gate Smashers — Multiple Access Protocols (Pure ALOHA)
> " "
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Pure ALOHA?](#1-what-is-pure-aloha)
2. [Network Model — Shared Medium](#2-network-model--shared-medium)
3. [Acknowledgement Mechanism](#3-acknowledgement-mechanism)
4. [Retransmission & Backoff](#4-retransmission--backoff)
5. [Historical & Exam-Relevant Facts](#5-historical--exam-relevant-facts)
6. [Vulnerable Time — Concept & Derivation](#6-vulnerable-time--concept--derivation)
7. [Efficiency of Pure ALOHA](#7-efficiency-of-pure-aloha)
8. [Where Pure ALOHA Sits in the Access Control Family](#8-where-pure-aloha-sits-in-the-access-control-family)
9. [🔴 Security Considerations & Modern Relevance](#9--security-considerations--modern-relevance)
10. [🧪 Practical Labs — Your Setup](#10--practical-labs--your-setup)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. What is Pure ALOHA?

- **Pure ALOHA** is a **Random Access Protocol** — an **Access Control** mechanism belonging to the MAC sub-layer of the **Data Link Layer**.
- "Random Access" means **no station is given a scheduled time slot**. Any station, the moment it has data, transmits **immediately**.
- There is **no carrier sensing** — a station does **not check** whether the channel is already busy before transmitting.

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

Because every station can transmit at will, with zero coordination and zero channel-sensing, **collisions are always possible** — this is the central problem the rest of the lecture's math (Vulnerable Time, Efficiency) exists to quantify.

---

## 2. Network Model — Shared Medium

- Multiple stations (say **A, B, C, D**) share a **single communication medium/channel**.
- Any of A, B, C, D can begin transmission the instant it receives data to send.

```
        A       B       C       D
         \      |       |      /
          \     |       |     /
           \____|_______|____/
                |
          Shared Channel/Medium
```

Since there's no scheduling and no carrier sensing, two or more stations may transmit **at overlapping times**, causing a **collision** — both messages become unusable.

---

## 3. Acknowledgement Mechanism

- Pure ALOHA **does include acknowledgements (ACK)**.
- Example: If **A sends data to C**:
  - If **no collision occurs**, C successfully receives the data and sends an **ACK back to A**.
  - A now knows it's safe to send further data.
- If **no ACK is received by the sender**, it implies:
  - A **collision occurred**.
  - **Neither station** received the data correctly, and **no ACK** was generated.

```
   No collision:   A ──data──► C ──ACK──► A     (success)
   Collision:       A ──data──►╳╳╳ (garbled)     No ACK received by A
```

---

## 4. Retransmission & Backoff

- If a collision happens (no ACK received), the sending station must **retransmit**.
- To avoid immediately colliding again, the station **waits for a random amount of time** before retransmitting.
- **Problem:** Careless/immediate retransmission increases the chance of repeated collisions.
- **Solution:** **Exponential Backoff** — used **only during retransmission after a collision**, not during a normal/first-time transmission.
  - (Exponential Backoff itself is a separate topic, covered in the following lecture.)

---

## 5. Historical & Exam-Relevant Facts

| Property           | Detail                                                                                       |
| ------------------ | -------------------------------------------------------------------------------------------- |
| Protocol type      | Random Access Protocol (Access Control sub-function of Data Link Layer)                      |
| Origin             | Introduced around **1970**                                                                   |
| Current status     | **Obsolete**, but modern protocols are conceptually based on it                              |
| Network scope      | **LAN-based** protocol                                                                       |
| Distance           | Short distances (LAN scale)                                                                  |
| Time consideration | Only **Transmission Time** is considered; **Propagation Time is ignored**                    |
| Time slot type     | **Fixed time slot** — transmission time assumed fixed for all stations, to simplify analysis |

---

## 6. Vulnerable Time — Concept & Derivation

### 6.1 What is Vulnerable Time?

**Vulnerable Time** = the time period during which a transmission is at risk of colliding with another station's transmission.

### 6.2 Setting Up the Example

- Message size = **1000 bits**
- Channel bandwidth = **100 Kbps**

**Transmission Time (TT) formula:**

```
Transmission Time = Message Size / Bandwidth
```

Calculation:

```
TT = 1000 bits / 100 Kbps
   = 1000 / (100 × 10^3)
   = 10 × 10^-3 seconds
   = 10 milliseconds
```

So this station transmits continuously for **10 ms**.

### 6.3 Why Collision Can Occur

```
                 ◄──────── 10 ms (before t) ────────►│◄──────── 10 ms (after t) ────────►
                  DANGER ZONE: a station starting     t   DANGER ZONE: any station starting
                  here could still be transmitting         here collides with current message
                  its last bit exactly when my first
                  bit begins
```

- Suppose a station starts transmitting at time **t**, and keeps transmitting until **t + 10 ms**.
- **During this entire 10 ms window**, if **any other station starts transmitting**, a collision occurs.
  - Even if only the **last bit** of the current message overlaps with the **first bit** of another station's message, the **entire message is corrupted** — collision affects the whole frame, not just the overlapping bit.
- It's also possible that **another station started slightly before time t**, and its **last bit** collides with **this station's first bit** — so there's a danger window **before** t too, equal to one more transmission time.

### 6.4 Final Vulnerable Time Formula

```
Vulnerable Time (VT) = 2 × Transmission Time (TT)
```

**Applying the numbers:**

```
VT = 2 × 10 ms = 20 ms
```

### 6.5 Interpretation

- If **any other station** transmits within this **20 ms vulnerable window**, a **collision will occur**.
- If the gap between transmissions is **greater than 20 ms**, **no collision occurs**.
- For a **collision-free environment**, only **one station should transmit within any 20 ms window** (per this numeric example).

---

## 7. Efficiency of Pure ALOHA

### 7.1 Efficiency Formula

Efficiency is denoted by **η (eta)**:

```
η = G × e^(-2G)
```

Where:

- **G** = number of stations that want to transmit data within a given time slot (offered load)
- The **−2** in the exponent comes directly from **Vulnerable Time = 2 × TT**.

### 7.2 Deriving Maximum Efficiency

**Step 1 — Differentiate (product rule):**

```
η = G × e^(-2G)

dη/dG = e^(-2G) × 1 + G × (-2) × e^(-2G)
      = e^(-2G) (1 - 2G)
```

**Step 2 — Set derivative to zero:**

```
e^(-2G) (1 - 2G) = 0

Since e^(-2G) is never zero:
1 - 2G = 0
G = 1/2
```

### 7.3 Meaning of G = 1/2

- On average, **maximum efficiency (least collision)** occurs when **half of the stations** attempt to transmit in a given period.
- Simplified example: with **2 stations**, efficiency is highest when **only 1 of the 2** transmits at a time.

### 7.4 Calculating Maximum Efficiency Value

```
η_max = (1/2) × e^(-2 × 1/2)
      = (1/2) × e^(-1)
```

Using **e ≈ 2.71**:

```
η_max = (1/2) × (1/2.71)
      = (1/2) × 0.368
      ≈ 0.184  →  18.4%
```

### 7.5 Interpretation of Result

- **Maximum efficiency of Pure ALOHA ≈ 18.4%**.
- Out of a **100 Kbps** channel, only about **18.4 Kbps** worth of data gets through successfully — the rest is **wasted in collisions**.
- This is considered **very low** compared to modern channel-access standards.

---

## 8. Where Pure ALOHA Sits in the Access Control Family

| Method             | Core Idea                                                                                                                                |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------------- |
| **Pure ALOHA**     | Send whenever ready, no carrier sensing, detect collisions after the fact via missing ACK                                                |
| **Slotted ALOHA**  | Improvement — transmission only allowed at the start of fixed time slots (halves the vulnerable time → doubles max efficiency to ~36.8%) |
| **CSMA/CD**        | Carrier Sense Multiple Access with Collision Detection — sense channel first, used in classic Ethernet                                   |
| **Token Ring/Bus** | A token is passed around; only the token holder may transmit                                                                             |

> Pure ALOHA is the **baseline/worst-case** random access scheme — every later scheme (Slotted ALOHA, CSMA, CSMA/CD) is essentially an optimization on top of this same "shared medium, collision-prone" problem.

---

## 9. 🔴 Security Considerations & Modern Relevance

Pure ALOHA itself is obsolete, but the **underlying shared-medium/contention problem** it models is directly relevant to several standard Layer 2 security topics:

| ALOHA Concept                                   | Related Security Topic                   | Notes                                                                                                                                                                                                        |
| ----------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **No carrier sensing before transmit**          | **Jamming / RF interference (wireless)** | A device transmitting continuously without sensing the channel mirrors how uncoordinated/interfering signals force collisions — relevant to understanding wireless interference and denial-of-service theory |
| **Collision as the failure mode**               | **Contention-based DoS theory**          | Understanding how forced collisions degrade a shared channel explains why flooding a contention-based medium reduces service for all users                                                                   |
| **Random backoff before retransmission**        | **Fairness in shared-channel access**    | A node that doesn't follow proper backoff timing can unfairly dominate a shared channel — a documented fairness issue studied in real 802.11 (Wi-Fi) MAC-layer research                                      |
| **ACK-based collision detection**               | **Trust in acknowledgement signals**     | Systems that treat "ACK received" as proof of success illustrate why acknowledgement integrity matters in protocol design                                                                                    |
| **Low efficiency under contention (18.4% max)** | **Capacity planning**                    | Understanding _why_ contention-based protocols have a low efficiency ceiling helps explain realistic throughput limits and resilience planning for shared-medium networks                                    |

> **Key takeaway:** Pure ALOHA is rarely encountered directly today (it's obsolete), but the **contention/collision model** it teaches is the conceptual foundation for understanding throughput limits, fairness, and interference on modern shared-medium networks like Wi-Fi.

---

## 10. 🧪 Practical Labs — Your Setup

### Lab 1 — Simulate & Compare Pure ALOHA vs Slotted ALOHA Efficiency

```bash
# Quick Python simulation to visually confirm the 18.4% vs 36.8%
# max efficiency difference between Pure and Slotted ALOHA
python3 - <<'EOF'
import numpy as np

def pure_aloha_efficiency(G):
    return G * np.exp(-2 * G)

def slotted_aloha_efficiency(G):
    return G * np.exp(-G)

Gs = np.linspace(0.01, 2, 200)
pure_max = max(pure_aloha_efficiency(Gs))
slotted_max = max(slotted_aloha_efficiency(Gs))

print(f"Pure ALOHA max efficiency:    {pure_max*100:.1f}%")
print(f"Slotted ALOHA max efficiency: {slotted_max*100:.1f}%")
EOF
```

### Lab 2 — Observe Real Shared-Medium Retry Behavior with Wireshark/tcpdump

```bash
# On your own Wi-Fi adapter, observe 802.11 MAC-layer retransmissions —
# a modern echo of ALOHA-style "collision → no ACK → retry" behavior
sudo airmon-ng start wlan0
sudo airodump-ng wlan0mon
# Look at the "Retry" flag on captured frames
```

### Lab 3 — Observe Collision Domain Behavior on Your Own LAN / Metasploitable2

```bash
# Generate concurrent traffic between your host and Metasploitable2
# to observe how a switched network limits collision domains compared
# to a hypothetical shared-bus (ALOHA-style) network
sudo tcpdump -i eth0 -e -c 50 host 192.168.56.101
```

### Lab 4 — Backoff Timing Demonstration

```bash
# Simple script demonstrating exponential backoff logic conceptually
python3 - <<'EOF'
import random, time

def send_with_backoff(max_attempts=5):
    for attempt in range(max_attempts):
        collision = random.random() < 0.5  # simulate 50% collision chance
        if not collision:
            print(f"Attempt {attempt+1}: Success, ACK received")
            return
        wait = random.uniform(0, 2**attempt)
        print(f"Attempt {attempt+1}: Collision. Backing off for {wait:.2f}s")
        time.sleep(0.1)  # shortened for demo purposes
    print("Max attempts reached, giving up")

send_with_backoff()
EOF
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                  PURE ALOHA — EXAM CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  PROTOCOL TYPE                                                         ║
║  ─────────────────────────────────────────────────────────           ║
║  Random Access Protocol (Access Control, Data Link Layer)            ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE PRINCIPLE                                                        ║
║  ─────────────────────────────────────────────────────────           ║
║  Transmit IMMEDIATELY when data is ready — NO carrier sensing,        ║
║  NO scheduling → collisions are always possible                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FORMULAS                                                          ║
║  ─────────────────────────────────────────────────────────           ║
║  Transmission Time (TT)  = Message Size / Bandwidth                   ║
║  Vulnerable Time (VT)    = 2 × TT                                     ║
║  Efficiency (η)          = G × e^(−2G)                                ║
║  G at Max Efficiency     = 1/2                                       ║
║  Maximum Efficiency      ≈ 18.4%                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY VT = 2×TT?                                                        ║
║  ─────────────────────────────────────────────────────────           ║
║  Danger zone exists BOTH before AND during own transmission window   ║
║  (a station starting just before you finishes overlapping your        ║
║   first bit; a station starting during your window overlaps too)     ║
╠══════════════════════════════════════════════════════════════════════╣
║  COLLISION HANDLING                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  ACK received       → success, no collision                          ║
║  No ACK received    → collision occurred                             ║
║  On collision       → wait RANDOM time → retransmit                  ║
║  Repeated collision → use EXPONENTIAL BACKOFF                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "Type of protocol"                    → Random Access Protocol      ║
║  "Layer it belongs to"                 → Data Link Layer (MAC sub)   ║
║  "Time considered (LAN)"               → Transmission Time only      ║
║  "Time NOT considered"                 → Propagation Time            ║
║  "Formula for Vulnerable Time"         → 2 × Transmission Time       ║
║  "Formula for Efficiency"              → G × e^(−2G)                 ║
║  "G value for max efficiency"          → 1/2                         ║
║  "Max efficiency value"                → 18.4%                       ║
║  "Backoff used when?"                  → Only during retransmission  ║
║  "Year introduced"                     → ~1970                       ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 33)

- [ ] Exponential Backoff Algorithm — detailed working
- [ ] Slotted ALOHA — how halving Vulnerable Time doubles max efficiency to 36.8%
- [ ] CSMA (Carrier Sense Multiple Access) — 1-persistent, non-persistent, p-persistent variants
- [ ] CSMA/CD — collision detection and Ethernet's backoff behavior

---

_Notes compiled from: Networking Course Lecture 32 — Pure ALOHA (Multiple Access Protocols)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
