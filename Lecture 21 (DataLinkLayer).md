# 🔗 Data Link Layer — Functionalities & Responsibilities

### Cybersecurity Student Notes | Networking Course — Lecture 21

> **Source:** Gate Smashers — Data Link Layer (OSI Model)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Where Data Link Layer Sits in OSI](#1-where-data-link-layer-sits-in-osi)
2. [The Core Idea: Hop-to-Hop, Not Source-to-Destination](#2-the-core-idea-hop-to-hop-not-source-to-destination)
3. [Responsibility 1 — Hop-to-Hop (Node-to-Node) Delivery](#3-responsibility-1--hop-to-hop-node-to-node-delivery)
4. [Responsibility 2 — Flow Control](#4-responsibility-2--flow-control)
5. [Responsibility 3 — Error Control](#5-responsibility-3--error-control)
6. [Responsibility 4 — Access Control](#6-responsibility-4--access-control)
7. [Responsibility 5 — Physical (MAC) Addressing](#7-responsibility-5--physical-mac-addressing)
8. [Responsibility 6 — Framing](#8-responsibility-6--framing)
9. [The Train Analogy — Full Recap](#9-the-train-analogy--full-recap)
10. [Data Link Layer vs Transport/Network Layer — Comparison](#10-data-link-layer-vs-transportnetwork-layer--comparison)
11. [🔴 Security Considerations & Modern Relevance](#11--security-considerations--modern-relevance)
12. [🧪 Practical Labs — Your Setup](#12--practical-labs--your-setup)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. Where Data Link Layer Sits in OSI

```
 ┌───────────────────────┐
 │   Application Layer    │  (Layer 7)
 ├───────────────────────┤
 │   Presentation Layer   │  (Layer 6)
 ├───────────────────────┤
 │   Session Layer        │  (Layer 5)
 ├───────────────────────┤
 │   Transport Layer      │  (Layer 4)
 ├───────────────────────┤
 │   Network Layer        │  (Layer 3)
 ├───────────────────────┤
 │ ► Data Link Layer  ◄   │  (Layer 2)  ← THIS LAYER
 ├───────────────────────┤
 │   Physical Layer       │  (Layer 1)
 └───────────────────────┘
```

- **Data Link Layer = 2nd layer from the bottom**, sandwiched between **Network Layer** (above) and **Physical Layer** (below).
- It **takes data from the Network Layer** and **hands it to the Physical Layer**, which then actually transmits it across the medium so the receiver can get it.

---

## 2. The Core Idea: Hop-to-Hop, Not Source-to-Destination

This is the single most important concept of this lecture — and the point most students get wrong in exams:

> **The Data Link Layer's job is NOT to get data from the original source all the way to the final destination.**
> **That is the Network Layer's job.**
> **The Data Link Layer's job is to get data from ONE node to the NEXT node — one hop at a time.**

```
Network A                    (Routers = hops/nodes)                Network B

[A1] [A2] [A3] [A4] ──► [R1] ──► [R2] ──► [R3] ──► [B1] [B2] [B3] [B4]
                          hop 1     hop 2    hop 3

A4 → B1 overall journey = Network Layer's responsibility (source→destination)
A4 → R1  segment        = Data Link Layer's responsibility (hop→hop)
R1 → R2  segment        = Data Link Layer's responsibility (hop→hop)
R2 → R3  segment        = Data Link Layer's responsibility (hop→hop)
R3 → B1  segment        = Data Link Layer's responsibility (hop→hop)
```

Every single functionality discussed below (delivery, flow control, error control) follows this **same hop-to-hop philosophy** — this is the thread that ties the whole topic together.

---

## 3. Responsibility 1 — Hop-to-Hop (Node-to-Node) Delivery

### Concept

- **Within a single network (LAN)**, communication between two hosts can be achieved using **Data Link Layer alone** — the Network Layer isn't even required.
  - Example: A1 talking to A2, or A1 talking to A3, inside the same network → possible purely via **MAC address**, handled entirely at the Data Link Layer.
- **Across networks** (the internet is a "network of networks" — one network might be in India, another in Australia, another in New Zealand), data cannot jump directly from source to destination. It must pass through intermediate devices — **routers**, also called **hops** or **nodes**.

### Diagram

```
Network A                                                    Network B
┌─────────────────┐                                    ┌─────────────────┐
│  [A1][A2][A3][A4]│                                    │ [B1][B2][B3][B4] │
└────────┬─────────┘                                    └────────┬────────┘
         │                                                        │
         └──►[R1]──►[R2]──►[R3]──────────────────────────────────┘
              hop     hop    hop

A4 (sender) wants to send a message to B1 (receiver).

Data Link Layer's job:
  A4 → R1   : "which node does the message reach FIRST?" → Data Link Layer's job
  R1 → R2   : "from this node, which node next?"          → Data Link Layer's job
  R2 → R3   : same pattern                                  → Data Link Layer's job
  R3 → B1   : final hop into destination network            → Data Link Layer's job

The overall "A4 ultimately needs to reach B1" plan = Network Layer's job.
```

### Key Exam Point

> Data Link Layer does **NOT** know or care about the final destination in the big picture — it only handles **"where does this go next, one hop at a time."**

---

## 4. Responsibility 2 — Flow Control

### Concept

**Flow control = regulating the speed/rate at which data is sent**, so that the receiving node isn't overwhelmed.

```
If A4 generates messages VERY FAST and sends them to R1:

  [A4] ──fast fast fast──► [R1 buffer]
                              ┌──────────┐
                              │▓▓▓▓▓▓▓▓▓▓│ ← buffer fills up
                              └──────────┘
                              Old/unprocessed messages get dropped
                              once buffer capacity is exceeded
                              (similar to a phone's message inbox —
                              if messages flood in, old ones may be
                              lost/overwritten once storage is full)
```

If the flow isn't controlled, the receiving node's buffer overflows and data is lost.

### Protocols Used for Flow Control (Data Link Layer)

| Protocol             | Core Idea (to be covered in depth later)                                                             |
| -------------------- | ---------------------------------------------------------------------------------------------------- |
| **Stop-and-Wait**    | Sender sends one frame, waits for acknowledgment before sending the next                             |
| **Go-Back-N**        | Sender can send multiple frames (a "window"), but if an error occurs, resends from that frame onward |
| **Selective Repeat** | Sender resends **only** the specific frame(s) that had an error, not the whole window                |

### ⚠️ Critical Distinction: Data Link Layer Flow Control vs Transport Layer Flow Control

This is explicitly called out as a point where students commonly go wrong:

| Layer               | Flow control scope                    | What it checks                                                                                         |
| ------------------- | ------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| **Transport Layer** | **Source → Destination** (end-to-end) | Capacity/window size of the **final destination machine** (e.g., B1)                                   |
| **Data Link Layer** | **Hop → Hop** (node → node)           | Capacity/buffer of the **immediate next node** (e.g., R1's buffer capacity), not the final destination |

> Data Link Layer controls flow **at every individual node along the path**, not just once at the very end. The message still has to physically traverse every hop — so every hop's capacity matters.

---

## 5. Responsibility 3 — Error Control

### Concept

**Error control = detecting (and correcting) errors that occur during transmission** — e.g., a bit that was `0` flips to `1` during transmission.

- **Single bit error** → only one bit changes.
- **Burst error** → multiple bits change.

### Why Detect Errors at Every Hop (Not Just at the Final Destination)?

This is presented as a key efficiency argument:

```
BAD approach (error checked only at final destination B1):

[A4] ──►[R1]──►[R2]──►[R3]──►[B1]  ← error found HERE (after full journey!)
                                     B1 sends "ACK: error, please resend"
                                     Message must travel the ENTIRE path again
                                     = highly time-consuming, wastes bandwidth
                                     across every hop it already crossed


GOOD approach (Data Link Layer checks error at EACH hop):

[A4] ──►[R1]  ← error detected HERE already!
              R1 immediately tells A4 to resend
              Message doesn't need to travel further before the
              problem is caught = much more efficient
```

> **This is why error checking happens at the Data Link Layer (hop-by-hop) in addition to the Transport Layer (end-to-end)** — catching an error early, at the nearest hop, avoids wasting the time/bandwidth of sending a corrupted message all the way to the final destination first.

### Methods Used

| Method                            | Used At                                             |
| --------------------------------- | --------------------------------------------------- |
| **CRC** (Cyclic Redundancy Check) | **Data Link Layer** (primary method discussed here) |
| **Checksum**                      | **Transport Layer**                                 |
| Parity bits                       | Basic error detection (supplementary)               |
| Hamming Code                      | Error detection **and correction**                  |

---

## 6. Responsibility 4 — Access Control

### Concept

**Access control = deciding who gets to use the shared communication channel, and when** — relevant whenever multiple devices share the same physical medium.

```
Shared medium ("thick wire" — high bandwidth shared channel):

  [B1]───┬───[B2]───┬───[B3]───┬───[B4]
         └───────────┴──────────┘
              shared channel

If B1 and B4 BOTH start sending messages AT THE SAME TIME:

  [B1] ──msg──►
                    ╳  COLLISION — both messages destroyed/corrupted
  [B4] ──msg──►

Neither message is usable after a collision.
```

Access control ensures **only one device transmits on the shared channel at a time** (or otherwise manages/coordinates access), while the others simply receive.

### Methods/Protocols Used

| Method                     | Description                                                                           |
| -------------------------- | ------------------------------------------------------------------------------------- |
| **CSMA/CD**                | Carrier Sense Multiple Access with **Collision Detection** — used in classic Ethernet |
| **ALOHA**                  | Pure ALOHA — send whenever ready, detect collisions after the fact                    |
| **Slotted ALOHA**          | Improvement over pure ALOHA — transmission only allowed in defined time slots         |
| **Token Ring / Token Bus** | A "token" is passed around; only the device holding the token may transmit            |

> Access control is required specifically **within a network** (LAN) — this is another Data Link Layer function that doesn't need the Network Layer at all.

---

## 7. Responsibility 5 — Physical (MAC) Addressing

### Concept

- **Physical Address / MAC Address** = a **48-bit address**.
- It is **fixed/constant** — assigned to the Network Interface Card (NIC) and doesn't change based on which network you connect to.
- **Within one network**, devices can communicate using MAC addresses alone (e.g., A1 talking to A3).
- **Across different networks**, MAC address **cannot** be used for addressing — this is where **logical addressing (IP address)**, handled at the Network Layer, becomes necessary instead.

```
MAC Address (Data Link Layer)        IP / Logical Address (Network Layer)
────────────────────────────         ──────────────────────────────────────
48-bit, fixed/constant                 Can change (based on network you join)
Identifies the physical NIC            Identifies your position/network
Works WITHIN a single network          Works ACROSS different networks
```

> **Note:** Having two NIC cards from the **same vendor** with duplicate/reissued MAC addresses is not something any legitimate vendor does — but two NIC cards from **different vendors** in one machine is perfectly normal/legal (this is just clarifying that MAC address assignment is vendor-controlled and expected to be globally unique).

---

## 8. Responsibility 6 — Framing

### Concept

- Data Link Layer's terminology for its data unit is a **Frame** (this process is called **Framing**).
- Compare with other layers:

```
Network Layer   → data unit = "Packet"
Data Link Layer → data unit = "Frame"
```

- When packets arrive from the Network Layer, the Data Link Layer:
  1. Divides/organizes them into **fixed-size units**.
  2. Adds a **Header** (front) and a **Tail** (end) around the data.
  3. Passes the resulting **Frame** down to the Physical Layer.

```
 ┌────────┬─────────────────────────┬────────┐
 │ HEADER │        DATA (Payload)    │  TAIL  │
 └────────┴─────────────────────────┴────────┘
              = one Frame
```

**Purpose of framing:** reliability — ensures data is properly organized and delivered correctly to the next hop, with metadata (header/tail) supporting error detection, addressing, etc.

---

## 9. The Train Analogy — Full Recap

The lecture uses a **coal train** analogy to tie everything together:

```
       HEADER                    FRAMES (coaches)                  TAIL
      (Engine)     [Coach 1][Coach 2][Coach 3][Coach 4]   (Man with green flag)
         🚂 ───────[▭][▭][▭][▭]──────────────────────────────────── 🚩

Train journey: Delhi → Agra → Gwalior → Mumbai

  Delhi → Agra     = Data Link Layer's responsibility (one hop)
  Agra → Gwalior   = Data Link Layer's responsibility (next hop)
  Gwalior → Mumbai = Data Link Layer's responsibility (final hop)

  "Delhi to Mumbai" as an overall journey = Network Layer's responsibility
```

- **Coaches** = Frames
- **Engine (front)** = Header
- **Man with green flag (end)** = Tail
- **Station-to-station movement** = hop-to-hop delivery (Data Link Layer)
- **The overall Delhi-to-Mumbai route** = Network Layer's job, NOT Data Link Layer's

---

## 10. Data Link Layer vs Transport/Network Layer — Comparison

| Functionality      | Data Link Layer (Layer 2)                      | Transport Layer (Layer 4) / Network Layer (Layer 3)         |
| ------------------ | ---------------------------------------------- | ----------------------------------------------------------- |
| **Delivery scope** | Hop-to-hop (node-to-node)                      | Source-to-destination (end-to-end) — Network/Transport      |
| **Flow control**   | Controls flow at **every intermediate node**   | Controls flow **only between source and final destination** |
| **Error control**  | **CRC** — checked at every hop                 | **Checksum** — checked once, end-to-end                     |
| **Addressing**     | MAC / Physical address (fixed, 48-bit)         | IP / Logical address (Network Layer)                        |
| **Data unit**      | Frame                                          | Packet (Network Layer) / Segment (Transport Layer)          |
| **Works within**   | A single network/LAN (no Network Layer needed) | Across multiple networks                                    |

---

## 11. 🔴 Security Considerations & Modern Relevance

Because the Data Link Layer operates hop-by-hop and relies heavily on **MAC addressing** and **shared-medium access**, it's a layer with several well-documented, standard attack/defense topics relevant to a security student:

| Data Link Layer Function                    | Related Security Topic                                                                           | Notes                                                                                                                                                                                                                                                   |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MAC Addressing**                          | **MAC Spoofing**                                                                                 | Since MAC addresses are meant to be fixed/unique, a host presenting a spoofed MAC can impersonate another device on the LAN — relevant to bypassing MAC-based access controls                                                                           |
| **Hop-to-hop delivery / MAC-IP mapping**    | **ARP Spoofing / ARP Cache Poisoning**                                                           | ARP maps IP (Network Layer) to MAC (Data Link Layer) _within_ a network — poisoning this mapping lets an attacker position themselves as a man-in-the-middle for hop-to-hop traffic (you already tested this concept in your Part 5 star-topology labs) |
| **Access Control (CSMA/CD, shared medium)** | **Collision-based DoS / jamming (wired); deauthentication attacks (wireless, 802.11 MAC layer)** | Deliberately flooding a shared medium or forcing repeated "access denied" conditions is a classic Layer 2 denial-of-service concept                                                                                                                     |
| **Framing**                                 | **Frame injection / VLAN hopping (802.1Q double-tagging)**                                       | Manipulating frame headers (e.g., VLAN tags) can let traffic cross network segments it shouldn't                                                                                                                                                        |
| **Flow control buffers**                    | **Buffer exhaustion at a node**                                                                  | Deliberately overwhelming a switch/router's buffer conceptually parallels the "old messages get dropped" scenario described in this lecture — relevant to understanding traffic-flooding DoS at Layer 2                                                 |
| **Error control (CRC)**                     | **CRC is for accidental errors, not tamper-proofing**                                            | CRC detects _unintentional_ bit errors — it is **not** a security/integrity mechanism against a deliberate attacker, who can recompute a valid CRC for altered data. Genuine integrity requires cryptographic checks (e.g., HMAC), not CRC              |

> **Key takeaway:** Almost every classic "Layer 2 attack" (ARP spoofing, MAC flooding, VLAN hopping) exists specifically _because_ the Data Link Layer trusts hop-local information (MAC addresses, frame headers) without cryptographic verification. This is exactly why Layer 2 security controls exist: **port security, DHCP snooping, Dynamic ARP Inspection (DAI), and IEEE 802.1X** on managed switches.

---

## 12. 🧪 Practical Labs — Your Setup

### Lab 1 — Observe MAC Addressing Directly

```bash
# View your NIC's fixed MAC (physical) address
ip link show eth0
# Compare with Metasploitable2's MAC by checking your ARP table after pinging it
ping -c 2 192.168.56.101
arp -a
```

### Lab 2 — Watch Hop-to-Hop Behavior via TTL / Traceroute

```bash
# Each hop is a Data Link Layer handoff to the next node
traceroute 8.8.8.8
# Compare to a same-network host — only 1 "hop" (no intermediate node needed)
traceroute 192.168.56.101
```

### Lab 3 — Observe Framing & Headers with Wireshark/tcpdump

```bash
# Capture raw frames and inspect Layer 2 headers (source MAC, dest MAC, EtherType)
sudo tcpdump -i eth0 -e -c 20
# The "-e" flag prints the Ethernet (Data Link Layer) header for each frame
```

### Lab 4 — ARP Spoofing as a Hop-to-Hop Trust Exploit (Own Lab Only)

```bash
# You already practiced this in Part 5 — worth re-running here with the
# Data Link Layer framing in mind: this attack works BECAUSE Data Link
# Layer addressing (MAC) is trusted locally without cryptographic proof.

sudo arpspoof -i eth0 -t 192.168.56.101 192.168.56.1 &
sudo tcpdump -i eth0 -e not arp
```

### Lab 5 — CRC vs Cryptographic Integrity (Conceptual Demo)

```bash
# Show that CRC only catches ACCIDENTAL corruption, not deliberate tampering
echo "original message" > msg.txt
crc32 msg.txt          # note the CRC value

# "Attacker" modifies the file AND can trivially recompute a matching CRC
# (CRC is NOT designed to resist intentional tampering)
echo "tampered message" > msg.txt
crc32 msg.txt           # a different but equally "valid" CRC — no cryptographic guarantee

# Compare with a cryptographic hash (this DOES detect tampering meaningfully
# when combined with a secret key, e.g. HMAC):
sha256sum msg.txt
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║             DATA LINK LAYER — EXAM CHEAT SHEET                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  POSITION IN OSI                                                      ║
║  ─────────────────────────────────────────────────────────           ║
║  Layer 2 — between Network Layer (L3) and Physical Layer (L1)        ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE PRINCIPLE                                                       ║
║  ─────────────────────────────────────────────────────────           ║
║  HOP-TO-HOP / NODE-TO-NODE delivery — NOT source-to-destination      ║
║  (Source-to-destination = Network Layer's job)                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  6 KEY RESPONSIBILITIES                                               ║
║  ─────────────────────────────────────────────────────────           ║
║  1. Hop-to-Hop Delivery     — node to node, not end to end           ║
║  2. Flow Control            — Stop&Wait, Go-Back-N, Selective Repeat ║
║  3. Error Control           — CRC (Data Link) vs Checksum (Transport)║
║  4. Access Control          — CSMA/CD, ALOHA, Slotted ALOHA, Token   ║
║  5. Physical (MAC) Address  — 48-bit, fixed, works within 1 network  ║
║  6. Framing                 — adds Header + Tail to Packet → Frame   ║
╠══════════════════════════════════════════════════════════════════════╣
║  FLOW CONTROL — LAYER COMPARISON                                      ║
║  ─────────────────────────────────────────────────────────           ║
║  Data Link Layer  → controls flow at EVERY intermediate node         ║
║  Transport Layer  → controls flow ONLY source ↔ final destination    ║
╠══════════════════════════════════════════════════════════════════════╣
║  ERROR CONTROL — LAYER COMPARISON                                     ║
║  ─────────────────────────────────────────────────────────           ║
║  Data Link Layer  → CRC, checked at EVERY hop (catches errors early) ║
║  Transport Layer  → Checksum, checked ONLY at final destination      ║
║  Why check twice? → Catching errors early (at nearest hop) avoids    ║
║                      resending the message across the WHOLE path     ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "Data unit of Data Link Layer"        → Frame                       ║
║  "Data unit of Network Layer"          → Packet                      ║
║  "Address used within a network"       → MAC / Physical Address      ║
║  "48-bit fixed address"                → MAC Address                 ║
║  "Handles collisions on shared medium" → Access Control (CSMA/CD)    ║
║  "Error detection method in Data Link" → CRC                         ║
║  "Error detection method in Transport" → Checksum                    ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 22)

- [ ] Stop-and-Wait Protocol — detailed working, timing diagrams
- [ ] Go-Back-N ARQ — sliding window, retransmission logic
- [ ] Selective Repeat ARQ — detailed comparison with Go-Back-N
- [ ] CRC — step-by-step calculation method
- [ ] CSMA/CD — detailed working and Ethernet collision handling

---

_Notes compiled from: Networking Course Lecture 21 — Data Link Layer (Functionalities)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
