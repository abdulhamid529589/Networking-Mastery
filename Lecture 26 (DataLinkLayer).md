# 🔗 Data Link Layer — Framing, Byte Stuffing & Bit Stuffing

### " "Networking Course — Lecture 26

> **Source:** Gate Smashers — Framing in Data Link Layer
> " "
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is the Data Link Layer?](#1-what-is-the-data-link-layer)
2. [What is Framing?](#2-what-is-framing)
3. [Why Framing is Needed](#3-why-framing-is-needed)
4. [Character-Oriented Framing (Byte Stuffing)](#4-character-oriented-framing-byte-stuffing)
   - [4.1 Basic Concept — Flag Delimiter](#41-basic-concept--flag-delimiter)
   - [4.2 Problem — Flag in Data](#42-problem--flag-in-data)
   - [4.3 Solution — ESC Stuffing](#43-solution--esc-stuffing)
   - [4.4 Problem — ESC in Data](#44-problem--esc-in-data)
   - [4.5 Complete Byte Stuffing Rules](#45-complete-byte-stuffing-rules)
5. [Bit-Oriented Framing (Bit Stuffing)](#5-bit-oriented-framing-bit-stuffing)
   - [5.1 Concept](#51-concept)
   - [5.2 How Bit Stuffing Works](#52-how-bit-stuffing-works)
   - [5.3 Bit Destuffing at Receiver](#53-bit-destuffing-at-receiver)
6. [Byte Stuffing vs Bit Stuffing](#6-byte-stuffing-vs-bit-stuffing)
7. [Frame Structure](#7-frame-structure)
8. [🔴 Attack Surface — Framing & Protocol Exploitation](#8--attack-surface--framing--protocol-exploitation)
9. [🧪 Practical Labs — Your Setup](#9--practical-labs--your-setup)
10. [Exam Cheat Sheet](#10-exam-cheat-sheet)

---

## 1. What is the Data Link Layer?

Data Link Layer (OSI Layer 2) sits between the Physical Layer and Network Layer.

```
OSI Stack:
┌─────────────────────┐
│   Network Layer     │  ← sends/receives Packets
├─────────────────────┤
│  DATA LINK LAYER    │  ← We are here — deals with FRAMES
├─────────────────────┤
│   Physical Layer    │  ← sends/receives raw Bits
└─────────────────────┘
```

### Major Responsibilities of Data Link Layer

| Responsibility          | What it means                               |
| ----------------------- | ------------------------------------------- |
| **Framing**             | Pack bits into frames (this lecture)        |
| **Physical Addressing** | MAC address — source and destination        |
| **Error Control**       | Detect & correct bit errors (CRC, parity)   |
| **Flow Control**        | Prevent sender overwhelming receiver        |
| **Access Control**      | Who gets to use the shared medium (CSMA/CD) |

---

## 2. What is Framing?

### Definition

> Framing = The process of **dividing a stream of bits into discrete units called frames**, so the receiver can identify where one frame ends and the next begins.

### The Postal System Analogy

```
Without framing:
  DEAR JOHN I LOVE YOU DEAR JANE PLEASE HELP ME
  Everything is one big continuous stream — impossible to tell which
  letter belongs to whom.

With framing (envelopes = frames):
  [DEAR JOHN I LOVE YOU]  [DEAR JANE PLEASE HELP ME]
  ←─── Envelope 1 ────→  ←──────── Envelope 2 ────────→
  Each letter is clearly separated by its envelope (frame boundary).
```

Frames are the "envelopes" of the Data Link Layer.

### What Comes INTO Physical Layer → What Comes OUT as Frames

```
From Network Layer:  [  Packet (bits)  ]

Physical Layer sends raw bits:
0110100110101001101...continuous bit stream...10110101

Data Link Layer must figure out:
"Where does Frame 1 end and Frame 2 begin?"
That's FRAMING.

After Framing:
[FLAG][Frame 1 data][FLAG] [FLAG][Frame 2 data][FLAG]
```

---

## 3. Why Framing is Needed

Without framing, the receiver gets a continuous stream of bits with no idea where one message ends and another begins.

```
Problem:
Sender sends:  Hello  +  World
Bits arrive:   0100100001100101011011000110110001101111...
                        (all merged together)

Receiver sees: ????? (no boundaries)

With Framing:
[FLAG][Hello bits][FLAG][FLAG][World bits][FLAG]
Receiver clearly knows: Frame 1 = "Hello", Frame 2 = "World"
```

**Frame delimiters** = special bit/byte patterns that mark frame boundaries (start and end).

---

## 4. Character-Oriented Framing (Byte Stuffing)

### 4.1 Basic Concept — Flag Delimiter

A special character called **FLAG** is placed at the beginning and end of each frame.

```
Data from upper layer:   A B C D

After framing:
[FLAG] A B C D [FLAG]

Receiver rule:
  "Everything between the first FLAG and second FLAG = my data"
  Accept: A B C D ✓
  Discard: the FLAG characters themselves
```

```
Multiple frames:
[FLAG] A B C D [FLAG] [FLAG] X Y Z [FLAG]
        Frame 1                Frame 2
```

### 4.2 Problem — Flag in Data

What if the actual data contains the FLAG character?

```
Data from upper layer:  A [FLAG] C
(The FLAG character appears inside the actual data)

Sender naively frames it:
[FLAG] A [FLAG] C [FLAG]
  ↑           ↑       ↑
Start      ← receiver thinks this is END FLAG!
                  C [FLAG] → C is now outside any frame → LOST

Receiver accepts: A  ← only this, thinks frame ended
Discards: C        ← data loss!
```

**Problem:** Receiver cannot distinguish between a **delimiter FLAG** and a **data FLAG**.

### 4.3 Solution — ESC Stuffing

Insert an **Escape (ESC)** character immediately before any FLAG that appears in the data.

```
Data:  A [FLAG] C

Sender stuffs ESC before the data FLAG:
[FLAG] A [ESC][FLAG] C [FLAG]
  ↑                      ↑
Start                   End

Receiver rule:
  "If ESC appears before a FLAG → treat that FLAG as DATA, not delimiter"
  "If no ESC before a FLAG → it is a real delimiter"

Receiver reads: A → then sees ESC → next char is FLAG → treat as data
Result: A [FLAG] C ✓ (correct, data recovered)
```

```
Step by step:
Original data:     A   FLAG   C
After ESC stuff:   A   ESC   FLAG   C
Transmitted:    [FLAG] A ESC FLAG C [FLAG]
                  ↑                   ↑
               delimiter           delimiter

Receiver:
  Sees FLAG → start of frame
  Reads A → data
  Reads ESC → next char is special
  Reads FLAG → because ESC before it → treat as DATA
  Reads C → data
  Sees FLAG (no ESC before it) → end of frame

Final received data: A FLAG C ✓
```

### 4.4 Problem — ESC in Data

What if the actual data contains the ESC character?

```
Data:  A [ESC] C

Sender must handle this too.
If sent as: [FLAG] A [ESC] C [FLAG]
Receiver sees ESC → "next char is special"
Reads C → treats C as if it's escaped → ERROR

Solution: Stuff another ESC before any ESC in data.
```

```
Data:   A  ESC  C
Stuffed: A  ESC  ESC  C
Transmitted: [FLAG] A [ESC][ESC] C [FLAG]

Receiver rule:
  "If ESC followed by ESC → treat second ESC as DATA, not escape signal"

Receiver reads:
  ESC → next is special
  ESC → because ESC before it → treat as DATA ESC
  C → data

Final received: A ESC C ✓ (correct)
```

### 4.5 Complete Byte Stuffing Rules

#### Sender Rules (Stuffing)

```
Rule 1: Add FLAG at the start of every frame
Rule 2: Add FLAG at the end of every frame
Rule 3: If data contains FLAG → insert ESC before that FLAG
Rule 4: If data contains ESC  → insert ESC before that ESC
```

#### Receiver Rules (Destuffing)

```
Rule 1: When FLAG received (with no ESC before it) → frame boundary
Rule 2: When ESC received → look at next character:
          - If next is FLAG → discard ESC, keep FLAG as DATA
          - If next is ESC  → discard first ESC, keep second ESC as DATA
```

#### Full Example (All Cases)

```
Original data:  A  FLAG  ESC  B

Sender stuffing process:
  A    → no special → send as A
  FLAG → FLAG in data → send ESC FLAG
  ESC  → ESC in data  → send ESC ESC
  B    → no special → send as B

Transmitted frame:
[FLAG] A [ESC][FLAG] [ESC][ESC] B [FLAG]

Receiver destuffing:
  FLAG → start of frame
  A    → data A
  ESC  → escape signal
  FLAG → ESC before it → DATA FLAG
  ESC  → escape signal
  ESC  → ESC before it → DATA ESC
  B    → data B
  FLAG → no ESC before it → end of frame

Received data: A FLAG ESC B ✓ (exactly original)
```

#### Overhead Calculation

Each stuffed character adds **1 extra byte** of overhead.

```
If data has 3 FLAG characters and 2 ESC characters:
  Overhead = 3 + 2 = 5 extra bytes added by stuffing
  Receiver removes all stuffed bytes during destuffing
```

---

## 5. Bit-Oriented Framing (Bit Stuffing)

### 5.1 Concept

Instead of working with whole characters (bytes), bit stuffing works at the individual **bit level**.

Used in protocols like: **HDLC (High-Level Data Link Control)**, **PPP**, **CAN bus**

**Frame delimiter (flag)** in bit stuffing = `01111110` (0 followed by six 1s)

```
Start of frame: 01111110
End of frame:   01111110

Problem: What if this pattern appears in the data?
Solution: Bit stuffing
```

### 5.2 How Bit Stuffing Works

**Rule:** After every **5 consecutive 1s** in the data, the sender inserts a **0**.

This ensures the pattern `011111` never appears naturally in data — only in the frame delimiter.

```
Original data:  0 1 1 1 1 1 0 0 1 1 1 1 1 1 0
                          ↑                 ↑
                      5 ones             6 ones (would look like delimiter!)

After bit stuffing:
  Count 1s consecutively:
  0 → reset count
  1 → count=1
  1 → count=2
  1 → count=3
  1 → count=4
  1 → count=5 → INSERT 0
  0 → (already inserted)
  0 → reset count
  1 → count=1
  1 → count=2
  1 → count=3
  1 → count=4
  1 → count=5 → INSERT 0
  1 → count=1 (after reset)
  0 → reset

Stuffed data: 0 1 1 1 1 1 [0] 0 0 1 1 1 1 1 [0] 1 0
                           ↑ inserted           ↑ inserted
```

#### Worked Example from Lecture

```
Input data:   0 1 1 1 1 1 1 0
                         ↑ ↑
              5 ones → stuff 0 after 5th one

Stuffed:      0 1 1 1 1 1 [0] 1 0
                           ↑
                     stuffed bit

Full frame:   [01111110] 0 1 1 1 1 1 0 1 0 [01111110]
               ↑ start                        ↑ end
               flag                           flag
```

### 5.3 Bit Destuffing at Receiver

**Receiver rule:** After receiving 5 consecutive 1s:

- If next bit is **0** → **discard** that 0 (it was stuffed), continue reading data
- If next bit is **1** → this is the **frame delimiter flag** (01111110 pattern) → end of frame

```
Receiver gets:  0 1 1 1 1 1 0 1 0
                          ↑
                After 5 ones, sees 0 → discard it (stuffed bit)
                Continue reading: 1 0

Recovered data: 0 1 1 1 1 1 1 0  ✓ (original restored)
```

---

## 6. Byte Stuffing vs Bit Stuffing

| Parameter            | Byte Stuffing (Character)                        | Bit Stuffing                          |
| -------------------- | ------------------------------------------------ | ------------------------------------- |
| **Unit**             | Byte (character)                                 | Bit                                   |
| **Flag size**        | 1 byte (e.g., `0x7E`)                            | 8 bits (`01111110`)                   |
| **Escape mechanism** | ESC character before FLAG/ESC in data            | Insert 0 after five 1s                |
| **Overhead**         | Variable — depends on FLAG/ESC frequency in data | Variable — depends on 1-run frequency |
| **Protocols**        | PPP (early), BSC, DDCMP                          | HDLC, PPP, CAN, SDLC                  |
| **Data type**        | Character/text data                              | Any binary data                       |
| **Complexity**       | Simple logic                                     | Simpler — only one rule               |
| **Used in**          | Character-oriented protocols                     | Bit-oriented protocols                |

---

## 7. Frame Structure

A complete Data Link Layer frame includes more than just the data:

```
┌──────────┬────────────┬────────────┬──────────┬─────────┬──────────┐
│  FLAG    │  Header    │    Data    │ Trailer  │  FCS    │  FLAG    │
│(delimiter)│(MAC addrs)│(IP packet) │          │(CRC)    │(delimiter)│
└──────────┴────────────┴────────────┴──────────┴─────────┴──────────┘

FLAG    = Frame boundary marker (byte or bit pattern)
Header  = Source MAC, Destination MAC, frame type
Data    = The actual payload (IP packet from Network Layer)
Trailer = End-of-frame marker (in some protocols)
FCS     = Frame Check Sequence (CRC for error detection)
```

### Ethernet Frame Structure (Real-world)

```
┌──────────┬──────────┬──────┬──────────────┬─────┐
│ Preamble │ Dest MAC │ Src  │   EtherType  │Data │ FCS │
│ 8 bytes  │ 6 bytes  │ MAC  │   2 bytes    │46-  │4B   │
│          │          │ 6B   │(IPv4=0x0800) │1500B│     │
└──────────┴──────────┴──────┴──────────────┴─────┘

Preamble = 10101010...10101011 (sync signal — physical layer sync)
Dest MAC = Who should receive this frame
Src MAC  = Who sent this frame
FCS      = CRC-32 error detection
```

---

## 8. 🔴 Attack Surface — Framing & Protocol Exploitation

### Why Framing Matters for Security

Framing vulnerabilities are a rich class of attacks. Improperly handled frame boundaries lead to critical bugs.

### Attack 1 — Frame Injection

```
Attacker crafts a malicious frame with forged source MAC:

[FLAG][MALICIOUS DATA][SRC MAC=victim][DST MAC=server][FLAG]

Switch forwards it based on MAC → server thinks victim sent it
Used in: ARP poisoning, MAC spoofing
```

### Attack 2 — Frame Boundary Confusion (Framing Attack)

```
If framing implementation is buggy:
  Attacker sends: [FLAG][DATA][ESC][FLAG][MALICIOUS][FLAG]

  Buggy receiver might not handle ESC-FLAG correctly
  → Interprets MALICIOUS as part of previous frame
  → Or as a new frame entirely

This class of bug causes: buffer overflows, data corruption, authentication bypass
```

### Attack 3 — Protocol Fuzzing (Malformed Frames)

```
Send deliberately malformed frames to find parsing bugs:

  Missing end FLAG:      [FLAG][DATA]  (no closing flag)
  Double FLAG:           [FLAG][FLAG][DATA][FLAG]
  ESC at end:            [FLAG][DATA][ESC][FLAG]
  Oversized frame:       [FLAG][10MB of data][FLAG]
  Undersized frame:      [FLAG][FLAG] (empty)

Each of these can crash a buggy implementation.
This is exactly what network fuzzers do.
```

### Attack 4 — HDLC/PPP Bit Stuffing Exploitation

```
In older WAN protocols using bit stuffing:
  Send data that forces maximum stuffing overhead:
  01111101111101111101111... (lots of five-1-sequences)

  Each five 1s → inserts a 0 → increases frame size
  Used to cause bandwidth exhaustion on links
```

### Attack 5 — Ethernet Frame Padding Attack

```
Ethernet minimum frame = 64 bytes
Shorter payloads get PADDED to 64 bytes

Old systems leaked memory content in padding:
  [Real data: 20 bytes][PADDING from RAM: 44 bytes]

  The padding could contain sensitive data from previous frames in memory!
  This is why modern NICs zero-fill padding.
```

### Attack 6 — VLAN Hopping (802.1Q Frame Manipulation)

```
Ethernet frames can carry 802.1Q VLAN tags:
[Dest MAC][Src MAC][802.1Q tag: VLAN ID][EtherType][Data][FCS]

Double-tagging attack:
[VLAN tag: 1][VLAN tag: 100][Data]
First switch strips tag 1 → forwards to VLAN 1 trunk
Second switch strips tag 100 → delivers to VLAN 100
Attacker in VLAN 1 reaches VLAN 100 — bypasses isolation!
```

---

## 9. 🧪 Practical Labs — Your Setup

### Lab 1 — See Real Ethernet Frames in Wireshark

```bash
# Start Wireshark, capture on eth0
sudo wireshark &

# Generate some traffic to Metasploitable2
ping 192.168.56.101
curl http://192.168.56.101

# In Wireshark — click any Ethernet packet
# Expand: "Ethernet II" section
# You will see:
#   Destination: [MAC address]
#   Source: [MAC address]
#   Type: IPv4 (0x0800) or ARP (0x0806)
# This is the frame HEADER (framing in action)

# To see raw frame bytes:
# Right-click packet → "Show Packet in New Window"
# Bottom panel shows hex dump of entire frame
```

### Lab 2 — Capture and Analyze Frame Boundaries

```bash
# Capture raw frames with tcpdump (shows frame-level data)
sudo tcpdump -i eth0 -XX -v 2>/dev/null | head -100
# -XX = print frame in hex AND ASCII
# -v  = verbose (show frame details)

# Look for:
# Frame start (Preamble not shown — stripped by NIC)
# Source/Destination MAC (first 12 bytes after preamble)
# EtherType (bytes 13-14: 0800=IPv4, 0806=ARP)
# FCS at end (last 4 bytes — CRC)
```

### Lab 3 — MAC Spoofing (Frame Header Manipulation)

```bash
# View your current MAC address
ip link show eth0 | grep ether
# Example: ether 08:00:27:xx:xx:xx

# Change your MAC (spoof it) — modifies src MAC in frames you send
sudo ip link set eth0 down
sudo ip link set eth0 address 00:11:22:33:44:55
sudo ip link set eth0 up

# Verify
ip link show eth0 | grep ether
# Now shows: ether 00:11:22:33:44:55

# All frames you now send will have this spoofed src MAC
# This fools switches and other devices about your identity
ping 192.168.56.101
# Capture in Wireshark → see spoofed MAC in frame header

# Restore original MAC (restart VM or set back)
sudo ip link set eth0 down
sudo ip link set eth0 address 08:00:27:original:mac
sudo ip link set eth0 up
```

### Lab 4 — Simulate Byte Stuffing in Python

```python
# Save as byte_stuffing.py and run: python3 byte_stuffing.py

FLAG = 0x7E  # 126 decimal — common flag byte
ESC  = 0x7D  # 125 decimal — common escape byte

def byte_stuff(data: bytes) -> bytes:
    """Sender: stuff the data, wrap with flags"""
    stuffed = [FLAG]
    for byte in data:
        if byte == FLAG:
            stuffed.append(ESC)
            stuffed.append(FLAG)
        elif byte == ESC:
            stuffed.append(ESC)
            stuffed.append(ESC)
        else:
            stuffed.append(byte)
    stuffed.append(FLAG)
    return bytes(stuffed)

def byte_destuff(frame: bytes) -> bytes:
    """Receiver: remove flags and unstuff"""
    data = []
    i = 1  # skip opening FLAG
    while i < len(frame) - 1:  # skip closing FLAG
        if frame[i] == ESC:
            i += 1
            data.append(frame[i])  # next byte is actual data
        else:
            data.append(frame[i])
        i += 1
    return bytes(data)

# Test cases
test_cases = [
    b"ABCD",                          # normal data
    bytes([0x41, FLAG, 0x43]),         # FLAG in data
    bytes([0x41, ESC, 0x43]),          # ESC in data
    bytes([FLAG, ESC, FLAG]),          # both FLAG and ESC in data
]

for original in test_cases:
    framed    = byte_stuff(original)
    recovered = byte_destuff(framed)

    print(f"Original:  {list(original)}")
    print(f"Framed:    {list(framed)}")
    print(f"Recovered: {list(recovered)}")
    print(f"Match: {'✓' if original == recovered else '✗ MISMATCH!'}")
    print()
```

### Lab 5 — Simulate Bit Stuffing in Python

```python
# Save as bit_stuffing.py and run: python3 bit_stuffing.py

def bit_stuff(data: str) -> str:
    """Sender: insert 0 after every 5 consecutive 1s"""
    stuffed = []
    count_ones = 0
    for bit in data:
        stuffed.append(bit)
        if bit == '1':
            count_ones += 1
            if count_ones == 5:
                stuffed.append('0')  # stuff a 0
                count_ones = 0
        else:
            count_ones = 0
    return ''.join(stuffed)

def bit_destuff(data: str) -> str:
    """Receiver: remove stuffed 0 after every 5 consecutive 1s"""
    destuffed = []
    count_ones = 0
    i = 0
    while i < len(data):
        bit = data[i]
        if bit == '1':
            destuffed.append(bit)
            count_ones += 1
            if count_ones == 5:
                i += 1  # skip next bit (it's the stuffed 0)
                count_ones = 0
        else:
            destuffed.append(bit)
            count_ones = 0
        i += 1
    return ''.join(destuffed)

# Test cases
test_cases = [
    "01111110",      # six 1s — would look like flag
    "011111011111",  # two runs of five 1s
    "0101010101",    # alternating — no stuffing needed
    "111111111",     # nine 1s — two stuffs needed
]

for original in test_cases:
    stuffed   = bit_stuff(original)
    recovered = bit_destuff(stuffed)

    print(f"Original:  {original}")
    print(f"Stuffed:   {stuffed}")
    print(f"Recovered: {recovered}")
    print(f"Match: {'✓' if original == recovered else '✗ MISMATCH!'}")
    print()
```

### Lab 6 — VLAN Hopping (Frame Manipulation)

```bash
# See VLAN tags in captured frames (if your network uses VLANs)
sudo tcpdump -i eth0 -e vlan
# -e shows link-level headers including 802.1Q VLAN tags

# Craft a double-tagged 802.1Q frame (educational — your lab only)
# Using Scapy:
sudo python3 - << 'EOF'
from scapy.all import *

# Double-tagged frame (VLAN hopping concept)
frame = Ether(dst="ff:ff:ff:ff:ff:ff") / \
        Dot1Q(vlan=1) / \
        Dot1Q(vlan=100) / \
        IP(dst="192.168.56.101") / \
        ICMP()

# Show the frame structure
frame.show()
# sendp(frame, iface="eth0")  # uncomment to actually send (lab only!)
EOF
```

---

## 10. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║         FRAMING — DATA LINK LAYER EXAM CHEAT SHEET                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHAT IS FRAMING?                                                    ║
║  Packing bits into discrete units (frames) with clear boundaries    ║
║  so receiver knows where one frame ends and next begins             ║
║  Postal analogy: frame = envelope around data                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  BYTE STUFFING (Character-Oriented Framing)                         ║
║  ─────────────────────────────────────────────────────             ║
║  FLAG = delimiter (marks frame start/end)                           ║
║  ESC  = escape character                                            ║
║                                                                      ║
║  SENDER Rules:                                                       ║
║    1. Wrap data with FLAG...FLAG                                    ║
║    2. FLAG in data → insert ESC before it → ESC FLAG               ║
║    3. ESC in data  → insert ESC before it → ESC ESC                ║
║                                                                      ║
║  RECEIVER Rules:                                                     ║
║    1. FLAG (no ESC before) → frame boundary                        ║
║    2. ESC FLAG             → treat FLAG as data                    ║
║    3. ESC ESC              → treat second ESC as data              ║
║                                                                      ║
║  Overhead: 1 extra byte per FLAG or ESC in data                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  BIT STUFFING (Bit-Oriented Framing)                                ║
║  ─────────────────────────────────────────────────────             ║
║  Flag pattern: 01111110 (0 + six 1s)                               ║
║                                                                      ║
║  SENDER Rule: After every 5 consecutive 1s → insert 0              ║
║  RECEIVER Rule: After 5 consecutive 1s:                            ║
║    - Next bit = 0 → discard it (was stuffed)                       ║
║    - Next bit = 1 → frame boundary (end flag)                      ║
║                                                                      ║
║  Used in: HDLC, PPP, CAN bus                                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  COMPARISON                                                          ║
║  Byte stuffing → works on whole bytes (characters)                 ║
║  Bit stuffing  → works on individual bits                          ║
║  Both → sender stuffs, receiver destuffs using SAME agreed rules   ║
╠══════════════════════════════════════════════════════════════════════╣
║  FRAME = FLAG + Header(MACs) + Data(IP packet) + FCS + FLAG        ║
║  PDU at Data Link Layer = FRAME                                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY ATTACKS ON FRAMING                                         ║
║  Frame injection  → forge src MAC in frame header                  ║
║  MAC spoofing     → change src MAC address in frames sent          ║
║  VLAN hopping     → double 802.1Q tags to cross VLAN boundaries    ║
║  Protocol fuzzing → malformed frame boundaries crash parsers       ║
║  Padding attack   → old NICs leaked memory in Ethernet padding     ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY RULE (Both types):                                              ║
║  Sender and receiver MUST agree on same rules BEFORE communication ║
║  Like rules of a game — both sides know them beforehand            ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Data Link Layer — Error Detection (CRC, Parity, Checksum)
- [ ] Data Link Layer — Error Correction (Hamming Code)
- [ ] Data Link Layer — Flow Control (Stop-and-Wait, Sliding Window)
- [ ] MAC Addressing — ARP, MAC table
- [ ] Network Layer — IP Addressing & Subnetting

---

_Notes compiled from: Networking Course Lecture 06 — Framing in Data Link Layer_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
" "
