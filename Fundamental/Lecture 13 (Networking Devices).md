# 🔗 Networking Devices — Switch

### " "Networking Course — Lecture 13

> **Source:** Gate Smashers — Networking Devices: Switch
> **Exam Relevance:** GATE · UGC NET · PSU Exams · University Exams · Interviews
> **Topic Area:** Data Link Layer — Multiport Intelligent Forwarding Device

---

## 📑 Table of Contents

1. [What is a Switch?](#1-what-is-a-switch)
2. [Switch = Multiport Bridge](#2-switch--multiport-bridge)
3. [Real-World Role — Connecting Endpoints to the Internet](#3-real-world-role--connecting-endpoints-to-the-internet)
4. [Full-Duplex Links — Why Collision Domain is Zero](#4-full-duplex-links--why-collision-domain-is-zero)
5. [Traffic Behavior — Address-Based Forwarding](#5-traffic-behavior--address-based-forwarding)
6. [Layer 2 by Default — Smart Switches (Layer 3)](#6-layer-2-by-default--smart-switches-layer-3)
7. [Summary Table — Switch vs Bridge vs Hub](#7-summary-table--switch-vs-bridge-vs-hub)
8. [Why This Matters for Cybersecurity](#8-why-this-matters-for-cybersecurity)
9. [Exam Cheat Sheet](#9-exam-cheat-sheet)

---

## 1. What is a Switch?

A **switch** is a device commonly seen in computer labs and offices, used to connect multiple devices in a network. It is fundamentally an **evolution of the bridge** (Part 12), with the same Layer 2 intelligence but far more ports and better performance characteristics.

---

## 2. Switch = Multiport Bridge

**Core concept: a switch is a multiport bridge.**

| Device     | Ports                                         | OSI Layer                                   |
| ---------- | --------------------------------------------- | ------------------------------------------- |
| **Bridge** | 2 ports (fixed) — connects exactly two LANs   | Physical + Data Link Layer                  |
| **Switch** | **Multiport** — 8, 24, 48, 52+ ports possible | Physical + Data Link Layer (same as bridge) |

```
Bridge:           [LAN 1] ══P1══[BRIDGE]══P2══ [LAN 2]

Switch:           [PC]──┐
                   [Laptop]──┼──[SWITCH: many ports]
                   [Printer]──┤
                   [Wireless AP]──┘
```

- Both bridge and switch are **Layer 2 devices** — they share **Physical Layer + Data Link Layer** functionality, meaning both can check **MAC addresses** to make forwarding decisions.
- The key structural difference is simply **port count**: a bridge is limited to 2 ports (by definition, connecting two LANs), while a switch is a **multiport** version of that same concept.

---

## 3. Real-World Role — Connecting Endpoints to the Internet

While a bridge is typically used to **connect two LANs together**, a switch is used to connect **individual end devices** — computers, laptops, printers, wireless access points, etc. — into the network.

**Typical real-world path to the internet:**

```
[Your Laptop] ──(cable)──► [Switch] ──► [Router] ──► www (Internet)
```

- You don't usually connect your laptop **directly** to the router — you connect to a **switch**, and the switch connects to the router.
- This is how end-user devices reach the wider internet in most real network setups (offices, labs, homes with multiple wired ports).

---

## 4. Full-Duplex Links — Why Collision Domain is Zero

**Switch ports operate as full-duplex links.**

```
4-port switch example: A, B, C, D connected

A ◄──────────────────► Switch   (full duplex: A→Switch AND Switch→A simultaneously)
B ◄──────────────────► Switch
C ◄──────────────────► Switch
D ◄──────────────────► Switch
```

**What is full duplex?** Data can travel **in both directions simultaneously** on the same link — e.g., data can go from A to the switch, while at the very same time, data can go from the switch back to A, with **no collision** between them.

**Dedicated circuits per conversation:**

```
If A sends a message to B → a dedicated circuit forms between A and B
If C sends a message to D → a SEPARATE dedicated circuit forms between C and D

These two circuits do NOT interfere with or collide with each other.
```

- Because each port has its own full-duplex link, and the switch creates dedicated circuits for each conversation, **the collision domain of a switch is effectively ZERO** — a major improvement over hubs (collision domain = `n`, see Part 11) and even over bridges (rare collisions via store-and-forward, see Part 12).

---

## 5. Traffic Behavior — Address-Based Forwarding

**Switches produce far less traffic than hubs**, because they use **physical (MAC) address**-based forwarding, just like a bridge.

```
HUB behavior (Part 11):
  A sends message to B ──► broadcast ──► reaches A, B, C, D (everyone)

SWITCH behavior:
  A sends message to B ──► switch checks its MAC address table
                        ──► forwards ONLY to B's port
                        ──► C and D do NOT receive the message
```

- The switch maintains a table (conceptually the same **learning mechanism** as the dynamic/transparent bridge from Part 12 — often called a **MAC address table** or **CAM table**) mapping MAC addresses to ports.
- Because of this selective, address-based forwarding: **traffic stays low** (only sent where needed) **and collisions stay low/zero** (full-duplex, dedicated circuits).
- This combination — **low traffic + (near) zero collision** — is what makes switches significantly more powerful/efficient than hubs.

---

## 6. Layer 2 by Default — Smart Switches (Layer 3)

**By default, a switch is a Layer 2 (Data Link Layer) device.**

- You may encounter sources online claiming a switch "includes the network layer" — this refers to an **enhanced/upgraded** category of switch, not the standard/default switch.
- These enhanced devices are called **"smart switches" (Layer 3 switches)** — they add **Layer 3 (Network Layer)** capability on top of standard Layer 2 switching.
- The standard Layer 3 (Network Layer) device is the **router** (covered separately) — a Layer 3 switch essentially blends switch-like port density with some router-like IP-based forwarding capability.

> **Exam tip:** Unless a question specifically says "Layer 3 switch" or "smart switch," assume a plain **"switch" = Layer 2 device** by default.

---

## 7. Summary Table — Switch vs Bridge vs Hub

| Property                  | Hub                         | Bridge                            | Switch                                                                 |
| ------------------------- | --------------------------- | --------------------------------- | ---------------------------------------------------------------------- |
| **OSI Layer**             | Physical (L1)               | Physical + Data Link (L1+L2)      | Physical + Data Link (L1+L2), by default                               |
| **Ports**                 | Multiport                   | 2 (fixed)                         | Multiport (8/24/48/52+)                                                |
| **MAC address awareness** | No                          | Yes                               | Yes                                                                    |
| **Forwarding**            | Broadcasts to all           | Address-based (table lookup)      | Address-based (table lookup)                                           |
| **Filtering**             | No                          | Yes                               | Yes                                                                    |
| **Link type**             | Shared, half-duplex         | Half-duplex segments joined       | **Full-duplex** per port                                               |
| **Collision domain**      | `n` (all connected devices) | Rare (store-and-forward)          | **Zero** (full-duplex, dedicated circuits)                             |
| **Traffic level**         | High (broadcast)            | Lower (filters same-side traffic) | Low (unicast to specific port)                                         |
| **Typical use**           | Legacy Star topology center | Connecting 2 LANs                 | Connecting many end devices (PCs, printers, APs) to the network/router |
| **Can be upgraded to**    | —                           | —                                 | Layer 3 / "Smart Switch"                                               |

---

## 8. Why This Matters for Cybersecurity

- **CAM table = the mechanism behind MAC flooding attacks:** The switch's MAC address table (CAM table) is exactly the target of the **MAC flooding** attack introduced in Part 5's practical labs (`macof`). Now that you understand _why_ the table exists (address-based forwarding to reduce traffic/collisions), it's clear why flooding it with fake entries forces the switch to fail open into **broadcast/hub-like behavior** — this is a direct, practical consequence of the mechanism described in this lecture.
- **Full-duplex/zero-collision ≠ secure by default:** A switch's near-zero collision domain and low broadcast traffic make **passive sniffing much harder than on a hub** — an attacker generally cannot simply plug in and see everyone's traffic. This is _why_ techniques like **ARP spoofing** and **MAC flooding** exist: they're specifically designed to defeat this selective-forwarding behavior and force visibility into traffic that wouldn't otherwise reach an attacker's port.
- **Layer 3 switches widen the attack surface conceptually:** Since "smart switches" add routing-like Layer 3 capability, they inherit some of the considerations relevant to routers (e.g., routing table integrity, inter-VLAN routing security) — a useful bridge (no pun intended) into router-focused security topics coming up next.

---

## 9. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                     SWITCH — EXAM CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI LAYER:  Physical + Data Link Layer (Layer 2) — DEFAULT           ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE IDENTITY                                                          ║
║  ─────────────────────────────────────────────────────────          ║
║      SWITCH = MULTIPORT BRIDGE                                        ║
║      Bridge  → 2 ports (fixed), connects 2 LANs                       ║
║      Switch  → many ports (8/24/48/52+), connects many END DEVICES    ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY PERFORMANCE FACTS                                                  ║
║  ─────────────────────────────────────────────────────          ║
║      Link type          → FULL DUPLEX (send + receive simultaneously)║
║      Collision domain    → ZERO (dedicated circuits per conversation) ║
║      Traffic             → LOW (unicast via MAC address table lookup) ║
║      Forwarding          → YES, address-based                        ║
║      Filtering           → YES, address-based                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  DEFAULT vs ENHANCED                                                     ║
║  ─────────────────────────────────────────────────────          ║
║      Default switch    → Layer 2 (Data Link Layer) device            ║
║      "Smart switch"     → Layer 3 (adds Network Layer capability)     ║
║      Router             → the standard/original Layer 3 device        ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK COLLISION DOMAIN COMPARISON (ALL DEVICES SO FAR)                ║
║  ─────────────────────────────────────────────────────          ║
║      Repeater / Hub  → collision domain = n                          ║
║      Bridge           → collisions RARE (store-and-forward)          ║
║      Switch            → collision domain = ZERO (full duplex)        ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Router — Layer 3, IP-based routing, routing tables, TTL field
- [ ] Gateway — role played by router/firewall/proxy (recap from Part 8)
- [ ] Firewall & IDS — building on the filtering concepts from Bridge/Switch

---

_Notes compiled from: Networking Course Lecture 13 — Networking Devices: Switch_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
