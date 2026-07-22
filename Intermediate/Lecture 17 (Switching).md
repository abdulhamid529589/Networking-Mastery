# 🔗 Switching — Packet Switching

### " "Networking Course — Lecture 17

> **Source:** Gate Smashers — Packet Switching
> **Exam Relevance:** GATE (high — theory + numericals) · UGC NET (high)
> **Topic Area:** Switching Techniques — Data Link Layer & Network Layer

---

## 📑 Table of Contents

1. [Packet Switching vs Circuit Switching — The Core Difference](#1-packet-switching-vs-circuit-switching--the-core-difference)
2. [OSI Layer — Data Link & Network Layer](#2-osi-layer--data-link--network-layer)
3. [Two Types of Packet Switching](#3-two-types-of-packet-switching)
4. [Store-and-Forward](#4-store-and-forward)
5. [The Restaurant Table Analogy — Efficiency](#5-the-restaurant-table-analogy--efficiency)
6. [Efficiency vs Delay Trade-off](#6-efficiency-vs-delay-trade-off)
7. [Pipelining](#7-pipelining)
8. [Total Time — Formula](#8-total-time--formula)
9. [Summary Table — Packet Switching vs Circuit Switching](#9-summary-table--packet-switching-vs-circuit-switching)
10. [Why This Matters for Cybersecurity](#10-why-this-matters-for-cybersecurity)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. Packet Switching vs Circuit Switching — The Core Difference

**Packet switching is fundamentally different from circuit switching** (Part 16).

- In **packet switching**, we talk about **packets** — the format in which data is actually transmitted.
- This concept relies on the **layered model** (OSI or TCP/IP): data originates at the **Application Layer**, passes through the **Transport Layer**, then the **Network Layer**, and finally exits via the **Physical Layer**.
- **What the Network Layer / Data Link Layer actually does:** they **divide the continuous stream of data into packets** — small, discrete units — before sending them onto the network.

```
Continuous data (from upper layers)
        │
        ▼
  [Divided into PACKETS: P1, P2, P3, P4...]
        │
        ▼
   Sent onto the network individually
```

**Contrast with circuit switching:** In circuit switching, once a circuit/connection is established, there is a **continuous flow** of data — no division into packets or segments. In packet switching, data is **always** transmitted in the form of discrete packets.

> **Golden rule (first and most important point):** Data is **always transmitted in packets**. If A wants to send data to B, that data is first divided into packets (e.g., 4 packets from one data stream) before transmission.

---

## 2. OSI Layer — Data Link & Network Layer

**Packet switching works at the Data Link Layer and/or the Network Layer** — NOT the Physical Layer (which is where circuit switching operates).

| Switching Type                          | OSI Layer           |
| --------------------------------------- | ------------------- |
| Circuit Switching (Part 16)             | Physical Layer      |
| **Packet Switching — Datagram Service** | **Network Layer**   |
| **Packet Switching — Virtual Circuit**  | **Data Link Layer** |

> **Exam tip:** This layer mapping is a very commonly tested fact — remember it as a set: **Physical → Circuit Switching**, **Network Layer → Datagram**, **Data Link Layer → Virtual Circuit**.

---

## 3. Two Types of Packet Switching

Packet switching is implemented in **two ways**:

| Type                 | Layer           | Importance                                                                     |
| -------------------- | --------------- | ------------------------------------------------------------------------------ |
| **Datagram Service** | Network Layer   | **Very important** — most commonly used, and the primary focus of the syllabus |
| **Virtual Circuit**  | Data Link Layer | Covered separately (mentioned as a follow-up topic)                            |

> The instructor notes that **Datagram Service** will be covered in detail in the **next video** — this lecture focuses on the shared fundamentals of packet switching in general.

---

## 4. Store-and-Forward

**Yes — packet switching uses the store-and-forward technique.**

**Example: A wants to send data to B**

```
A divides data into 4 packets.

Unlike circuit switching, NO connection is established beforehand.
A simply sends the packets directly — starting with the nearest switch, S1.

[A] ──(Packet)──► [S1: Switch]
```

**What does S1 do when the packet arrives?**

1. **Store** — the packet is placed into the switch's **buffer** (memory).
2. **Why store it?** So the switch can **decide** the best path/route for forwarding — using the concept of a **routing table**.
3. **Process** — the switch consults its routing table to determine how/where to route the packet.
4. **Forward** — the switch then sends the packet onward to the next hop.

**Contrast with circuit switching:**

```
Circuit switching:  Connection established FIRST (e.g., dialing a phone)
                     → THEN continuous data transmission, no store-and-forward

Packet switching:    NO connection established first
                      → each switch STORES → PROCESSES → FORWARDS every packet
```

---

## 5. The Restaurant Table Analogy — Efficiency

The instructor uses a memorable analogy to explain the **efficiency** trade-off between the two switching types:

> **Packet Switching = walking into a restaurant and searching for an open table.** You didn't reserve anything in advance — you simply look for available seating when you arrive.

> **Circuit Switching = reserving a table in advance** (e.g., in the morning, reserving for 5:00 PM). If you actually arrive at 7:00 PM instead, the table sat **reserved and unused** from 5:00–7:00 PM — nobody else could use it, even though it wasn't actively being used.

This directly illustrates **why circuit switching has lower efficiency** (Part 16) — reserved resources go to waste when not actively in use — while **packet switching has higher efficiency**, since no resources are pre-reserved; packets simply use whatever path/switch capacity is available when they arrive.

---

## 6. Efficiency vs Delay Trade-off

Because of the **store-and-forward** mechanism:

| Property       | Circuit Switching                                                      | Packet Switching                                                                                                |
| -------------- | ---------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| **Efficiency** | Low (resources reserved even when idle)                                | **High** (no wasted reservation — store-and-forward uses resources on demand)                                   |
| **Delay**      | Low (no intermediate storing/processing — data flows straight through) | **Higher** (each switch must store, process via routing table, and then forward — this takes time at every hop) |

**Why is delay higher in packet switching?**

- Each packet is temporarily held in a switch's **buffer** for some (typically short) amount of time before being processed and forwarded.
- **The more switches a packet passes through, the more this delay accumulates** — each additional hop adds its own store-and-forward delay.

> **Key trade-off to remember:** Packet switching **trades some delay for much higher efficiency**, compared to circuit switching's low delay but low efficiency.

---

## 7. Pipelining

**Packet switching uses the concept of pipelining** to further improve efficiency and increase the effective number of packets that can be in transit at once.

**How pipelining works:**

```
A needs to send 4 packets to B.

WITHOUT pipelining (sequential):
  Send Packet 1 → wait for it to fully arrive → THEN send Packet 2 → ...

WITH pipelining (parallel):
  A sends Packet 1 to S1.
  While S1 is forwarding Packet 1 onward (e.g., to S3),
  A does NOT wait — A immediately sends Packet 2 as well.

  This happens in PARALLEL, not one-at-a-time sequentially.
```

- Pipelining means that while one switch is busy forwarding an earlier packet further down the path, the **sender doesn't wait** — it continues sending subsequent packets right away.
- This **overlapping/parallel transmission** significantly **increases efficiency** and **reduces the overall delay** compared to a strictly sequential (wait-for-each-packet) approach.

> The instructor notes this concept becomes much clearer with a **worked numerical example** (to be covered separately alongside datagram service).

---

## 8. Total Time — Formula

**Packet switching's total time formula differs from circuit switching's** (Part 16).

| Switching Type        | Total Time Components                                                        |
| --------------------- | ---------------------------------------------------------------------------- |
| **Circuit Switching** | Setup Time + **Transmission Time** + Propagation Delay + Tear-Down Time      |
| **Packet Switching**  | **Transmission Time (multiplied by number of switches)** + Propagation Delay |

**Why does Transmission Time change?**

```
Circuit switching:
  Connection is established ONCE (setup phase).
  After that, A transmits data ONCE — it flows straight through
  without being re-transmitted at each intermediate device.
  → Transmission Time is used only ONCE.

Packet switching:
  A transmits Packet 1 to S1.
  S1 STORES, PROCESSES, and RE-TRANSMITS it onward to S3.
  S3 STORES, PROCESSES, and RE-TRANSMITS it further.
  ...and so on, at EVERY switch the packet passes through.

  → Transmission Time is effectively incurred AT EACH SWITCH.
```

**Formula:**

```
Total Time = (n × Transmission Time) + Propagation Delay

Where n = number of switches the packet passes through
      (i.e., how many times the packet must be re-transmitted
       along its path, once per switch)
```

> **Exam tip:** This `n ×` multiplier on transmission time — where `n` is the number of switches in the network — is the key numerical distinction to remember versus circuit switching, where transmission time is only counted **once** because the whole path is already reserved/connected in advance.

---

## 9. Summary Table — Packet Switching vs Circuit Switching

| Property               | Circuit Switching (Part 16)                      | Packet Switching                                                               |
| ---------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------ |
| **OSI Layer**          | Physical Layer                                   | Data Link Layer (Virtual Circuit) / Network Layer (Datagram)                   |
| **Connection setup**   | Required first (dedicated path)                  | Not required — no pre-established connection                                   |
| **Data format**        | Continuous flow (no division)                    | Divided into discrete **packets**                                              |
| **Path**               | Dedicated, exclusive                             | No fixed/dedicated path (per-switch routing decisions)                         |
| **Store-and-forward**  | No                                               | **Yes** (every switch: store → process → forward)                              |
| **Headers required**   | No                                               | **Yes** (each packet needs addressing, since routing decisions happen per-hop) |
| **Efficiency**         | Low (idle reserved resources wasted)             | **High** (no wasted reservation)                                               |
| **Delay**              | Low (no intermediate processing)                 | **Higher** (accumulated store-and-forward delay per switch)                    |
| **Pipelining**         | Not applicable                                   | **Yes** — improves efficiency/reduces delay further                            |
| **Total time formula** | Setup + Transmission + Propagation + Tear-down   | **(n × Transmission) + Propagation**                                           |
| **Best suited for**    | Telephone networks (continuous, real-time voice) | Computer networks (email, web traffic, messaging, bursty data)                 |

---

## 10. Why This Matters for Cybersecurity

- **Every hop = an inspection point:** Because packet switching requires headers (source/destination addressing) at every packet and involves per-hop store-and-forward processing, this is _exactly_ what makes modern network security tooling possible — firewalls, IDS, and routers can inspect headers and payload at each hop, unlike circuit-switched connections where no such natural inspection point exists (Part 16). This is foundational to understanding where and how packet inspection, filtering, and logging happen in real networks.
- **Store-and-forward buffers as an attack surface:** Since every switch/router buffers packets before forwarding, buffer/queue exhaustion is a real attack vector — this is the conceptual basis for certain **denial-of-service** techniques that flood intermediate devices with more packets than their buffers/processing capacity can handle.
- **Datagram vs Virtual Circuit — relevant to later topics:** The distinction between **Datagram Service** (Network Layer, connectionless — this is how IP itself fundamentally works) and **Virtual Circuit** (Data Link Layer, connection-oriented) is foundational once you study **IP spoofing** and connectionless-protocol-based attacks later — connectionless datagram delivery (like UDP/IP) is exactly why source IP addresses can be spoofed more easily than in connection-oriented protocols.

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║               PACKET SWITCHING — EXAM CHEAT SHEET                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE IDENTITY: Data is ALWAYS transmitted in the form of PACKETS    ║
║  (unlike circuit switching's continuous flow)                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI LAYER MAPPING (MEMORIZE THIS SET)                                ║
║  ─────────────────────────────────────────────────────────          ║
║      Circuit Switching        → Physical Layer                       ║
║      Datagram Service          → Network Layer                       ║
║      Virtual Circuit           → Data Link Layer                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY MECHANISM: STORE-AND-FORWARD                                       ║
║  ─────────────────────────────────────────────────────          ║
║      Every switch: STORE (buffer) → PROCESS (routing table)          ║
║                     → FORWARD                                         ║
║      No pre-established connection needed (unlike circuit switching) ║
╠══════════════════════════════════════════════════════════════════════╣
║  EFFICIENCY vs DELAY (OPPOSITE OF CIRCUIT SWITCHING)                    ║
║  ─────────────────────────────────────────────────────          ║
║      Efficiency → HIGH (no idle reserved resources — "no advance     ║
║                    table reservation," unlike circuit switching)     ║
║      Delay       → HIGHER (accumulates at every switch, store-and-   ║
║                    forward processing time per hop)                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  PIPELINING                                                              ║
║  ─────────────────────────────────────────────────────          ║
║      Sender does NOT wait for one packet to fully arrive before      ║
║      sending the next — packets move in PARALLEL through the         ║
║      network → boosts efficiency, reduces overall delay              ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOTAL TIME FORMULA (NUMERICALS)                                        ║
║  ─────────────────────────────────────────────────────          ║
║      Circuit Switching: Setup + Transmission + Propagation + Teardown║
║      Packet Switching:  (n × Transmission) + Propagation             ║
║           where n = number of switches along the path                ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Datagram Service — detailed working (Network Layer, connectionless)
- [ ] Virtual Circuit — detailed working (Data Link Layer, connection-oriented)
- [ ] Datagram vs Virtual Circuit — full comparison
- [ ] Numerical worked examples on packet switching total time (with pipelining)

---

_Notes compiled from: Networking Course Lecture 17 — Packet Switching_
_Source: Gate Smashers YouTube (original lecture in Hindi; notes translated and structured in English for consistency with the rest of this notes series)_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
