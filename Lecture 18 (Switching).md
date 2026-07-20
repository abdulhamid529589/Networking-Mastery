# 🔗 Switching — Datagram Service vs Virtual Circuit

### " "Networking Course — Lecture 18

> **Source:** Gate Smashers — Datagram Switching & Virtual Circuit
> **Exam Relevance:** GATE (high — very frequently tested comparison) · UGC NET (high)
> **Topic Area:** Packet Switching — Two Implementation Approaches

---

## 📑 Table of Contents

1. [Recap — Two Types of Packet Switching](#1-recap--two-types-of-packet-switching)
2. [Datagram Service — Connectionless](#2-datagram-service--connectionless)
3. [Virtual Circuit — Connection-Oriented](#3-virtual-circuit--connection-oriented)
4. [Virtual Circuit = A "Mix" of Packet and Circuit Switching](#4-virtual-circuit--a-mix-of-packet-and-circuit-switching)
5. [Ordering — Out of Order vs Sequential](#5-ordering--out-of-order-vs-sequential)
6. [Overhead — Header Comparison](#6-overhead--header-comparison)
7. [Packet Loss](#7-packet-loss)
8. [Real-World Usage / Examples](#8-real-world-usage--examples)
9. [Cost](#9-cost)
10. [Efficiency](#10-efficiency)
11. [Delay](#11-delay)
12. [Master Comparison Table](#12-master-comparison-table)
13. [Why This Matters for Cybersecurity](#13-why-this-matters-for-cybersecurity)
14. [Exam Cheat Sheet](#14-exam-cheat-sheet)

---

## 1. Recap — Two Types of Packet Switching

As introduced in Part 17, **Packet Switching** can be implemented in two different ways:

1. **Datagram Switching (Datagram Service)** — Network Layer
2. **Virtual Circuit** — Data Link Layer

**Shared fundamental of both:** Data to be sent is **never sent directly** — it is first **divided into small packets**, and then those packets are sent over the network. This lecture explains **how each of these two approaches differs** in handling those packets.

---

## 2. Datagram Service — Connectionless

**The single most important keyword for Datagram Service: "Connectionless."**

**Connectionless means: there is NO reservation of resources.**

```
A wants to send data to B:
   1. Data is divided into packets (P1, P2, P3, P4...)
   2. Packets are sent DIRECTLY toward B
   3. NO CPU, Buffer, or Bandwidth is reserved in advance anywhere in the network
```

- Everything is handled **"on demand."**
- **On demand** means: as soon as Packet 1 arrives at a switch, it goes into that switch's **buffer** — this is the **store-and-forward** method (Part 17).
  - **Store** → requires a **buffer**
  - **Process** → requires **CPU**
  - **Transmit** → requires **bandwidth**
- All of these resources are obtained **on demand**, at the moment they're needed — **not pre-reserved**.
- **Consequence:** If a resource (e.g., buffer) is already in use when Packet 1 arrives, **Packet 1 must wait**.

---

## 3. Virtual Circuit — Connection-Oriented

**The single most important keyword for Virtual Circuit: "Connection-Oriented."**

**Connection-Oriented means: there IS a reservation, made before actual data transmission.**

```
A wants to send data to B (Virtual Circuit):
   1. Data is STILL divided into packets (this is still packet switching)
   2. BEFORE sending data packets, a "Global Packet" is sent first
   3. This Global Packet travels all the way to B
   4. Along the way, at EVERY switch it passes through, it RESERVES:
        - Buffer
        - CPU
        - Memory
```

- This is the key distinguishing behavior: a **reservation phase** happens first (via the Global Packet), establishing a fixed path with reserved resources at each switch **before** the actual data packets are sent.

---

## 4. Virtual Circuit = A "Mix" of Packet and Circuit Switching

**Virtual Circuit is described as roughly 50% Packet Switching + 50% Circuit Switching:**

| Component                  | Why                                                                                                                                |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| **~50% Packet Switching**  | Data is still divided into packets before being sent                                                                               |
| **~50% Circuit Switching** | A reservation process occurs first (like circuit switching's setup phase, Part 16), reserving buffer/CPU/memory along a fixed path |

> **Exam tip:** "Virtual Circuit is a hybrid/mix of packet switching and circuit switching" is a commonly tested one-line conceptual fact.

---

## 5. Ordering — Out of Order vs Sequential

### Datagram Service → Out of Order (possible)

```
Packet 1, 2, 3, 4 sent toward B.

Since there's NO reserved fixed path, each packet can independently
follow a DIFFERENT route, decided at each switch using its
ROUTING TABLE:

  P1 → Path A
  P2 → Path B
  P3 → Path C
  P4 → Path D  (possibly different from P3 too)
```

- Because different packets may take different paths (based on each switch's independent routing table decisions), they **may not arrive at B in the same order they were sent**.
- **Out-of-order arrival is possible** (e.g., P4 might arrive before P1).
- **Advantage of this behavior:** Better **traffic management** — since all available paths can be explored/utilized, the network is used more efficiently overall (load is spread across multiple paths rather than concentrated on one).

### Virtual Circuit → Sequential (Same Order)

```
Since the Global Packet already reserved ONE FIXED PATH,
ALL subsequent data packets follow that SAME path.

→ P1 arrives first, then P2, then P3... in the exact order sent.
```

- Because every packet follows the identical, pre-reserved path, packets **always arrive in the same sequential order** they were sent — no reordering needed at the destination.

---

## 6. Overhead — Header Comparison

### Datagram Service → HIGH Overhead

**What is "overhead"?** Extra data added on top of the actual payload — primarily via **headers**.

```
Example: Sending 100 bytes of actual data.
Can it be sent as just 100 bytes? NO — headers must be added.
```

- A **header** contains addressing/control information: **source address, destination address, TTL field, fragmentation info**, and more.
- **Analogy:** A header is like an **envelope** used for courier — it's "extra weight"/extra portion added around the actual item being sent.
- **Why headers are MORE in Datagram Service:** Since every packet may follow a **different path**, **every single packet** must carry its own **complete header** — each packet independently needs to know its own destination and routing information, since there's no shared pre-established path to rely on.
- **Result:** More headers → **higher overhead** relative to actual payload (e.g., if the header itself is 10 bytes and you're sending small chunks, that overhead becomes proportionally significant).

### Virtual Circuit → LOWER Overhead

```
Only the FIRST packet carries a "Global Header"
   (used during the initial reservation/setup by the Global Packet)

Remaining packets only need a smaller "LOCAL header"
   (since the path is already fixed/known via the earlier reservation)
```

- Because the path is already established, subsequent packets don't need to repeat full addressing/routing information — a lighter **local header** suffices.
- **Result:** Overhead is **lower** than Datagram Service, though not entirely zero (unlike pure circuit switching, which needs no headers at all — Part 16).

---

## 7. Packet Loss

| Service              | Packet Loss | Why                                                                                                                                                                                                                                    |
| -------------------- | ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Datagram Service** | **More**    | Each packet follows an independent path — P1 doesn't know which route P2 is taking, and vice versa. With multiple different, uncoordinated paths in use, there are more opportunities for a packet to be lost somewhere along the way. |
| **Virtual Circuit**  | **Less**    | Every packet follows the exact same, already-known, reserved path — since the route is fixed and known in advance, there's inherently less chance of a packet going astray or being lost.                                              |

---

## 8. Real-World Usage / Examples

### Datagram Service

- **Widely used today** — this is the foundation of the modern **Internet**.
- The Internet fundamentally relies on the **IP Network** (the entire IP addressing/classing system, and the Network Layer in general, is built on **Datagram Service** principles).

### Virtual Circuit

- Historically used technologies: **Frame Relay**, **X.25**, **ATM (Asynchronous Transfer Mode — NOT "Automated Teller Machine")**.
- Some of these (e.g., Frame Relay) are still in limited use, but **Datagram Service (IP-based)** is overwhelmingly the dominant approach used today.

---

## 9. Cost

| Service              | Cost       | Why                                                                                                                                                                                  |
| -------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Virtual Circuit**  | **Higher** | Every time a connection needs to be made, **resources must be reserved again** (CPU, bandwidth, buffer) — this reservation process has a cost associated with it, every single time. |
| **Datagram Service** | **Lower**  | No reservation is ever made — everything is handled **on demand**, so there's no repeated reservation cost.                                                                          |

---

## 10. Efficiency

| Service              | Efficiency | Why                                                                                                                                                                                                                                                          |
| -------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Datagram Service** | **Higher** | Since there's no advance reservation, resources are never left idle/wasted — if a source pauses sending data, those resources immediately become available for others to use (recall the restaurant-table analogy from Part 17: no wasted reserved seating). |
| **Virtual Circuit**  | **Lower**  | Reserved resources (CPU, bandwidth, buffer) sit dedicated to a specific connection whether or not they are actively being used at any given moment — this wastes capacity, reducing overall efficiency.                                                      |

---

## 11. Delay

| Service              | Delay    | Why                                                                                                                                                                                                                                                |
| -------------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Virtual Circuit**  | **Less** | Since the path and resources are already reserved in advance, packets don't need to wait for resource availability at each switch — like arriving at a restaurant where your table is already reserved: you go straight to your seat with no wait. |
| **Datagram Service** | **More** | Packets may need to **wait** at a switch if the required resource (buffer/CPU) is currently busy/unavailable at that moment — like walking into a restaurant without a reservation and having to wait for a table to become free.                  |

---

## 12. Master Comparison Table

| Property                           | Datagram Service                                | Virtual Circuit                                               |
| ---------------------------------- | ----------------------------------------------- | ------------------------------------------------------------- |
| **OSI Layer**                      | Network Layer                                   | Data Link Layer                                               |
| **Connection type**                | Connectionless — NO reservation                 | Connection-Oriented — reservation via Global Packet           |
| **Resource allocation**            | On-demand                                       | Reserved in advance                                           |
| **Path per packet**                | Can differ (independent routing per packet)     | Same fixed path for all packets                               |
| **Packet ordering at destination** | Out of order possible                           | Sequential / same order (guaranteed)                          |
| **Traffic management**             | Better (uses/explores multiple paths)           | Less flexible (single fixed path)                             |
| **Header overhead**                | High (every packet needs a full header)         | Lower (Global Header on first packet, Local Header afterward) |
| **Packet loss**                    | More likely                                     | Less likely                                                   |
| **Cost**                           | Lower (no repeated reservation cost)            | Higher (reservation cost every connection)                    |
| **Efficiency**                     | Higher (no idle reserved resources)             | Lower (idle reservation wastes capacity)                      |
| **Delay**                          | More (possible waiting for on-demand resources) | Less (resources pre-reserved, no waiting)                     |
| **Real-world examples**            | Internet / IP Network (dominant today)          | Frame Relay, X.25, ATM (legacy/limited use)                   |

---

## 13. Why This Matters for Cybersecurity

- **Connectionless = spoofable source addressing:** Since Datagram Service (the foundation of IP/the modern Internet) is connectionless and every packet is independently routed with its own header, there is no persistent, verified "circuit" tying a source address to a real session — this is the structural reason **IP spoofing** is possible at the network layer, and why protocols built on top of IP (like TCP) had to add their own connection-oriented handshaking (SYN/ACK) to compensate for this lack of built-in verification.
- **Out-of-order delivery and attack/defense implications:** Because datagram-based routing allows packets to take different paths and arrive out of order, this is directly why **higher-layer protocols (TCP)** need sequence numbers to reassemble data correctly — and why **packet reassembly** is a genuine area of security concern (e.g., fragmentation-based evasion techniques against IDS/firewalls exploit exactly this kind of out-of-order/fragmented delivery behavior).
- **Virtual Circuit-style networks and modern parallels:** The reservation/connection-oriented model of Virtual Circuit conceptually parallels modern **VPN tunnels** and **MPLS** (Multiprotocol Label Switching) — both establish a defined path with reserved treatment, which is one reason such approaches are sometimes preferred for security-sensitive, predictable traffic flows over plain best-effort IP routing.

---

## 14. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║       DATAGRAM SERVICE vs VIRTUAL CIRCUIT — EXAM CHEAT SHEET         ║
╠══════════════════════════════════════════════════════════════════════╣
║  ONE-WORD IDENTITY                                                       ║
║  ─────────────────────────────────────────────────────────          ║
║      Datagram Service  → CONNECTIONLESS (no reservation)              ║
║      Virtual Circuit   → CONNECTION-ORIENTED (Global Packet reserves ║
║                           path first)                                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  FULL COMPARISON (MEMORIZE AS A SET)                                    ║
║  ─────────────────────────────────────────────────────          ║
║      Property          │ Datagram      │ Virtual Circuit             ║
║      ───────────────────┼────────────────┼───────────────────         ║
║      OSI Layer           │ Network        │ Data Link                 ║
║      Reservation          │ None           │ Yes (via Global Packet)  ║
║      Packet path           │ Independent    │ Same fixed path          ║
║      Order at destination  │ Out of order   │ Sequential               ║
║      Header overhead        │ HIGH           │ Lower                   ║
║      Packet loss             │ MORE           │ Less                    ║
║      Cost                     │ Lower          │ HIGHER                 ║
║      Efficiency                │ HIGHER         │ Lower                  ║
║      Delay                      │ MORE           │ Less                    ║
║      Real-world example          │ Internet / IP  │ Frame Relay, X.25, ATM ║
╠══════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICK                                                            ║
║  ─────────────────────────────────────────────────────          ║
║      "Datagram = no reservation = cheap + efficient + but messier    ║
║       (out-of-order, more headers, more loss, more delay)"           ║
║      "Virtual Circuit = reserve first = orderly + reliable           ║
║       but costs more and is less efficient"                          ║
║      Virtual Circuit = ~50% Packet Switching + ~50% Circuit Switching║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Message Switching — comparison with circuit and packet switching
- [ ] TCP vs UDP — how connection-oriented vs connectionless plays out at the Transport Layer
- [ ] IP Addressing / Classful Addressing

---

_Notes compiled from: Networking Course Lecture 18 — Datagram Switching & Virtual Circuit_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
