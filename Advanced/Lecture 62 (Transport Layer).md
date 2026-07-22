# 🌐 Transport Layer — Responsibilities & Functionalities

### Networking Course — Lecture 62

> **Source:** Gate Smashers — Transport Layer

> **Topic Area:** OSI Model — Layer 4 (Transport Layer)

---

## 📑 Table of Contents

1. [Where Transport Layer Fits](#1-where-transport-layer-fits)
2. [Responsibility 1 — End-to-End / Port-to-Port / Process-to-Process Delivery](#2-responsibility-1--end-to-end--port-to-port--process-to-process-delivery)
3. [Responsibility 2 — Reliability (TCP vs UDP)](#3-responsibility-2--reliability-tcp-vs-udp)
4. [Responsibility 3 — Error Control (Checksum)](#4-responsibility-3--error-control-checksum)
5. [Responsibility 4 — Flow Control](#5-responsibility-4--flow-control)
6. [Responsibility 5 — Congestion Control](#6-responsibility-5--congestion-control)
7. [Responsibility 6 — Segmentation](#7-responsibility-6--segmentation)
8. [Responsibility 7 — Multiplexing & De-multiplexing](#8-responsibility-7--multiplexing--de-multiplexing)
9. [OSI Layer Comparison — What Each Layer Does](#9-osi-layer-comparison--what-each-layer-does)
10. [TCP vs UDP — Quick Comparison](#10-tcp-vs-udp--quick-comparison)
11. [Why This Matters for Cybersecurity](#11-why-this-matters-for-cybersecurity)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. Where Transport Layer Fits

**Transport Layer = Layer 4 in the OSI model.**

```
OSI Model (top to bottom):
  7 — Application Layer   ← data originates here (user/applications)
  6 — Presentation Layer
  5 — Session Layer
  4 — TRANSPORT LAYER     ← THIS LECTURE
  3 — Network Layer       ← passes data down to here
  2 — Data Link Layer
  1 — Physical Layer
```

**What transport layer does in one sentence:** Takes data from the Application Layer (user/applications) and passes it down to the Network Layer — while providing services like reliability, ordering, error checking, and flow control along the way.

---

## 2. Responsibility 1 — End-to-End / Port-to-Port / Process-to-Process Delivery

**This is the most fundamental responsibility of the transport layer.**

### What Each Lower Layer Does (for context)

| Layer               | Delivery Type                                      | Scope                                                       |
| ------------------- | -------------------------------------------------- | ----------------------------------------------------------- |
| **Data Link Layer** | Node-to-node / Hop-to-hop                          | Between two directly connected nodes                        |
| **Network Layer**   | Host-to-host / Machine-to-machine                  | Between two machines (using IP addresses)                   |
| **Transport Layer** | **End-to-end / Port-to-port / Process-to-process** | Between two specific applications/processes on two machines |

### Why Port Numbers Are Needed

```
Scenario:
  Machine A1 (Network A) wants to communicate with Machine B1 (Network B)

Network layer delivers the packet to B1's machine.
BUT: multiple applications may be running on B1
  → Web browser (port 80)
  → Email client (port 25)
  → FTP client (port 21)
  → Some custom app (port 120)

The transport layer uses PORT NUMBERS to direct the message
to the exact application/process it belongs to.
```

**Example:**

- A1 sends a mail via Gmail → uses **SMTP protocol → Port 25**
- The message goes to B1's machine (Network Layer's job)
- On B1, it is delivered to **port 120** (the target application) — not to the browser, not to FTP
- **This precise application-to-application routing = transport layer's job**

### Port Number Basics

- Port numbers are **16-bit numbers** (0 to 65535)
- Some are **well-known/reserved** (e.g., SMTP = 25, HTTP = 80, FTP = 21)
- Some are **registered port numbers**
- Some are **dynamic/private** (used temporarily by applications)

> **Key exam terminology — all three mean the same thing:**
>
> - End-to-end delivery
> - Port-to-port delivery
> - Process-to-process delivery

---

## 3. Responsibility 2 — Reliability (TCP vs UDP)

Transport layer uses **two primary protocols:**

| Protocol | Full Name                     | Nature                                |
| -------- | ----------------------------- | ------------------------------------- |
| **TCP**  | Transmission Control Protocol | Connection-oriented, reliable         |
| **UDP**  | User Datagram Protocol        | Connectionless, unreliable (but fast) |

### The Problem with Network Layer (IP)

- The IP protocol (Network Layer) is **unreliable** — it sends messages without any guarantee
- Messages can be lost, delayed, or arrive out of order
- This is similar to old postal mail — no guarantee of delivery or order

### How TCP Provides Reliability

**TCP guarantees three things:**

**1. All packets arrive (no loss):**

```
Sender sends: P1, P2, P3, P4
Receiver must receive: P1, P2, P3, P4
If any packet is missing → TCP detects it and requests retransmission
```

**2. In-order delivery:**

```
Sender sends in order: A → B → C → D (e.g., spelling "LIVE")
Receiver must receive: A, B, C, D → prints "LIVE"

If received out of order: E, V, I, L → prints "EVIL"
(completely different meaning!)

Transport layer ensures correct order is maintained.
```

**3. Connection-oriented (before sending any data):**

```
Like a telephone call:
  Step 1: Dial the number (connection setup / handshake)
  Step 2: Wait for connection to establish (dial tone = acknowledgment)
  Step 3: Start talking (data transfer)

TCP always sets up a connection BEFORE transferring data.
```

### When to Use UDP

- When reliability is NOT needed (speed is more important)
- Examples: live video streaming, online gaming, DNS lookups, VoIP
- Application layer handles any reliability logic if needed

---

## 4. Responsibility 3 — Error Control (Checksum)

**Transport layer detects errors using the CHECKSUM method.**

```
How checksum works:
  1. Sender calculates a checksum value from the data
  2. Sends the checksum along with the data
  3. Receiver receives the data + checksum
  4. Receiver independently recalculates the checksum on the received data
  5. Compares: if checksums match → no error ✅
               if checksums don't match → error detected ❌
```

### Transport Layer vs Data Link Layer Error Control

| Property       | Data Link Layer                                          | Transport Layer                                |
| -------------- | -------------------------------------------------------- | ---------------------------------------------- |
| **Method**     | CRC (Cyclic Redundancy Check)                            | Checksum                                       |
| **Scope**      | Every hop (node-to-node) — each intermediate node checks | End-to-end — only source and destination check |
| **Who checks** | Every intermediate node on the path                      | Only the final receiver                        |

> **Important distinction:** Data link layer checks for errors at every hop. Transport layer checks only at the final destination — this is **end-to-end** error control.

---

## 5. Responsibility 4 — Flow Control

**Flow control manages the rate at which the sender sends data so the receiver is not overwhelmed.**

**Methods used:** Stop-and-Wait, Go-Back-N (GBN), Selective Repeat (SR)

**Mechanism — Advertising the Window:**

```
Receiver → tells Sender: "My window (capacity) is X"

Window = how many packets + what size the receiver can handle at once

Receiver advertises TWO things:
  1. Maximum size of each message
  2. How many messages it can receive at once

Sender adjusts its transmission rate accordingly.
```

```
Without flow control:
  Fast sender → overwhelmed slow receiver → packets dropped → retransmission
  = wasted bandwidth, poor performance

With flow control:
  Sender knows receiver's capacity → sends accordingly → no overflow
```

---

## 6. Responsibility 5 — Congestion Control

**Congestion control prevents the network itself from being overwhelmed** (different from flow control, which is about the receiver's capacity).

```
Flow Control:     Protects the RECEIVER from being overwhelmed
Congestion Control: Protects the NETWORK from being overwhelmed
```

**Key algorithm:** **AIMD — Additive Increase Multiplicative Decrease**

- When network is not congested → slowly increase sending rate (additive increase)
- When congestion is detected → rapidly decrease sending rate (multiplicative decrease)

> Detailed algorithms (AIMD, Slow Start, etc.) are covered in dedicated later lectures.

---

## 7. Responsibility 6 — Segmentation

**Transport layer divides the continuous stream of data from the Application Layer into discrete segments.**

```
Data from Application Layer:
  Arrives as a CONTINUOUS STREAM of bits: 0101010101010101...
  (No defined boundaries — just an unbroken flow)

Transport Layer (TCP/UDP) does segmentation:
  Takes chunks of the stream
  Adds a HEADER (containing port numbers, sequence numbers, checksums, etc.)
  Creates a SEGMENT = Header + Data chunk
  Sends segment down to Network Layer
  Then takes the next chunk → creates another segment → sends → repeat
```

```
Why segmentation matters:
  → Network layer expects discrete units (packets), not raw streams
  → Segmentation makes addressing, error checking, and ordering possible
  → Each segment can take a different path and be reassembled in order at destination
```

> **Terminology note:** At the Transport Layer, the data unit is called a **segment** (TCP) or **datagram** (UDP). At the Network Layer, it becomes a **packet**. At the Data Link Layer, it becomes a **frame**.

---

## 8. Responsibility 7 — Multiplexing & De-multiplexing

**Multiplexing (at sender side) = Many-to-One**
**De-multiplexing (at receiver side) = One-to-Many**

### Multiplexing (Sender)

```
Multiple applications running on one machine:
  App 1 (Gmail/SMTP)     → sending data
  App 2 (Web browser)    → sending data
  App 3 (FTP client)     → sending data
  App 4 (Custom app)     → sending data

Machine has ONE network connection (one channel to the internet).

Transport layer MULTIPLEXES all these streams:
  → Takes data from multiple applications
  → Tags each with the correct source/destination port
  → Sends them all through the single channel
```

### De-multiplexing (Receiver)

```
One channel → multiple messages arriving continuously
Each message is tagged with a destination port number

Transport layer DE-MULTIPLEXES:
  → Reads destination port of each incoming segment
  → Delivers to the correct application/process

  Port 25  → delivered to email application
  Port 80  → delivered to web browser
  Port 120 → delivered to custom app
```

---

## 9. OSI Layer Comparison — What Each Layer Does

| Layer              | Delivery Scope                    | Unit               | Protocol Examples |
| ------------------ | --------------------------------- | ------------------ | ----------------- |
| **Data Link (L2)** | Node-to-node (hop-to-hop)         | Frame              | Ethernet, Wi-Fi   |
| **Network (L3)**   | Host-to-host (machine-to-machine) | Packet             | IP                |
| **Transport (L4)** | Process-to-process (port-to-port) | Segment / Datagram | TCP, UDP          |

---

## 10. TCP vs UDP — Quick Comparison

| Property               | TCP                                           | UDP                                     |
| ---------------------- | --------------------------------------------- | --------------------------------------- |
| **Full name**          | Transmission Control Protocol                 | User Datagram Protocol                  |
| **Connection type**    | Connection-oriented (handshake first)         | Connectionless (no setup)               |
| **Reliability**        | Reliable (guarantees delivery)                | Unreliable (best-effort)                |
| **Ordering**           | In-order delivery guaranteed                  | No ordering guarantee                   |
| **Error control**      | Yes (checksum + retransmission)               | Checksum only (no retransmission)       |
| **Flow control**       | Yes                                           | No                                      |
| **Congestion control** | Yes                                           | No                                      |
| **Speed**              | Slower (overhead of all above)                | Faster (minimal overhead)               |
| **Use cases**          | Web (HTTP/HTTPS), Email (SMTP), File transfer | Live streaming, DNS, VoIP, Gaming       |
| **Analogy**            | Registered post (tracked, guaranteed)         | Regular post (no tracking, may be lost) |

---

## 11. Why This Matters for Cybersecurity

- **Port scanning = transport layer reconnaissance:** Tools like `nmap` discover open ports by probing transport layer endpoints. Understanding ports is the foundation of network reconnaissance — every service runs on a specific port, and knowing which ports are open reveals what services/attack surface is exposed.

- **TCP handshake attacks:** TCP's three-way handshake (SYN → SYN-ACK → ACK) is directly exploited in **SYN flood attacks** — an attacker sends thousands of SYN packets without completing the handshake, exhausting the server's connection table and causing a DoS. This is one of the most common network-layer DoS techniques.

- **UDP-based amplification attacks:** Because UDP is connectionless and has no source verification, attackers can spoof source IPs to redirect large amplified responses to victims — **DNS amplification**, **NTP amplification**, and **SSDP amplification** (from Lecture 43) all exploit this UDP property.

- **Port numbers reveal application identity:** During a pentest, finding port 25 open tells you SMTP is running, port 22 = SSH, port 3306 = MySQL, etc. Misconfigurations on these ports are standard exploitation targets.

- **Checksum and integrity:** The transport layer's checksum provides basic integrity checking, but it is not cryptographically secure — it can be spoofed. This is why application-layer security (TLS/HTTPS) is needed on top: transport layer error control is for accidental corruption, not intentional tampering.

- **Multiplexing and session hijacking:** Because multiple sessions (identified by port pairs) flow through one connection, session hijacking attacks exploit transport layer identifiers — an attacker who can predict or sniff the TCP sequence numbers can inject packets into an existing session.

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║          TRANSPORT LAYER — EXAM CHEAT SHEET                          ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI LAYER:  Layer 4 — between Application (L7) and Network (L3)    ║
║  DATA UNIT:  Segment (TCP) / Datagram (UDP)                          ║
╠══════════════════════════════════════════════════════════════════════╣
║  7 RESPONSIBILITIES (MEMORISE ALL)                                    ║
║  ─────────────────────────────────────────────────────────          ║
║  1. END-TO-END DELIVERY                                              ║
║     = Port-to-port = Process-to-process                             ║
║     Uses PORT NUMBERS (16-bit) to identify applications             ║
║     e.g. SMTP = port 25, HTTP = port 80                             ║
║                                                                        ║
║  2. RELIABILITY (via TCP)                                             ║
║     → All packets delivered (no loss)                               ║
║     → In-order delivery (sequence numbers)                          ║
║     → Connection-oriented (handshake before data)                   ║
║     UDP = unreliable but fast (use when reliability not needed)     ║
║                                                                        ║
║  3. ERROR CONTROL                                                       ║
║     Method: CHECKSUM (end-to-end, only at destination)              ║
║     vs Data Link Layer: CRC (every hop)                             ║
║                                                                        ║
║  4. FLOW CONTROL                                                        ║
║     Protects: RECEIVER from being overwhelmed                       ║
║     Method: Advertising the window (receiver tells sender capacity) ║
║     Algorithms: Stop-and-Wait, Go-Back-N, Selective Repeat          ║
║                                                                        ║
║  5. CONGESTION CONTROL                                                  ║
║     Protects: NETWORK from being overwhelmed                        ║
║     Algorithm: AIMD (Additive Increase Multiplicative Decrease)     ║
║                                                                        ║
║  6. SEGMENTATION                                                        ║
║     Continuous data stream from Application Layer                   ║
║     → Divided into discrete SEGMENTS with headers                   ║
║     Header contains: port numbers, sequence numbers, checksum       ║
║                                                                        ║
║  7. MULTIPLEXING & DE-MULTIPLEXING                                      ║
║     Multiplexing (sender):   Many apps → one channel (many-to-one)  ║
║     De-multiplexing (receiver): one channel → correct app (one-to-many) ║
╠══════════════════════════════════════════════════════════════════════╣
║  TCP vs UDP ONE-LINER                                                   ║
║  TCP = reliable, ordered, connection-oriented, slower               ║
║  UDP = unreliable, unordered, connectionless, faster                ║
╠══════════════════════════════════════════════════════════════════════╣
║  LAYER DELIVERY COMPARISON                                              ║
║  Data Link  → Node-to-node (hop-to-hop)                             ║
║  Network    → Host-to-host (machine-to-machine) using IP            ║
║  Transport  → Process-to-process (port-to-port) ← THIS LAYER        ║
╠══════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICK FOR 7 RESPONSIBILITIES:                                   ║
║  "Every Reliable Engineer Controls Flow, Segments, Multiplexes"     ║
║   E nd-to-end                                                        ║
║   R eliability (TCP/UDP)                                             ║
║   E rror control (checksum)                                          ║
║   C ongestion control (AIMD)                                         ║
║   F low control (window)                                             ║
║   S egmentation                                                      ║
║   M ultiplexing / De-multiplexing                                    ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] TCP — Three-way handshake (SYN, SYN-ACK, ACK) in detail
- [ ] TCP Header — all fields explained
- [ ] UDP — Header, use cases, comparison with TCP
- [ ] Port numbers — Well-known, registered, dynamic ranges
- [ ] Flow control algorithms — Stop-and-Wait, Go-Back-N, Selective Repeat
- [ ] Congestion control — AIMD, Slow Start, Fast Retransmit

---

_Notes compiled from: Networking Course Lecture 62 — Transport Layer_

_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
