# 🌐 Token Ring — IEEE 802.5

### " "Networking Course — Lecture 38

> **Source:** Gate Smashers — Token Ring / IEEE 802.5
> " "/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Token Ring?](#1-what-is-token-ring)
2. [Ring Topology — How Devices Connect](#2-ring-topology--how-devices-connect)
3. [Token Passing — The Core Access Method](#3-token-passing--the-core-access-method)
4. [Token Release Strategies](#4-token-release-strategies)
5. [Data Frame Format](#5-data-frame-format)
6. [Token Frame Format](#6-token-frame-format)
7. [Access Control (AC) Byte — Deep Dive](#7-access-control-ac-byte--deep-dive)
8. [Frame Status (FS) Field](#8-frame-status-fs-field)
9. [Monitor Station](#9-monitor-station)
10. [Other Technical Details](#10-other-technical-details)
11. [Token Ring vs Ethernet — Side-by-Side](#11-token-ring-vs-ethernet--side-by-side)
12. [🔴 Security Context](#12--security-context)
13. [🧪 Practical Labs](#13--practical-labs)
14. [Exam Cheat Sheet](#14-exam-cheat-sheet)

---

## 1. What is Token Ring?

### Overview

```
Token Ring is a Data Link Layer LAN protocol
where stations share a ring-shaped medium using
a rotating permission slip called a TOKEN.

Only the station holding the token may transmit.
→ Collisions are IMPOSSIBLE by design.
```

### Key Identity Facts

| Property               | Value                                        |
| ---------------------- | -------------------------------------------- |
| Standard               | **IEEE 802.5**                               |
| Originally Designed By | **IBM, 1984**                                |
| OSI Layer              | **Data Link Layer**                          |
| Topology               | **Ring**                                     |
| Access Method          | **Token Passing**                            |
| Direction              | **Unidirectional** (one way only)            |
| Data Rate              | **4 Mbps and 16 Mbps** (exam default)        |
| Frame Size             | **Variable** (bounded by Token Holding Time) |
| Encoding               | **Differential Manchester**                  |
| Acknowledgement        | Piggybacking + FS field bits                 |

### Why Token Ring Matters for Exams

A very common exam question compares Data Link Layer protocols by topology and access method:

| Protocol       | Standard  | Topology      | Access Method     |
| -------------- | --------- | ------------- | ----------------- |
| **Token Ring** | **802.5** | **Ring**      | **Token Passing** |
| Ethernet       | 802.3     | Bus (or Star) | CSMA/CD           |
| Wi-Fi          | 802.11    | Star (via AP) | CSMA/CA           |

---

## 2. Ring Topology — How Devices Connect

### Physical Layout

```
           [Station A]
          ↗             ↘
   [Station D]         [Station B]
          ↖             ↙
           [Station C]

Arrow direction = data travel direction (unidirectional)
Each station is both a receiver AND a repeater.
```

### Unidirectional Flow Rule

```
If clockwise direction is chosen:
  A → B → C → D → A → B → ...

A frame sent by A passes through B, C, D
and eventually RETURNS to A.

The sender (A) is responsible for removing its own frame
after it has completed the loop.
```

### Each Station Acts as a Repeater

```
Station receives signal from previous station
↓
Reads the destination address
↓
If it's not for this station: regenerate signal, pass it on
If it IS for this station: copy frame into buffer, pass it on
↓
Signal continues around the ring
```

---

## 3. Token Passing — The Core Access Method

### What is a Token?

```
A token is a special 3-byte control frame
that continuously circulates around the ring.

Think of it as a PERMISSION SLIP:
  Token available (free) → any ready station may capture it
  Token held by station  → only that station may transmit
  No station has token   → nothing is transmitted
```

### Step-by-Step Token Passing

```
Step 1: Token circulates (no one transmitting)
            Token → A → B → C → D → A → ...

Step 2: Station B has data ready
            B waits until token arrives at B

Step 3: B captures the token
            B removes it from the ring

Step 4: B starts transmitting its data frame
            Frame → C → D → A → ... → back to B
            (all intermediate stations copy if addressed to them)

Step 5: Frame returns to B (full loop complete)
            B checks FS field — was frame received?

Step 6: B removes the frame from the ring

Step 7: B releases (regenerates) the free token
            Token → C → D → A → B → ...

Step 8: Next ready station captures the token
            Process repeats
```

### Real-World Analogy

> Token passing is like a **speaking stick** in a group meeting.
> Only the person holding the stick may speak.
> When done, they pass it to the next person.
> Everyone else must wait silently — no interruptions, no chaos.

### Why Token Passing Eliminates Collisions

```
Ethernet (CSMA/CD):
  Multiple stations can transmit simultaneously → COLLISION → detect → retry

Token Ring:
  Only ONE token exists on the ring at any time
  Only the token HOLDER can transmit
  → Two stations CANNOT transmit simultaneously
  → Collisions are physically impossible
```

---

## 4. Token Release Strategies

After finishing transmission, a station releases the token in one of two ways:

### Early Token Release

```
Station B finishes sending all its frames
↓
IMMEDIATELY releases the token
↓
Token available for next station right away

Analogy: Pass the speaking stick the moment you finish your sentence.
```

### Delayed Token Release (Single Token)

```
Station B transmits a frame
↓
WAITS for that frame to travel the full ring and return
↓
B confirms: "My frame has propagated the entire ring"
↓
Only THEN releases the token

Analogy: Wait for everyone to nod in acknowledgement before
         passing the speaking stick.
```

### Comparison Table

| Property            | Early Release         | Delayed Release          |
| ------------------- | --------------------- | ------------------------ |
| When token released | After last frame sent | After last frame returns |
| Ring utilization    | Higher                | Lower                    |
| Typical speed       | 16 Mbps               | 4 Mbps                   |
| Exam default answer | 16 Mbps version       | 4 Mbps version           |
| Complexity          | Simpler               | More conservative        |

> **Exam note:** Numericals on backoff and THT (Token Holding Time) appear in GATE and UGC-NET.

---

## 5. Data Frame Format

Token Ring uses **two different frame formats** — one for data, one for the token itself.

### Data Frame Structure

```
┌────┬────┬────┬──────┬──────┬──────────┬─────┬────┬────┐
│ SD │ AC │ FC │ Dest │ Src  │   Data   │ CRC │ ED │ FS │
│ 1B │ 1B │ 1B │  6B  │  6B  │ Variable │ 4B  │ 1B │ 1B │
└────┴────┴────┴──────┴──────┴──────────┴─────┴────┴────┘
      ↑                                              ↑
   Contains                                     Unique to
  token bit,                                  Token Ring —
  priority,                                  tells sender
  monitor bit                              if frame was received
```

### Field-by-Field Reference

| Field                | Size     | Purpose                                                 |
| -------------------- | -------- | ------------------------------------------------------- |
| SD — Start Delimiter | 1 byte   | Synchronize / alert receiver; start of frame            |
| AC — Access Control  | 1 byte   | Priority bits, token bit, monitor bit, reservation bits |
| FC — Frame Control   | 1 byte   | Is this a **data** frame or a **control** frame?        |
| Destination Address  | 6 bytes  | MAC address of intended recipient (48-bit)              |
| Source Address       | 6 bytes  | MAC address of sender (48-bit)                          |
| Data                 | Variable | 0 bytes min; max = limited by Token Holding Time        |
| CRC                  | 4 bytes  | Cyclic Redundancy Check — error detection (32 bits)     |
| ED — End Delimiter   | 1 byte   | End of frame; contains I bit and E bit                  |
| FS — Frame Status    | 1 byte   | A bit (address recognized) + C bit (frame copied)       |

### Start Delimiter (SD) — Detail

```
Uses special non-data bit patterns (violations of Differential Manchester)
that can NEVER appear in normal data content.
This guarantees the receiver can always identify where a frame starts,
even without a length field to guide it.
```

### End Delimiter (ED) — Key Bits

```
I bit (Intermediate): = 1 if more frames follow from the same station
                       = 0 if this is the last frame in the transmission

E bit (Error):        Set to 1 by ANY intermediate station that detects
                      a CRC or format error while the frame passes through
                      Tells the sender: "your frame got corrupted on the way"
```

### Data Field — Variable Size is an Advantage

```
Ethernet:     Max data = 1500 bytes (MTU — hard limit)

Token Ring:   Max data = limited only by Token Holding Time (THT)
              THT = max time a station may hold the token
              Larger THT → larger allowed frame → more data per turn

Why this matters:
  Applications needing large file transfers benefit from
  Token Ring's flexibility vs Ethernet's fixed 1500-byte cap.
  This is considered Token Ring's key structural advantage.
```

---

## 6. Token Frame Format

The token is a tiny **3-byte frame** — just enough to pass permission around the ring:

```
┌─────────────────┬─────────────────┬─────────────────┐
│  Start Delimiter│  Access Control │   End Delimiter  │
│    1 byte       │  1 byte (T=0)   │    1 byte        │
└─────────────────┴─────────────────┴─────────────────┘
        Total: 3 bytes

Key: Token bit (T) in AC field = 0  →  this is a FREE token
     Token bit (T) in AC field = 1  →  this is a DATA frame
```

### How a Token Becomes a Data Frame

```
Station B captures the free token (T = 0)
↓
B flips the Token bit: T = 0 → T = 1
↓
B appends: FC + Dest + Src + Data + CRC + ED + FS
↓
Full data frame transmitted onto the ring
```

---

## 7. Access Control (AC) Byte — Deep Dive

The AC byte is the most information-dense field in Token Ring:

```
Bit positions (7 to 0):
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ P │ P │ P │ T │ M │ R │ R │ R │
│ 7 │ 6 │ 5 │ 4 │ 3 │ 2 │ 1 │ 0 │
└───┴───┴───┴───┴───┴───┴───┴───┘
```

| Bits | Name              | Size   | Meaning                                                    |
| ---- | ----------------- | ------ | ---------------------------------------------------------- |
| 7–5  | Priority (PPP)    | 3 bits | Priority of this token/frame (0–7, higher = more priority) |
| 4    | Token bit (T)     | 1 bit  | 0 = free token · 1 = data frame                            |
| 3    | Monitor bit (M)   | 1 bit  | Used by monitor station to detect orphaned frames          |
| 2–0  | Reservation (RRR) | 3 bits | Waiting stations request priority for next token           |

### Priority System

```
Token priority = PPP bits in current token
Station's frame priority = station's own priority level

Rule: A station can only capture the token if
      its frame priority ≥ current token priority

Priority range: 0 (lowest) to 7 (highest)

Example:
  Token priority = 3
  Station A priority = 2 → CANNOT capture token
  Station B priority = 4 → CAN capture token
```

### Reservation Bits

```
While a data frame is circulating, stations waiting to transmit
can WRITE their priority into the RRR bits as the frame passes.

When the current sender finishes and releases a token,
it sets the new token's priority = value in RRR bits.

This allows high-priority stations to pre-book the next token
without disrupting the current transmission.
```

### Monitor Bit (M) — How Orphaned Frames Are Caught

```
Frame enters ring, M = 0

Monitor station sees frame for FIRST TIME
  → Monitor sets M = 1
  → Frame continues circulating

Frame comes around again (full loop), M is already 1
  → Monitor knows: this frame went around TWICE
  → Original sender must have crashed — frame is ORPHANED
  → Monitor REMOVES the frame from the ring
  → Monitor releases a fresh token
```

---

## 8. Frame Status (FS) Field

The FS field is **unique to Token Ring** — Ethernet has no equivalent. It gives the sender implicit delivery confirmation without a separate ACK packet.

### FS Bit Layout

```
┌───┬───┬───┬───┬───┬───┬───┬───┐
│ A │ C │ r │ r │ A │ C │ r │ r │
└───┴───┴───┴───┴───┴───┴───┴───┘
  ↑   ↑           ↑   ↑
  │   │           │   └── Frame Copied (duplicate for reliability)
  │   │           └────── Address Recognized (duplicate for reliability)
  │   └────────────────── Frame Copied (C)
  └────────────────────── Address Recognized (A)
  r = Reserved (ignored)

A and C bits are duplicated for error resilience.
```

### A and C Bit Meanings

| A   | C   | Meaning                                                                         |
| --- | --- | ------------------------------------------------------------------------------- |
| 0   | 0   | Destination station **not on ring** or **not active**                           |
| 1   | 0   | Destination **recognized its address** but **did not copy** (buffer full, busy) |
| 1   | 1   | Frame **successfully delivered and copied** ✓                                   |
| 0   | 1   | Invalid / error state                                                           |

### How the Sender Uses FS

```
Sender transmits frame → frame circulates ring
↓
Destination sets A=1 (sees its own address) and C=1 (copies frame)
↓
Frame returns to sender
↓
Sender reads FS field:
  A=1, C=1 → SUCCESS — frame was received
  A=1, C=0 → Destination was there but couldn't accept (retry later)
  A=0, C=0 → Destination not on ring (alert administrator)
↓
Sender removes frame from ring
Sender releases token
```

---

## 9. Monitor Station

### The Problem Without a Monitor

```
Scenario 1 — Orphaned frame:
  Station B transmits a frame
  Station B crashes mid-loop
  Frame circulates endlessly
  No one removes it (sender is dead)
  Token never released → RING DEADLOCK

Scenario 2 — Corrupt frame:
  Frame gets corrupted (CRC mismatch)
  Every station rejects it
  Frame keeps going around
  Consuming bandwidth forever
```

### Monitor Station Solution

```
One station is elected as the ACTIVE MONITOR
All others are STANDBY MONITORS

Active Monitor responsibilities:
  1. Remove orphaned frames     (using M bit tracking)
  2. Remove corrupt frames      (persistent frames with E bit set)
  3. Regenerate lost token      (if ring goes silent > threshold time)
  4. Purge the ring             (send Purge frame to reset all stations)
  5. Ring recovery              (after break or failure event)
```

### Monitor Election

```
If active monitor fails:
  Ring goes silent (no token, no frames)
  Standby monitors detect silence after timeout
  All standbys send "Claim Token" frames
  Station with HIGHEST MAC address wins
  New active monitor purges ring and releases fresh token
```

---

## 10. Other Technical Details

### Differential Manchester Encoding

```
Standard Manchester:
  1 = transition Low → High at mid-bit
  0 = transition High → Low at mid-bit

Differential Manchester:
  Transition at START of bit period = 0
  No transition at START of bit period = 1
  ALWAYS has transition at MID-PERIOD (for clock sync)

Why Differential Manchester?
  ✓ Self-clocking (clock extracted from signal itself)
  ✓ Error detection (missing transition = detectable)
  ✓ Polarity independent (works even if wires are swapped)
  ✓ No DC component (better for transformer coupling)
```

### Acknowledgement in Token Ring

```
Token Ring has NO separate explicit ACK frame.

Two implicit confirmation mechanisms:
  1. Frame Status (FS) field — A and C bits set by destination
     Sender checks these bits when frame returns

  2. Piggybacking — when ACKs are needed at higher layers,
     acknowledgement is attached to the next DATA frame going back
     rather than sent as a standalone packet
     → reduces total number of frames on ring
     → less traffic overhead
```

### Variable Frame Size vs Fixed (Ethernet)

```
Ethernet:
  MTU = 1500 bytes (hard maximum for data field)
  If your payload is larger → must fragment at IP layer

Token Ring:
  No fixed MTU
  Maximum = how much you can send within Token Holding Time (THT)
  Larger THT setting → larger frames allowed
  More flexible for bulk data transfers

This is the most commonly cited ADVANTAGE of Token Ring over Ethernet.
```

---

## 11. Token Ring vs Ethernet — Side-by-Side

| Property              | Token Ring (IEEE 802.5) | Ethernet (IEEE 802.3)         |
| --------------------- | ----------------------- | ----------------------------- |
| Designed              | IBM, 1984               | Xerox/DEC/Intel, 1983         |
| Standard              | IEEE 802.5              | IEEE 802.3                    |
| Topology              | **Ring**                | **Bus** (or Star)             |
| Access Method         | **Token Passing**       | **CSMA/CD**                   |
| Collisions            | **Impossible**          | Possible (detected, handled)  |
| Direction             | **Unidirectional**      | Bidirectional                 |
| Data Rate             | 4 Mbps, 16 Mbps         | 10 Mbps → 400 Gbps            |
| Frame Size            | **Variable** (THT)      | Fixed max 1500 bytes          |
| Acknowledgement       | FS bits + Piggybacking  | None (LAN, no ACK)            |
| Encoding              | Differential Manchester | Manchester / NRZ              |
| Monitor Station       | **Yes (required)**      | No                            |
| Orphan frame handling | Monitor removes         | No equivalent                 |
| Token frame           | 3 bytes (SD+AC+ED)      | N/A                           |
| Frame Status field    | **Yes (A, C bits)**     | No                            |
| Priority mechanism    | Yes (PPP + RRR bits)    | No                            |
| Collision-free        | ✓                       | ✗                             |
| Deterministic access  | ✓ (bounded wait time)   | ✗ (CSMA/CD is probabilistic)  |
| Current relevance     | Legacy / rare           | Dominant (Gigabit/10G in use) |

---

## 12. 🔴 Security Context

### Token Ring in Cybersecurity Perspective

```
Token Ring networks are largely legacy today, but understanding
their security model informs how you think about:
  → Token-based authentication systems
  → Ring topologies in storage networks (Fibre Channel)
  → Deterministic industrial control networks
  → How access control prevents certain attack classes
```

### Attacks Impossible in Token Ring (that plague Ethernet)

```
1. COLLISION-BASED ATTACKS
   Ethernet: attacker can flood network, causing constant collisions
   Token Ring: impossible — only token holder transmits

2. ARP SPOOFING (reduced impact)
   Ethernet: ARP broadcasts reach all stations simultaneously
   Token Ring: frames follow the ring, station-by-station
   Monitor station can detect unusual token/frame patterns

3. CSMA/CD JAMMING
   Ethernet: jam signal can disrupt all stations
   Token Ring: no jam signal concept — token controls access
```

### Attacks Specific to Token Ring

```
1. TOKEN THEFT
   If attacker is ON the ring and captures the token repeatedly,
   they can monopolize all bandwidth (DoS against other stations)

   Tool concept (Python pseudocode):
   while True:
       wait_for_token()
       capture_token()
       send_garbage_frames()  # exhaust THT
       release_token()
       capture_again_immediately()

   Defense: Token Holding Time (THT) limits each station's turn

2. MONITOR IMPERSONATION
   If active monitor is taken offline:
   → Attacker can become the new monitor
   → Controls orphan frame removal, ring recovery
   → Can selectively DROP frames from other stations

   Defense: Secure monitor election, audit who wins Claim Token

3. PRIORITY ESCALATION
   Attacker sets reservation bits (RRR) to max priority (7)
   Steals every token before legitimate stations get it
   Starves low-priority stations

   Defense: Monitor priority levels in traffic analysis
```

### Token Ring in Modern Context — Industrial Networks

```
PROFIBUS, Modbus Token Ring, ARCNET:
  Industrial control systems (ICS) still use token-passing ring concepts
  Petroleum plants, power grids, factory floors

Cybersecurity implication:
  If your pentest scope includes ICS/SCADA:
  → Token ring variants may be present
  → Understand token passing before sending any frames
  → Disrupting the token on an industrial ring = physical danger

  Tools: Wireshark (with appropriate dissectors), SCADA-specific tools
  Never inject frames without explicit permission on ICS networks
```

### Reconnaissance on Legacy Token Ring Segments

```bash
# If you encounter a token ring segment in a pentest
# (rare but possible in legacy enterprise/government environments)

# Wireshark filter for token ring traffic
# (Wireshark can decode 802.5 frames)
# Filter: tr    ← Token Ring display filter

# Check for token ring interface
ip link show | grep -i "token\|tr[0-9]"

# Old-school: token ring interface names
# eth0 = Ethernet
# tr0  = Token Ring (legacy Linux naming)

# Capture on token ring interface
sudo tcpdump -i tr0 -w token_ring_capture.pcap

# Analyze frame types
# Look for AC field, token frames (3 bytes), data frames
```

### Mapping Token Ring to Modern Security Concepts

```
Token Passing → Access Control / Mutual Exclusion
  Modern parallel: Mutex locks, semaphores in software
  Security parallel: Token-based auth (JWT, OAuth tokens)

Monitor Station → Centralized IDS/IPS
  Watches for anomalies, removes bad traffic, recovers from failures
  Modern parallel: SIEM, Network IDS (Snort, Suricata)

Frame Status (A/C bits) → Delivery Confirmation
  Modern parallel: SMTP delivery receipts, TCP ACK, read receipts

Token Holding Time → Rate Limiting
  No single station can monopolize indefinitely
  Modern parallel: API rate limits, QoS policies
```

---

## 13. 🧪 Practical Labs

### Lab 1 — Token Ring Frame Simulator (Python)

```python
# Save as token_ring_sim.py — run: python3 token_ring_sim.py
# Simulates a 4-station token ring: A → B → C → D → A

import time
import random

class Station:
    def __init__(self, name, has_data=False):
        self.name = name
        self.has_data = has_data
        self.received_frames = []

    def __repr__(self):
        return f"Station({self.name}, data={'YES' if self.has_data else 'no'})"

def run_token_ring(stations, rounds=3):
    """Simulate token passing for given number of rounds"""
    print("=" * 60)
    print("TOKEN RING SIMULATION")
    print(f"Stations: {' → '.join(s.name for s in stations)} → (back to {stations[0].name})")
    print("=" * 60)

    token_holder = 0  # index into stations list

    for round_num in range(1, rounds + 1):
        print(f"\n--- Round {round_num} ---")
        print(f"Token at: {stations[token_holder].name}")

        # Find next station with data, starting from current token holder
        transmitted = False
        for i in range(len(stations)):
            idx = (token_holder + i) % len(stations)
            station = stations[idx]

            if station.has_data:
                print(f"\n✓ {station.name} captures token")
                print(f"  → {station.name} transmits data frame")

                # Simulate frame traveling the ring
                dest_idx = (idx + 2) % len(stations)  # 2 hops away
                dest = stations[dest_idx]
                print(f"  → Frame travels: ", end="")

                for j in range(1, len(stations) + 1):
                    hop = stations[(idx + j) % len(stations)]
                    print(f"{hop.name}", end="")
                    if j < len(stations):
                        print(" → ", end="")

                print()

                # Destination copies the frame
                dest.received_frames.append(f"from_{station.name}")
                print(f"  → {dest.name} copies frame (A=1, C=1)")
                print(f"  → Frame returns to {station.name}")
                print(f"  → {station.name} reads FS: A=1, C=1 → SUCCESS ✓")
                print(f"  → {station.name} removes frame, releases token")

                station.has_data = False
                token_holder = (idx + 1) % len(stations)
                transmitted = True
                time.sleep(0.3)
                break

        if not transmitted:
            print(f"  No station has data — token passes to {stations[(token_holder + 1) % len(stations)].name}")
            token_holder = (token_holder + 1) % len(stations)

    print("\n" + "=" * 60)
    print("SIMULATION COMPLETE")
    print("Frames received:")
    for s in stations:
        if s.received_frames:
            print(f"  {s.name}: {s.received_frames}")

# Run simulation
stations = [
    Station("A", has_data=True),
    Station("B", has_data=False),
    Station("C", has_data=True),
    Station("D", has_data=False),
]

run_token_ring(stations, rounds=4)
```

### Lab 2 — AC Byte Decoder (Python)

```python
# Save as ac_byte_decoder.py
# Decodes the Access Control byte of a Token Ring frame

def decode_ac_byte(ac_byte: int):
    """Decode all fields of the AC byte"""
    priority    = (ac_byte >> 5) & 0b111    # bits 7-5
    token_bit   = (ac_byte >> 4) & 0b1      # bit 4
    monitor_bit = (ac_byte >> 3) & 0b1      # bit 3
    reservation = ac_byte & 0b111           # bits 2-0

    print(f"\nAC Byte: 0x{ac_byte:02X} = {ac_byte:08b}b")
    print(f"         P P P T M R R R")
    print(f"         {ac_byte>>5 & 1} {ac_byte>>6 & 1} {ac_byte>>7 & 1} {token_bit} {monitor_bit} {reservation & 4 >> 2} {reservation & 2 >> 1} {reservation & 1}")
    print()
    print(f"  Priority (PPP):     {priority}  {'(highest)' if priority == 7 else '(lowest)' if priority == 0 else ''}")
    print(f"  Token bit (T):      {token_bit}  → {'FREE TOKEN (no data)' if token_bit == 0 else 'DATA FRAME (station transmitting)'}")
    print(f"  Monitor bit (M):    {monitor_bit}  → {'Monitor has seen this frame (orphan if seen again!)' if monitor_bit == 1 else 'First time around ring'}")
    print(f"  Reservation (RRR):  {reservation}  → {'Priority ' + str(reservation) + ' station waiting' if reservation > 0 else 'No reservation'}")

# Test cases from lecture concepts
print("=" * 55)
print("TEST 1: Free token (T=0, P=0, M=0, R=0)")
decode_ac_byte(0b00000000)   # 0x00

print("\n" + "=" * 55)
print("TEST 2: Data frame (T=1, P=3, M=0, R=0)")
decode_ac_byte(0b01110000)   # priority 3, token=1

print("\n" + "=" * 55)
print("TEST 3: Orphaned frame (T=1, M=1 — monitor will remove!)")
decode_ac_byte(0b01111000)   # token=1, monitor=1

print("\n" + "=" * 55)
print("TEST 4: High priority reservation (R=7 — someone is waiting urgently)")
decode_ac_byte(0b00000111)   # reservation=7
```

### Lab 3 — Frame Status Analyzer

```python
# Save as fs_analyzer.py
# Decode Frame Status (FS) field and determine delivery outcome

def decode_fs_byte(fs_byte: int):
    """Decode Frame Status byte"""
    # A and C bits appear twice (bits 7,6 and bits 3,2)
    a_bit = (fs_byte >> 7) & 1   # primary A bit (bit 7)
    c_bit = (fs_byte >> 6) & 1   # primary C bit (bit 6)

    print(f"\nFS Byte: 0x{fs_byte:02X} = {fs_byte:08b}b")
    print(f"         A C r r A C r r")
    print(f"  A (Address Recognized): {a_bit}")
    print(f"  C (Frame Copied):       {c_bit}")
    print()

    if a_bit == 0 and c_bit == 0:
        status = "❌ DESTINATION NOT ON RING / NOT ACTIVE"
        advice = "Check if destination station is powered on and connected"
    elif a_bit == 1 and c_bit == 0:
        status = "⚠️  DESTINATION RECOGNIZED BUT DID NOT COPY"
        advice = "Destination may have full buffer or be busy — retry later"
    elif a_bit == 1 and c_bit == 1:
        status = "✅ FRAME SUCCESSFULLY DELIVERED AND COPIED"
        advice = "Sender can remove frame and release token"
    else:
        status = "⛔ INVALID STATE (A=0, C=1 is an error)"
        advice = "Error in frame status — may indicate ring corruption"

    print(f"  Result:  {status}")
    print(f"  Action:  {advice}")

# Test all four combinations
scenarios = [
    (0b00000000, "Destination offline"),
    (0b10000000, "Destination saw it but couldn't copy"),
    (0b11000000, "Perfect delivery"),
    (0b01000000, "Invalid state"),
]

for fs_val, description in scenarios:
    print("\n" + "=" * 55)
    print(f"Scenario: {description}")
    decode_fs_byte(fs_val)
```

### Lab 4 — Compare Token Ring vs Ethernet Scale

```bash
# Understand why Token Ring offered deterministic access
# at the cost of speed, vs Ethernet's probabilistic CSMA/CD

python3 - << 'EOF'
# Token Ring: 4 Mbps, 4 stations, equal sharing
# Ethernet (10Mbps): 4 stations, with possible collisions

token_ring_rate_mbps = 4
ethernet_rate_mbps = 10
stations = 4

# Token Ring: each station gets exactly 1/N of bandwidth (deterministic)
tr_per_station = token_ring_rate_mbps / stations
print(f"Token Ring ({token_ring_rate_mbps} Mbps, {stations} stations):")
print(f"  Each station guaranteed: {tr_per_station} Mbps")
print(f"  Access pattern: DETERMINISTIC (bounded wait time)")
print(f"  Max wait: {stations-1} × token_holding_time\n")

# Ethernet: each station competes; under load, collisions reduce throughput
# Practical throughput under heavy load ~ 37% of nominal (Ethernet)
eth_practical = ethernet_rate_mbps * 0.37
eth_per_station = eth_practical / stations
print(f"Ethernet ({ethernet_rate_mbps} Mbps, {stations} stations, heavy load):")
print(f"  Practical throughput: ~{eth_practical:.1f} Mbps (collision overhead)")
print(f"  Each station gets approx: {eth_per_station:.2f} Mbps")
print(f"  Access pattern: PROBABILISTIC (unbounded wait, starvation possible)")
print()
print("Conclusion:")
print("  Token Ring: SLOWER but FAIR and PREDICTABLE")
print("  Ethernet:   FASTER but subject to collision and starvation under load")
print("  → Token Ring preferred in real-time / industrial applications")
print("  → Ethernet won in general-purpose LAN due to speed and cost")
EOF
```

### Lab 5 — Wireshark: Observe Token Ring Concepts in 802.11 ACK

```bash
# We don't have actual Token Ring hardware, but we can observe
# analogous MAC-layer acknowledgement behavior in Wi-Fi (802.11)
# which also uses explicit ACK frames (like Token Ring's FS bits)

# Capture Wi-Fi traffic on your lab adapter
# (monitor mode required)
sudo ip link set wlan0 down
sudo iw dev wlan0 set type monitor
sudo ip link set wlan0 up

# Capture management + data + ACK frames
sudo tcpdump -i wlan0 -w wifi_ack_demo.pcap &

# Open Wireshark and apply filter:
# wlan.fc.type_subtype == 0x1D    ← 802.11 ACK frames
# Compare to Token Ring FS field concept:
# 802.11 ACK = separate frame sent after data received
# Token Ring FS = bits SET WITHIN returning data frame

# Stop capture
sudo kill %1

# Restore adapter
sudo iw dev wlan0 set type managed
sudo ip link set wlan0 up

echo "In Wireshark:"
echo "  802.11 ACK filter: wlan.fc.type_subtype == 0x1D"
echo "  Notice: ACK is a SEPARATE tiny frame (like Token Ring's 3-byte token)"
echo "  Token Ring FS bits = embedded delivery confirmation (more efficient)"
```

---

## 14. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║           TOKEN RING (IEEE 802.5) — EXAM CHEAT SHEET                ║
╠══════════════════════════════════════════════════════════════════════╣
║  IDENTITY                                                            ║
║  Standard:    IEEE 802.5                                            ║
║  Designed by: IBM, 1984                                             ║
║  Layer:       Data Link Layer                                       ║
║  Topology:    RING (unidirectional)                                 ║
║  Access:      TOKEN PASSING                                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY PROPERTIES                                                      ║
║  Direction:   Unidirectional (clockwise OR anticlockwise, not both) ║
║  Data Rate:   4 Mbps (delayed release) · 16 Mbps (early release)   ║
║  Frame Size:  Variable (limited by Token Holding Time)              ║
║  Collisions:  IMPOSSIBLE (only token holder transmits)              ║
║  Encoding:    Differential Manchester                                ║
║  Acknowledgement: Piggybacking + FS field (A and C bits)           ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOKEN PASSING RULES                                                 ║
║  → Token circulates continuously                                    ║
║  → Only token holder may transmit                                   ║
║  → After transmission → remove frame → release token               ║
║  → No token = no transmission = ZERO collisions                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOKEN RELEASE                                                       ║
║  Early:   Release immediately after last frame sent (16 Mbps)      ║
║  Delayed: Release after last frame returns (4 Mbps)                ║
╠══════════════════════════════════════════════════════════════════════╣
║  DATA FRAME FORMAT (sizes in bytes)                                  ║
║  SD(1) + AC(1) + FC(1) + Dest(6) + Src(6) + Data(var) + CRC(4)   ║
║                                          + ED(1) + FS(1)           ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOKEN FRAME = 3 bytes: SD(1) + AC(1, T=0) + ED(1)               ║
╠══════════════════════════════════════════════════════════════════════╣
║  ACCESS CONTROL (AC) BYTE: P P P T M R R R                         ║
║  PPP = Priority (0–7)   T = Token bit (0=free, 1=data)            ║
║  M   = Monitor bit      RRR = Reservation bits                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  FRAME STATUS (FS) — A and C bits                                   ║
║  A=0, C=0 → Destination not on ring                                ║
║  A=1, C=0 → Recognized but didn't copy (busy/buffer full)         ║
║  A=1, C=1 → Successfully delivered ✓                               ║
╠══════════════════════════════════════════════════════════════════════╣
║  MONITOR STATION                                                     ║
║  → Uses M bit to detect orphaned frames (frame passed twice)       ║
║  → Removes corrupt/orphaned frames                                  ║
║  → Regenerates lost token                                           ║
║  → Elected by highest MAC address when active monitor fails        ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOKEN RING vs ETHERNET                                             ║
║  802.5 → Ring, Token Passing, No collisions, Variable frame        ║
║  802.3 → Bus,  CSMA/CD,      Collisions possible, 1500B max MTU   ║
╠══════════════════════════════════════════════════════════════════════╣
║  TOKEN RING ADVANTAGE OVER ETHERNET                                  ║
║  ✓ No collisions (deterministic access)                            ║
║  ✓ Variable frame size (no 1500-byte MTU limit)                    ║
║  ✓ Priority mechanism (high-priority stations get token first)     ║
║  ✓ Bounded latency (max wait = (N-1) × THT)                       ║
║  ✗ Slower (4/16 Mbps vs Gigabit Ethernet today)                   ║
║  ✗ Single point of failure (token loss = ring down)                ║
║  ✗ Complex (monitor needed, token management overhead)             ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] HDLC — High-level Data Link Control (another 802 protocol)
- [ ] CSMA/CD — Full collision detection walkthrough (Ethernet MAC)
- [ ] CSMA/CA — Wireless LAN access control (IEEE 802.11)
- [ ] Ethernet Frame Format — Field-by-field (IEEE 802.3)
- [ ] Subnetting — Dividing IP networks (follows IP Addressing series)

---

_Notes compiled from: Networking Course Lecture 42 — Token Ring / IEEE 802.5_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
