# 🔗 IP Addressing — Classful Addressing (Solving Questions)

### Cybersecurity Student Notes | Networking Course — Lecture 19

> **Source:** Gate Smashers — Classful IP Addressing (Solved Question)
> **Exam Relevance:** GATE (high — numericals) · UGC NET · Banking Exams · University Exams
> **Topic Area:** Network Layer — IP Addressing

---

## 📑 Table of Contents

1. [What is Classful IP Addressing?](#1-what-is-classful-ip-addressing)
2. [IP Address Structure](#2-ip-address-structure)
3. [Class Ranges — Quick Reference](#3-class-ranges--quick-reference)
4. [Step-by-Step Problem-Solving Approach](#4-step-by-step-problem-solving-approach)
5. [Worked Example — IP Address: 201.20.30.40](#5-worked-example--ip-address-2012030-40)
   - [Step 1 — Find the Class](#step-1--find-the-class)
   - [Step 2 — Find the Default Mask](#step-2--find-the-default-mask)
   - [Step 3 — Find the Network ID (AND Operation)](#step-3--find-the-network-id-and-operation)
   - [Step 4 — Find the 4th Host](#step-4--find-the-4th-host)
   - [Step 5 — Find the Last Host](#step-5--find-the-last-host)
   - [Step 6 — Find the Broadcast Addresses](#step-6--find-the-broadcast-addresses)
6. [Broadcast Address Types — Limited vs Direct](#6-broadcast-address-types--limited-vs-direct)
7. [Reserved IP Addresses in a Network (Summary)](#7-reserved-ip-addresses-in-a-network-summary)
8. [General Formulas for Any Classful Address](#8-general-formulas-for-any-classful-address)
9. [Why This Matters for Cybersecurity](#9-why-this-matters-for-cybersecurity)
10. [Exam Cheat Sheet](#10-exam-cheat-sheet)

---

## 1. What is Classful IP Addressing?

**Classful IP Addressing** is the original, pre-CIDR system of dividing the IPv4 address space into fixed classes (A, B, C, D, E). Each class has a predetermined **default subnet mask** and a fixed split between **Network bits** and **Host bits**.

- This lecture works through a **full numerical problem** of the type commonly set in GATE, UGC NET, banking, and university exams.
- The same step-by-step method applies to **any** classful IP addressing question.

---

## 2. IP Address Structure

An IPv4 address is **32 bits** long, divided into **four octets** (groups of 8 bits each), written in **dotted-decimal notation**:

```
  201   .   20   .   30   .   40
  ─────     ───     ───     ───
  8 bits   8 bits  8 bits  8 bits
  ──────────────────────────────
           32 bits total
```

Each decimal number represents 8 binary bits — hence the name **octet** (octet = 8).

---

## 3. Class Ranges — Quick Reference

**To find the class of an IP address, look ONLY at the first octet.**

| Class | First Octet Range | Default Mask                        | Network / Host Bits            |
| ----- | ----------------- | ----------------------------------- | ------------------------------ |
| **A** | 0 – 127           | 255.0.0.0                           | 8 bits network / 24 bits host  |
| **B** | 128 – 191         | 255.255.0.0                         | 16 bits network / 16 bits host |
| **C** | 192 – 223         | 255.255.255.0                       | 24 bits network / 8 bits host  |
| D     | 224 – 239         | (Multicast — not for general hosts) | —                              |
| E     | 240 – 255         | (Reserved/Experimental)             | —                              |

> **Exam tip:** You must remember the range boundaries — especially **Class C: 192 to 223**, as it is by far the most commonly tested.

---

## 4. Step-by-Step Problem-Solving Approach

When given any IP address and asked to find various values, always follow this order:

```
Step 1 → Find the CLASS (look at the first octet only)
Step 2 → Find the DEFAULT MASK (depends on class)
Step 3 → AND the IP address with the mask → NETWORK ID
Step 4 → Find any specific HOST (Network ID + host number)
Step 5 → Find the LAST HOST (last IP address − 1)
Step 6 → Find BROADCAST ADDRESSES (limited and direct)
```

---

## 5. Worked Example — IP Address: 201.20.30.40

### Step 1 — Find the Class

Look at the **first octet** only: **201**

```
Class C range: 192 – 223
201 falls within 192–223 → Class C
```

### Step 2 — Find the Default Mask

Class C default mask = **255.255.255.0**

```
255 . 255 . 255 . 0
 ↑      ↑     ↑    ↑
All 1s All 1s All 1s All 0s
(Network bits)       (Host bits)
```

### Step 3 — Find the Network ID (AND Operation)

**Perform bitwise AND** between the IP address and the default mask:

```
IP Address:  201  .  20  .  30  .  40
Mask:        255  . 255  . 255  .   0
             ─────────────────────────
             AND  AND    AND    AND

Result:      201  .  20  .  30  .   0
```

**Shortcut (saves exam time):**

- `Any value AND 255 = that same value` (because 255 = `11111111` in binary — ANDing any value with all-1s returns the value unchanged).
- `Any value AND 0 = 0` (because 0 = `00000000` in binary — ANDing with all-0s always gives 0).

**So:**

- `201 AND 255 = 201` (copy as-is)
- `20 AND 255 = 20` (copy as-is)
- `30 AND 255 = 30` (copy as-is)
- `40 AND 0 = 0`

```
Network ID = 201.20.30.0
```

> Think of this as the "address of the organization" — let's call this organization **ABC.com**. The rest of the world identifies this entire network by the address **201.20.30.0**.

### Step 4 — Find the 4th Host

In **Class C**, only the **last (4th) octet** represents hosts — the first three octets are the network ID and **must not be changed**.

```
Host addresses within 201.20.30.0:

 .0   → Network ID (reserved — not assignable to any host)
 .1   → 1st host
 .2   → 2nd host
 .3   → 3rd host
 .4   → 4th host  ← ANSWER
  ...
 .254 → Last host
 .255 → Broadcast (reserved — not assignable to any host)
```

**Why start from .1 (not .0)?** Because `.0` is the **Network ID** itself — no host can be assigned this address; it identifies the network as a whole.

```
4th Host = 201.20.30.4
```

> **Quick rule:** For the Nth host in Class C, simply append `.N` to the network base address. (`10th host = 201.20.30.10`, and so on.)

### Step 5 — Find the Last Host

The last octet in Class C can hold values from `0` to `255` (256 possible values total in 8 bits).

```
Maximum possible last octet value = 255   (= 11111111 in binary)
BUT: .255 is reserved as the BROADCAST address
→ The last ASSIGNABLE host address = .255 − 1 = .254
```

```
Last Host = 201.20.30.254
```

### Step 6 — Find the Broadcast Addresses

There are **two types** of broadcast addresses — see Section 6 for full explanation.

```
Limited Broadcast Address = 255.255.255.255   (always fixed — same for all networks)
Direct Broadcast Address  = 201.20.30.255     (last IP address of THIS specific network)
```

---

## 6. Broadcast Address Types — Limited vs Direct

### Limited Broadcast

- **Address:** `255.255.255.255` (all bits = 1) — fixed, same for every network.
- **Used when:** A host wants to send a message to **every host within its own organization/network** — an internal broadcast.
- **Router behavior:** The router does **not** forward this beyond the local network — it stays internal.

```
Host in ABC.com → wants to message everyone in ABC.com
→ Sets destination = 255.255.255.255
→ Router broadcasts internally only, does not cross to XYZ.com
```

### Direct Broadcast

- **Address:** `Network ID with all host bits set to 1` — i.e., the **last IP address** of the target network.
  - For `201.20.30.0 / Class C`: Direct Broadcast = **201.20.30.255**
- **Used when:** A host from **a different network** (e.g., `XYZ.com`) wants to broadcast to **all hosts in another network** (e.g., `ABC.com`).
- The sending host puts `201.20.30.255` as the destination address — when the packet reaches the router of ABC.com, the router knows to broadcast it to everyone on that network.

```
Host in XYZ.com → wants to message EVERYONE in ABC.com
→ Sets destination = 201.20.30.255 (Direct Broadcast of ABC.com)
→ Router of ABC.com receives it and broadcasts to all of ABC.com
```

> **Exam tip:** Limited broadcast (`255.255.255.255`) is always fixed — so exams almost always ask about the **Direct Broadcast**, which varies per network. Direct Broadcast = the **last IP address of the target network**.

---

## 7. Reserved IP Addresses in a Network (Summary)

For any Class C network (using `201.20.30.0` as the example):

| IP Address             | Purpose              | Assignable to a Host? |
| ---------------------- | -------------------- | --------------------- |
| `201.20.30.0`          | **Network ID**       | ❌ No                 |
| `201.20.30.1`          | First host           | ✅ Yes                |
| `201.20.30.2` – `.253` | Regular hosts        | ✅ Yes                |
| `201.20.30.254`        | Last host            | ✅ Yes                |
| `201.20.30.255`        | **Direct Broadcast** | ❌ No                 |

- **Total IP addresses in Class C:** 256 (0–255)
- **Usable host addresses:** 256 − 2 = **254** (subtract Network ID and Broadcast)

---

## 8. General Formulas for Any Classful Address

| Task                        | Method                                                                                   |
| --------------------------- | ---------------------------------------------------------------------------------------- |
| **Find the class**          | Look at first octet only (compare against class ranges)                                  |
| **Find default mask**       | A→255.0.0.0 / B→255.255.0.0 / C→255.255.255.0                                            |
| **Find Network ID**         | IP AND Mask (use shortcut: AND with 255 = same, AND with 0 = 0)                          |
| **Find Nth host**           | Network ID + N in the host portion                                                       |
| **Find Last host**          | Replace all host bits with 1s, then subtract 1 (i.e., the second-to-last possible value) |
| **Find Direct Broadcast**   | Replace all host bits with 1s (last IP address of the network)                           |
| **Find Limited Broadcast**  | Always `255.255.255.255`                                                                 |
| **Usable hosts in Class C** | 256 − 2 = 254                                                                            |
| **Usable hosts in Class B** | 65536 − 2 = 65534                                                                        |
| **Usable hosts in Class A** | 16,777,216 − 2 = 16,777,214                                                              |

---

## 9. Why This Matters for Cybersecurity

- **Network reconnaissance starts here:** Tools like `nmap`, `arp-scan`, and `netdiscover` need to know the **network range** to scan — this is exactly the Network ID calculation. Knowing `201.20.30.0/24` tells the tool which 254 host addresses to probe. This is the math under every host-discovery scan you run in your lab.
- **Broadcast as an attack vector:** The distinction between limited and direct broadcast is directly relevant to **Smurf attacks** — a classic amplification DoS technique where an attacker spoofs a victim's IP as the source and sends pings to a network's **direct broadcast address** (`201.20.30.255`), causing every host on that network to reply to the victim, flooding it with traffic.
- **Subnetting for network segmentation:** Classful addressing is the foundation for understanding **CIDR/subnetting**, where organizations divide networks into smaller segments to **contain breach impact** (a key concept in network security design — compromising one subnet shouldn't immediately expose others). This lecture's AND-operation method is also the core of subnetting math.
- **IP spoofing awareness:** Since the Network ID identifies "which network" a host belongs to, understanding classful ranges lets you quickly recognize when a claimed source IP is suspicious — e.g., a packet arriving on an interface with a source IP from a completely different class/network than expected.

---

## 10. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           CLASSFUL IP ADDRESSING — EXAM CHEAT SHEET                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  CLASS RANGES (first octet only)                                         ║
║  ─────────────────────────────────────────────────────────          ║
║      Class A: 0–127      Mask: 255.0.0.0         Hosts: ~16.7 M     ║
║      Class B: 128–191    Mask: 255.255.0.0       Hosts: 65,534      ║
║      Class C: 192–223    Mask: 255.255.255.0     Hosts: 254         ║
║      Class D: 224–239    Multicast (no mask/hosts)                  ║
║      Class E: 240–255    Reserved/Experimental                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  AND SHORTCUT (saves exam time)                                          ║
║  ─────────────────────────────────────────────────────          ║
║      X AND 255 = X   (copy the value as-is)                         ║
║      X AND 0   = 0   (always zero)                                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  SOLVED EXAMPLE SUMMARY (IP: 201.20.30.40)                            ║
║  ─────────────────────────────────────────────────────          ║
║      Class               → C (201 is in 192–223)                    ║
║      Default Mask         → 255.255.255.0                           ║
║      Network ID           → 201.20.30.0                             ║
║      4th Host             → 201.20.30.4                             ║
║      Last Host            → 201.20.30.254                           ║
║      Limited Broadcast    → 255.255.255.255  (always fixed)         ║
║      Direct Broadcast     → 201.20.30.255    (last IP of network)   ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY RULES                                                               ║
║  ─────────────────────────────────────────────────────          ║
║      .0  of network → Network ID (NOT assignable to any host)       ║
║      .255 of network → Direct Broadcast (NOT assignable to any host)║
║      Usable hosts = Total IPs − 2                                   ║
║      Direct Broadcast ≠ Limited Broadcast (255.255.255.255)         ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Subnetting / CIDR — going beyond classful divisions
- [ ] Private vs Public IP ranges (Class A/B/C private blocks)
- [ ] IPv6 addressing overview
- [ ] ARP — how IP maps to MAC (the link between Layer 3 and Layer 2)

---

_Notes compiled from: Networking Course Lecture 19 — Classful IP Addressing (Solved Question)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
