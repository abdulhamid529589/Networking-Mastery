# 🔗 Networking Devices — Cables (UTP, Coaxial, Fiber Optic)

### Cybersecurity Student Notes | Networking Course — Lecture 09

> **Source:** Gate Smashers — Networking Devices: Cable
> **Exam Relevance:** GATE · UGC NET · PSU Exams · University Exams · CompTIA Network+
> **Topic Area:** Physical Layer — Transmission Media

---

## 📑 Table of Contents

1. [Why We Use Cables](#1-why-we-use-cables)
2. [Decoding Cable Naming — `<Rate> Base <Distance>`](#2-decoding-cable-naming--rate-base-distance)
3. [Baseband vs Broadband](#3-baseband-vs-broadband)
4. [Unshielded Twisted Pair (UTP) Cable](#4-unshielded-twisted-pair-utp-cable)
5. [Coaxial Cable](#5-coaxial-cable)
6. [Fiber Optic Cable](#6-fiber-optic-cable)
7. [Attenuation](#7-attenuation)
8. [Collision Domain on a Cable](#8-collision-domain-on-a-cable)
9. [Can Cables Filter Traffic?](#9-can-cables-filter-traffic)
10. [OSI Layer Placement](#10-osi-layer-placement)
11. [Comparison Table](#11-comparison-table)
12. [Why This Matters for Cybersecurity](#12-why-this-matters-for-cybersecurity)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. Why We Use Cables

Cables are the **physical medium** used to connect devices in a network — they carry signals (electrical or light) that represent the data/packets being transmitted. Common cable types covered here:

- **Unshielded Twisted Pair (UTP)**
- **Shielded Twisted Pair (STP)**
- **Coaxial cable**
- **Fiber optic cable**

> The exam focus is **not** memorizing cable names — it's understanding how to **decode the naming convention** (e.g., `10BaseT`, `100BaseFX`) and answering conceptual questions about attenuation, collision, filtering, and OSI layer.

---

## 2. Decoding Cable Naming — `<Rate> Base <Distance>`

Cable standards are typically named in the format: **`[Speed] Base [Distance/Type]`**

Example: **`10BaseT`**

| Part     | Meaning                                                                                                            |
| -------- | ------------------------------------------------------------------------------------------------------------------ |
| **10**   | **Bandwidth / data rate** = 10 Mbps (10 megabits per second) — how many bits can be transmitted per second.        |
| **Base** | **Baseband** transmission (see Section 3) — only one signal can travel on the wire at a time.                      |
| **T**    | Distance the signal can travel before attenuation sets in — for UTP, `T` conventionally represents **100 meters**. |

> **Bandwidth** = the maximum number of bits that can be transmitted per second over the medium.

---

## 3. Baseband vs Broadband

| Mode          | Meaning                                                                                                                                                                                                                                                                                      |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Baseband**  | Only **one signal** can be transmitted on the wire **at a time**. If a sender is transmitting to a receiver, the receiver cannot simultaneously transmit back — they can communicate, but not _at the same time_. If two signals try to use the wire simultaneously, a **collision** occurs. |
| **Broadband** | **Multiple signals** can travel on the medium **in parallel** at the same time, without colliding.                                                                                                                                                                                           |

> Most of the cable standards discussed here (`10BaseT`, `100BaseFX`, etc.) use **Base** = baseband transmission.

---

## 4. Unshielded Twisted Pair (UTP) Cable

- Carries signal as an **electrical signal**.
- Common standards: **`10BaseT`** and **`100BaseT`**

| Standard   | Bandwidth | Distance before attenuation |
| ---------- | --------- | --------------------------- |
| `10BaseT`  | 10 Mbps   | 100 meters (`T`)            |
| `100BaseT` | 100 Mbps  | 100 meters (`T`)            |

- **Usage:** Commonly used in **Ethernet LANs** — local area networks. Historically defined as "less than 1–2 km," but modern office/organizational LANs are generally considered **less than 10 km**.
- **Example application:** Can be used as the cable in **Token Bus** or **Token Ring** networks (topologies covered in Part 6).

---

## 5. Coaxial Cable

- Also carries signal as an **electrical signal**.
- Common standards: **`10Base2`** and **`10Base5`**

| Standard  | Bandwidth | Distance before attenuation |
| --------- | --------- | --------------------------- |
| `10Base2` | 10 Mbps   | 200 meters (`2` = 200m)     |
| `10Base5` | 10 Mbps   | 500 meters (`5` = 500m)     |

- Compared to UTP, coaxial cable allows the signal to travel a **greater distance** before attenuation occurs (200m or 500m vs. UTP's 100m).
- Note: These are **example standards** — modern cabling also exists at **gigabit** rates; the exam expects you to know **how to decode** the naming pattern (`rate` / `base or broadband` / `distance`), not memorize every possible modern variant.

---

## 6. Fiber Optic Cable

- Carries signal as a **light signal**, NOT electrical.
- **Speed of transmission = speed of light** = `3 × 10⁸ meters/second`
- Example standard: **`100BaseFX`**

| Part of `100BaseFX` | Meaning                                                                 |
| ------------------- | ----------------------------------------------------------------------- |
| **100**             | 100 Mbps bandwidth                                                      |
| **Base**            | Baseband transmission                                                   |
| **FX**              | Fiber channel — distance ≈ **1.9 km (approx. 2 km)** before attenuation |

- Other variants exist (e.g., `1000BaseFX`) with higher bandwidth.
- Because it uses light instead of electrical signals, fiber optic cable is generally **far less susceptible to electromagnetic interference** and can travel much greater distances than copper-based cables (UTP/coaxial) before attenuation.

---

## 7. Attenuation

**Attenuation** = the **weakening of a signal's strength/energy** as it travels a distance through a cable, eventually preventing it from traveling further without regeneration.

```
Signal strength
   │╲
   │ ╲___
   │     ╲____
   │          ╲______  ← signal too weak to be reliably received
   └──────────────────────► Distance
        (e.g., 100m for UTP, 200/500m for coaxial, ~2km for fiber)
```

- Each cable type has a **maximum effective distance** before attenuation becomes significant (this is exactly the `T` / `2` / `5` / `FX` distance value encoded in the naming standard).
- This is why **repeaters** (Part 6, Part 8) are used — to regenerate a weakening signal and extend the effective transmission distance.

> **Exam tip:** "Attenuation" = **loss of signal strength over distance**. This is a very frequently asked one-line definition question.

---

## 8. Collision Domain on a Cable

- If `n` devices are connected to a shared cable (baseband, multipoint — see Bus topology in Part 6), and **all `n` devices transmit at the same time**, all their signals can collide.
- **Maximum possible collisions = `n`** (i.e., all connected devices can participate in a collision).
- This shared collision "zone" is called the **collision domain**.

> **Exam tip:** "What is the maximum number of devices that can participate in a collision on a cable with n devices?" → **`n`** (all of them).

---

## 9. Can Cables Filter Traffic?

**No — cables cannot filter traffic.**

```
[LAN 1] ══════ Cable ══════ [LAN 2]

If a signal enters this cable from LAN 1,
it will move onward to LAN 2 — the cable
has NO way to stop it or decide otherwise.
```

- **Filtering** means deciding whether a signal/packet should be allowed to move further (e.g., based on IP address or MAC address).
- Cables are **pure hardware** — there is **no software** running on them.
- A cable **cannot check an IP address or a MAC address** — it has no intelligence at all. Its only purpose is to provide a physical connection/medium.
- This is a key distinction from devices like bridges, switches, and routers (Part 8), which _do_ have the software intelligence to filter/forward traffic selectively.

---

## 10. OSI Layer Placement

**Cables operate at the Physical Layer (Layer 1) of the OSI model.**

- No software, no MAC address checking, no IP address checking — the basic purpose of a cable is purely to provide **connectivity** (transmit raw signals).
- This is a **very frequently asked exam question**: _"In which OSI layer does a cable operate?"_ → **Physical Layer**.

---

## 11. Comparison Table

| Property                            | UTP                          | Coaxial                       | Fiber Optic                                   |
| ----------------------------------- | ---------------------------- | ----------------------------- | --------------------------------------------- |
| **Signal type**                     | Electrical                   | Electrical                    | Light                                         |
| **Example standards**               | `10BaseT`, `100BaseT`        | `10Base2`, `10Base5`          | `100BaseFX`, `1000BaseFX`                     |
| **Speed**                           | 10 / 100 Mbps                | 10 Mbps                       | 100 Mbps+ (up to speed of light transmission) |
| **Max distance before attenuation** | 100 m                        | 200 m (Base2) / 500 m (Base5) | ~1.9–2 km                                     |
| **Common use case**                 | Ethernet LAN, Token Ring/Bus | Legacy Ethernet               | High-speed / long-distance backbone links     |
| **Interference resistance**         | Lower                        | Moderate                      | High (immune to EM interference)              |

---

## 12. Why This Matters for Cybersecurity

- **Physical-layer attack surface:** Understanding cable types and their signal characteristics is foundational to physical security assessments — e.g., copper (UTP/coaxial) cables emit electromagnetic signals that can theoretically be intercepted with physical access and specialized equipment, while fiber is comparatively much harder to tap without detection (this is why fiber is often preferred for sensitive backbone links).
- **No filtering = no security at Layer 1:** Since cables have zero intelligence, all security controls (firewalls, ACLs, IDS) must be implemented at higher layers/devices — this reinforces why physical layer security (locked server rooms, cable management, tamper-evident conduits) is still a real part of a defense-in-depth strategy, even though the cable itself can't "decide" anything.
- **Collision domains and reconnaissance:** Understanding collision domains ties directly back to why legacy shared-medium networks (Part 6 — Bus topology, hub-based Star topology) were trivially easy to sniff — every device sat in the same collision/broadcast domain.

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║                     CABLES — EXAM CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  NAMING CONVENTION:  [Speed] Base [Distance/Type]                    ║
║  ─────────────────────────────────────────────────────────          ║
║      10BaseT   → 10 Mbps, Baseband, ~100m (UTP)                      ║
║      100BaseT  → 100 Mbps, Baseband, ~100m (UTP)                     ║
║      10Base2   → 10 Mbps, Baseband, ~200m (Coaxial)                  ║
║      10Base5   → 10 Mbps, Baseband, ~500m (Coaxial)                  ║
║      100BaseFX → 100 Mbps, Baseband, ~1.9-2km (Fiber, light signal)  ║
╠══════════════════════════════════════════════════════════════════════╣
║  BASEBAND vs BROADBAND                                                ║
║  ─────────────────────────────────────────────────────          ║
║      Baseband  → ONE signal at a time (collision risk)               ║
║      Broadband → MULTIPLE signals in parallel                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY DEFINITIONS                                                        ║
║  ─────────────────────────────────────────────────────          ║
║      Attenuation      → signal strength weakens over distance         ║
║      Collision domain → max collisions on n devices = n               ║
║      Filtering        → cables CANNOT filter (pure hardware,          ║
║                          no MAC/IP awareness)                         ║
║      OSI Layer        → Cables work at PHYSICAL LAYER (Layer 1)       ║
╠══════════════════════════════════════════════════════════════════════╣
║  SIGNAL TYPE BY CABLE                                                   ║
║  ─────────────────────────────────────────────────────          ║
║      UTP / Coaxial → Electrical signal                                ║
║      Fiber Optic    → Light signal (speed = 3×10⁸ m/s)                ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Repeater — detailed working, OSI layer, why/where it's used
- [ ] Hub — active vs passive, limitations
- [ ] Bridge vs Switch — detailed comparison

---

_Notes compiled from: Networking Course Lecture 09 — Networking Devices: Cable_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
