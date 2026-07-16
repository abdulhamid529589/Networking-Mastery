# 🔗 Networking Devices — Bridge

### Cybersecurity Student Notes | Networking Course — Lecture 12

> **Source:** Gate Smashers — Networking Devices: Bridge
> **Exam Relevance:** UGC NET (high) · PSU/Bank IT Officer exams (high) · GATE (low priority)
> **Topic Area:** Physical + Data Link Layer — Traffic Filtering Device

---

## 📑 Table of Contents

1. [What is a Bridge?](#1-what-is-a-bridge)
2. [Why "Bridge"? Connecting Two Different LANs](#2-why-bridge-connecting-two-different-lans)
3. [OSI Layer — Physical + Data Link](#3-osi-layer--physical--data-link)
4. [Forwarding on a Bridge](#4-forwarding-on-a-bridge)
5. [Filtering on a Bridge](#5-filtering-on-a-bridge)
6. [Static Bridge](#6-static-bridge)
7. [Dynamic (Transparent) Bridge — Learning Process](#7-dynamic-transparent-bridge--learning-process)
8. [Static vs Dynamic Bridge — Comparison](#8-static-vs-dynamic-bridge--comparison)
9. [Collision Domain of a Bridge](#9-collision-domain-of-a-bridge)
10. [Loop Prevention — Spanning Tree Protocol](#10-loop-prevention--spanning-tree-protocol)
11. [Summary Table](#11-summary-table)
12. [Why This Matters for Cybersecurity](#12-why-this-matters-for-cybersecurity)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. What is a Bridge?

A **bridge** is a networking device used to **connect two LANs**. Unlike a hub (which can also connect LANs but has no intelligence), a bridge introduces **address-based decision-making** for the first time in this device series.

> **Key restriction to remember:** Bridges are generally used to connect **exactly two** LANs (the word "two" is emphasized as an important exam point).

---

## 2. Why "Bridge"? Connecting Two Different LANs

The name comes from the real-world concept of a **bridge/flyover** — something that connects two separate areas.

A key exam point: a bridge can connect **two different types of LANs** — for example:

```
[Token Ring LAN]  ═══[Bridge]═══  [Token Bus / Ethernet LAN]
   (Ring topology)                  (Bus topology)
```

This means a bridge can join dissimilar LAN technologies (e.g., a Ring-topology network on one side and a Bus/Ethernet-topology network on the other), which a simple hub or repeater setup could not meaningfully bridge together.

---

## 3. OSI Layer — Physical + Data Link

**Bridges operate at both the Physical Layer AND the Data Link Layer.**

- Because of **data link layer** capability, a bridge **can check the MAC address** of a packet.
- This is the fundamental capability that separates a bridge from a repeater/hub (which are pure Layer 1 devices with no addressing awareness — Parts 10, 11).

```
Example setup:
   [M1][M2][M3][M4] ── P1 ──[BRIDGE]── P2 ── [M5][M6][M7][M8]
```

- `P1` and `P2` are the bridge's **ports/interfaces**.
- Every packet carries a **source MAC address** and a **destination MAC address** — the bridge inspects these to make its forwarding/filtering decision.

---

## 4. Forwarding on a Bridge

**Yes, bridges forward packets** — but the decision is now **intelligent**, based on MAC address, not automatic like a hub/repeater.

**Example: M1 sends to M6**

```
M1 (on P1 side) ──► [Bridge checks destination MAC: M6]
Bridge determines M6 is on the P2 side ──► forwards packet to P2
```

The bridge checks whether the destination MAC address is on the **same side** or the **opposite side**, and forwards accordingly.

---

## 5. Filtering on a Bridge

**This is the key new capability introduced by bridges** (not present in repeaters or hubs).

**"Filter" = block/stop a packet that doesn't need to go further.**

**Example: M1 sends to M3 (both on the same side, P1)**

```
M1 (P1 side) ──► [Bridge checks destination MAC: M3]
Bridge determines M3 is ALSO on the P1 side
Bridge does NOT forward the packet to P2 — it is FILTERED (blocked/stopped)
```

- Since M1 and M3 are on the **same side**, there's no need to send the packet across to the other LAN — doing so would create **unnecessary traffic**.
- This filtering capability exists **only because of the data link layer / MAC address checking** — a repeater or hub cannot do this at all.

---

## 6. Static Bridge

A **static bridge** maintains a table of **MAC addresses mapped to ports** — similar in concept to how a router maintains a routing table.

**Key characteristic: the table is filled in MANUALLY by the network administrator.**

```
Static Bridge Table (example):
┌────────────┬──────┐
│ MAC Address│ Port │
├────────────┼──────┤
│ M1         │ P1   │
│ M2         │ P1   │
│ M3         │ P1   │
│ M4         │ P1   │
│ M5         │ P2   │
│ M6         │ P2   │
│ M7         │ P2   │
│ M8         │ P2   │
└────────────┴──────┘
```

- The administrator manually types which machine is connected to which port.
- The bridge then uses this table to **forward** (e.g., M1 → M5 goes P1 to P2) or **filter** (e.g., M1 → M3 stays within P1, blocked from crossing).

### Problems with Static Bridges

| Problem                    | Why it's an issue                                                                                                                                                                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **MAC address changes**    | Devices can have multiple MAC addresses (e.g., separate Wi-Fi vs. wired interface addresses), and MAC addresses _can_ be changed (even though this isn't common). If a MAC address changes, the administrator must **manually update** the table entry. |
| **Interface/port changes** | If a machine is physically moved (e.g., unplugged from one lab and moved to another), its connected port changes (e.g., from P1 to P2), and the administrator must **manually retype** this mapping.                                                    |
| **No learning**            | The static bridge has **no ability to learn or adapt on its own** — every change requires manual administrator intervention, which does not scale well.                                                                                                 |

> Because of these limitations, **static bridges are used less** in practice — **dynamic (transparent) bridges** are preferred.

---

## 7. Dynamic (Transparent) Bridge — Learning Process

A **dynamic (transparent) bridge** starts with an **empty table** and **learns automatically** over time.

### Step-by-Step Learning Process

**Step 1 — First packet, unknown destination:**

```
M1 (P1 side) sends a packet to M6 (destination unknown to the bridge)

Bridge table is EMPTY — bridge doesn't know where M6 is.
Action: BROADCAST the packet to all sides (like a hub would).

While broadcasting, the bridge ALSO learns:
"This packet came FROM M1, and it arrived on port P1"
→ New table entry created: M1 → P1
```

**Step 2 — Destination replies (acknowledgment):**

```
M6 sends a reply back to M1.
Source address = M6, arriving on port P2.

Bridge learns:
"M6 is the source of this reply, and it arrived on port P2"
→ New table entry created: M6 → P2
```

**Step 3 — Table builds up over time:**

As more devices communicate, the bridge continues to observe **source addresses and their arrival ports**, gradually building a complete table (e.g., M7 → P2, M4 → P1, etc.) — entirely **without administrator intervention**.

### Key Learning Behavior

- **When the bridge doesn't know where a destination is** → it **broadcasts** (forwards to all sides), just like a hub would in that moment.
- **Once the bridge has learned the mapping** (from observing source addresses on replies) → it can properly **forward** or **filter** based on the table, just like a static bridge would.

---

## 8. Static vs Dynamic Bridge — Comparison

| Property                      | Static Bridge                                     | Dynamic (Transparent) Bridge                           |
| ----------------------------- | ------------------------------------------------- | ------------------------------------------------------ |
| **Table population**          | Manual (network administrator types entries)      | Automatic (bridge learns from source addresses)        |
| **Initial state**             | Fully configured by admin before use              | Empty — learns as traffic flows                        |
| **Startup performance**       | Immediate — table is ready from the start         | Slower initially — takes time to learn/build the table |
| **Adaptability**              | Poor — any MAC/port change requires manual update | Good — automatically adapts to changes over time       |
| **Real-world preference**     | Used less                                         | **Preferred** — no administrator interference needed   |
| **Handles device MAC change** | Requires manual reconfiguration                   | Learns the new mapping automatically over time         |

> **Exam tip:** Dynamic bridges are slower to start (must learn), but perform better long-term due to automatic adaptability — this trade-off is a common comparison question.

---

## 9. Collision Domain of a Bridge

**Collisions are very rare in a bridge** (unlike repeaters/hubs, where max collisions = `n`).

**Why?** Bridges use a **store-and-forward** strategy:

```
Repeater/Hub:  Signal arrives ──► immediately relayed (no buffer) ──► COLLISION RISK

Bridge:        Packet arrives ──► STORED in buffer/memory
               Another packet arrives ──► ALSO stored in buffer
               Bridge processes and forwards each in turn ──► NO COLLISION
```

- A bridge has its **own buffer/memory**.
- Incoming packets are **stored** first, then **forwarded** after processing — this is the **store-and-forward** technique.
- Because packets are queued and processed rather than instantly relayed, **collisions are avoided almost entirely**, in sharp contrast to repeaters/hubs where collision domain = `n`.

---

## 10. Loop Prevention — Spanning Tree Protocol

Bridges use the **Bridge Protocol Data Unit (BPDU)** to build a **Spanning Tree**, which is used to **prevent loops**.

**Why is this needed?**

```
When multiple paths exist between LAN segments (e.g., redundant bridge links),
a packet can get stuck circulating endlessly in a LOOP.
```

- **Routers** solve a similar problem using the **TTL (Time to Live)** field, which limits how many hops a packet can take before being discarded.
- **Bridges do not have a TTL field.** Instead, they use the **Spanning Tree Protocol (STP)**, built using BPDU exchanges, to determine the correct path for forwarding and **eliminate loops entirely** by disabling redundant paths, rather than relying on a hop-count limit.

---

## 11. Summary Table

| Property                  | Bridge                                                 |
| ------------------------- | ------------------------------------------------------ |
| **OSI Layer**             | Physical Layer + Data Link Layer                       |
| **Hardware/Software**     | Hardware + Software                                    |
| **MAC address awareness** | Yes                                                    |
| **Forwarding**            | Yes — based on MAC address table                       |
| **Filtering**             | Yes — blocks packets destined for the same side        |
| **Table types**           | Static (manual) or Dynamic/Transparent (self-learning) |
| **Collision domain**      | Very rare — uses store-and-forward (buffer/memory)     |
| **Loop prevention**       | Spanning Tree Protocol, built via BPDU                 |
| **Typical use**           | Connecting two (possibly different-technology) LANs    |

---

## 12. Why This Matters for Cybersecurity

- **First appearance of access-control-like behavior:** The bridge's filtering capability is the conceptual ancestor of firewall/ACL-based traffic control — understanding _why_ a device needs Layer 2 intelligence (MAC awareness) to selectively block traffic is foundational before studying actual security devices (Firewall, IDS — covered in later parts).
- **Learning tables and spoofing implications:** The dynamic/transparent bridge's auto-learning behavior (mapping MAC address → port based on observed source addresses) is architecturally the same underlying mechanism switches use to build their **CAM/MAC address tables** — this is exactly the mechanism abused in **MAC flooding attacks** (see Part 5's lab on `macof`), where an attacker overwhelms the table with fake source MACs to force the device into broadcast (hub-like) behavior.
- **Spanning Tree Protocol as an attack surface:** STP itself has known attack classes (e.g., BPDU spoofing / STP manipulation attacks) that can be used to alter a network's logical topology or create denial-of-service conditions — worth keeping in mind as a topic to explore further once you study switches and enterprise LAN security in more depth.

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                     BRIDGE — EXAM CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI LAYER:  Physical Layer + Data Link Layer — Hardware + Software  ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE IDENTITY                                                          ║
║  ─────────────────────────────────────────────────────────          ║
║      Connects TWO (typically) LANs — can be DIFFERENT LAN types      ║
║      (e.g., Token Ring ↔ Token Bus/Ethernet)                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY CAPABILITIES (NEW vs repeater/hub)                               ║
║  ─────────────────────────────────────────────────────          ║
║      Forwarding → YES, based on MAC address table                    ║
║      Filtering  → YES (first device in this series that CAN filter!) ║
╠══════════════════════════════════════════════════════════════════════╣
║  STATIC vs DYNAMIC BRIDGE                                              ║
║  ─────────────────────────────────────────────────────          ║
║      Static  → admin manually types MAC-to-port table; no learning   ║
║      Dynamic → starts EMPTY, learns via broadcast + observing         ║
║                source MAC + arrival port on replies                   ║
║      Dynamic = slower start, but better long-term (preferred)        ║
╠══════════════════════════════════════════════════════════════════════╣
║  COLLISION DOMAIN                                                        ║
║  ─────────────────────────────────────────────────────          ║
║      Bridge uses STORE-AND-FORWARD (has buffer/memory)                ║
║      → Collisions are RARE (unlike repeater/hub where max = n)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  LOOP PREVENTION                                                        ║
║  ─────────────────────────────────────────────────────          ║
║      Bridge has NO TTL field (unlike routers)                        ║
║      Uses BPDU → builds SPANNING TREE to prevent loops               ║
╠══════════════════════════════════════════════════════════════════════╣
║  NOTE (per instructor): High-yield for UGC NET / PSU / Bank IT        ║
║  Officer exams. Low priority for GATE.                                ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Switch — full comparison with bridge, CAM table, multiport bridge concept
- [ ] Router — Layer 3, IP-based forwarding/filtering, TTL field, routing tables
- [ ] Firewall & IDS — building on the filtering concept introduced here

---

_Notes compiled from: Networking Course Lecture 12 — Networking Devices: Bridge_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
