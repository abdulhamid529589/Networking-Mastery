# Ethernet Frame Format — Comprehensive Study Guide

### Computer Networks | Data Link Layer | IEEE 802.3

---

## Table of Contents

1. [What is Ethernet?](#1-what-is-ethernet)
2. [Types of Ethernet](#2-types-of-ethernet)
3. [Ethernet Frame Format — Complete Structure](#3-ethernet-frame-format--complete-structure)
4. [Field-by-Field Deep Dive](#4-field-by-field-deep-dive)
5. [Who Adds Which Field?](#5-who-adds-which-field)
6. [Frame Size Calculations](#6-frame-size-calculations)
7. [MAC Address and Hop-by-Hop Delivery](#7-mac-address-and-hop-by-hop-delivery)
8. [Why Minimum Data is 46 Bytes — CSMA/CD Link](#8-why-minimum-data-is-46-bytes--csmacd-link)
9. [Ethernet vs Other Protocols](#9-ethernet-vs-other-protocols)
10. [Exam Questions & Answers](#10-exam-questions--answers)
11. [Quick Summary Cheatsheet](#11-quick-summary-cheatsheet)

---

## 1. What is Ethernet?

| Property              | Value                                                             |
| --------------------- | ----------------------------------------------------------------- |
| Layer                 | **Data Link Layer**                                               |
| Standard              | **IEEE 802.3**                                                    |
| Year Introduced       | **1983**                                                          |
| Medium Access Control | **CSMA/CD** (Carrier Sense Multiple Access / Collision Detection) |
| Topology              | **Bus** (primary); Star also used                                 |
| Data Rate Range       | 1 Mbps to **400 Gbps** (400 Gbps launched 2017)                   |
| Used In               | **LAN** (Local Area Network) technology                           |

Ethernet is one of the most important Data Link Layer protocols for competitive and university exams.

### Layer-wise Protocol Comparison

| Layer         | Popular Protocols                                                |
| ------------- | ---------------------------------------------------------------- |
| Application   | HTTP, SMTP, FTP, DNS                                             |
| Transport     | TCP, UDP                                                         |
| Network       | IPv4, IPv6                                                       |
| **Data Link** | **Ethernet (IEEE 802.3)**, HDLC, IEEE 802.11 (Wi-Fi), Token Ring |
| Physical      | Cables, signals                                                  |

### Why CSMA/CD in Ethernet?

Ethernet operates in a LAN where multiple devices share the same medium. Since it is a wired LAN:

- **No acknowledgements** are used — adding ACKs in LAN would generate too much overhead traffic
- Instead, **collision detection** is used — if two stations transmit simultaneously, the collision is detected and handled
- CSMA/CD allows multiple devices to share the channel efficiently

---

## 2. Types of Ethernet

### Naming Convention

```
[Speed] Base [Cable Length or Type]

Examples:
  10Base2   → 10 Mbps, Baseband, 200 m max
  10Base5   → 10 Mbps, Baseband, 500 m max
  100BaseT  → 100 Mbps, Baseband, Twisted pair
```

### Ethernet Generations

| Name                 | Speed        | Cable                | Notes                         |
| -------------------- | ------------ | -------------------- | ----------------------------- |
| 10Base2              | 10 Mbps      | Coaxial              | **Thin Ethernet**, 200 m max  |
| 10Base5              | 10 Mbps      | Coaxial              | **Thick Ethernet**, 500 m max |
| 10BaseT              | 10 Mbps      | Twisted Pair         | Standard Ethernet             |
| **Fast Ethernet**    | **100 Mbps** | Twisted Pair / Fiber | IEEE 802.3u                   |
| **Gigabit Ethernet** | **1 Gbps**   | Twisted Pair / Fiber | IEEE 802.3z/ab                |
| **10G Ethernet**     | **10 Gbps**  | Twisted Pair / Fiber | IEEE 802.3ae                  |
| **400G Ethernet**    | **400 Gbps** | Fiber                | Launched 2017                 |

**Exam tip:** These speeds and names are frequently asked as direct MCQs.

---

## 3. Ethernet Frame Format — Complete Structure

### Visual Diagram

```
┌────────────┬─────┬──────────────┬──────────────┬────────┬──────────────────┬─────┐
│  Preamble  │ SFD │ Destination  │   Source     │ Length │      Data        │ CRC │
│  7 bytes   │ 1 B │  Address     │  Address     │ 2 bytes│   46–1500 bytes  │ 4 B │
│  (56 bits) │(8b) │  6 bytes     │  6 bytes     │(16 bits│                  │(32b)│
│            │     │  (48 bits)   │  (48 bits)   │        │                  │     │
└────────────┴─────┴──────────────┴──────────────┴────────┴──────────────────┴─────┘
│←────────── Physical Layer adds ──────────────→│←── Data Link Layer adds ─────────→│
         (8 bytes total)                              (minimum 18 bytes overhead)
```

### Field Summary Table

| Field               | Size              | Added By        | Purpose                             |
| ------------------- | ----------------- | --------------- | ----------------------------------- |
| Preamble            | 7 bytes (56 bits) | Physical Layer  | Synchronization / alerting receiver |
| SFD                 | 1 byte (8 bits)   | Physical Layer  | Signals start of actual frame       |
| Destination Address | 6 bytes (48 bits) | Data Link Layer | MAC address of next-hop receiver    |
| Source Address      | 6 bytes (48 bits) | Data Link Layer | MAC address of sender               |
| Length              | 2 bytes (16 bits) | Data Link Layer | Length of data field                |
| Data (Payload)      | 46–1500 bytes     | Data Link Layer | Actual data being transmitted       |
| CRC                 | 4 bytes (32 bits) | Data Link Layer | Error detection                     |

---

## 4. Field-by-Field Deep Dive

### Field 1: Preamble (7 bytes = 56 bits)

**Pattern:** `10101010 10101010 10101010 10101010 10101010 10101010 10101010`

All 56 bits follow the alternating `1010...` pattern.

**Purpose:**

- Alerts the receiver that a frame is coming
- Allows the receiver to synchronize its clock with the sender
- Was added later (originally only SFD existed) when network usage increased and synchronization became critical

**Added by:** Physical Layer

**Key point:** This is a constant, fixed bit pattern — it never carries data.

---

### Field 2: SFD — Start Frame Delimiter (1 byte = 8 bits)

**Pattern:** `10101011`

Notice the last two bits are `11` instead of `10` — this is intentional.

**Purpose:**

- Signals the **end of preamble** and the **start of the actual Ethernet frame**
- Acts like a "flag" saying: "The real frame starts after me"
- Assumes this exact bit pattern will never appear in data

**Added by:** Physical Layer

**Exam trick:** Preamble ends with `...10` pattern; SFD ends with `...11` — the `11` at the end is the delimiter marker.

---

### Field 3: Destination Address (6 bytes = 48 bits)

This is the **MAC address** (also called Physical Address or NIC Address) of the **next-hop device**, not necessarily the final destination.

**Important:** In Data Link Layer, delivery is **hop-by-hop** (node-to-node), NOT end-to-end. So this address changes at every hop. (See Section 7 for detailed explanation.)

**Types of MAC addresses:**

- **Unicast** — one specific device
- **Multicast** — group of devices
- **Broadcast** — `FF:FF:FF:FF:FF:FF` (all devices on segment)

---

### Field 4: Source Address (6 bytes = 48 bits)

**MAC address of the sending device** — specifically the device sending _this hop_ of the frame.

This also changes at every hop. When a router forwards a frame, it replaces the Source Address with its own MAC address.

---

### Field 5: Length (2 bytes = 16 bits)

**Range:** 0 to 2^16 − 1 = 0 to **65,535**

Specifies the **length of the Data field** in bytes.

**Why needed?**

- Ethernet frames are **variable length** (data is 46–1500 bytes)
- The receiver needs to know where the data ends and CRC begins
- In fixed-length protocols, length fields aren't needed; in variable-length (like Ethernet), they are

**Compare with IPv4:** IPv4 has a Total Length field for the same reason — variable-length packets need length information.

**Note:** In newer versions of Ethernet (802.3), this field can also carry an **EtherType** value (if value > 1500) identifying the upper-layer protocol (e.g., 0x0800 = IPv4, 0x0806 = ARP, 0x86DD = IPv6).

---

### Field 6: Data / Payload (46–1500 bytes)

This is the **actual data being transmitted** — the payload from the upper layer (typically an IP packet).

**Minimum: 46 bytes**
**Maximum: 1500 bytes**

**Why minimum 46 bytes?** → This is directly linked to CSMA/CD's collision detection requirement (see Section 8 for full explanation).

**What if data is less than 46 bytes?**
→ **Padding** is added to bring the data field up to 46 bytes. The padding bytes are zeros and are ignored by the receiver.

**The 1500-byte maximum** is called the **MTU — Maximum Transmission Unit** of Ethernet.

---

### Field 7: CRC — Cyclic Redundancy Check (4 bytes = 32 bits)

Also called **FCS — Frame Check Sequence**.

**Purpose:** Error detection — allows the receiver to check if the frame was corrupted during transmission.

**How it works:**

1. Sender runs a mathematical algorithm on the frame data → generates a 32-bit CRC value
2. Sender appends CRC to the frame
3. Receiver runs the same algorithm on the received data
4. If receiver's calculated CRC ≠ received CRC → error detected → frame discarded

**Important:** CRC detects errors but does **not** correct them. Error correction requires protocols like HDLC or ARQ at higher layers.

**CRC is NOT an encryption** — it's purely for error detection.

---

## 5. Who Adds Which Field?

This is a very common exam question.

```
PHYSICAL LAYER adds:
  ├── Preamble (7 bytes)
  └── SFD (1 byte)
  Total: 8 bytes

DATA LINK LAYER adds:
  ├── Destination Address (6 bytes)
  ├── Source Address (6 bytes)
  ├── Length (2 bytes)
  ├── Data / Payload (46–1500 bytes)
  └── CRC (4 bytes)
  Overhead (excluding data): 18 bytes
```

---

## 6. Frame Size Calculations

This section is extremely important for numerical exam questions.

### Component Sizes

```
Preamble          =  7 bytes  (Physical Layer)
SFD               =  1 byte   (Physical Layer)
─────────────────────────────
Physical Layer    =  8 bytes total

Destination Addr  =  6 bytes
Source Addr       =  6 bytes
Length            =  2 bytes
CRC               =  4 bytes
─────────────────────────────
DLL Overhead      = 18 bytes (excluding data)

Data (minimum)    = 46 bytes
Data (maximum)    = 1500 bytes
```

### Minimum Frame Size Calculations

```
Case 1: Data Link Layer fields only (most common exam answer)
  = DLL Overhead + Minimum Data
  = 18 + 46
  = 64 bytes  ← Most frequently asked answer

Case 2: Including Physical Layer fields
  = Physical Layer + DLL Overhead + Minimum Data
  = 8 + 18 + 46
  = 72 bytes

Summary:
  Minimum frame (DLL only)  = 64 bytes
  Minimum frame (with PHY)  = 72 bytes
```

### Maximum Frame Size Calculations

```
Case 1: Data Link Layer fields only
  = DLL Overhead + Maximum Data
  = 18 + 1500
  = 1518 bytes  ← Most frequently asked answer

Case 2: Including Physical Layer fields
  = Physical Layer + DLL Overhead + Maximum Data
  = 8 + 18 + 1500
  = 1526 bytes

Summary:
  Maximum frame (DLL only)  = 1518 bytes
  Maximum frame (with PHY)  = 1526 bytes
```

### Quick Reference Table

| Scenario              | Minimum      | Maximum        |
| --------------------- | ------------ | -------------- |
| Data only             | 46 bytes     | 1500 bytes     |
| DLL frame (no PHY)    | **64 bytes** | **1518 bytes** |
| Full frame (with PHY) | 72 bytes     | 1526 bytes     |

### Memory Formula

```
Frame (DLL) = Dest(6) + Src(6) + Length(2) + Data(46 to 1500) + CRC(4)
            = 18 + Data
Min = 18 + 46  = 64 bytes
Max = 18 + 1500 = 1518 bytes

Full frame = 8 (PHY) + Frame (DLL)
Min = 8 + 64  = 72 bytes
Max = 8 + 1518 = 1526 bytes
```

---

## 7. MAC Address and Hop-by-Hop Delivery

### The Key Concept

| Layer               | Delivery Type                          | Address Used                           |
| ------------------- | -------------------------------------- | -------------------------------------- |
| Network Layer (IP)  | **End-to-end** (Source to Destination) | IP Address — does NOT change           |
| **Data Link Layer** | **Hop-by-hop** (node to node)          | **MAC Address — CHANGES at every hop** |

### Example: Source → X1 → X2 → Destination

```
Network:
Source ──────── X1 ──────── X2 ──────── Destination
(IP: S)      (Router)    (Router)        (IP: D)
```

#### Hop 1: Source → X1

```
Source Address:      MAC of Source
Destination Address: MAC of X1

IP Source:      IP of Source   ← doesn't change (Network Layer)
IP Destination: IP of Dest     ← doesn't change (Network Layer)
```

But how does Source know the MAC of X1?
→ Source knows X1's **IP address** (it's the default gateway)
→ Source uses **ARP (Address Resolution Protocol)** to discover X1's MAC from its IP
→ ARP request is broadcast; X1 replies with its MAC

#### Hop 2: X1 → X2

```
Source Address:      MAC of X1      ← changed!
Destination Address: MAC of X2      ← changed!
```

#### Hop 3: X2 → Destination

```
Source Address:      MAC of X2      ← changed!
Destination Address: MAC of Dest    ← changed!
```

### Key Rules

1. **IP addresses never change** throughout the journey (Network Layer responsibility)
2. **MAC addresses change at every hop** (Data Link Layer responsibility)
3. **ARP** is used to resolve IP → MAC at each hop
4. Data Link Layer's job: deliver from _this node_ to _next node_ only

---

## 8. Why Minimum Data is 46 Bytes — CSMA/CD Link

This is the most important theoretical explanation in Ethernet Frame Format.

### CSMA/CD Collision Detection Requirement

For a station to successfully detect a collision, it must **still be transmitting** when the collision signal arrives back at it.

**Condition:**

```
Transmission Time ≥ 2 × Propagation Delay

or equivalently:

Tt ≥ 2 × Tp
```

**Why 2 × Tp?**

```
Station A ─────────────────────────────────── Station B
          ←────── Propagation Delay (Tp) ────→

Worst case scenario:
  A starts transmitting
  Signal travels toward B (takes Tp time)
  Just before signal reaches B, B starts transmitting
  Collision occurs near B
  Collision signal travels back to A (takes another Tp)
  Total time for A to learn about collision = 2 × Tp

  A must still be transmitting at this point to detect it
  → Tt must be ≥ 2 × Tp
```

### Connecting to 46-Byte Minimum

The standard 10 Mbps Ethernet was designed for up to 2500 m with 4 repeaters:

- Maximum propagation delay = 25.6 µs
- Round trip time (2 × Tp) = 51.2 µs

At 10 Mbps:

```
Minimum frame bits = Transmission Rate × Round Trip Time
                   = 10 × 10^6 bits/sec × 51.2 × 10^-6 sec
                   = 512 bits
                   = 64 bytes total frame

64 bytes total − 18 bytes overhead = 46 bytes minimum data
```

Therefore: **Minimum data = 46 bytes** ensures Transmission Time ≥ 2 × Propagation Time for collision detection.

If data < 46 bytes → Padding is added to reach 46 bytes.

---

## 9. Ethernet vs Other Protocols

### Ethernet vs IEEE 802.11 (Wi-Fi)

| Property           | Ethernet (802.3)     | Wi-Fi (802.11)     |
| ------------------ | -------------------- | ------------------ |
| Medium             | Wired (copper/fiber) | Wireless           |
| MAC protocol       | CSMA/**CD**          | CSMA/**CA**        |
| Collision handling | Detect after         | Avoid before       |
| Acknowledgements   | Not used (LAN)       | Used (RTS/CTS/ACK) |
| Topology           | Bus / Star           | Star (via AP)      |

### PDU Names by Layer

| Layer                   | Data Unit Name    |
| ----------------------- | ----------------- |
| Application / Transport | Message / Segment |
| **Network**             | **Packet**        |
| **Data Link**           | **Frame**         |
| Physical                | Bits              |

Always use "frame" when discussing Ethernet — not "packet."

---

## 10. Exam Questions & Answers

**Q1: What is Ethernet? Which layer does it belong to?**

Ethernet is a Data Link Layer protocol defined in IEEE 802.3 (1983). It is a LAN technology that uses CSMA/CD for medium access control.

---

**Q2: What are the fields of the Ethernet frame and their sizes?**

```
Preamble:             7 bytes   (Physical Layer)
SFD:                  1 byte    (Physical Layer)
Destination Address:  6 bytes   (MAC of next-hop receiver)
Source Address:       6 bytes   (MAC of sender)
Length:               2 bytes   (length of data field)
Data:                 46–1500 bytes (payload)
CRC:                  4 bytes   (error detection)
```

---

**Q3: What is the minimum and maximum size of an Ethernet frame?**

- Minimum (DLL only): **64 bytes** (18 overhead + 46 data)
- Maximum (DLL only): **1518 bytes** (18 overhead + 1500 data)
- Minimum (including Physical Layer): 72 bytes
- Maximum (including Physical Layer): 1526 bytes

---

**Q4: Why is the minimum data size 46 bytes?**

For CSMA/CD to detect collisions, the transmission time must be at least 2 × propagation delay (round trip time). For 10 Mbps Ethernet with a maximum network span, this requires a minimum frame of 64 bytes. Subtracting 18 bytes of overhead leaves 46 bytes minimum for data. If actual data is less than 46 bytes, padding is added.

---

**Q5: What is Preamble and why is it used?**

Preamble is a 7-byte field with the alternating bit pattern `10101010...` (56 bits). It is added by the Physical Layer to alert and synchronize the receiver before the actual frame arrives. Originally only SFD was used; Preamble was added later as network usage grew.

---

**Q6: What is SFD?**

SFD (Start Frame Delimiter) is a 1-byte field with bit pattern `10101011`. It marks the end of the preamble and the beginning of the actual Ethernet frame. The last two bits `11` distinguish it from the preamble's `10` pattern.

---

**Q7: Does the MAC address change during transmission? Does the IP address?**

- **MAC address: YES** — changes at every hop because the Data Link Layer is responsible for node-to-node (hop-by-hop) delivery
- **IP address: NO** — remains constant throughout the journey because the Network Layer is responsible for end-to-end delivery

---

**Q8: How does a source know the MAC address of the next-hop router?**

The source knows the next-hop's IP address (default gateway). It uses **ARP (Address Resolution Protocol)** to discover the corresponding MAC address. ARP sends a broadcast request asking "Who has IP address X?" and the device with that IP replies with its MAC address.

---

**Q9: What is the purpose of CRC in Ethernet?**

CRC (Cyclic Redundancy Check) is a 4-byte field used for **error detection**. The sender computes a 32-bit CRC value over the frame and appends it. The receiver recalculates CRC and compares — if they differ, the frame is corrupted and discarded.

---

**Q10: What is 10Base2?**

10Base2 refers to Thin Ethernet: 10 Mbps speed, Baseband signaling, 200 m maximum cable length. Uses coaxial cable.

---

**Q11: Which fields does the Physical Layer add vs the Data Link Layer?**

- **Physical Layer adds:** Preamble (7 bytes) + SFD (1 byte) = 8 bytes total
- **Data Link Layer adds:** All remaining fields (Destination Address, Source Address, Length, Data, CRC)

---

**Q12: What is the MTU of Ethernet?**

MTU (Maximum Transmission Unit) = **1500 bytes** — the maximum size of the data/payload field in an Ethernet frame.

---

**Q13: A data payload is 30 bytes. What happens?**

Since 30 bytes < 46 bytes (minimum), **padding** is added to bring the data field to 46 bytes. The total minimum DLL frame size remains 64 bytes.

---

**Q14: Calculate the total frame size if data is 500 bytes (including Physical Layer fields).**

```
Preamble + SFD    =  8 bytes
Destination Addr  =  6 bytes
Source Addr       =  6 bytes
Length            =  2 bytes
Data              =  500 bytes
CRC               =  4 bytes
─────────────────────────────
Total             =  526 bytes
```

---

## 11. Quick Summary Cheatsheet

```
ETHERNET FRAME FORMAT — QUICK REFERENCE
═══════════════════════════════════════════════════════════════

Standard:     IEEE 802.3 (1983)
Layer:        Data Link Layer
MAC Method:   CSMA/CD
Topology:     Bus (primary)
Speed Range:  1 Mbps → 400 Gbps

FRAME STRUCTURE:
┌──────────┬─────┬──────┬──────┬────────┬──────────────┬─────┐
│Preamble  │ SFD │ Dest │ Src  │ Length │    Data      │ CRC │
│ 7 bytes  │ 1 B │  6 B │  6 B │  2 B   │ 46–1500 B    │ 4 B │
│ PHY adds │PHY  │      │      │        │              │     │
└──────────┴─────┴──────┴──────┴────────┴──────────────┴─────┘
 ←── 8 B ──→      ←────────────── 18 B overhead ──────────────→

PREAMBLE:   10101010... (7 bytes, alternating, PHY layer)
SFD:        10101011    (1 byte, ends with 11, PHY layer)

KEY SIZES:
  Minimum data:      46 bytes  (padding if less)
  Maximum data:    1500 bytes  (= Ethernet MTU)
  DLL overhead:      18 bytes  (dest + src + length + CRC)
  Min DLL frame:     64 bytes  (18 + 46) ← exam answer
  Max DLL frame:   1518 bytes  (18 + 1500) ← exam answer
  Min full frame:    72 bytes  (8 PHY + 64 DLL)
  Max full frame:  1526 bytes  (8 PHY + 1518 DLL)

MAC ADDRESS:
  Size:   48 bits = 6 bytes
  Type:   Physical / NIC / Hardware address
  Changes at every hop (hop-by-hop delivery)
  IP address does NOT change (end-to-end delivery)
  ARP: resolves IP → MAC at each hop

WHY MIN 46 BYTES?
  CSMA/CD requires: Tt ≥ 2 × Tp
  At 10 Mbps: minimum 64 bytes total frame
  64 − 18 (overhead) = 46 bytes data

CRC: 4 bytes, 32 bits, error DETECTION only

ETHERNET TYPES:
  10Base2  → 10 Mbps, Thin Ethernet, 200 m
  10Base5  → 10 Mbps, Thick Ethernet, 500 m
  Fast     → 100 Mbps
  Gigabit  → 1 Gbps
  10G      → 10 Gbps
  400G     → 400 Gbps (2017)
```

---

_Topic: Ethernet Frame Format | Subject: Computer Networks | Data Link Layer — IEEE 802.3_
