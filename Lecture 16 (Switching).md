# 🔗 Switching — Circuit Switching

### Cybersecurity Student Notes | Networking Course — Lecture 16

> **Source:** Gate Smashers — Circuit Switching
> **Exam Relevance:** GATE (high — theory + numericals) · UGC NET (high)
> **Topic Area:** Switching Techniques — Physical Layer

---

## 📑 Table of Contents

1. [Origin — Designed for Telephone Networks](#1-origin--designed-for-telephone-networks)
2. [How Circuit Switching Works — The Telephone Exchange Model](#2-how-circuit-switching-works--the-telephone-exchange-model)
3. [Key Property #1 — Physical Layer Only](#3-key-property-1--physical-layer-only)
4. [Key Property #2 — Dedicated Path](#4-key-property-2--dedicated-path)
5. [Key Property #3 — Contiguous (In-Order) Flow](#5-key-property-3--contiguous-in-order-flow)
6. [Key Property #4 — No Headers Needed](#6-key-property-4--no-headers-needed)
7. [Key Property #5 — Efficiency is LOW](#7-key-property-5--efficiency-is-low)
8. [Key Property #6 — Delay is LOW](#8-key-property-6--delay-is-low)
9. [Why Circuit Switching Isn't Used for Modern Computer Networks](#9-why-circuit-switching-isnt-used-for-modern-computer-networks)
10. [Numerical — Total Time Formula](#10-numerical--total-time-formula)
11. [Summary Table](#11-summary-table)
12. [Why This Matters for Cybersecurity](#12-why-this-matters-for-cybersecurity)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. Origin — Designed for Telephone Networks

**Circuit switching was originally designed for telephone networks**, not computer networks.

- This predates the widespread use of the **OSI model** and **TCP/IP model**, which were developed for **computer networks** (PCs, laptops, and other devices sharing data).
- Before computer networks became dominant, the **telephone network** was the primary large-scale communication network — and it relied on physical circuit-based connections, established via **circuit switching**.

---

## 2. How Circuit Switching Works — The Telephone Exchange Model

```
[T1] ──► [Telephone Exchange (local)] ──► [Telephone Exchange (other city)] ──► [T5]
```

- Every landline telephone (`T`) connects to a local **telephone exchange**.
- Telephone exchanges are themselves interconnected (e.g., a city's exchange connects to another city's exchange).
- **Process:** You pick up the receiver, dial the number, and a **connection setup** process occurs (the dial tone represents this setup happening).

**Example: T1 wants to talk to T5**

- The exchange devices act like **multiplexers** — many users can connect through them.
- Once dialing completes, a **complete, dedicated path** is established: T1's port → through the exchanges → T5's port.
- This is the **circuit** being "switched" (established) — hence the name **circuit switching**.

**Setup Time:** The very first phase of any circuit-switched connection is the time required to establish this dedicated path — referred to as **Setup Time**. (Relevant for GATE/UGC NET numericals — see Section 10.)

---

## 3. Key Property #1 — Physical Layer Only

**Circuit switching operates purely at the Physical Layer.**

- There is no concept of "layers" here in the way computer networks use them (application → transport → network → data link → physical, with data broken into segments/packets/frames).
- The essential idea: devices are **connected**, and once connected, you simply **flow** data (voice, in the original telephone case) directly — no layered processing.

---

## 4. Key Property #2 — Dedicated Path

**Circuit switching always uses a dedicated path.**

```
T1 ══════════[dedicated circuit]══════════ T5
```

- Once the connection is established, this path is **reserved exclusively** for this communication.
- You can continue talking / flowing data for as long as the call remains connected — until you **disconnect**.
- No other user can use this specific reserved path while it's active — it functions like a "delicately" (deliberately) dedicated, exclusive channel, not a shared/point-to-multipoint arrangement.

---

## 5. Key Property #3 — Contiguous (In-Order) Flow

**Data flows continuously and strictly in order — never out of order.**

```
Sending: P1 → P2 → P3
Receiving: MUST arrive as P1 → P2 → P3 (never P2 → P3 → P1, etc.)
```

- Unlike computer network packet-based systems where data is divided into segments/packets/frames (which, in packet switching, _can_ arrive out of order and get reassembled), circuit switching has **no such division**.
- Analogy: On a phone call, you say **"hello"** before **"how are you?"** — the words/data must arrive in the exact order they were sent, because it's a **continuous, real-time flow**, not discrete independently-routed packets.

---

## 6. Key Property #4 — No Headers Needed

**Circuit switching does not require headers (no source/destination addressing per unit of data).**

**Compare with computer networks:**

| Layer                 | Addressing normally required  |
| --------------------- | ----------------------------- |
| Network Layer         | Source IP, Destination IP     |
| Transport Layer       | Source Port, Destination Port |
| Data Link Layer       | Source MAC, Destination MAC   |
| **Circuit Switching** | **None needed**               |

**Why no headers are needed:**

- Once **setup** is complete, you have already **reserved**:
  - The **port** (dedicated to this connection until termination)
  - The **bandwidth** of the channel/link
  - The **buffer/resources** along the path
- Since the entire path is exclusively yours for the duration of the connection, there's no need to tell any intermediate device "where this data should go" — addressing in headers is only needed when a device along the way must **decide** how to forward the data. In circuit switching, **that decision was already made once, during setup** — no further decisions are needed.

> **Trade-off:** Setup is time-consuming, but once complete, transmission becomes extremely simple and efficient in terms of overhead (no per-packet headers needed).

---

## 7. Key Property #5 — Efficiency is LOW

**Circuit switching has low efficiency.**

**Why?** Because of the same **reservation** behavior described above:

```
Example: You're on a phone call and say "wait 2 minutes, I'll finish something else."
You don't hang up — the connection remains active.
The bandwidth, port, and resources remain RESERVED for you...
...even though NO data is actually being transmitted during that time.
```

- Reserved resources **cannot be shared** with anyone else while idle — this **wastes** capacity that could otherwise be used by other users, reducing overall network **efficiency**.

---

## 8. Key Property #6 — Delay is LOW

**Circuit switching has low delay** (in terms of the "in transit" portion of communication).

**Why?**

- There is **no intermediate node processing** — no store-and-forward. Once the dedicated path exists, data flows straight through, port to port, without being stopped, buffered, or processed by any device along the way.
- This is in direct contrast to **packet switching** (a later topic), where intermediate devices (like routers) **store** each packet, process it, and **forward** it — introducing per-node processing delay at every hop.
- Because circuit switching has **no such stopping/processing**, the actual data-flow delay is minimal once the circuit is established.

---

## 9. Why Circuit Switching Isn't Used for Modern Computer Networks

- Since circuit switching works **purely at the physical layer**, it **cannot** divide/route data flexibly the way modern applications need (email, WhatsApp messages, web traffic, etc.).
- **Efficiency** is the most important requirement for computer networks handling many simultaneous, often bursty (non-continuous) communications — and circuit switching's resource-reservation model is fundamentally inefficient for this use case.
- **Conclusion:** Circuit switching is **best suited for telephone networks** (continuous, real-time voice), but is **not suitable** for typical computer network communication like email, instant messaging, or general internet/data traffic — those rely on **packet switching** instead.

---

## 10. Numerical — Total Time Formula

For GATE/UGC NET numerical questions, the **total time** for a circuit-switched transmission is calculated as:

```
Total Time = Setup Time + Transmission Time + Propagation Delay + Tear-Down Time
```

| Component             | Meaning                                                                                                                                                             |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Setup Time**        | Time taken to establish the dedicated connection/reservation (this value is typically **given** in the problem — it cannot be derived from other given quantities). |
| **Transmission Time** | `Message size ÷ Bandwidth` — how long it takes to actually push the message data through the connection.                                                            |
| **Propagation Delay** | `Distance ÷ Velocity (speed of signal propagation)` — the time for the signal itself to physically travel the distance.                                             |
| **Tear-Down Time**    | Time taken to **terminate** the connection and **free/release** all previously reserved resources (bandwidth, port, buffer, etc.).                                  |

> **Exam tip:** Memorize this exact **4-component sum**. Setup time is almost always a given value in the problem; the other three components (transmission, propagation, tear-down) may require calculation from provided message size, bandwidth, distance, and velocity.

---

## 11. Summary Table

| Property               | Circuit Switching                                                   |
| ---------------------- | ------------------------------------------------------------------- |
| **OSI Layer**          | Physical Layer only                                                 |
| **Path type**          | Dedicated (exclusive, reserved)                                     |
| **Data order**         | Strictly in-order / contiguous flow                                 |
| **Headers required?**  | No — resources pre-reserved during setup                            |
| **Efficiency**         | Low (idle reserved resources can't be shared)                       |
| **Delay (in-transit)** | Low (no intermediate store-and-forward)                             |
| **Best suited for**    | Telephone networks (continuous, real-time voice)                    |
| **Not suited for**     | Typical computer network data (email, messaging, web traffic)       |
| **Total time formula** | Setup Time + Transmission Time + Propagation Delay + Tear-Down Time |

---

## 12. Why This Matters for Cybersecurity

- **Resource exhaustion/DoS parallels:** The efficiency weakness described here (reserved-but-idle resources) is conceptually similar to modern **resource-exhaustion denial-of-service** patterns — e.g., an attacker holding open connections/sessions without using them (a version of this idea appears in attacks like **Slowloris**, which exploits how servers reserve connection resources for slow/idle clients).
- **Dedicated-path security intuition:** Circuit switching's "dedicated path" property is conceptually similar to why **VPN tunnels** or **dedicated leased lines** are considered more resistant to eavesdropping than shared/broadcast media — no other party shares the same reserved channel, unlike the broadcast-domain issues explored in Parts 5, 6, 11, and 15.
- **No headers = no in-flight inspection point:** Because circuit switching has no per-packet headers/addressing once the circuit is set up, there's no natural point for **deep packet inspection** or per-packet filtering mid-stream (unlike packet-switched networks, where routers/firewalls inspect headers at every hop) — worth keeping in mind conceptually when packet switching and firewall/IDS placement are covered later.

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                CIRCUIT SWITCHING — EXAM CHEAT SHEET                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  ORIGIN:  Designed for TELEPHONE networks (pre-dates computer         ║
║  networks / OSI / TCP-IP relevance)                                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  6 KEY PROPERTIES                                                        ║
║  ─────────────────────────────────────────────────────────          ║
║      1. Physical Layer only (no layered processing)                   ║
║      2. Dedicated path (exclusive, reserved end-to-end)               ║
║      3. Contiguous / strictly in-order data flow                      ║
║      4. NO headers needed (resources pre-reserved at setup)           ║
║      5. LOW efficiency (idle reserved resources = wasted capacity)    ║
║      6. LOW delay (no intermediate store-and-forward processing)      ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOTAL TIME FORMULA (NUMERICALS)                                        ║
║  ─────────────────────────────────────────────────────          ║
║      Total Time = Setup Time                                          ║
║                  + Transmission Time (message size ÷ bandwidth)       ║
║                  + Propagation Delay (distance ÷ velocity)            ║
║                  + Tear-Down Time                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  BEST USE CASE vs POOR USE CASE                                         ║
║  ─────────────────────────────────────────────────────          ║
║      BEST:  Telephone networks (continuous, real-time voice)          ║
║      POOR:  Modern computer network data (email, IM, web traffic)     ║
║             → because of LOW EFFICIENCY, not because it's "wrong",    ║
║               it's simply the wrong tool for bursty data              ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Packet Switching — store-and-forward, headers required, higher efficiency
- [ ] Message Switching — comparison with circuit and packet switching
- [ ] Circuit vs Packet vs Message Switching — full comparison table

---

_Notes compiled from: Networking Course Lecture 16 — Circuit Switching_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
