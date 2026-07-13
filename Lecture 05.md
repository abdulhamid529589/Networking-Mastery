# 🔗 Network Topologies — Mesh & Star

### Cybersecurity Student Notes | Networking Course — Lecture 05

> **Source:** Gate Smashers — Topology (Part 1)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Topology?](#1-what-is-topology)
2. [Mesh Topology](#2-mesh-topology)
   - [2.1 Structure & Diagram](#21-structure--diagram)
   - [2.2 Number of Cables — Formula](#22-number-of-cables--formula)
   - [2.3 Number of Ports — Formula](#23-number-of-ports--formula)
   - [2.4 Reliability](#24-reliability)
   - [2.5 Cost](#25-cost)
   - [2.6 Security](#26-security)
   - [2.7 Communication Type](#27-communication-type)
   - [2.8 Advantages & Disadvantages](#28-advantages--disadvantages)
3. [Star Topology (Hub Topology)](#3-star-topology-hub-topology)
   - [3.1 Structure & Diagram](#31-structure--diagram)
   - [3.2 Number of Cables & Ports](#32-number-of-cables--ports)
   - [3.3 Reliability — Single Point of Failure](#33-reliability--single-point-of-failure)
   - [3.4 Cost, Security, Communication Type](#34-cost-security-communication-type)
4. [Topology Comparison Table](#4-topology-comparison-table)
5. [Point-to-Point vs Multipoint](#5-point-to-point-vs-multipoint)
6. [🔴 Attack Surface by Topology](#6--attack-surface-by-topology)
7. [🧪 Practical Labs — Your Setup](#7--practical-labs--your-setup)
8. [Exam Formulas & Cheat Sheet](#8-exam-formulas--cheat-sheet)

---

## 1. What is Topology?

**Topology** = The arrangement/layout of devices in a network and how they are connected to each other.

Two aspects:

- **Physical Topology** → Actual physical layout of cables and devices
- **Logical Topology** → How data actually flows (may differ from physical)

### Types of Topologies (Overview)

```
Mesh ── Star ── Bus ── Ring ── Hybrid
```

| Topology   | Key Characteristic                            |
| ---------- | --------------------------------------------- |
| **Mesh**   | Every device connected to every other device  |
| **Star**   | All devices connected to a central hub/switch |
| **Bus**    | All devices share one common cable            |
| **Ring**   | Devices connected in a circular chain         |
| **Hybrid** | Combination of two or more topologies         |

> UGC NET, GATE, and university exams frequently test topology — especially **formula-based questions** on mesh topology. Don't underestimate it.

---

## 2. Mesh Topology

### 2.1 Structure & Diagram

Every device has a **dedicated point-to-point link** to every other device. No sharing.

```
4-Node Mesh:

    [A]───────[B]
     │╲       /│
     │  ╲   /  │
     │    ╳    │
     │  /   ╲  │
     │/       ╲│
    [C]───────[D]

Links: A-B, A-C, A-D, B-C, B-D, C-D = 6 cables
Each node has 3 ports (connected to the other 3)
```

```
5-Node Mesh:

    [A]────[B]
    │╲╲  /╱│
    │  ╲╱  │
    │  [E] │
    │  /╲  │
    │╱╱  ╲╲│
    [C]────[D]

Links: 5×4/2 = 10 cables
Each node has 4 ports
```

---

### 2.2 Number of Cables — Formula

#### Formula

```
Number of cables = NC2 = n(n-1)/2

Where n = number of nodes/devices
```

#### Why this formula?

- Each device connects to every OTHER device → (n-1) connections per device
- But each cable is shared between 2 devices → divide by 2
- This is identical to the **complete graph edge formula** in graph theory

#### Examples

| Nodes (n) | Cables = n(n-1)/2   |
| --------- | ------------------- |
| 4         | 4×3/2 = **6**       |
| 5         | 5×4/2 = **10**      |
| 10        | 10×9/2 = **45**     |
| 20        | 20×19/2 = **190**   |
| 100       | 100×99/2 = **4950** |

> 10 devices = 45 cables. This is why mesh doesn't scale — cost explodes.

---

### 2.3 Number of Ports — Formula

#### Formula

```
Ports per device = n - 1
Total ports      = n × (n-1)
```

#### Why?

Each device needs one port for each of its connections. With n devices, each connects to (n-1) others → needs (n-1) ports.

#### Examples

| Nodes (n) | Ports per device | Total ports |
| --------- | ---------------- | ----------- |
| 4         | 3                | 12          |
| 5         | 4                | 20          |
| 10        | 9                | 90          |

---

### 2.4 Reliability

**Reliability = Very High (HIGHEST among all topologies)**

```
A wants to send to D:

Primary path:   A ──────────────────► D
If cable fails: A ──► B ──► D          (via B)
                A ──► C ──► D          (via C)
                A ──► B ──► C ──► D    (longer path)
```

- Multiple **redundant paths** exist between any two nodes
- **Single cable failure** does NOT stop communication
- Network continues operating — just reroutes
- This property = **fault tolerance**

> **Exam question:** "Which topology provides highest reliability?" → **Mesh**

---

### 2.5 Cost

**Cost = Very High**

Cost drivers:

- **Cables** → n(n-1)/2 cables needed (grows quadratically)
- **Ports** → n(n-1) total ports needed
- **Installation labor** → massive cable management
- **Maintenance** → more cables = more failure points to monitor

```
Cost comparison:
Mesh > Star > Ring > Bus
(most expensive to least expensive)
```

---

### 2.6 Security

**Security = High**

```
A sends message directly to D via dedicated cable A-D:
  [A] ──(private wire)──► [D]

Does B see this message? NO.
Does C see this message? NO.

Each link is DEDICATED — no sharing → no eavesdropping possible
without physical access to that specific cable.
```

- Point-to-point dedicated links = private channel between each pair
- No passive sniffing possible from another node (unlike bus/hub)
- **Most secure topology** among traditional layouts

---

### 2.7 Communication Type

**Point-to-Point (P2P)** — NOT multipoint/multicast

```
Point-to-Point:   [A] ──dedicated wire──► [D]   (only A and D use this wire)

Multipoint:       [A]─┬─[B]─┬─[C]─┬─[D]        (all share one wire)
                      └─────┴─────┘
                         shared bus
```

- In mesh, every cable is **dedicated** between exactly 2 devices
- **No cable sharing** → no collisions
- Does **NOT support multicasting** (sending one message to many simultaneously on shared medium)

---

### 2.8 Advantages & Disadvantages

| ✅ Advantages                        | ❌ Disadvantages                |
| ------------------------------------ | ------------------------------- |
| Highest reliability (fault tolerant) | Very expensive (cables + ports) |
| High security (dedicated links)      | Difficult installation          |
| No traffic congestion                | High maintenance (many cables)  |
| Easy fault isolation                 | Doesn't scale well              |
| Simultaneous communication possible  | No multicasting support         |

**Real-world use:** Internet backbone (ISP interconnections), military networks, critical infrastructure — where reliability and security matter more than cost.

---

## 3. Star Topology (Hub Topology)

### 3.1 Structure & Diagram

All devices connect to a **central device** (hub or switch). No direct device-to-device links.

```
          [A]
           │
    [D] ──[HUB]── [B]
           │
          [C]

Central device = Hub (Layer 1) or Switch (Layer 2)
```

```
Message flow (A → C via Hub):
  [A] ──► [HUB] ──► [C]
              │
              └──► [B] (also receives if hub — broadcast)
              └──► [D] (also receives if hub — broadcast)
```

---

### 3.2 Number of Cables & Ports

#### Cables

```
Number of cables = n (one per device)

4 devices → 4 cables
10 devices → 10 cables
```

**Why?** Each device has exactly ONE cable going to the central hub.

#### Ports on each device

```
Ports per end device = 1 (one connection to hub)
Total ports on end devices = n
Hub ports = n (one per connected device)
```

---

### 3.3 Reliability — Single Point of Failure

**Reliability = LOW**

```
If HUB fails:
  [A]  [B]  [C]  [D]  ← all isolated, cannot communicate
        ☠️ HUB DEAD ☠️

Single Point of Failure (SPOF) = Hub
```

**Single Point of Failure (SPOF):**

> One component whose failure brings down the entire system.

In star topology, the hub is the SPOF:

- Hub fails → entire network fails
- This is the biggest weakness of star topology

**Mitigation:** Use **redundant central devices** (two switches in parallel) or **switch instead of hub** for better management.

---

### 3.4 Cost, Security, Communication Type

#### Cost

```
Cost = Moderate (normal)

Cable cost:     Low (only n cables, not n(n-1)/2)
Hub/Switch cost: Moderate to High (depending on port count and type)
Overall:         Less than Mesh, manageable
```

#### Security

**Security = LOW (when using Hub)**

```
A sends message to C:
Hub broadcasts to ALL ports → A, B, C, D ALL receive the frame

[A] sends "Hello C" ──► HUB broadcasts ──► [B] receives ← security issue!
                                        ──► [C] receives ✓
                                        ──► [D] receives ← security issue!
```

- **Hub** = dumb Layer 1 device — broadcasts to ALL ports
- Any device on the network can see all traffic → passive sniffing works
- **Hub has no intelligence** to decide "send only to C"

**With Switch (Layer 2):**

```
Switch learns MAC addresses → builds CAM table
A sends to C → Switch looks up C's MAC → sends ONLY to C's port
B and D don't receive it → better security
```

> **Hub** = broadcast → insecure
> **Switch** = selective forwarding → more secure (but still vulnerable to ARP spoofing)

#### Communication Type

**Point-to-Point** — same as mesh

- Each device has dedicated cable to the hub
- No cable sharing between end devices
- Hub then forwards (broadcast for hub, unicast for switch)
- Does **NOT support native multipoint** (though hub effectively broadcasts = forced multipoint)

---

## 4. Topology Comparison Table

| Parameter        | Mesh               | Star (Hub)     | Bus        | Ring            |
| ---------------- | ------------------ | -------------- | ---------- | --------------- |
| **Cables**       | n(n-1)/2           | n              | 1          | n               |
| **Ports/device** | n-1                | 1              | 1          | 2               |
| **Total ports**  | n(n-1)             | n              | n          | n               |
| **Reliability**  | Highest ⭐⭐⭐⭐⭐ | Low ⭐⭐       | Low ⭐⭐   | Moderate ⭐⭐⭐ |
| **SPOF**         | None               | Hub/Switch     | Bus cable  | Any node        |
| **Cost**         | Very High          | Moderate       | Low        | Moderate        |
| **Security**     | High               | Low (Hub)      | Very Low   | Moderate        |
| **Comm. type**   | Point-to-Point     | Point-to-Point | Multipoint | Point-to-Point  |
| **Scalability**  | Very Low           | High           | Moderate   | Low             |
| **Maintenance**  | Hard               | Easy           | Easy       | Moderate        |
| **Real use**     | Backbone, Military | Office LAN     | Legacy     | Token Ring      |

---

## 5. Point-to-Point vs Multipoint

This distinction comes up frequently in exams:

### Point-to-Point

```
[Device A] ──dedicated link──► [Device B]

- One sender, one receiver on each link
- No sharing of the medium
- Private, no collision
- Examples: Mesh links, Star links (device to hub)
```

### Multipoint (Shared / Multi-access)

```
[A]───┬───[B]───┬───[C]───┬───[D]
      └─────────┴─────────┘
              shared bus

- Multiple devices share ONE medium
- Collision possible → need CSMA/CD
- Anyone can see anyone's traffic
- Examples: Bus topology, Hub-based networks (hub broadcasts = effectively multipoint)
```

| Property       | Point-to-Point     | Multipoint       |
| -------------- | ------------------ | ---------------- |
| Medium sharing | No                 | Yes              |
| Collision      | No                 | Possible         |
| Privacy        | High               | Low              |
| Multicasting   | Not natural        | Natural          |
| Examples       | Mesh, leased lines | Bus, Hub network |

---

## 6. 🔴 Attack Surface by Topology

### Mesh Topology Attacks

| Attack                | Difficulty     | Why                                                 |
| --------------------- | -------------- | --------------------------------------------------- |
| **Passive sniffing**  | Very Hard      | Dedicated links — must tap specific cable           |
| **Physical wiretap**  | Hard           | Must access specific physical cable between 2 nodes |
| **DoS via cable cut** | Hard to impact | Redundant paths exist — resilient                   |
| **Node compromise**   | Moderate       | Compromise one node → only see that node's traffic  |

**Security assessment:** Mesh is the most attack-resistant topology at physical level. But if you compromise a node (not the cable), you can still intercept that node's communications.

---

### Star Topology (Hub) Attacks

| Attack                     | Difficulty             | Why                                          |
| -------------------------- | ---------------------- | -------------------------------------------- |
| **Passive sniffing**       | **Very Easy**          | Hub broadcasts everything to all ports       |
| **ARP Spoofing**           | Not even needed on hub | Already see all traffic                      |
| **Hub port monitoring**    | Easy                   | Plug into any port → see all traffic         |
| **DoS via hub attack**     | Easy                   | Crash/flood hub → entire network down (SPOF) |
| **Physical access to hub** | Single target          | Control hub = control entire network         |

```bash
# On a hub-based network — passive sniff (NO attack needed!)
sudo tcpdump -i eth0 -v
# You will see ALL traffic on the hub network
# This is why hub networks are a serious security risk

# On your VirtualBox lab — simulate hub behavior with ARP spoof
# (switches need ARP spoof first to expose all traffic)
sudo arpspoof -i eth0 -t 192.168.56.101 192.168.56.1 &
sudo tcpdump -i eth0 -v
# Now you see traffic just like a hub network
```

### Star Topology (Switch) Attacks

| Attack                   | Difficulty      | Why                                       |
| ------------------------ | --------------- | ----------------------------------------- |
| **ARP Spoofing**         | Moderate        | Trick switch's CAM table → become MitM    |
| **MAC Flooding**         | Moderate        | Overflow CAM table → switch acts like hub |
| **Port mirroring abuse** | If admin access | Mirror all traffic to your port           |
| **VLAN hopping**         | Harder          | Need 802.1Q double-tagging trick          |

```bash
# MAC Flooding — force switch to act like a hub
sudo macof -i eth0
# macof generates thousands of random MAC addresses
# Switch CAM table fills up → switch starts broadcasting (like hub)
# Now passive sniffing works on the switch network!

# Monitor the attack effect
sudo tcpdump -i eth0 | wc -l
# Packet count should spike as switch starts broadcasting everything
```

---

## 7. 🧪 Practical Labs — Your Setup

### Lab 1 — Identify Your VirtualBox Topology

```bash
# Your VirtualBox lab IS a Star topology:
# Parrot OS ──► VirtualBox virtual switch ◄── Metasploitable2

# The VirtualBox NAT/Host-Only network acts like a SWITCH (not hub)
# That's why you need ARP spoofing to see Metasploitable2's traffic

# Verify your network setup
ip addr show eth0
# Note your IP: 192.168.56.x (Host-Only range)

# See all devices on your star topology
nmap -sn 192.168.56.0/24
# Your Parrot OS + Metasploitable2 = 2 nodes in star topology
```

### Lab 2 — SPOF Simulation (Star Topology Weakness)

```bash
# Simulate single point of failure — disable the "hub" (your NIC)
# First, ping Metasploitable2 continuously
ping 192.168.56.101 &

# Now disable your network interface (simulates hub failure)
sudo ip link set eth0 down

# Ping fails immediately — this is the SPOF effect
# Re-enable:
sudo ip link set eth0 up
# Ping resumes — network restored

# In a real star network: if the switch fails, ALL nodes lose connectivity
```

### Lab 3 — MAC Flooding (Force Switch to Act Like Hub)

```bash
# Install macof (part of dsniff)
sudo apt install dsniff

# Start capturing BEFORE flooding
sudo tcpdump -i eth0 -w /tmp/before_flood.pcap &
sleep 5

# Start MAC flooding
sudo macof -i eth0 &
FLOOD_PID=$!
sleep 10

# Capture DURING flooding
sudo tcpdump -i eth0 -w /tmp/during_flood.pcap &
sleep 10

# Stop flooding
kill $FLOOD_PID

# Compare packet counts
tcpdump -r /tmp/before_flood.pcap | wc -l
tcpdump -r /tmp/during_flood.pcap | wc -l
# During flood: higher count = switch broadcasting = acting like hub
```

### Lab 4 — Mesh Topology Security Simulation

```bash
# Mesh topology security = dedicated links
# Simulate: can Parrot OS sniff traffic NOT meant for it?

# WITHOUT ARP spoofing — try to sniff Metasploitable2's traffic
sudo tcpdump -i eth0 not arp and not broadcast

# You will only see traffic TO or FROM your machine
# Traffic between Metasploitable2 and gateway → NOT visible
# This simulates the security of point-to-point dedicated links

# WITH ARP spoofing — now you CAN see it (breaks the mesh security model)
sudo arpspoof -i eth0 -t 192.168.56.101 192.168.56.1 &
sudo tcpdump -i eth0 not arp
# Now you see Metasploitable2's traffic = topology security bypassed
```

### Lab 5 — Map Topology of Your Network

```bash
# Discover network topology through scanning
nmap -sn 192.168.56.0/24 -oX /tmp/hosts.xml

# See connections (star topology visible through shared gateway)
traceroute 192.168.56.101
# Only 1 hop = directly connected (star topology — no intermediate routers)

traceroute 8.8.8.8
# Multiple hops = WAN topology (hybrid — star at edges, mesh at core)
# Each hop is a router = a node in a larger topology
```

---

## 8. Exam Formulas & Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              TOPOLOGY — EXAM CHEAT SHEET                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  MESH TOPOLOGY FORMULAS                                              ║
║  ─────────────────────────────────────────────────────────          ║
║  Cables (links)    = n(n-1)/2   = NC2                               ║
║  Ports per device  = n - 1                                          ║
║  Total ports       = n(n-1)                                         ║
║                                                                      ║
║  Quick examples:                                                     ║
║    n=4  → cables=6,  ports/device=3,  total ports=12               ║
║    n=5  → cables=10, ports/device=4,  total ports=20               ║
║    n=10 → cables=45, ports/device=9,  total ports=90               ║
╠══════════════════════════════════════════════════════════════════════╣
║  STAR TOPOLOGY FORMULAS                                              ║
║  ─────────────────────────────────────────────────────          ║
║  Cables            = n                                              ║
║  Ports per device  = 1                                              ║
║  Total ports       = n (end devices) + n (hub ports) = 2n          ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOPOLOGY QUICK COMPARISON                                           ║
║  ─────────────────────────────────────────────────────          ║
║  Reliability:  Mesh > Ring > Star > Bus                             ║
║  Cost:         Mesh > Star > Ring > Bus                             ║
║  Security:     Mesh > Star(Switch) > Ring > Star(Hub) > Bus        ║
║  Scalability:  Star > Bus > Ring > Mesh                             ║
║  Cables used:  Bus(1) < Ring(n) = Star(n) << Mesh(n(n-1)/2)        ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY EXAM FACTS                                                      ║
║  ─────────────────────────────────────────────────────          ║
║  Mesh   → Point-to-Point, no SPOF, highest reliability             ║
║  Star   → Point-to-Point, SPOF = central device                    ║
║  Hub    → broadcasts to ALL (insecure, easy to sniff)              ║
║  Switch → sends to specific port (more secure than hub)            ║
║  SPOF   → Single Point of Failure (star topology weakness)         ║
║  Mesh cables formula = complete graph edges formula                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY QUICK MAP                                                  ║
║  ─────────────────────────────────────────────────────          ║
║  Hub network   → passive sniff works (no attack needed)            ║
║  Switch network→ need ARP spoof or MAC flood first                 ║
║  Mesh network  → need physical wire tap or node compromise         ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Part 2 of Topology)

- [ ] Bus Topology — single shared cable, CSMA/CD
- [ ] Ring Topology — token passing, FDDI
- [ ] Hybrid Topology — real-world combinations
- [ ] Logical vs Physical Topology differences

---

_Notes compiled from: Networking Course Lecture 05 — Topology Part 1 (Mesh & Star)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
