# 🔗 Switching Techniques — Message Switching

### Cybersecurity Student Notes | Networking Course — Lecture 19

> **Source:** Gate Smashers — Message Switching (Part 19)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Switching?](#1-what-is-switching)
2. [Timeline: How Switching Evolved](#2-timeline-how-switching-evolved)
3. [Recap: Circuit Switching](#3-recap-circuit-switching)
4. [Message Switching](#4-message-switching)
   - [4.1 Core Concept — Store & Forward](#41-core-concept--store--forward)
   - [4.2 Structure & Flow Diagram](#42-structure--flow-diagram)
   - [4.3 Storage Medium Evolution](#43-storage-medium-evolution)
   - [4.4 Reliability & Resource Reservation](#44-reliability--resource-reservation)
   - [4.5 Delay Analysis](#45-delay-analysis)
   - [4.6 Advantages & Disadvantages](#46-advantages--disadvantages)
5. [Message Switching vs Circuit Switching](#5-message-switching-vs-circuit-switching)
6. [Message Switching → Packet Switching](#6-message-switching--packet-switching)
7. [🔴 Security Considerations & Modern Relevance](#7--security-considerations--modern-relevance)
8. [🧪 Practical Labs — Real-World Store-and-Forward Systems](#8--practical-labs--real-world-store-and-forward-systems)
9. [Exam Cheat Sheet](#9-exam-cheat-sheet)

---

## 1. What is Switching?

**Switching** = the method by which data is moved from a source device to a destination device across a network of intermediate nodes (switches/routers), when a direct dedicated wire between every pair of devices isn't practical.

Three classical switching techniques, studied in this order historically:

```
Circuit Switching ──► Message Switching ──► Packet Switching
```

| Technique             | Core Idea                                                                      |
| --------------------- | ------------------------------------------------------------------------------ |
| **Circuit Switching** | Reserve a dedicated path first, then transfer data                             |
| **Message Switching** | Store-and-forward the _entire_ message hop by hop                              |
| **Packet Switching**  | Break the message into small packets, forward in parallel over multiple routes |

> Message switching is the historical **bridge** between circuit switching and packet switching — direct exam questions on it are less common than on circuit/packet switching, but understanding it is essential to understanding _why_ packet switching works the way it does.

---

## 2. Timeline: How Switching Evolved

```
1876 ─────────────────────────────────────────────► Present
  │                    │                    │
Circuit Switching   Message Switching   Packet Switching
(Telephone era)     (~1960s onward)     (Modern networks —
                     Store & Forward     Datagram + Virtual
                     on hard disk → RAM  Circuit)
```

- **Circuit switching** was designed for **telephone networks**, where devices are physically wired and a **connection must be established** before any communication.
- **Message switching** emerged around the **1960s**, introducing the **Store-and-Forward** concept.
- **Packet switching** refined message switching further by splitting messages into smaller **packets**, enabling **parallelism**.

---

## 3. Recap: Circuit Switching

Quick reminder before contrasting it with message switching:

1. Communication **begins with connection setup** — a dedicated path is created between sender and receiver.
2. Only **after** the connection is fully established does **data transfer** (continuous) begin.
3. Designed for **telephone networks** — devices are already interconnected; calling someone creates a **point-to-point path**.
4. Once the path exists, continuous data transfer (talking) happens over it.

**Key drawback:** Resources (bandwidth, channel, CPU) are **reserved for the entire session**, even during silence → inefficient use of the network.

---

## 4. Message Switching

### 4.1 Core Concept — Store & Forward

> Rather than reserving a path and sending data continuously, the **entire message** is transmitted **hop by hop**. Each intermediate node **stores** the full message, **processes** it to decide the next hop, then **forwards** it.

This is why message switching is described as having **"hop-by-hop delivery."**

No dedicated end-to-end connection is ever created — unlike circuit switching.

---

### 4.2 Structure & Flow Diagram

Suppose device **A** wants to send data to device **D**, through two intermediate switches/routers:

```
4-Node Path (Message Switching):

  [A] ──msg──► [Switch 1] ──msg──► [Switch 2] ──msg──► [D]
                 (store)              (store)
                 (process)            (process)
                 (forward)            (forward)
```

Step-by-step:

1. **A** transmits the **entire message** to the next hop.
2. That hop (switch/router) **stores** the complete message (historically on disk, later in RAM).
3. It **processes** the message to determine the best onward route.
4. It **forwards** the message to the next hop.
5. The next hop **repeats** the same store → process → forward cycle.
6. This continues until the message finally reaches **D**.

```
Circuit Switching flow:     [A]═══════reserved path═══════►[D]   (continuous)

Message Switching flow:     [A]──►[N1]──►[N2]──►[N3]──►[D]      (hop-by-hop,
                              store  store  store              stop-and-go)
                              +proc  +proc  +proc
```

---

### 4.3 Storage Medium Evolution

| Era                     | Storage Medium | Characteristic                                               |
| ----------------------- | -------------- | ------------------------------------------------------------ |
| Early message switching | **Hard Disk**  | Very slow — caused significant delay at every hop            |
| Later message switching | **RAM**        | Much faster — substantially reduced per-hop processing delay |

This shift from disk-based to RAM-based intermediate storage was a major factor in reducing overall latency in store-and-forward systems.

---

### 4.4 Reliability & Resource Reservation

**No reservation** of bandwidth, path, or CPU is made in advance — unlike circuit switching.

```
Circuit Switching:  Bandwidth reserved for ENTIRE session, even when idle
                     [═══ RESERVED ═══════════════════ IDLE ═══════════]

Message Switching:   No reservation — network resources used only
                     when actually forwarding a message, freeing up
                     capacity for other traffic in between hops
```

- Because there's no fixed reserved path, **different messages can take different routes** dynamically.
- This means the network as a whole is **utilized more efficiently** than in circuit switching.

---

### 4.5 Delay Analysis

Because every intermediate node must **store** and **process** the message before forwarding it, delay is inherently **higher** than circuit switching (once a circuit is set up, data flows continuously with no per-hop stop).

```
Delay sources per hop:
  Store time  → writing full message to disk/RAM
  Process time → deciding next route
  Forward time → transmitting to next node

Total delay = Σ (store + process + forward) across ALL hops
```

> **Trade-off:** Message switching sacrifices **speed/delay** in exchange for better **efficiency** (no wasted reserved resources).

---

### 4.6 Advantages & Disadvantages

| ✅ Advantages                                    | ❌ Disadvantages                                                |
| ------------------------------------------------ | --------------------------------------------------------------- |
| No reservation → better resource efficiency      | Higher delay (store + process at every hop)                     |
| Better network utilization (dynamic routing)     | Requires storage capacity at every node                         |
| No wasted bandwidth during "idle" periods        | Not suitable for real-time communication                        |
| Network traffic distributed across varied routes | Whole message must arrive before it can move on (no pipelining) |

---

## 5. Message Switching vs Circuit Switching

| Feature              | Circuit Switching                               | Message Switching                                        |
| -------------------- | ----------------------------------------------- | -------------------------------------------------------- |
| Connection setup     | Required before data transfer                   | **Not required** — no dedicated path                     |
| Resource reservation | Yes (bandwidth, CPU, path reserved for session) | **No reservation**                                       |
| Delivery style       | Continuous, over a fixed path                   | **Hop-by-hop**, store-and-forward                        |
| Efficiency           | Lower (resources reserved even when idle)       | **Higher** (no wasted reservation)                       |
| Delay                | Lower (once connected, data flows continuously) | **Higher** (every hop stores + processes)                |
| Route flexibility    | Fixed path for the whole session                | Can use **different routes** dynamically                 |
| Designed for         | Telephone networks                              | Early data networks (telegrams, early computer networks) |

---

## 6. Message Switching → Packet Switching

Packet switching is essentially a **refinement of message switching**:

- In message switching, the **entire message** is sent hop-by-hop using store-and-forward.
- In packet switching, that same message is **broken into smaller packets**, so that:
  - Multiple packets can travel **in parallel** (parallelism / pipelining).
  - Packets can take **different routes simultaneously**.
  - Overall **delay is reduced** and **efficiency increases** further.

```
Message Switching:  [ ENTIRE MESSAGE ]  ──►  hop ──► hop ──► hop  ──►  Destination
                       (one large block, sequential)

Packet Switching:   [P1][P2][P3][P4]   ──►  parallel hops/routes  ──►  Destination
                       (small independent units, concurrent)
```

| Parameter          | Message Switching       | Packet Switching                  |
| ------------------ | ----------------------- | --------------------------------- |
| Unit transmitted   | Whole message           | Small packets                     |
| Parallelism        | No                      | Yes                               |
| Delay              | Higher                  | Lower                             |
| Storage needed/hop | Large (full message)    | Small (per packet)                |
| Modern usage       | Historically superseded | Basis of the modern Internet (IP) |

---

## 7. 🔴 Security Considerations & Modern Relevance

Message switching, as a _dedicated network-layer technique_, is essentially **obsolete** — it isn't deployed today the way circuit or packet switching are (there's no live "message-switched network" to scan or probe the way you can with a hub/switch LAN). So there isn't a hands-on "attack the topology" lab here in the way star/mesh topologies have.

That said, the **store-and-forward** _concept_ is still very much alive in modern systems — and understanding it conceptually matters for security:

| Concept from Message Switching                  | Modern Equivalent                                                     | Security Implication                                                                                                                   |
| ----------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Store-and-forward, hop-by-hop                   | **SMTP email relaying** (MTAs store and forward mail between servers) | Email can be **read/logged at any intermediate mail server** unless end-to-end encrypted (PGP/S-MIME)                                  |
| Intermediate node stores full message           | **Message queues** (RabbitMQ, Kafka, SQS)                             | Queued messages sitting on disk/RAM at a broker are a **data-at-rest exposure point** if the broker is compromised                     |
| No dedicated path / dynamic routing             | **Internet routing (BGP)**                                            | Traffic can be **rerouted** through untrusted intermediate networks — relevant to **traffic interception / BGP hijacking** discussions |
| Full message held at each hop before forwarding | **Store-and-forward malware scanning gateways / mail gateways**       | Legitimate defensive use — but also a single point where a compromised gateway could **inspect or alter** all passing messages         |

**Key takeaway for a security student:** any "store-and-forward" design — whether it's 1960s message switching or a modern message broker — introduces a **window where data at rest, on an intermediate node, is exposed** to whoever controls that node. This is why **end-to-end encryption** (rather than just hop-by-hop/transport encryption) matters: it protects the _message contents_ even if an intermediate store is compromised.

---

## 8. 🧪 Practical Labs — Real-World Store-and-Forward Systems

Since there's no message-switched LAN to build in VirtualBox, these labs let you observe **store-and-forward behavior** in systems you already run (your own mail server / message queue / Metasploitable2), so you can see the concept in action safely on your own infrastructure.

### Lab 1 — Observe SMTP as a Store-and-Forward System

```bash
# On your own mail server / test SMTP setup (e.g., Postfix on a test VM)
# Watch the mail queue — this IS store-and-forward in action
mailq
# or
postqueue -p

# Each queued message is "stored" before being "forwarded" to the next hop (relay)
# Tail the mail log to see store -> process -> forward happen in real time
sudo tail -f /var/log/mail.log
```

### Lab 2 — Inspect a Message Queue (Kafka/RabbitMQ) as Modern Store-and-Forward

```bash
# If you run RabbitMQ locally for one of your MERN/PERN projects:
sudo rabbitmqctl list_queues name messages

# This shows messages sitting "at rest" in the broker — the modern
# analogue of a message-switching node storing data before forwarding it.
# Ask yourself: if this broker were compromised, could an attacker
# read every queued message? (Answer: yes, unless payloads are encrypted
# end-to-end before being queued.)
```

### Lab 3 — Compare Hop-by-Hop Delay: Traceroute as a Store-and-Forward Analogy

```bash
# Each hop in a traceroute is a router that receives, processes, and forwards
# the packet — conceptually similar to a message-switching node's job,
# just done at packet granularity instead of whole-message granularity.

traceroute 8.8.8.8
# Note the per-hop latency — this is the modern, packet-level version
# of the "delay per hop" idea discussed in section 4.5.

# Compare to a directly connected host (1 hop, minimal delay):
traceroute 192.168.56.101
```

### Lab 4 — Metasploitable2: Observe a Legacy Mail Service

```bash
# Metasploitable2 ships with an old Sendmail service — useful for
# observing (on your own isolated lab VM only) how a legacy MTA
# handles store-and-forward mail relaying.

nmap -p 25 -sV 192.168.56.101
# Confirms an SMTP service is listening

# Connect and inspect banner / relay behavior (own lab VM only)
nc 192.168.56.101 25
```

---

## 9. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║               MESSAGE SWITCHING — EXAM CHEAT SHEET                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  ORDER OF EVOLUTION                                                   ║
║  ─────────────────────────────────────────────────────────           ║
║  Circuit Switching  →  Message Switching  →  Packet Switching        ║
║  (Telephone era)       (~1960s, Store &      (Modern networks)       ║
║                         Forward)                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY CONCEPT                                                          ║
║  ─────────────────────────────────────────────────────────           ║
║  Message Switching = Store-and-Forward, Hop-by-Hop delivery          ║
║  Storage evolved:  Hard Disk (slow) → RAM (fast)                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  vs CIRCUIT SWITCHING                                                 ║
║  ─────────────────────────────────────────────────────────           ║
║  Connection setup:     Not required (message switching)              ║
║  Resource reservation: None (message switching)                      ║
║  Delay:                Higher (message switching)                    ║
║  Efficiency:            Higher (message switching)                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  vs PACKET SWITCHING                                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  Unit sent:  Whole message (msg switching) vs small packets (pkt)    ║
║  Parallelism: No (msg switching) vs Yes (pkt switching)              ║
║  Message switching = direct PREDECESSOR of packet switching          ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "Store and forward" strategy         → Message Switching            ║
║  "Hop by hop delivery"                → Message Switching            ║
║  "No reservation, better efficiency"  → Message Switching            ║
║  "Higher delay due to per-hop store"  → Message Switching            ║
║  "Predecessor of packet switching"    → Message Switching            ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 7)

- [ ] Packet Switching — Datagram approach
- [ ] Packet Switching — Virtual Circuit approach
- [ ] Comparison: Circuit vs Message vs Packet Switching (full summary table)

---

_Notes compiled from: Networking Course Lecture 06 — Switching Part (Message Switching)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
