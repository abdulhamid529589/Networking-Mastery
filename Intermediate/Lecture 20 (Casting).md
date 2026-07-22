# 📡 Types of Casting — Unicast, Broadcast & Multicast

### " "Networking Course — Lecture 20

> **Source:** Gate Smashers — Types of Casting (Unicast, Broadcast, Multicast)
> " "
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is "Casting" in Networking?](#1-what-is-casting-in-networking)
2. [Unicast](#2-unicast)
   - [2.1 Concept](#21-concept)
   - [2.2 Diagram & Example](#22-diagram--example)
3. [Broadcast](#3-broadcast)
   - [3.1 Limited Broadcast](#31-limited-broadcast)
   - [3.2 Direct Broadcast](#32-direct-broadcast)
   - [3.3 Limited vs Direct Broadcast](#33-limited-vs-direct-broadcast)
4. [Multicast](#4-multicast)
   - [4.1 Concept](#41-concept)
   - [4.2 IGMP — Internet Group Management Protocol](#42-igmp--internet-group-management-protocol)
   - [4.3 Class D Addressing](#43-class-d-addressing)
5. [Comparison Table — Unicast vs Broadcast vs Multicast](#5-comparison-table--unicast-vs-broadcast-vs-multicast)
6. [🔴 Security Considerations & Modern Relevance](#6--security-considerations--modern-relevance)
7. [🧪 Practical Labs — Your Setup](#7--practical-labs--your-setup)
8. [Exam Cheat Sheet](#8-exam-cheat-sheet)

---

## 1. What is "Casting" in Networking?

**Casting** = the manner in which data is addressed and delivered from a sender to one or more receivers on a network.

There are three fundamental types:

| Type          | Meaning                                                        |
| ------------- | -------------------------------------------------------------- |
| **Unicast**   | One sender → **one** specific receiver                         |
| **Broadcast** | One sender → **all** hosts (in own network or another network) |
| **Multicast** | One sender → a **specific group** of interested receivers      |

> This is a **frequently tested theoretical topic** — especially in **UGC NET** and **GATE**-style exams. It doesn't require heavy formula work like Mesh topology, but the **concepts and address types** are commonly asked directly.

---

## 2. Unicast

### 2.1 Concept

**Unicast = one-to-one communication.**

- A unique **IP address** maps to a **single specific host**.
- The most common form of communication on computer networks — because the core purpose of networking is usually **sharing data/files between two specific parties**.
- Even though messages _can_ be sent to a group, **day-to-day communication** (e.g., sending a message, loading a webpage, sending an email to one person) is fundamentally **unicast**.

### 2.2 Diagram & Example

```
Network A (Class A):  90.0.0.0
Network B:             (separate network)

        Network A                         Network B
   ┌───────────────────┐            ┌───────────────────┐
   │   [M1] ──────────────message───────────► [M2]        │
   │                    │            │                     │
   └───────────────────┘            └───────────────────┘

M1 wants to send data to M2 → uses M2's UNIQUE IP address
→ ONE-to-ONE communication = Unicast
```

- Source host = **M1**
- Destination host = **M2**
- Delivery is addressed to **exactly one** unique IP — no other host processes this as "meant for me."

---

## 3. Broadcast

**Broadcast** = sending a message to **all** hosts on a network (as opposed to just one, in unicast).

Broadcast is of **two types**:

```
Broadcast
   │
   ├── Limited Broadcast
   └── Direct Broadcast
```

---

### 3.1 Limited Broadcast

**Limited Broadcast** = sending a message to **all hosts within your OWN (local) network only**.

- Uses the special address:

```
255.255.255.255   (all 1s — "limited broadcast address")
```

- This address is **identical/reserved regardless of which network you're on** — it simply means _"everyone on my local network."_
- The message **never leaves the local network** (routers do not forward this address onward).

```
Network A: 90.0.0.0
┌─────────────────────────────────────────┐
│   [M1] ──broadcast──► [Host2]            │
│            │  ├──────► [Host3]           │
│            │  └──────► [Host4]           │
│    Destination address used:              │
│    255.255.255.255  (limited broadcast)   │
└─────────────────────────────────────────┘

M1 wants to message ALL hosts in its OWN network
→ Destination = 255.255.255.255
```

---

### 3.2 Direct Broadcast

**Direct Broadcast** = sending a message to **all hosts in a DIFFERENT (remote) network** — i.e., you are outside that network, but you want every host inside it to receive your message.

- The **direct broadcast address** for a target network is formed by:

```
Direct Broadcast Address = Network ID with ALL HOST BITS set to 1
```

- Example:

```
Target Network ID:        90.0.0.0        (Class A)
                           ↓ (set all host bits to 1)
Direct Broadcast Address: 90.255.255.255
```

```
Network A                              Network B: 90.0.0.0
┌───────────────┐                     ┌─────────────────────────┐
│    [M1]  ──────────message───────────►  ALL hosts in Network B │
│                │  Destination =      │   [Host1] [Host2] [Host3]│
└───────────────┘  90.255.255.255      └─────────────────────────┘

M1 (in Network A) wants to message EVERYONE in Network B (a network
it is NOT part of) → uses Network B's DIRECT BROADCAST ADDRESS
```

> **Key distinction:**
>
> - **Limited broadcast** → stays inside **your own** network, uses `255.255.255.255`.
> - **Direct broadcast** → targets **another** network, uses that network's ID with all host bits set to 1 (e.g., `90.255.255.255`).

---

### 3.3 Limited vs Direct Broadcast

| Feature                 | Limited Broadcast                                  | Direct Broadcast                           |
| ----------------------- | -------------------------------------------------- | ------------------------------------------ |
| Target                  | Hosts in **own** network                           | Hosts in **another/remote** network        |
| Address used            | `255.255.255.255` (all 1s)                         | `<Network ID>.255.255.255` (host bits = 1) |
| Crosses router boundary | **No** (stays local)                               | **Yes** (intended for a different network) |
| Example                 | Sending to all in Network A while inside Network A | Sending from Network A to all of Network B |

---

## 4. Multicast

### 4.1 Concept

**Multicast** = sending a message to a **specific group** of interested receivers — not everyone (unlike broadcast), and not just one host (unlike unicast).

> Real-life analogy: A **research group** working on a common topic. If you want to send an update, you don't email the entire organization (broadcast) or just one person (unicast) — you email **that specific group** (multicast).

Other real-world examples:

- **News/TV broadcast** channels → conceptually more like "broadcast" (open to anyone tuned in).
- **A mailing list / group email** sent only to members of a particular group → this is **multicast** — targeting a _similar kind of audience_ rather than literally everyone.

---

### 4.2 IGMP — Internet Group Management Protocol

Multicast group membership is managed using:

```
IGMP = Internet Group Management Protocol
```

- IGMP allows hosts to **join** or **leave** a multicast group.
- Routers use IGMP to know **which hosts on a network are interested** in receiving traffic for a particular multicast group, so they only forward multicast traffic where there are actual listeners.

---

### 4.3 Class D Addressing

Multicast uses a **dedicated address class**:

```
Class D → reserved exclusively for multicast addressing
```

- Class D IP addresses are used to identify a **multicast group** (not an individual host).
- Hosts that want to receive traffic for that group must **join** the group (via IGMP); only those hosts will process/receive the multicast traffic.
- This lets similar/interested hosts be **targeted together**, without flooding the entire network (as broadcast would).

---

## 5. Comparison Table — Unicast vs Broadcast vs Multicast

| Parameter            | Unicast                                          | Broadcast                                                             | Multicast                                            |
| -------------------- | ------------------------------------------------ | --------------------------------------------------------------------- | ---------------------------------------------------- |
| **Delivery**         | One sender → one receiver                        | One sender → **all** hosts                                            | One sender → a **specific group**                    |
| **Address type**     | Unique host IP                                   | `255.255.255.255` (limited) / Network ID + all host bits = 1 (direct) | Class D address                                      |
| **Scope**            | Point-to-point                                   | Local network (limited) or a remote network (direct)                  | Only members of the group                            |
| **Efficiency**       | High (only intended host processes it)           | Low (every host must process it)                                      | Moderate–High (only interested hosts process it)     |
| **Managed by**       | Routing / ARP                                    | Broadcast domain rules                                                | IGMP                                                 |
| **Typical use case** | Web browsing, file transfer, email to one person | Network discovery (e.g., DHCP discover), alerts to all local hosts    | Video conferencing, group email, IPTV, group updates |

---

## 6. 🔴 Security Considerations & Modern Relevance

### Unicast

- Because unicast is addressed to exactly one host, an attacker generally needs to be **on-path** (e.g., via ARP spoofing on a switch) to intercept it — it isn't inherently visible to other hosts, unlike broadcast traffic.

### Broadcast

Broadcast traffic is visible to **every host in the broadcast domain**, which has real security implications:

| Concern                                          | Explanation                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Broadcast storms**                             | Excessive broadcast traffic (e.g., from a misconfigured device or loop) can flood a LAN and degrade performance/availability                                                                                                                                                                                                                                                                                                                                     |
| **Information exposure**                         | Protocols like **ARP** and **DHCP Discover** rely on broadcast — any host on the segment can passively observe these broadcasts and learn about other devices on the network                                                                                                                                                                                                                                                                                     |
| **Directed/Direct broadcast abuse (historical)** | Direct broadcast was historically abused in amplification-style denial-of-service attacks (e.g., the well-known "Smurf attack" from the 1990s, which spoofed a victim's address and sent pings to a network's direct broadcast address so that _every host_ replied to the victim at once). This is why, for decades, **routers have directed broadcast forwarding disabled by default** (a standard hardening setting, e.g. Cisco's `no ip directed-broadcast`) |

### Multicast

- Because only "joined" hosts are supposed to receive multicast traffic, **IGMP itself can be abused** if unauthenticated — a host could send bogus IGMP join/leave messages, which is why enterprise switches implement **IGMP snooping** (only forward multicast frames to ports that have valid group members) as both a **performance and security control**.

> **Key takeaway for a security student:** Broadcast and (to a lesser extent) multicast traffic are inherently "louder" than unicast — more hosts see them by design, which means misconfiguration or spoofing has a wider blast radius. This is why **broadcast domains are segmented** (via VLANs) and why **directed broadcast is disabled** on modern routers.

---

## 7. 🧪 Practical Labs — Your Setup

### Lab 1 — Observe Broadcast Traffic (ARP is a Limited Broadcast)

```bash
# ARP requests are a real-world example of LIMITED broadcast
# (destination = FF:FF:FF:FF:FF:FF at Layer 2 / 255.255.255.255 concept at L3)

sudo tcpdump -i eth0 arp -v
# Every ARP request you see here is a broadcast — visible to ALL hosts
# on your local network (192.168.56.0/24 in your VirtualBox lab)
```

### Lab 2 — Trigger and Observe a Broadcast (Own Lab Only)

```bash
# Send a limited broadcast ping on your Host-Only network
ping -b 192.168.56.255 -c 4
# (-b enables broadcast pings; requires appropriate permissions)

# Watch it arrive at Metasploitable2 (and any other lab host) simultaneously
sudo tcpdump -i eth0 icmp -v
```

### Lab 3 — Identify Your Network's Broadcast Address

```bash
# See your subnet + calculated broadcast address
ip addr show eth0
# e.g. 192.168.56.10/24 → broadcast address = 192.168.56.255

# Or explicitly with ifconfig
ifconfig eth0 | grep -i broadcast
```

### Lab 4 — Observe Multicast Traffic

```bash
# Multicast addresses fall in the Class D range: 224.0.0.0 – 239.255.255.255
# Many local services use multicast, e.g. mDNS (224.0.0.251)

sudo tcpdump -i eth0 net 224.0.0.0/4 -v
# Watch for mDNS/SSDP multicast chatter on your local network —
# a good real-world way to see Class D addressing in action
```

### Lab 5 — Confirm Directed Broadcast is Disabled (Defensive Check)

```bash
# On a router/lab device you control, confirm directed broadcast forwarding
# is off (best practice, prevents Smurf-style amplification)

# Example on a Cisco-style device (if you have GNS3/Packet Tracer set up):
show running-config interface <if>
# Look for: "no ip directed-broadcast"  (should be present/default on modern IOS)
```

---

## 8. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           TYPES OF CASTING — EXAM CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  UNICAST                                                              ║
║  ─────────────────────────────────────────────────────────           ║
║  One-to-one communication                                            ║
║  Uses a unique IP address of the destination host                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  BROADCAST — 2 TYPES                                                  ║
║  ─────────────────────────────────────────────────────────           ║
║  Limited Broadcast:                                                    ║
║    - Sends to ALL hosts in OWN network                                ║
║    - Address = 255.255.255.255                                        ║
║                                                                        ║
║  Direct Broadcast:                                                     ║
║    - Sends to ALL hosts in ANOTHER network                            ║
║    - Address = <Network ID> with all HOST bits set to 1               ║
║    - Example: Network 90.0.0.0 → Direct broadcast = 90.255.255.255   ║
╠══════════════════════════════════════════════════════════════════════╣
║  MULTICAST                                                            ║
║  ─────────────────────────────────────────────────────────           ║
║  Sends to a SPECIFIC GROUP of interested hosts                        ║
║  Managed by: IGMP (Internet Group Management Protocol)               ║
║  Addressing: Class D  (224.0.0.0 – 239.255.255.255)                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  QUICK EXAM ANSWERS                                                    ║
║  ─────────────────────────────────────────────────────────           ║
║  "One to one"                         → Unicast                      ║
║  "255.255.255.255"                    → Limited Broadcast            ║
║  "Network ID + all host bits = 1"     → Direct Broadcast address     ║
║  "Sent to a specific group"           → Multicast                    ║
║  "Protocol used for multicast groups" → IGMP                         ║
║  "Address class used for multicast"   → Class D                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 21)

- [ ] Anycast (bonus 4th type — nearest-node delivery)
- [ ] Broadcast Domain vs Collision Domain
- [ ] VLANs and broadcast domain segmentation
- [ ] IGMP versions (v1/v2/v3) and IGMP snooping in depth

---

_Notes compiled from: Networking Course Lecture 20 — Types of Casting (Unicast, Broadcast, Multicast)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
