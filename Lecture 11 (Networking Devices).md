# 🔗 Networking Devices — Hub

### Cybersecurity Student Notes | Networking Course — Lecture 11

> **Source:** Gate Smashers — Networking Devices: Hub
> **Exam Relevance:** UGC NET (high) · GATE (secondary — instructor notes this topic is mainly for Electronics students) · PSU Exams
> **Topic Area:** Physical Layer — Multiport Signal Relay Device

---

## 📑 Table of Contents

1. [What is a Hub?](#1-what-is-a-hub)
2. [Hub vs Repeater — Key Differences](#2-hub-vs-repeater--key-differences)
3. [Does a Hub Forward?](#3-does-a-hub-forward)
4. [Does a Hub Filter?](#4-does-a-hub-filter)
5. [Collision Domain of a Hub](#5-collision-domain-of-a-hub)
6. [Summary Table](#6-summary-table)
7. [Why This Matters for Cybersecurity](#7-why-this-matters-for-cybersecurity)
8. [Exam Cheat Sheet](#8-exam-cheat-sheet)

---

## 1. What is a Hub?

A **hub**, like a repeater, operates purely at the **Physical Layer (Layer 1)** of the OSI model — it is **pure hardware**, with **no software** running on it, and therefore **no data link layer functionality** (no MAC address awareness).

```
        [A]
         │
  [D]──[HUB]──[B]     ← this is the Star topology from Part 5
         │
        [C]
```

A hub is commonly used as the **central device in a Star topology** (covered in Part 5), connecting multiple devices together.

---

## 2. Hub vs Repeater — Key Differences

### 2.1 Port Count — "Hub is a Multiport Repeater"

> **Core concept:** A **hub is essentially a multiport repeater.**

| Device       | Ports                                                                  |
| ------------ | ---------------------------------------------------------------------- |
| **Repeater** | 2 ports (fixed, by default — see Part 10)                              |
| **Hub**      | **Multiport** — commonly available as 4-port, 8-port, 16-port, or more |

This is the fundamental structural difference: a repeater connects exactly two segments/devices, while a hub can connect **many** devices simultaneously (as seen in Star topology).

### 2.2 Extra Functionality (Dedicated Device)

Because a hub is a **dedicated networking device** (unlike a simple repeater), it typically offers small extra conveniences that a repeater does not, such as:

- **Status LEDs** — small indicator lights that show whether a port/link has a problem, or whether power is on/off.

A plain repeater has no such diagnostic indicators — if a wire is cut or has an issue, the repeater itself gives no indication of the problem.

---

## 3. Does a Hub Forward?

**Yes — a hub always forwards**, just like a repeater.

```
A ──(message to D)──► [HUB] ──(forwarded)──► D
```

When a device sends data, and it reaches the hub, the hub forwards it onward.

---

## 4. Does a Hub Filter?

**No — a hub cannot filter**, for the same fundamental reason as a repeater: it is pure hardware with no software.

**What does "filter" mean?** Think of a household filter — it blocks unwanted material and only lets the intended substance through. A network filtering device would only send data to its **intended destination**, blocking it from unnecessary paths.

**A hub cannot do this.** Instead, it **broadcasts**:

```
[A] sends "Hello B" to the hub:

Hub broadcasts to ALL ports:
   ──► B  (intended recipient)
   ──► C  (also receives — should not have!)
   ──► D  (also receives — should not have!)
```

- A hub **cannot check a MAC address or any interface information** — it has no way to determine which specific port the destination device is on.
- Its only behavior is to **simply broadcast the message to everyone** connected to it.
- **Consequence:** Because every message goes to every device (whether intended or not), **overall network traffic stays high** — this is true for both the hub and the repeater, since both simply forward+broadcast without any selectivity.

---

## 5. Collision Domain of a Hub

**Yes, collisions are possible inside a hub.**

```
[A]──┐
[B]──┼──[HUB]   ← if A, B, C, D all transmit at the same time,
[C]──┤            all 4 signals collide with each other
[D]──┘
```

- If multiple devices connected to the hub transmit **at the same time**, their signals will **collide** within the hub.
- **Maximum possible collisions = `n`**, where `n` = the number of devices connected to the hub.
- This mirrors the repeater's collision domain behavior (Part 10) — since a hub is just a repeater with more ports, it inherits the same lack of buffering/store-and-forward capability, and therefore the same collision characteristics, just scaled to however many ports are connected.

---

## 6. Summary Table

| Property                  | Repeater                       | Hub                                       |
| ------------------------- | ------------------------------ | ----------------------------------------- |
| **OSI Layer**             | Physical Layer (Layer 1)       | Physical Layer (Layer 1)                  |
| **Hardware/Software**     | Pure hardware                  | Pure hardware                             |
| **Ports**                 | 2 (fixed)                      | Multiport (4, 8, 16, 26+...)              |
| **Extra diagnostics**     | None                           | Status LEDs (link/power indicators)       |
| **Forwarding**            | Always forwards                | Always forwards                           |
| **Filtering**             | Cannot filter                  | Cannot filter (broadcasts to all)         |
| **Collision domain**      | n (all devices on both sides)  | n (all connected devices)                 |
| **Cost**                  | Cheap                          | Cheap (cheaper than bridge/switch/router) |
| **Typical topology role** | Extending a segment's distance | Central device in Star topology           |

> **Bottom line:** A hub = a **multiport repeater** with a couple of extra conveniences (diagnostic LEDs), but it shares the exact same core limitations as a repeater — no filtering, no MAC address awareness, full broadcast, and a shared collision domain across all connected devices.

---

## 7. Why This Matters for Cybersecurity

This lecture directly reinforces the Star-topology security weakness already covered in Part 5:

- **Trivial passive sniffing:** Since a hub broadcasts every frame to every connected port with zero filtering, **any device on a hub-based network can capture all traffic passing through it** with a simple packet capture tool — no attack technique (like ARP spoofing or MAC flooding, needed on switches) is required at all.
- **Single collision domain = full traffic visibility:** Because all connected devices share one collision domain (`n` devices, `n` max collisions), an attacker plugged into any port has visibility into the entire hub segment's traffic — this is exactly why hub-based networks are considered legacy/insecure and have been almost entirely replaced by switches in modern deployments.
- **Contrast with switches (next part):** This sets up the key security distinction covered in later parts — a **switch** uses a MAC address table to forward selectively, dramatically reducing (but not eliminating — see MAC flooding, Part 5) this exposure.

---

## 8. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                       HUB — EXAM CHEAT SHEET                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI LAYER:  Physical Layer (Layer 1) — Pure Hardware                ║
╠══════════════════════════════════════════════════════════════════════╣
║  CORE IDENTITY                                                          ║
║  ─────────────────────────────────────────────────────────          ║
║      HUB = MULTIPORT REPEATER                                        ║
║      Repeater → 2 ports (fixed)                                      ║
║      Hub      → many ports (4/8/16/26+)                              ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FACTS                                                              ║
║  ─────────────────────────────────────────────────────          ║
║      Forwarding         → YES, always forwards                       ║
║      Filtering          → NO — broadcasts to ALL ports                ║
║                            (cannot check MAC address)                 ║
║      Collision domain   → n (all connected devices — max collisions) ║
║      Extra feature      → Status LEDs (link/power) — repeater lacks  ║
║                            this since it's not a "dedicated device"  ║
║      Cost                → Cheap (hardware only, like repeater)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  COMMON UGC NET-STYLE T/F QUESTIONS                                    ║
║  ─────────────────────────────────────────────────────          ║
║      "Hub can filter"          → FALSE                                ║
║      "Hub can forward"         → TRUE                                 ║
║      "Hub works on Layer 1"    → TRUE                                 ║
║      "Hub has software"        → FALSE                                ║
║      "Hub is a two-port device"→ FALSE (it's multiport)               ║
╠══════════════════════════════════════════════════════════════════════╣
║  NOTE (per instructor): This topic is mainly relevant for             ║
║  Electronics students — the points above are sufficient depth for     ║
║  UGC NET / PSU exams; no need to go deeper.                          ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Bridge — how filtering capability first appears, MAC-based decisions
- [ ] Switch — full comparison with hub, MAC address table, CAM table
- [ ] Router — Layer 3, IP-based forwarding and filtering

---

_Notes compiled from: Networking Course Lecture 11 — Networking Devices: Hub_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
