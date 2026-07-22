# 🔗 Network Topologies — Bus & Ring

### Networking Course — Lecture 06

> **Source:** Gate Smashers — Topology (Part 2)

> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Recap — Topology So Far](#1-recap--topology-so-far)
2. [Bus Topology](#2-bus-topology)
   - [2.1 Structure & Diagram](#21-structure--diagram)
   - [2.2 Number of Cables — Formula](#22-number-of-cables--formula)
   - [2.3 Number of Ports — Formula](#23-number-of-ports--formula)
   - [2.4 Reliability](#24-reliability)
   - [2.5 Security](#25-security)
   - [2.6 Cost](#26-cost)
   - [2.7 Terminator & Repeater](#27-terminator--repeater)
   - [2.8 Collision & Multipoint Nature](#28-collision--multipoint-nature)
   - [2.9 Advantages & Disadvantages](#29-advantages--disadvantages)
3. [Ring Topology](#3-ring-topology)
   - [3.1 Structure & Diagram](#31-structure--diagram)
   - [3.2 Number of Cables & Ports](#32-number-of-cables--ports)
   - [3.3 Reliability](#33-reliability)
   - [3.4 Security, Cost, Direction](#34-security-cost-direction)
   - [3.5 Token Ring](#35-token-ring)
4. [Topology Comparison Table](#4-topology-comparison-table)
5. [🔴 Attack Surface by Topology](#5--attack-surface-by-topology)
6. [🧪 Practical Labs — Your Setup](#6--practical-labs--your-setup)
7. [Exam Formulas & Cheat Sheet](#7-exam-formulas--cheat-sheet)

---

## 1. Recap — Topology So Far

| Topology | Cables   | Ports  | Reliability | Security  | Cost      |
| -------- | -------- | ------ | ----------- | --------- | --------- |
| Mesh     | n(n-1)/2 | n(n-1) | Highest     | High      | Very High |
| Star     | n        | 2n     | Low (SPOF)  | Depends\* | Moderate  |

\*Star security depends on hub (broadcasts, insecure) vs switch (selective, more secure).

This part covers the two remaining traditional physical topologies: **Bus** and **Ring**.

---

## 2. Bus Topology

### 2.1 Structure & Diagram

All devices connect to a single shared central cable — the **backbone**.

```
Backbone / Coaxial Cable ("thick Ethernet")
══════╦═══════╦═══════╦═══════╦══════
      │       │       │       │
    [tap]   [tap]   [tap]   [tap]
      │       │       │       │
     [A]     [B]     [C]     [D]
      (drop line = wire from tap to device)

[T]═══...═══[T]   ← Terminator at each end of the backbone
```

| Component          | Description                                                                                                                        |
| ------------------ | ---------------------------------------------------------------------------------------------------------------------------------- |
| **Backbone cable** | The central cable. Also called **coaxial cable** / **thick Ethernet wire** (needs high bandwidth since multiple devices share it). |
| **Drop line**      | Wire connecting a device to the backbone.                                                                                          |
| **Tap**            | Electrical device joining a drop line to the backbone.                                                                             |
| **Terminator**     | Placed at each end of the backbone to absorb the signal and prevent reflection.                                                    |

---

### 2.2 Number of Cables — Formula

```
Number of cables = n + 1

Where n = number of devices
(n drop lines + 1 backbone cable)
```

| Devices (n) | Cables = n + 1 |
| ----------- | -------------- |
| 4           | **5**          |
| 10          | **11**         |
| 20          | **21**         |

> ⚠️ **Exam trap:** Students commonly forget the `+1` for the backbone itself and only count drop lines.

---

### 2.3 Number of Ports — Formula

```
Ports per device = 1
Total ports       = n
```

Each device only needs one port to connect its single drop line to the shared backbone.

---

### 2.4 Reliability

**Reliability = Very Low**

```
A ──┬── B ──┬── C ──┬── D
    │       │       │
 backbone segment fails here (✗)
    │
A can still reach the local segment,
but if the BACKBONE itself breaks →
entire network stops working.
```

- Single point of failure = **the backbone cable**
- If the backbone fails at any point, the **whole network goes down** — there is no alternate path (unlike mesh)
- This is the same fundamental weakness as star topology's hub, but here the SPOF is the _cable itself_, not a device

---

### 2.5 Security

**Security = Very Low (Lowest of all topologies)**

```
A sends "Hello B" on the backbone:

A ──► [backbone broadcasts to everyone] ──► B  (intended recipient)
                                        ──► C  (also receives!)
                                        ──► D  (also receives!)

Only B's NIC keeps the frame (matching destination address);
C and D's NICs normally discard it — but a listening NIC
in "promiscuous mode" on C or D can capture it anyway.
```

- A cable **cannot filter or direct traffic** — filtering requires an intelligent device (switch, router)
- Every message is broadcast to **all devices** on the segment
- Any device in promiscuous mode can passively sniff all traffic — this is the same problem as **hub-based star networks**

---

### 2.6 Cost

**Cost = Low (cheaper than Mesh, cheaper than Star in cabling)**

- Fewer total cables than mesh (`n+1` vs `n(n-1)/2`)
- No expensive central device (hub/switch) required — just cable, taps, and terminators
- Historically the cheapest LAN topology to deploy for small networks

---

### 2.7 Terminator & Repeater

| Device         | Purpose                                                                                                                                                                    |
| -------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Terminator** | Installed at each end of the backbone; absorbs the signal so it doesn't reflect back down the wire and cause interference.                                                 |
| **Repeater**   | Regenerates/boosts a weakening signal so it can travel further — e.g., a cable segment limited to ~500m can be extended to 1km by joining two segments through a repeater. |

```
Segment 1 (500m) ══[Repeater]══ Segment 2 (500m)  =  Effective 1km bus
```

---

### 2.8 Collision & Multipoint Nature

Bus is a **multipoint network** — one wire, multiple devices sharing it simultaneously (as opposed to mesh/star's point-to-point links).

```
Multipoint sharing:

[A]──┬──[B]──┬──[C]──┬──[D]
     └───────┴───────┘
        one shared wire

If A, B, C, D all transmit at once → collision
```

- **Maximum theoretical collisions ≈ n** (if all n devices transmit simultaneously)
- Collision-handling mechanisms used on bus networks:
  - **Token passing** — a token controls transmission rights
  - **CSMA/CD** (Carrier Sense Multiple Access with Collision Detection) — devices listen before transmitting, and detect/react to collisions

---

### 2.9 Advantages & Disadvantages

| ✅ Advantages                      | ❌ Disadvantages                                     |
| ---------------------------------- | ---------------------------------------------------- |
| Cheap — least cabling              | Entire network fails if backbone breaks              |
| Easy to install for small networks | No security — broadcasts to everyone                 |
| Requires less cable than mesh/star | High collision potential (multipoint)                |
| Good for small, temporary LANs     | Limited cable length (needs repeaters)               |
|                                    | Difficult to troubleshoot a break in a long backbone |

**Real-world use:** Mostly **legacy** — early Ethernet (10BASE2/10BASE5) LANs. Modern networks use switched star topology instead.

---

## 3. Ring Topology

### 3.1 Structure & Diagram

Formed by connecting **both ends of a bus** together into a closed loop.

```
        [A]
       /    \
    [D]      [B]
       \    /
        [C]

One device (e.g., a designated node) may act as a MONITOR,
overseeing the ring's operation.
Data typically flows in ONE direction around the loop.
```

> **Key concept:** Ring = Bus with its two open ends joined into a circle.

---

### 3.2 Number of Cables & Ports

```
Cables = n + 1   (same as bus: n drop lines + 1 backbone/loop, conceptually)
Ports  = n       (1 port per device)
```

| Devices (n) | Cables | Ports |
| ----------- | ------ | ----- |
| 4           | 5      | 4     |
| 10          | 11     | 10    |

---

### 3.3 Reliability

**Reliability = Low**

```
A ── B ── C ── D ── (back to A)
        ✗ break here

If the link between B and C breaks:
Data is an ELECTRIC SIGNAL on a closed circuit.
When the circuit breaks, NO device can communicate —
even A and B, which are not near the break, are affected,
because the whole ring depends on circuit continuity.
```

- Any single link failure can potentially disrupt the **entire ring**, not just the two devices at the break
- This is a key exam distinction from bus, where only the broken segment/backbone stops functioning — in ring, the closed electrical loop itself is the dependency

---

### 3.4 Security, Cost, Direction

| Property      | Ring Topology                                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **Security**  | Low — same broadcast problem as bus; a transmitted message reaches every device on the ring.                                          |
| **Cost**      | Roughly similar to bus (comparable cabling requirements).                                                                             |
| **Direction** | **Unidirectional** — data travels in only one direction around the ring (a key difference from bus, which has no inherent direction). |

---

### 3.5 Token Ring

- Ring topology commonly uses a **token** — a special control frame — to regulate which device may transmit at any given time.
- This access method is called **Token Ring**.
- It reduces (but does not eliminate, compared to mesh) the chance of collisions, since only the device holding the token can transmit.

---

## 4. Topology Comparison Table

| Parameter          | Mesh               | Star (Hub)     | Bus                | Ring                     |
| ------------------ | ------------------ | -------------- | ------------------ | ------------------------ |
| **Cables**         | n(n-1)/2           | n              | n + 1              | n + 1                    |
| **Ports/device**   | n-1                | 1              | 1                  | 1                        |
| **Total ports**    | n(n-1)             | n              | n                  | n                        |
| **Reliability**    | Highest ⭐⭐⭐⭐⭐ | Low ⭐⭐       | Very Low ⭐        | Low ⭐⭐                 |
| **SPOF**           | None               | Hub/Switch     | Backbone cable     | Any single link          |
| **Cost**           | Very High          | Moderate       | Low                | Low–Moderate             |
| **Security**       | High               | Low (Hub)      | Very Low           | Low                      |
| **Comm. type**     | Point-to-Point     | Point-to-Point | Multipoint         | Point-to-Point (loop)    |
| **Direction**      | N/A                | N/A            | No fixed direction | Unidirectional           |
| **Access control** | N/A                | N/A            | CSMA/CD, Token     | Token Ring               |
| **Scalability**    | Very Low           | High           | Moderate           | Low                      |
| **Real use**       | Backbone, Military | Office LAN     | Legacy Ethernet    | Legacy Token Ring / FDDI |

---

## 5. 🔴 Attack Surface by Topology

### Bus Topology Attacks

| Attack                     | Difficulty    | Why                                                                                                                                                      |
| -------------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Passive sniffing**       | **Very Easy** | The backbone broadcasts every frame to all connected devices — functionally identical to a hub network. Any NIC in promiscuous mode captures everything. |
| **DoS via backbone cut**   | Very Easy     | Cutting or damaging the single backbone cable takes down the _entire_ network — no redundancy exists.                                                    |
| **Physical tap injection** | Moderate      | An attacker with physical access can add an unauthorized tap/drop line to eavesdrop or inject traffic.                                                   |
| **Collision-based DoS**    | Moderate      | Flooding the shared medium with traffic increases collisions for all devices, degrading throughput for everyone.                                         |

> Bus networks share the same fundamental sniffing weakness as hub-based star networks — a shared broadcast medium — because both are effectively "multipoint" from a traffic-visibility standpoint.

---

### Ring Topology Attacks

| Attack                               | Difficulty | Why                                                                                                                                                              |
| ------------------------------------ | ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Passive sniffing**                 | Easy       | Every frame traverses the entire ring (or at least passes near many nodes) before reaching its destination or being removed by the sender — visibility is broad. |
| **DoS via single link break**        | Very Easy  | Breaking one link disrupts the whole closed-loop circuit, unlike bus where only the affected segment stops.                                                      |
| **Monitor/token station compromise** | Moderate   | If the ring depends on a designated monitor station to manage the token, compromising or disabling that node can disrupt token circulation for the whole ring.   |
| **Token manipulation**               | Harder     | Historically, forging or withholding a token could deny transmission rights to other stations (legacy Token Ring era).                                           |

---

## 6. 🧪 Practical Labs — Your Setup

> Note: Modern LANs (including your VirtualBox lab) are switched star topologies, not bus/ring. These labs simulate the _behavior_ of bus/ring networks so you can observe the underlying security properties described above.

### Lab 1 — Simulate a Bus/Hub-Style Broadcast Domain

```bash
# A hub (or bus backbone) broadcasts to everyone — simulate this
# by forcing your switched lab network into a broadcast-like state.

sudo apt install dsniff

# Flood the switch's CAM table so it starts broadcasting like a bus/hub
sudo macof -i eth0 &
sleep 10

# Now passively capture — you should start seeing traffic
# that wasn't originally destined for you (bus/hub-like behavior)
sudo tcpdump -i eth0 -v
```

### Lab 2 — Simulate Bus Topology Single Point of Failure

```bash
# The bus backbone = SPOF. Simulate a "cable cut" on your host-only adapter.

ping 192.168.56.101 &

# "Cut the backbone" by disabling your NIC
sudo ip link set eth0 down

# Ping fails immediately for ALL simulated devices sharing this "backbone"
# Restore:
sudo ip link set eth0 up
```

### Lab 3 — Observe Broadcast Traffic Visibility (Bus/Ring Security Weakness)

```bash
# Capture only broadcast and multicast traffic — this is what EVERY
# device on a real bus or ring network would also see, by design.

sudo tcpdump -i eth0 broadcast or multicast -v

# Compare this to Lab 4 in Part 5 (mesh point-to-point simulation),
# where only traffic to/from your own machine was visible.
# The contrast demonstrates why bus/ring/hub networks are
# inherently less private than mesh or switched point-to-point links.
```

### Lab 4 — Map Your Lab as a Logical Bus

```bash
# Even on a switch, you can reason about it as a logical bus
# for VLAN or broadcast-domain analysis.

nmap -sn 192.168.56.0/24

# Everything within the same broadcast domain (VLAN/subnet)
# effectively shares "bus-like" broadcast visibility for ARP,
# DHCP, and other broadcast traffic — even though physically
# it's a switched star.
arping -c 3 192.168.56.101
```

---

## 7. Exam Formulas & Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              BUS & RING TOPOLOGY — EXAM CHEAT SHEET                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  BUS TOPOLOGY FORMULAS                                                ║
║  ─────────────────────────────────────────────────────────          ║
║  Cables            = n + 1   (n drop lines + 1 backbone)             ║
║  Ports per device  = 1                                                ║
║  Total ports       = n                                                ║
║                                                                        ║
║  Quick examples:                                                       ║
║    n=4  → cables=5,  ports=4                                          ║
║    n=10 → cables=11, ports=10                                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  RING TOPOLOGY FORMULAS                                               ║
║  ─────────────────────────────────────────────────────          ║
║  Cables            = n + 1   (same structure as bus)                 ║
║  Ports per device  = 1                                                ║
║  Total ports       = n                                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY EXAM FACTS                                                        ║
║  ─────────────────────────────────────────────────────          ║
║  Bus   → Multipoint, SPOF = backbone cable, cheapest topology        ║
║  Ring  → Point-to-point loop, unidirectional, uses Token Ring        ║
║  Bus   → uses CSMA/CD or token passing for collision control         ║
║  Ring  → single link break can disrupt the WHOLE ring (closed circuit)║
║  Terminator → absorbs signal at bus cable ends (prevents reflection) ║
║  Repeater   → regenerates signal to extend cable distance            ║
╠══════════════════════════════════════════════════════════════════════╣
║  RELIABILITY / SECURITY / COST RANKING (ALL 4 TOPOLOGIES)            ║
║  ─────────────────────────────────────────────────────          ║
║  Reliability:  Mesh > Ring > Star > Bus                              ║
║  Cost:         Mesh > Star > Ring ≈ Bus                              ║
║  Security:     Mesh > Star(Switch) > Ring > Star(Hub) > Bus         ║
║  Scalability:  Star > Bus > Ring > Mesh                              ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY QUICK MAP                                                    ║
║  ─────────────────────────────────────────────────────          ║
║  Bus network   → passive sniff trivially easy (shared backbone)      ║
║  Ring network  → broad visibility as frame traverses the loop        ║
║  Hub network   → passive sniff works (no attack needed)              ║
║  Switch network→ need ARP spoof or MAC flood first                   ║
║  Mesh network  → need physical wire tap or node compromise           ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 3 of Topology)

- [ ] Hybrid Topology — real-world combinations
- [ ] Logical vs Physical Topology differences
- [ ] Comparing all 5 topologies together (final revision)

---

_Notes compiled from: Networking Course Lecture 06 — Topology Part 2 (Bus & Ring)_

_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
