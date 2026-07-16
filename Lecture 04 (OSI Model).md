# ⚡ Physical Layer — OSI Model Layer 1

### Cybersecurity Student Notes | Networking Course — Lecture 04

> **Source:** Gate Smashers — Physical Layer & Its Functionalities
> **Exam Relevance:** GATE · University Exams · Interviews · CompTIA ITF+ / Network+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Where Does Physical Layer Sit?](#1-where-does-physical-layer-sit)
2. [What Does Physical Layer Do? — Core Idea](#2-what-does-physical-layer-do--core-idea)
3. [Physical Layer Functionalities](#3-physical-layer-functionalities)
   - [3.1 Cables & Connectors](#31-cables--connectors)
   - [3.2 Physical Topology](#32-physical-topology)
   - [3.3 Hardware Devices](#33-hardware-devices)
   - [3.4 Transmission Modes](#34-transmission-modes)
   - [3.5 Multiplexing & Demultiplexing](#35-multiplexing--demultiplexing)
   - [3.6 Encoding](#36-encoding)
4. [Bits to Signals — The Core Job](#4-bits-to-signals--the-core-job)
5. [Transmission Media — Guided vs Unguided](#5-transmission-media--guided-vs-unguided)
6. [Signal Properties](#6-signal-properties)
7. [Physical Layer vs Upper Layers — Key Difference](#7-physical-layer-vs-upper-layers--key-difference)
8. [🔴 Physical Layer Attack Surface](#8--physical-layer-attack-surface)
9. [🧪 Practical Labs — Your Setup](#9--practical-labs--your-setup)
10. [Exam & Interview Cheat Sheet](#10-exam--interview-cheat-sheet)

---

## 1. Where Does Physical Layer Sit?

### Position in OSI Model

```
OSI MODEL
──────────────────────────────────────────────────
Layer 7 │ Application   │  HTTP, FTP, DNS, SMTP
Layer 6 │ Presentation  │  TLS, Encoding
Layer 5 │ Session       │  Session management
Layer 4 │ Transport     │  TCP, UDP
Layer 3 │ Network       │  IP, Routing
Layer 2 │ Data Link     │  Ethernet, MAC, ARP
──────────────────────────────────────────────────
Layer 1 │ PHYSICAL      │  ← We are here
──────────────────────────────────────────────────
           Cables, Signals, Hardware
```

### Sender vs Receiver Perspective

```
SENDER SIDE                          RECEIVER SIDE
───────────                          ─────────────
Application   ← first layer          Application   ← last layer
Presentation                         Presentation
Session                              Session
Transport                            Transport
Network                              Network
Data Link                            Data Link
PHYSICAL  ← LAST layer adds          PHYSICAL  ← FIRST layer
             functionality              receives signal, converts to bits
                │                              ▲
                ▼                              │
          [Signal on wire] ──────────────────►─┘
```

**Key insight:**

- On the **sender** side → Physical layer is the **last** to process data (adds its functionality)
- On the **receiver** side → Physical layer is the **first** to process the incoming signal

---

## 2. What Does Physical Layer Do? — Core Idea

### The Fundamental Job

Above the Physical Layer (in Data Link Layer) → we have **Frames** (which contain **bits**)

Physical Layer's job:

```
SENDER:
  Bits (from Data Link) → Convert → Signals → Send over media

RECEIVER:
  Signals (from media) → Convert → Bits → Pass up to Data Link
```

### Hardware vs Software — The Big Distinction

| Layer                 | Deals with                      | Type                |
| --------------------- | ------------------------------- | ------------------- |
| Application–Data Link | Logical/virtual data, protocols | Software + Hardware |
| **Physical**          | Real, tangible, physical things | **Mostly Hardware** |

> Physical layer = **tangible world**. You can touch and see everything it deals with: cables, connectors, signals, hubs, repeaters.

**Example:** Encryption cannot happen at Physical Layer — encryption is software logic, belongs to Presentation Layer (Layer 6).

---

## 3. Physical Layer Functionalities

### 3.1 Cables & Connectors

#### Types of Cables

| Cable Type                 | Signal Type     | Medium             | Speed         | Max Distance | Use Case             |
| -------------------------- | --------------- | ------------------ | ------------- | ------------ | -------------------- |
| **Twisted Pair (UTP/STP)** | Electrical      | Copper             | Up to 10Gbps  | 100m         | LAN (Ethernet)       |
| **Coaxial**                | Electrical      | Copper + shielding | Up to 10Gbps  | 500m         | Cable TV, older LANs |
| **Optical Fiber**          | Light (photons) | Glass/plastic      | Up to 100Tbps | Kilometers   | Backbone, WAN, MAN   |

#### Twisted Pair Categories

| Category   | Max Speed      | Typical Use   |
| ---------- | -------------- | ------------- |
| **Cat5e**  | 1 Gbps         | Home networks |
| **Cat6**   | 10 Gbps (55m)  | Office LAN    |
| **Cat6a**  | 10 Gbps (100m) | Data centers  |
| **Cat7/8** | 25–40 Gbps     | Data centers  |

#### Types of Connectors

| Connector        | Cable                | Layer            |
| ---------------- | -------------------- | ---------------- |
| **RJ-45**        | UTP/STP twisted pair | Physical (LAN)   |
| **BNC**          | Coaxial              | Physical (older) |
| **SC / LC / ST** | Optical fiber        | Physical (fiber) |
| **USB**          | Various              | Physical         |

```
RJ-45 (most common):
  ┌──────────┐
  │ 12345678 │ ← 8 pins (4 pairs of twisted wire)
  └──────────┘
  Used in Ethernet cables connecting to your NIC, switch, router
```

#### UTP vs STP vs Coaxial vs Fiber

| Property                  | UTP         | STP        | Coaxial        | Optical Fiber  |
| ------------------------- | ----------- | ---------- | -------------- | -------------- |
| Shielding                 | None        | Foil/braid | Braided shield | None needed    |
| EMI immunity              | Low         | Moderate   | High           | Immune         |
| Cost                      | Cheapest    | Moderate   | Moderate       | Most expensive |
| Weight                    | Light       | Moderate   | Heavy          | Very light     |
| Installation              | Easy        | Moderate   | Harder         | Needs skill    |
| Tap difficulty (security) | Easy to tap | Harder     | Moderate       | **Very hard**  |

---

### 3.2 Physical Topology

**Topology** = How physical devices are connected to each other.

#### Point-to-Point

```
[Device A] ──────────── [Device B]
Dedicated link, only these two share the media.
Example: Leased line between two offices.
Security: More secure — no other devices on the link.
```

#### Bus Topology

```
[PC1]─┬─[PC2]─┬─[PC3]─┬─[PC4]
      │       │       │
  ════╪═══════╪═══════╪════  ← Shared bus (single cable)
Multi-point — all devices share one medium.
```

- **Collision possible** — only one can transmit at a time (CSMA/CD)
- **Security risk** — every device sees all traffic (passive sniffing easy)

#### Star Topology

```
         [PC1]
           │
[PC2] ── [Switch/Hub] ── [PC3]
           │
         [PC4]
Central device (switch/hub) manages connections.
```

- **Hub** → broadcasts to all (easy to sniff all traffic)
- **Switch** → sends only to intended recipient (harder to sniff — needs ARP spoof)

#### Ring Topology

```
[PC1] ──► [PC2] ──► [PC3] ──► [PC4] ──► back to PC1
Data travels in one direction around the ring.
Token Ring (IEEE 802.5) — mostly legacy now.
```

#### Mesh Topology

```
[PC1] ─── [PC2]
  │   ╲ ╱   │
  │    ╳    │
  │   ╱ ╲   │
[PC3] ─── [PC4]
Every device connected to every other device.
Point-to-point between each pair.
```

- **Most reliable** — multiple paths exist
- **Most expensive** — n(n-1)/2 connections needed
- **Most secure** — dedicated links, hard to intercept without physical access

#### Hybrid Topology

Combination of two or more topologies. Most real-world networks are hybrid.

---

### 3.3 Hardware Devices

#### Hub

```
[PC1] ──┐
[PC2] ──┤── [HUB] ──► Broadcasts to ALL ports
[PC3] ──┘
Layer: Physical (Layer 1)
```

- **Dumb device** — no intelligence, just repeats signal to all ports
- **Security nightmare** — anyone connected to the hub sees ALL traffic
- **Replaced by switches** in modern networks
- **Attacker's dream** — passive sniffing works perfectly on hub networks

#### Repeater

```
[Signal weakens] ──► [REPEATER] ──► [Signal amplified & regenerated]
```

- Amplifies/regenerates degraded signal (fights **attenuation**)
- Works at Layer 1 — no understanding of data, just boosts signal
- Used to extend cable length beyond standard limits

#### Attenuation

> Signal loses energy as it travels through a medium. The farther it goes, the weaker it gets.

```
Strong signal ──────────────────────────────► Weak signal
[Sender]       distance →→→→→→→→→→→→       [Receiver far away]
                              ↑
                          Repeater here
                          (regenerates signal)
```

---

### 3.4 Transmission Modes

How data flows between sender and receiver on a link:

#### Simplex

```
[Sender] ──────────────────► [Receiver]
Only ONE direction. Receiver can NEVER send back.
```

- **Examples:** TV broadcast, keyboard → monitor, radio broadcast
- **No two-way communication possible**

#### Half-Duplex

```
[Device A] ◄────────────────► [Device B]
Both can send AND receive, but NOT at the same time.
One transmits → other must wait.
```

- **Examples:** Walkie-talkie ("Over"), old CB radio
- **Collision possible** if both try to transmit simultaneously

#### Full-Duplex

```
[Device A] ──────────────────► [Device B]
[Device A] ◄────────────────── [Device B]
Both send AND receive simultaneously — two separate channels.
```

- **Examples:** Phone call, modern Ethernet, internet browsing
- **No collision** — separate channels for each direction
- **Most efficient** for networking

| Mode            | Direction                | Simultaneous? | Example         |
| --------------- | ------------------------ | ------------- | --------------- |
| **Simplex**     | One-way only             | N/A           | TV, Keyboard    |
| **Half-Duplex** | Both ways, one at a time | No            | Walkie-talkie   |
| **Full-Duplex** | Both ways                | Yes           | Phone, Ethernet |

---

### 3.5 Multiplexing & Demultiplexing

#### The Problem Without Multiplexing

```
If 4 machines want to communicate:
  Machine1 → needs own cable to each partner → 3 cables
  Machine2 → 3 cables
  Machine3 → 3 cables
  Total: too many cables, very expensive
```

#### Solution: Multiplexing

Combine multiple signals into ONE channel — split the channel logically.

```
[Sender1] ─┐                              ┌─► [Receiver1]
[Sender2] ─┤──► [MULTIPLEXER] ──single──► [DEMULTIPLEXER] ─► [Receiver2]
[Sender3] ─┘       (MUX)        channel       (DEMUX)         [Receiver3]

n signals → 1 combined signal → n signals again
```

#### Types of Multiplexing

##### FDM — Frequency Division Multiplexing

```
Frequency spectrum divided into bands:
│──────│──────│──────│──────│  ← shared cable
  Ch1    Ch2    Ch3    Ch4     ← different frequencies, simultaneous
```

- Each channel gets its own frequency band
- All signals travel simultaneously
- Used in: FM radio, cable TV, ADSL internet
- **Analogue** signals

##### TDM — Time Division Multiplexing

```
Time slots assigned to each channel:
│ Ch1 │ Ch2 │ Ch3 │ Ch1 │ Ch2 │ Ch3 │  ← time
Each channel gets its own time slot (take turns at full speed)
```

- Digital signals
- Used in: telephone networks, T1/E1 lines

##### WDM — Wavelength Division Multiplexing

```
Different colors of light on same fiber:
│ λ1(red) + λ2(green) + λ3(blue) │  ← single fiber
```

- Used in **optical fiber** networks
- DWDM (Dense WDM) — 80+ channels on one fiber
- Used in submarine cables, long-haul WAN links

| Type         | Signal   | Division   | Used In                         |
| ------------ | -------- | ---------- | ------------------------------- |
| **FDM**      | Analogue | Frequency  | FM radio, Cable TV              |
| **TDM**      | Digital  | Time slots | Telephony, T1 lines             |
| **WDM/DWDM** | Light    | Wavelength | Optical fiber, submarine cables |
| **CDM**      | Digital  | Code       | 3G cellular, GPS                |

---

### 3.6 Encoding

Converting data into a signal format suitable for transmission.

#### Why Encoding is Needed

```
Your laptop generates: Digital data (discrete — 0s and 1s)
Transmission medium may need: Analogue signal (continuous wave)
OR
Digital signal (but different format)
```

#### The 4 Types of Encoding

##### Digital-to-Digital

Represent digital data as digital signals.

```
Data:    1   0   1   1   0
NRZ-L: ─┐ ┌─┐ ┌─┐─┐
        │ │ │ │ │ │
        └─┘ └─┘ └─┘

Manchester: Rising edge = 1, Falling edge = 0
Used in: Ethernet (IEEE 802.3)
```

##### Analogue-to-Digital (Digitization)

Convert analogue signal (voice, audio) into digital data.

```
Voice (continuous wave) → Sampling → Quantization → Binary
Used in: VoIP, CD audio, digital telephony (PCM)
```

##### Digital-to-Analogue (Modulation)

Convert digital data to analogue signal for transmission over analogue medium.

```
Digital 1,0,1,1 → Modulated wave (carrier signal modified)
Types:
  AM (Amplitude Modulation) → change amplitude
  FM (Frequency Modulation)  → change frequency  ← FM radio
  PM (Phase Modulation)      → change phase
Used in: Modems, old dial-up internet, radio
```

##### Analogue-to-Analogue

Modulate one analogue signal with another.

```
Audio signal modulates a carrier radio frequency wave
Used in: AM/FM radio broadcast
```

---

## 4. Bits to Signals — The Core Job

```
DATA LINK LAYER passes BITS to Physical Layer:
0 1 0 1 1 0 0 1 0 1 1 1 0 0 ...

PHYSICAL LAYER:
Step 1: Take the bit stream
Step 2: Choose encoding scheme
Step 3: Convert bits → signal (electrical/light/radio)
Step 4: Transmit over chosen medium (wire/fiber/air)

At RECEIVER:
Step 1: Receive signal from medium
Step 2: Decode signal → bits
Step 3: Pass bits UP to Data Link Layer
```

---

## 5. Transmission Media — Guided vs Unguided

### Guided Media (Wired)

Signal is guided along a physical path (cable).

```
[Sender] ════════════════════ [Receiver]
            physical cable
```

| Medium            | Signal     | Bandwidth     | Attenuation | EMI Resistance |
| ----------------- | ---------- | ------------- | ----------- | -------------- |
| **UTP Copper**    | Electrical | Moderate      | High        | Low            |
| **Coaxial**       | Electrical | Moderate-High | Moderate    | High           |
| **Optical Fiber** | Light      | Very High     | Very Low    | Immune         |

### Unguided Media (Wireless)

Signal travels through air/space — no physical path.

```
[Sender] ))))  (((( [Receiver]
         radio waves / microwave / infrared
```

| Medium            | Frequency       | Range          | Example            |
| ----------------- | --------------- | -------------- | ------------------ |
| **Radio waves**   | 3Hz – 1GHz      | Long           | AM/FM radio, Wi-Fi |
| **Microwave**     | 1GHz – 300GHz   | Line of sight  | 4G/5G, satellite   |
| **Infrared**      | 300GHz – 400THz | Very short     | TV remote, IrDA    |
| **Visible light** | 400THz – 700THz | Short, indoors | Li-Fi              |

---

## 6. Signal Properties

Physical Layer works with signals — these properties matter:

| Property        | Definition                     | Security Relevance                       |
| --------------- | ------------------------------ | ---------------------------------------- |
| **Amplitude**   | Height/strength of wave        | Signal jamming attacks reduce amplitude  |
| **Frequency**   | Cycles per second (Hz)         | Frequency jamming targets specific bands |
| **Wavelength**  | Distance of one cycle          | Determines range and absorption          |
| **Phase**       | Position of wave in cycle      | Phase modulation used in some encoding   |
| **Bandwidth**   | Range of frequencies available | Higher = more data capacity              |
| **Attenuation** | Signal loss over distance      | Attackers exploit weak signals           |
| **Noise**       | Unwanted interference          | SNR (Signal-to-Noise Ratio) matters      |

### Shannon's Theorem (Channel Capacity)

```
C = B × log₂(1 + S/N)

Where:
  C = Maximum channel capacity (bps)
  B = Bandwidth (Hz)
  S/N = Signal-to-Noise Ratio

Higher noise → Lower capacity → More errors
```

---

## 7. Physical Layer vs Upper Layers — Key Difference

```
UPPER LAYERS (2-7)          PHYSICAL LAYER (1)
──────────────────          ──────────────────
Software + Hardware         Mostly Hardware
Logical/virtual data        Physical signals
Protocols, headers          Cables, connectors, signals
Encryption (Layer 6)        No encryption (hardware only)
IP addresses (Layer 3)      No addresses (just raw bits)
MAC addresses (Layer 2)     No addresses (just bits)
Can be simulated/emulated   Requires physical medium
```

> **Interview question:** "Can encryption happen at the Physical Layer?"
> **Answer:** No — encryption is software logic. It happens at Presentation Layer (Layer 6). Physical Layer just deals with raw signal transmission.

---

## 8. 🔴 Physical Layer Attack Surface

### Attacks Targeting Physical Layer

| Attack                                      | Description                                                    | How                             | Defense                                                |
| ------------------------------------------- | -------------------------------------------------------------- | ------------------------------- | ------------------------------------------------------ |
| **Wiretapping / Cable Tap**                 | Physically tapping a copper cable to intercept signals         | Induction coil or direct splice | Use fiber (light can't be tapped easily), encrypt data |
| **Fiber Optic Tapping**                     | Bending fiber causes light leakage — captured by photodetector | Physical access to cable        | Monitor optical power levels, use OTDR                 |
| **Signal Jamming**                          | Broadcasting noise on the same frequency → disrupts wireless   | Jammer device                   | Frequency hopping (FHSS), DSSS, shielding              |
| **Rogue Physical Device**                   | Planting a hardware keylogger, network tap, or rogue switch    | Physical access                 | Physical security, CCTV, lock server rooms             |
| **Evil Twin AP**                            | Fake Wi-Fi AP broadcasting stronger signal                     | hostapd + stronger antenna      | 802.1X auth, WPA3, certificate-based auth              |
| **Deauthentication Attack**                 | Sending 802.11 deauth frames → kicks clients off Wi-Fi         | aireplay-ng                     | 802.11w (Management Frame Protection)                  |
| **Electromagnetic Eavesdropping (TEMPEST)** | Capturing EM emissions from monitors/cables                    | Specialized antenna             | TEMPEST shielding (used by military/govt)              |
| **USB Drop Attack**                         | Dropping infected USB drives for victims to plug in            | BadUSB, Rubber Ducky            | Disable USB ports, user awareness training             |
| **Hub Sniffing**                            | On a hub network, all traffic visible to all ports             | Wireshark on any port           | Replace hubs with switches                             |
| **Attenuation-based DoS**                   | Cutting or damaging cables → loss of connectivity              | Physical access                 | Redundant links, physical security                     |

### Physical Security = Physical Layer Security

```
If attacker has PHYSICAL ACCESS → game over
  → They can tap cables
  → They can plug in rogue devices
  → They can steal hardware
  → They can cold boot attack RAM

Physical Layer Security means:
  ✓ Lock server rooms
  ✓ Use cable locks on devices
  ✓ Disable unused physical ports (switch ports, USB ports)
  ✓ Use fiber optic (harder to tap than copper)
  ✓ Monitor for rogue devices
  ✓ CCTV + access logs
```

### Wi-Fi Physical Layer Attacks (Wireless Medium)

```bash
# Check what Wi-Fi channels are in use near you
sudo iwlist wlan0 scan | grep -E "ESSID|Channel|Signal"

# Enable monitor mode for Wi-Fi analysis
sudo airmon-ng start wlan0
# Now wlan0 becomes wlan0mon — can capture all 802.11 frames

# Capture all Wi-Fi frames (management + data + control)
sudo airodump-ng wlan0mon
# This shows all SSIDs, their channels, MACs, encryption types

# Deauthentication flood (test on YOUR OWN network only)
sudo aireplay-ng --deauth 0 -a <YOUR_AP_MAC> wlan0mon
# This exploits the unprotected management frame vulnerability (Physical/L2)
```

---

## 9. 🧪 Practical Labs — Your Setup

### Lab 1 — Understand Your Physical Layer Setup

```bash
# See physical network interfaces on Parrot OS
ip link show
# eth0 or enp0s3 = Ethernet (wired — copper UTP)
# wlan0           = Wi-Fi (wireless — radio waves)
# lo              = Loopback (virtual — no physical layer)

# Check NIC details (driver, speed, duplex mode)
ethtool eth0
# Look for: Speed: 1000Mb/s, Duplex: Full
# Full = Full-Duplex transmission mode!
# 1000Mb/s = Gigabit Ethernet

# Check link status
ethtool eth0 | grep "Link detected"
```

### Lab 2 — Observe Signal Attenuation Effect

```bash
# Ping with increasing packet sizes to observe performance
ping -c 10 192.168.56.101                  # default 64 bytes
ping -c 10 -s 1024 192.168.56.101          # 1KB packets
ping -c 10 -s 32768 192.168.56.101         # 32KB packets
ping -c 10 -s 65507 192.168.56.101         # max size

# Look at round trip times — larger packets = more processing
# In real networks, larger packets = more attenuation effects
```

### Lab 3 — See Multiplexing in Action

```bash
# Multiple processes sending data simultaneously = multiplexing in practice
# Open multiple connections to Metasploitable2 at once

# Terminal 1: HTTP download
wget http://192.168.56.101/ -O /dev/null &

# Terminal 2: FTP connection
ftp 192.168.56.101 &

# Terminal 3: SSH connection
ssh msfadmin@192.168.56.101

# Now watch in Wireshark — multiple data streams on same interface
sudo wireshark -i eth0 &
# Different colors = different sessions = demultiplexing by port numbers
# This is Transport layer demux, but the Physical layer carries all of them on 1 wire
```

### Lab 4 — Wi-Fi Signal Analysis (Wireless Physical Layer)

```bash
# See available wireless networks and their signal strength (amplitude)
sudo iwlist wlan0 scan | grep -E "ESSID|Signal level|Frequency|Channel"

# Signal level example output:
# Signal level=-65 dBm  ← strength (closer to 0 = stronger)
# Frequency:2.412 GHz   ← which frequency band (FDM concept!)
# Channel:1             ← which channel within the band

# Plot signal over time using wavemon tool
sudo apt install wavemon
wavemon
# Watch signal strength fluctuate — this is attenuation in real time!
```

### Lab 5 — Hub vs Switch Behavior (Physical vs Data Link)

```bash
# Simulate hub behavior — see all traffic on the network
# On switch networks, you need ARP spoofing first

# With ARP spoofing active (from Lab in previous lectures):
sudo arpspoof -i eth0 -t 192.168.56.101 192.168.56.1 &
sudo arpspoof -i eth0 -t 192.168.56.1 192.168.56.101 &

# Now capture — you're acting like a hub (seeing all traffic)
sudo tcpdump -i eth0 -v
# Compare: without ARP spoof, you'd only see traffic to/from YOU

# This shows WHY hub (Physical Layer device) is insecure:
# Hub = all traffic visible to everyone (like being on same bus topology)
# Switch = traffic isolated per port (need Layer 2 attack to overcome)
```

### Lab 6 — USB Attack Concept (Physical Layer)

```bash
# See what USB devices are connected to your system
lsusb
# List of all USB devices — if unexpected device appears → rogue hardware

# Monitor for new USB device insertions (detect hardware attacks)
sudo udevadm monitor --subsystem-match=usb

# Plug/unplug something → see the event appear in real time
# In a real environment: unknown USB = potential BadUSB / keylogger

# Check USB storage auto-mount (disable for security)
sudo systemctl status udisks2
# Disable auto-mount:
# gsettings set org.gnome.desktop.media-handling automount false
```

---

## 10. Exam & Interview Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              PHYSICAL LAYER — EXAM CHEAT SHEET                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  POSITION                                                            ║
║  OSI Layer 1 — Lowest layer                                         ║
║  Sender: LAST layer to add functionality                            ║
║  Receiver: FIRST layer to process signal                            ║
╠══════════════════════════════════════════════════════════════════════╣
║  6 MAIN FUNCTIONALITIES                                              ║
║  1. Cables & Connectors  (UTP, Coaxial, Fiber | RJ45, BNC, LC)     ║
║  2. Physical Topology    (Point-to-Point, Bus, Star, Ring, Mesh)    ║
║  3. Hardware Devices     (Repeater, Hub)                            ║
║  4. Transmission Modes   (Simplex, Half-Duplex, Full-Duplex)        ║
║  5. Multiplexing         (FDM, TDM, WDM)                           ║
║  6. Encoding             (Digital↔Digital, Digital↔Analogue)        ║
╠══════════════════════════════════════════════════════════════════════╣
║  TRANSMISSION MODES                                                  ║
║  Simplex     → One direction only       (TV, keyboard)             ║
║  Half-Duplex → Both but not same time   (Walkie-talkie)            ║
║  Full-Duplex → Both simultaneously      (Phone, Ethernet)           ║
╠══════════════════════════════════════════════════════════════════════╣
║  MULTIPLEXING TYPES                                                  ║
║  FDM → Frequency Division  → Analogue → FM radio, Cable TV         ║
║  TDM → Time Division       → Digital  → Telephony, T1 lines        ║
║  WDM → Wavelength Division → Light    → Optical fiber, submarine   ║
╠══════════════════════════════════════════════════════════════════════╣
║  CABLES                                                              ║
║  UTP    → Cheapest, EMI sensitive, easy to tap                      ║
║  Coax   → Shielded, moderate cost                                   ║
║  Fiber  → Fastest, immune to EMI, hardest to tap (MOST SECURE)     ║
╠══════════════════════════════════════════════════════════════════════╣
║  HARDWARE DEVICES                                                    ║
║  Hub      → Layer 1, broadcasts to ALL ports (insecure)            ║
║  Repeater → Layer 1, amplifies/regenerates weak signal             ║
║  Switch   → Layer 2 (NOT physical layer)                           ║
║  Router   → Layer 3 (NOT physical layer)                           ║
╠══════════════════════════════════════════════════════════════════════╣
║  PHYSICAL LAYER PDU = BITS (0s and 1s)                             ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY ATTACKS                                                    ║
║  Wiretapping   → Tap copper cable (UTP easy, fiber hard)           ║
║  Jamming       → Disrupt wireless signal frequency                 ║
║  Rogue device  → Plug in hardware tap / keylogger                  ║
║  Evil Twin     → Stronger fake AP overpowers real AP               ║
║  Deauth attack → 802.11 management frames unprotected              ║
║  Hub sniffing  → All traffic visible on hub networks               ║
║  TEMPEST       → Capture EM emissions from devices                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY EXAM DISTINCTIONS                                               ║
║  Physical Layer = Hardware (tangible)                               ║
║  Upper Layers  = Software + Hardware (logical)                      ║
║  No encryption at Physical Layer (encryption = software = L6)      ║
║  No IP/MAC addresses at Physical Layer (just raw bits)             ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Data Link Layer (Layer 2) — Framing, MAC addressing, Error detection
- [ ] ARP — Address Resolution Protocol (IP → MAC)
- [ ] Ethernet — IEEE 802.3 in depth
- [ ] Wi-Fi — IEEE 802.11 in depth
- [ ] Network Layer (Layer 3) — IP addressing, subnetting

---

_Notes compiled from: Networking Course Lecture 04 — Physical Layer & Functionalities_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
