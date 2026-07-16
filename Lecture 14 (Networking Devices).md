# 🔗 Networking Devices — Router

### Cybersecurity Student Notes | Networking Course — Lecture 14

> **Source:** Gate Smashers — Networking Devices: Router
> **Exam Relevance:** UGC NET (high) · GATE (basic level) · PSU Exams · University Exams
> **Topic Area:** Network Layer — Inter-Network Routing Device

---

## 📑 Table of Contents

1. [What is a Router? — Think "Internet"](#1-what-is-a-router--think-internet)
2. [OSI Layer — Physical + Data Link + Network](#2-osi-layer--physical--data-link--network)
3. [Forwarding on a Router](#3-forwarding-on-a-router)
4. [Flooding — When the Router Can't Decide](#4-flooding--when-the-router-cant-decide)
5. [Filtering on a Router — The ARP Request Example](#5-filtering-on-a-router--the-arp-request-example)
6. [Routing Protocols (Brief)](#6-routing-protocols-brief)
7. [Collision Domain of a Router](#7-collision-domain-of-a-router)
8. [Router Interface IP Addressing](#8-router-interface-ip-addressing)
9. [Summary Table — Router vs Switch vs Bridge vs Hub](#9-summary-table--router-vs-switch-vs-bridge-vs-hub)
10. [Why This Matters for Cybersecurity](#10-why-this-matters-for-cybersecurity)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. What is a Router? — Think "Internet"

**Key mental model:** Whenever the topic is **Router**, by default think **"Internet."**

**What is the Internet?** A collection of different, interconnected **networks**.

| Device Category                  | Scope                                                                  |
| -------------------------------- | ---------------------------------------------------------------------- |
| Hub, Repeater (Physical Layer)   | Within a LAN                                                           |
| Bridge, Switch (Data Link Layer) | Within/between LANs                                                    |
| **Router (Network Layer)**       | **Between different networks — the Internet, WAN (Wide Area Network)** |

A router's core purpose is to **connect two (or more) different networks** together — this is fundamentally a **WAN-scale** concern, in contrast to the LAN-scale devices covered in Parts 10–13.

---

## 2. OSI Layer — Physical + Data Link + Network

**Routers operate at THREE layers:**

| Layer             | Capability                |
| ----------------- | ------------------------- |
| Physical Layer    | Basic signal transmission |
| Data Link Layer   | Can check **MAC address** |
| **Network Layer** | Can check **IP Address**  |

**Comparison to earlier devices:**

| Device           | Layers                                 | Layer 3 (IP)?                             |
| ---------------- | -------------------------------------- | ----------------------------------------- |
| Bridge           | 2 (Physical + Data Link)               | No                                        |
| Switch (default) | 2 (Physical + Data Link)               | No (unless "smart switch"/Layer 3 switch) |
| **Router**       | **3 (Physical + Data Link + Network)** | **Yes**                                   |

> **Key exam distinction:** MAC addresses are used within a **Local Area Network**. IP addresses are used when talking about **Wide Area Network / the Internet / WWW** — accessing Google, Facebook, YouTube, Amazon, etc. all rely on **IP addressing**, which is why routers (Network Layer, IP-aware) are the core device of internetworking.

---

## 3. Forwarding on a Router

**Yes, routers forward packets** — using **IP addresses**, not MAC addresses, as the basis for the decision.

```
[Network A: Host HA] ────► [Router] ────► [Network B: Host HB]

Packet contains:
   Source Address = HA's IP Address
   Destination Address = HB's IP Address
```

**How does the router decide where to send it?** Using its **Routing Table**.

- A **routing table** is information the router maintains about **which networks it is connected to**.
- Using this table, the router determines **in which direction** (i.e., out of which interface) the packet should be forwarded.
- **Important:** A router is not limited to just two ports/interfaces (unlike a bridge) — **multiple networks can be connected** to a single router, each via its own interface, and the routing table governs the forwarding decision across all of them.

---

## 4. Flooding — When the Router Can't Decide

If the router is **unable to determine** which direction a packet should go (i.e., no matching route), it can perform **flooding**.

**Flooding = sending the packet out in all directions** — conceptually similar to broadcasting.

> This is the router-level equivalent of what a hub does by default (Part 11) or what a dynamic bridge does before it has learned a MAC-to-port mapping (Part 12) — except at the router, flooding is a fallback behavior used only when the routing table has no known path.

---

## 5. Filtering on a Router — The ARP Request Example

**Yes, routers can filter packets** — they can decide whether to forward a packet or stop it, based on the routing table.

**Worked example — why filtering matters here:**

- When a host wants to communicate over the internet, it typically knows the **IP address** of its **default gateway** (the router) but **not its MAC address**.
- To discover the MAC address corresponding to a known IP address, the host sends an **ARP Request** (Address Resolution Protocol) — essentially asking: _"I have this IP address; whose MAC address is this?"_

```
[Host in Network A] ──ARP Request──► [Router]

ARP Request: "I have IP address X — who has this? Tell me your MAC address."

Router's behavior: Does NOT forward this request onward to Network B.
It STOPS it here — this is FILTERING.
```

- **Why does the router stop it?** Because **ARP requests are only meaningful within a single local network** — they have no purpose being forwarded to a different network, since ARP resolves local Layer 2 (MAC) addresses.
- This decision — **whether to send the packet forward (forwarding) or stop it (filtering)** — is made possible because the router can inspect addressing information and apply logic via its routing table, exactly as a bridge/switch does with MAC addresses (Parts 12–13), but at the IP/network scope.

---

## 6. Routing Protocols (Brief)

Routers rely on **routing protocols** to build and maintain their routing tables. Two mentioned here (at an introductory level only):

- **RIP (Routing Information Protocol)**
- **Distance Vector Routing Protocol**

> **Instructor's note:** Detailed coverage of routing protocols belongs to the dedicated **Network Layer** section of the course. For **UGC NET** and basic-level **GATE** questions, the points in this lecture are sufficient — deep protocol mechanics are covered separately.

---

## 7. Collision Domain of a Router

**No, packets cannot collide inside a router.**

**Why?** Like the bridge (Part 12), the router uses the **store-and-forward** method:

```
Packet 1 arrives ──► STORED in router's memory
Packet 2 arrives ──► ALSO stored in memory (no collision)

Router processes stored packets using its routing table,
then decides the outgoing direction for each — one at a time,
with no simultaneous collision on the medium.
```

- The router **stores** incoming packets in memory/buffer, **processes** them against the routing table, and then forwards them appropriately.
- Because packets are queued and processed rather than instantaneously relayed onto a shared medium, **no collision occurs** within the router — consistent with the general pattern that devices with buffering/store-and-forward capability (bridge, switch, router) avoid the collision problems inherent to pure hardware relay devices (repeater, hub).

---

## 8. Router Interface IP Addressing

**Key concept: a router's interface gets its IP address FROM the network it connects to — not from anywhere else.**

```
[Network A] ══(Interface 1)══[ROUTER]══(Interface 2)══ [Network B]

Interface 1's IP address → taken from Network A's IP address range
Interface 2's IP address → taken from Network B's IP address range
```

- Each network (A, B, etc.) has a pool/range of possible IP addresses available.
- **One IP address from that network's range** is assigned to the router's corresponding interface, allowing that interface to participate in and communicate with that specific network.
- The router does **not** invent or import an IP address from an unrelated source — the interface's address must belong to the network it is directly connected to, so that hosts on that network can properly route traffic through it.

---

## 9. Summary Table — Router vs Switch vs Bridge vs Hub

| Property                  | Hub                     | Bridge                                      | Switch             | Router                                                    |
| ------------------------- | ----------------------- | ------------------------------------------- | ------------------ | --------------------------------------------------------- |
| **OSI Layer**             | L1 (Physical)           | L1+L2                                       | L1+L2 (default)    | **L1+L2+L3**                                              |
| **Address type used**     | None                    | MAC                                         | MAC                | **MAC + IP**                                              |
| **Scope**                 | LAN segment             | Between 2 LANs                              | Within/across LANs | **Between different networks (WAN/Internet)**             |
| **Forwarding**            | Broadcast, always       | Address-based                               | Address-based      | Address-based (routing table)                             |
| **Filtering**             | No                      | Yes                                         | Yes                | **Yes** (e.g., stops ARP requests from crossing networks) |
| **Fallback when unknown** | N/A (always broadcasts) | Broadcast (dynamic bridge, before learning) | N/A                | **Flooding**                                              |
| **Collision domain**      | `n`                     | Rare (store-and-forward)                    | Zero (full duplex) | **None (store-and-forward)**                              |
| **Table used**            | None                    | MAC address table                           | MAC/CAM table      | **Routing table**                                         |
| **Ports**                 | Multiport               | 2 (fixed)                                   | Multiport          | Multiport (connects multiple networks)                    |

---

## 10. Why This Matters for Cybersecurity

- **ARP scope boundary = a real security-relevant fact:** The reason a router filters (drops) ARP requests instead of forwarding them is directly why **ARP spoofing attacks** (used in earlier labs, Part 5) are inherently confined to the **local network segment** — an attacker cannot ARP-spoof a host on a different network across a router without also compromising something on that remote segment. This is foundational to understanding the blast radius of Layer 2 attacks.
- **Routing tables as a target:** Just as MAC/CAM tables are targeted by MAC flooding, **routing tables** can be targeted by **route injection/route poisoning** attacks (e.g., abusing insecure dynamic routing protocol implementations) — a natural extension of this lecture's routing table concept into offensive security topics you'll study later in more advanced networking/security coursework.
- **Default gateway trust:** Since hosts implicitly trust the IP-to-MAC mapping they get for their default gateway (via ARP), this is exactly the mechanism abused in classic **man-in-the-middle attacks** — an attacker who can convincingly answer the ARP request on behalf of the router's IP (as covered in Part 5's `arpspoof` lab) can intercept traffic meant for the real gateway.
- **Store-and-forward at Layer 3:** Understanding that routers buffer and process packets (rather than instantly relaying signals) is relevant background for topics like traffic shaping, QoS, and — from a security angle — why routers are a natural place to implement access control lists (ACLs) and firewall rules, since they already inspect and make per-packet decisions.

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                    ROUTER — EXAM CHEAT SHEET                          ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI LAYER:  Physical + Data Link + Network Layer (Layer 3)          ║
║  MENTAL MODEL: Router = Internet / WAN (connects different networks) ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY CAPABILITIES                                                        ║
║  ─────────────────────────────────────────────────────────          ║
║      Forwarding  → YES, based on ROUTING TABLE (IP address-based)     ║
║      Flooding     → fallback when routing table has NO known path     ║
║      Filtering    → YES (e.g., blocks ARP requests from crossing      ║
║                      into a different network — ARP is LOCAL only)    ║
╠══════════════════════════════════════════════════════════════════════╣
║  ADDRESSING                                                              ║
║  ─────────────────────────────────────────────────────          ║
║      MAC Address → used within a LAN                                  ║
║      IP Address   → used for WAN / Internet / inter-network traffic   ║
║      Router interface IP → assigned from the connected network's      ║
║                             own IP address range (not external)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  ROUTING PROTOCOLS (brief mention only)                                ║
║  ─────────────────────────────────────────────────────          ║
║      RIP (Routing Information Protocol)                               ║
║      Distance Vector Routing Protocol                                 ║
║      (Detailed protocol mechanics = separate Network Layer topic)     ║
╠══════════════════════════════════════════════════════════════════════╣
║  COLLISION DOMAIN                                                        ║
║  ─────────────────────────────────────────────────────          ║
║      Router uses STORE-AND-FORWARD (like bridge)                      ║
║      → NO collision occurs inside a router                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  DEVICE PROGRESSION SO FAR (LAYERS)                                      ║
║  ─────────────────────────────────────────────────────          ║
║      Repeater / Hub → Layer 1 only                                     ║
║      Bridge / Switch → Layer 1 + 2                                     ║
║      Router          → Layer 1 + 2 + 3                                 ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Gateway — recap and detail (role played by router/firewall/proxy)
- [ ] Firewall — types and detailed working
- [ ] IDS vs IPS — detection vs prevention
- [ ] Modem — modulation/demodulation in detail

---

_Notes compiled from: Networking Course Lecture 14 — Networking Devices: Router_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
