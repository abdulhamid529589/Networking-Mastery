# 🔍 Parity Check & Hamming Distance

### Cybersecurity Student Notes | Networking Course — Lecture 08

> **Source:** Gate Smashers — Single Parity Bit & Hamming Distance
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Single Parity Bit — Overview](#1-single-parity-bit--overview)
2. [Even Parity vs Odd Parity](#2-even-parity-vs-odd-parity)
3. [How Parity Works — Sender Side](#3-how-parity-works--sender-side)
4. [How Parity Works — Receiver Side](#4-how-parity-works--receiver-side)
5. [What Parity Can and Cannot Detect](#5-what-parity-can-and-cannot-detect)
6. [Hamming Distance](#6-hamming-distance)
   - [6.1 Definition & Calculation](#61-definition--calculation)
   - [6.2 Minimum Hamming Distance](#62-minimum-hamming-distance)
   - [6.3 Detection Capability Formula](#63-detection-capability-formula)
   - [6.4 Full Worked Example](#64-full-worked-example)
7. [Valid vs Invalid Code Words](#7-valid-vs-invalid-code-words)
8. [🔴 Security Context](#8--security-context)
9. [🧪 Practical Labs](#9--practical-labs)
10. [Exam Cheat Sheet](#10-exam-cheat-sheet)

---

## 1. Single Parity Bit — Overview

### What is it?

Single Parity Bit is the **simplest and least expensive** error detection method.

```
m bits of data  +  1 parity bit  =  (m+1) bits transmitted
```

### Why "Least Expensive"?

Cost here means **overhead** (extra bits sent per data block):

| Method            | Overhead                                     |
| ----------------- | -------------------------------------------- |
| **Single Parity** | **1 bit per m-bit block** ← least            |
| 2D Parity         | Multiple bits (one per row + one per column) |
| Checksum          | 16 bits                                      |
| CRC               | 16 or 32 bits                                |

More redundant bits → better detection → more expensive (more bandwidth used for overhead).

For applications that **can tolerate** some errors (audio, video streaming) and want minimal overhead → Single Parity is an option.

---

## 2. Even Parity vs Odd Parity

Both sender and receiver must **agree** on which type before communication.

### Even Parity

> The total number of **1s** in the transmitted code word (data + parity bit) must be **EVEN**.

```
Data:        1 0 1 0    → count 1s → 2 ones (even)
Parity bit:  0          → adding 0 keeps count even
Code word:   1 0 1 0 | 0   ← transmitted

Data:        1 1 1 0    → count 1s → 3 ones (odd)
Parity bit:  1          → adding 1 makes count even (4 ones total)
Code word:   1 1 1 0 | 1   ← transmitted
```

### Odd Parity

> The total number of **1s** must be **ODD**.

```
Data:        1 0 1 0    → 2 ones (even)
Parity bit:  1          → adding 1 makes count odd (3 total)
Code word:   1 0 1 0 | 1

Data:        1 1 1 0    → 3 ones (odd)
Parity bit:  0          → adding 0 keeps count odd
Code word:   1 1 1 0 | 0
```

### Which is Used?

- **Even Parity** is more commonly used in practice
- Both work the same way — just a convention both sides agree to

---

## 3. How Parity Works — Sender Side

### Step-by-Step Process

```
Step 1: Count the number of 1s in the data word.
Step 2: Determine the required parity bit:
          Even parity: parity = 0 if count is even, 1 if count is odd
          Odd parity:  parity = 1 if count is even, 0 if count is odd
Step 3: Append parity bit to the data → form the code word.
Step 4: Transmit the code word.
```

### Examples (Even Parity)

| Data   | Count of 1s | Parity Bit | Code Word |
| ------ | ----------- | ---------- | --------- |
| `1010` | 2 (even)    | `0`        | `1010 0`  |
| `1110` | 3 (odd)     | `1`        | `1110 1`  |
| `0000` | 0 (even)    | `0`        | `0000 0`  |
| `1111` | 4 (even)    | `0`        | `1111 0`  |
| `1011` | 3 (odd)     | `1`        | `1011 1`  |
| `0101` | 2 (even)    | `0`        | `0101 0`  |

---

## 4. How Parity Works — Receiver Side

### Step-by-Step Process

```
Step 1: Receive the code word (data + parity bit).
Step 2: Count ALL 1s in the entire received code word.
Step 3: Check the agreed parity rule:
          Even parity: count should be even
          Odd parity:  count should be odd
Step 4: If check passes → accept (no error detected)
        If check fails → error detected → request retransmission
```

### Example — Error Detected

```
Sender transmits:    1 1 1 0 1    (data=1110, parity=1, total 1s=4 ✓ even)

Noise flips bit 1:   0 1 1 0 1    (first bit changed 1→0)
                     ↑

Receiver counts 1s:  0+1+1+0+1 = 3  (ODD — should be EVEN)
Result: ERROR DETECTED ✓
```

### Example — Error NOT Detected (2 bits flip)

```
Sender transmits:    1 1 1 0 1    (total 1s = 4, even ✓)

Noise flips bits 1&2: 0 0 1 0 1   (two bits changed)
                      ↑ ↑

Receiver counts 1s:  0+0+1+0+1 = 2  (EVEN — passes parity check!)
Result: NO ERROR DETECTED ✗ (but there IS an error — parity MISSED it)
```

---

## 5. What Parity Can and Cannot Detect

### Detection Rule (Key Exam Point)

```
Single Parity detects errors when the NUMBER OF FLIPPED BITS IS ODD.

Can detect:    1 bit flip (single bit error)
               3 bit flips
               5 bit flips
               7 bit flips
               Any ODD number of flips

Cannot detect: 2 bit flips
               4 bit flips
               6 bit flips
               Any EVEN number of flips
```

### Why?

```
Even number of flips → each pair of flips cancels out in the parity count
                     → total 1s count changes by an even number
                     → parity check still passes → ERROR MISSED

Odd number of flips  → total 1s count changes by odd number
                     → parity check FAILS → ERROR DETECTED
```

### Can Parity Correct Errors?

**NO.** Parity can only DETECT, NOT correct.

```
Receiver detects error (parity fails).
But receiver knows ONLY that "some bit changed".
Does NOT know WHICH bit changed.
Cannot fix it — must request retransmission (ARQ).
```

### Summary Table

| Flipped Bits | Count | Detected? | Corrected? |
| ------------ | ----- | --------- | ---------- |
| 1            | Odd   | ✅ Yes    | ❌ No      |
| 2            | Even  | ❌ No     | ❌ No      |
| 3            | Odd   | ✅ Yes    | ❌ No      |
| 4            | Even  | ❌ No     | ❌ No      |
| 5            | Odd   | ✅ Yes    | ❌ No      |
| Any odd      | Odd   | ✅ Yes    | ❌ No      |
| Any even     | Even  | ❌ No     | ❌ No      |

---

## 6. Hamming Distance

### 6.1 Definition & Calculation

> **Hamming Distance** between two code words = number of bit positions in which they **differ**.

### How to Calculate

```
Method: XOR the two code words → count the number of 1s in the result.

XOR truth table:
  Same bits  → 0  (no difference)
  Diff bits  → 1  (difference here)
```

### Worked Examples

**Example 1:**

```
Code word A: 0 0 0 0
Code word B: 1 1 1 1

XOR result:  1 1 1 1

Count 1s in XOR = 4
Hamming Distance = 4
```

**Example 2:**

```
Code word A: 0 1 0 1
Code word B: 1 0 0 0

XOR:         1 1 0 1

Count 1s = 3
Hamming Distance = 3
```

**Example 3:**

```
Code word A: 1 1 0 0 1
Code word B: 1 0 0 0 1

XOR:         0 1 0 0 0

Count 1s = 1
Hamming Distance = 1
```

---

### 6.2 Minimum Hamming Distance

> **Minimum Hamming Distance (d_min)** of a code = the **smallest** Hamming Distance between **any two valid code words** in the entire set.

This is the most important metric for evaluating an error detection scheme.

```
Valid code word set (even parity, 4-bit data):
00000, 00011, 00101, 00110,
01001, 01010, 01100, 01111,
10001, 10010, 10100, 10111,
11000, 11011, 11101, 11110

Find the pair with the SMALLEST Hamming Distance.
That value = minimum Hamming Distance of the code.
```

### Calculating d_min for Even Parity Example

```
Take any two valid code words close to each other:
  00000 and 11000 → XOR = 11000 → 2 ones → distance = 2
  00000 and 00011 → XOR = 00011 → 2 ones → distance = 2

  Can you find any pair with distance = 1? NO.
  (Changing 1 bit of a valid code word makes total 1s odd → invalid)

  Minimum distance = 2
```

**Key insight for single parity:**

- Any valid code word has **even** number of 1s
- Changing **1 bit** → odd number of 1s → NOT a valid code word
- So the minimum gap between any two valid code words = **2**
- Therefore: d_min = **2** for single parity

---

### 6.3 Detection Capability Formula

> **A code with minimum Hamming Distance d_min can detect up to (d_min − 1) bit errors.**

```
Formula:
  Detectable errors = d_min − 1

For single parity (d_min = 2):
  Detectable errors = 2 − 1 = 1

This matches what we already know: single parity detects only 1-bit errors.
```

### Why This Formula Works

```
If d_min = 2, that means:
  Every pair of valid code words differs in at least 2 positions.

  If 1 bit is flipped:
    Result is NOT a valid code word (distance to nearest valid = 1, not 0)
    Receiver knows error occurred ✓

  If 2 bits are flipped:
    Result MIGHT be a valid code word (2 flips can reach another valid code word)
    Receiver cannot distinguish: "error happened" vs "different data was sent"
    ERROR MISSED ✗
```

```
Visualization (d_min = 2):

Valid code words: ●        ●        ●
                 (distance between any two ≥ 2)

If 1 bit flips: ●  ×      ●        ●
                     ↑
                 Not a valid code word → detected

If 2 bits flip: ●         ●        ●
                ↑←─ 2 ──→↑
                Result lands on another valid code word → missed
```

---

### 6.4 Full Worked Example

**Question:** Given these valid code words: `{000, 011, 101, 110}`

1. Calculate all pairwise Hamming distances
2. Find d_min
3. How many bit errors can be detected?

```
All pairs and their XOR:

000 XOR 011 = 011 → 2 ones → distance = 2
000 XOR 101 = 101 → 2 ones → distance = 2
000 XOR 110 = 110 → 2 ones → distance = 2
011 XOR 101 = 110 → 2 ones → distance = 2
011 XOR 110 = 101 → 2 ones → distance = 2
101 XOR 110 = 011 → 2 ones → distance = 2

All distances = 2
d_min = 2

Detectable errors = d_min − 1 = 2 − 1 = 1

This code can detect any single bit error.
Cannot detect 2-bit errors (they land on another valid code word).
```

---

## 7. Valid vs Invalid Code Words

### The Concept

Not every possible bit pattern is a **valid code word**. The parity constraint filters out invalid patterns.

```
For 4-bit data with 1 even parity bit:
Total possible 5-bit patterns: 2⁵ = 32
Valid code words (even parity): 2⁴ = 16  (half of all patterns)
Invalid code words:              16  (the other half — odd parity patterns)
```

### Why This Matters for Detection

```
Valid code words:   ●    ●    ●    ●    ●    (16 out of 32)
Invalid patterns:        ×    ×    ×    ×    (16 out of 32)

If receiver gets an INVALID pattern → knows error occurred
If receiver gets a VALID pattern   → thinks no error (but might be wrong!)
```

### Complete Valid Code Word Set (4-bit data, even parity)

```
Data    Parity  Code Word   1s count
0000  |   0   |  00000   |    0  (even ✓)
0001  |   1   |  00011   |    2  (even ✓)
0010  |   1   |  00101   |    2  (even ✓)
0011  |   0   |  00110   |    2  (even ✓)
0100  |   1   |  01001   |    2  (even ✓)
0101  |   0   |  01010   |    2  (even ✓)
0110  |   0   |  01100   |    2  (even ✓)
0111  |   1   |  01111   |    4  (even ✓)
1000  |   1   |  10001   |    2  (even ✓)
1001  |   0   |  10010   |    2  (even ✓)
1010  |   0   |  10100   |    2  (even ✓)
1011  |   1   |  10111   |    4  (even ✓)
1100  |   0   |  11000   |    2  (even ✓)
1101  |   1   |  11011   |    4  (even ✓)
1110  |   1   |  11101   |    4  (even ✓)
1111  |   0   |  11110   |    4  (even ✓)
```

---

## 8. 🔴 Security Context

### Parity is NOT Security

```
Parity detects accidental errors.
Parity CANNOT detect intentional manipulation by an attacker.

Why?
  Attacker flips 2 bits → parity still passes → undetected
  Attacker can choose WHICH 2 bits to flip to change meaning while passing parity

Example attack:
  Data: price = $100 = 01100100 (ASCII encoding)
  Attacker flips 2 bits strategically → price = $900
  Even parity still passes
  Transaction goes through with wrong amount
```

### Parity in RAM — ECC vs Non-ECC

```
ECC RAM (Error Correcting Code):
  Uses Hamming code (not just parity)
  Can CORRECT single bit errors in RAM
  Used in servers, workstations with critical data

Non-ECC RAM (standard):
  Uses simple parity or no parity at all
  Cannot correct — only detect (or not even that)

Security relevance:
  Rowhammer attack: repeatedly accessing RAM rows causes bit flips in nearby rows
  On non-ECC RAM: attacker can flip bits in page tables → escalate privileges!
  On ECC RAM: single bit flips auto-corrected, harder to exploit
```

### Hamming Distance in Cryptography

```
Hamming Distance concept is used in:

1. Password similarity checking
   "password" vs "password1" → HD = 1 (add 1 char)
   Small Hamming Distance = passwords too similar

2. Fuzzy hashing (SSDEEP)
   Compare malware samples: similar code has small Hamming Distance
   Used in malware detection

3. Error-correcting codes in cryptography
   Lattice-based cryptography (post-quantum) uses HD
   McEliece cryptosystem based on Hamming Distance

4. Side-channel attacks
   Power analysis uses Hamming Distance/Weight of data being processed
   More 1s → more power consumed → attacker infers secret key bits
```

### Hamming Weight (Related Concept)

```
Hamming Weight of a code word = number of 1s in it
                               = Hamming Distance from all-zeros

Hamming Weight("10110") = 3
Hamming Weight("00000") = 0
Hamming Weight("11111") = 5

Used in:
  Side-channel attacks (power analysis)
  Popcount operations in CPUs
  Bloom filters in network security
```

---

## 9. 🧪 Practical Labs

### Lab 1 — Parity Generator & Checker in Python

```python
# Save as parity_complete.py — run: python3 parity_complete.py

def generate_parity(data_bits: list, mode: str = 'even') -> int:
    """Sender: compute parity bit for given data"""
    ones = data_bits.count(1)
    if mode == 'even':
        return 0 if ones % 2 == 0 else 1
    else:  # odd parity
        return 1 if ones % 2 == 0 else 0

def check_parity(code_word: list, mode: str = 'even') -> bool:
    """Receiver: verify parity of received code word"""
    ones = code_word.count(1)
    if mode == 'even':
        return ones % 2 == 0
    else:
        return ones % 2 != 0

def simulate_transmission(data: list, flip_positions: list = [], mode: str = 'even'):
    """Simulate send → (possible noise) → receive"""
    parity_bit = generate_parity(data, mode)
    code_word = data + [parity_bit]

    # Simulate noise (flip specified bits)
    received = code_word.copy()
    for pos in flip_positions:
        received[pos] = 1 - received[pos]

    error_detected = not check_parity(received, mode)
    actual_error = (received != code_word)

    print(f"  Data:          {data}")
    print(f"  Parity bit:    {parity_bit} ({mode} parity)")
    print(f"  Code word:     {code_word}")
    print(f"  Bits flipped:  {flip_positions if flip_positions else 'none'}")
    print(f"  Received:      {received}")
    print(f"  Actual error:  {'YES' if actual_error else 'NO'}")
    print(f"  Error detected:{'YES ✓' if error_detected else 'NO ✗' + (' ← MISSED!' if actual_error else '')}")
    print()

# Test scenarios
data = [1, 1, 1, 0]
print("=== EVEN PARITY TESTS ===\n")
print("Scenario 1: No error")
simulate_transmission(data, [])

print("Scenario 2: Single bit error (position 0)")
simulate_transmission(data, [0])

print("Scenario 3: Double bit error (positions 0,1) — parity FAILS to detect!")
simulate_transmission(data, [0, 1])

print("Scenario 4: Triple bit error (positions 0,1,2)")
simulate_transmission(data, [0, 1, 2])

print("Scenario 5: 4-bit error — parity FAILS again!")
simulate_transmission(data, [0, 1, 2, 3])
```

### Lab 2 — Hamming Distance Calculator

```python
# Save as hamming_distance.py — run: python3 hamming_distance.py

def hamming_distance(word1: str, word2: str) -> int:
    """Calculate Hamming Distance between two binary strings"""
    if len(word1) != len(word2):
        raise ValueError("Code words must be same length")
    xor = int(word1, 2) ^ int(word2, 2)
    return bin(xor).count('1')

def min_hamming_distance(code_words: list) -> tuple:
    """Find minimum Hamming Distance in a set of code words"""
    min_dist = float('inf')
    min_pair = None

    for i in range(len(code_words)):
        for j in range(i + 1, len(code_words)):
            d = hamming_distance(code_words[i], code_words[j])
            if d < min_dist:
                min_dist = d
                min_pair = (code_words[i], code_words[j])

    return min_dist, min_pair

def detection_capability(d_min: int) -> int:
    return d_min - 1

def correction_capability(d_min: int) -> int:
    return (d_min - 1) // 2

# --- Tests ---

# Example 1: Manual calculation
print("=== HAMMING DISTANCE EXAMPLES ===\n")
pairs = [
    ("0000", "1111"),
    ("0101", "1000"),
    ("11001", "10001"),
]
for a, b in pairs:
    d = hamming_distance(a, b)
    xor = bin(int(a,2) ^ int(b,2))[2:].zfill(len(a))
    print(f"  {a} XOR {b} = {xor} → Hamming Distance = {d}")

# Example 2: Even parity code words (4-bit data)
print("\n=== EVEN PARITY CODE ANALYSIS ===\n")
even_parity_codes = [
    "00000", "00011", "00101", "00110",
    "01001", "01010", "01100", "01111",
    "10001", "10010", "10100", "10111",
    "11000", "11011", "11101", "11110"
]

d_min, pair = min_hamming_distance(even_parity_codes)
print(f"  Total valid code words:  {len(even_parity_codes)}")
print(f"  Minimum Hamming Distance: {d_min}")
print(f"  Closest pair: {pair[0]} and {pair[1]}")
print(f"  Can DETECT up to:  {detection_capability(d_min)} bit error(s)")
print(f"  Can CORRECT up to: {correction_capability(d_min)} bit error(s)")

# Example 3: Custom code words
print("\n=== CUSTOM CODE WORD SET ===\n")
custom = ["000", "011", "101", "110"]
d_min, pair = min_hamming_distance(custom)
print(f"  Code words: {custom}")
print(f"  d_min = {d_min} (pair: {pair})")
print(f"  Detects:  {detection_capability(d_min)} bit error(s)")
print(f"  Corrects: {correction_capability(d_min)} bit error(s)")
```

### Lab 3 — Rowhammer Concept (Parity/ECC Relevance)

```bash
# Understand Rowhammer — why ECC RAM matters for security

# Check if your system has ECC RAM
sudo dmidecode --type 17 | grep -i "error correction"
# Or:
sudo dmidecode --type 17 | grep -i ECC

# Also check:
sudo dmidecode | grep -i "error correcting"

# If output shows "None" → no ECC → vulnerable to Rowhammer in theory
# If output shows "Single-bit ECC" → single bit errors auto-corrected

# Check memory errors (if ECC is present, it logs corrections)
sudo mcelog --client 2>/dev/null || echo "mcelog not available"

# Alternative: edac-utils for ECC error reporting
sudo apt install edac-utils 2>/dev/null
edac-util -s 0
```

### Lab 4 — Hamming Weight in Side-Channel Context

```python
# Save as hamming_weight_demo.py
# Demonstrates why Hamming Weight matters in side-channel attacks

def hamming_weight(value: int) -> int:
    """Count 1s in binary representation (power consumption proxy)"""
    return bin(value).count('1')

# AES S-box output values (simplified example)
# In real AES, power consumption correlates with Hamming Weight of data
print("=== HAMMING WEIGHT & POWER ANALYSIS CONCEPT ===\n")
print("Byte value | Binary       | Hamming Weight | Simulated Power")
print("-" * 65)

import random
test_values = [0x00, 0x01, 0x0F, 0x55, 0xAA, 0xFF, 0x42, 0x7E]
for val in test_values:
    hw = hamming_weight(val)
    power = 50 + hw * 10 + random.randint(-5, 5)  # simulated power (mW)
    print(f"  0x{val:02X}       | {val:08b}     | {hw}              | ~{power} mW")

print("\nAttacker observation:")
print("  Higher Hamming Weight → Higher power consumption")
print("  By measuring power and knowing correlation → can infer secret key bits")
print("  This is Simple Power Analysis (SPA) / Differential Power Analysis (DPA)")
```

### Lab 5 — See Parity in Network Packets

```bash
# TCP/UDP use checksum (not parity), but let's observe error detection in action

# Generate traffic and see checksum verification in Wireshark
sudo wireshark &
# Filter: tcp.checksum_bad == 1 OR udp.checksum_bad == 1

# Intentionally create bad checksum with Scapy
sudo python3 - << 'EOF'
from scapy.all import *

# Craft UDP packet with bad checksum
pkt = IP(dst="192.168.56.101") / UDP(sport=1234, dport=9999, chksum=0xBAD0) / "Hello"
send(pkt)
print("Sent UDP packet with bad checksum 0xBAD0")
print("Check Wireshark → it should flag 'Bad checksum'")
print("This is the Transport Layer equivalent of parity check failing")
EOF
```

---

## 10. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║          PARITY & HAMMING DISTANCE — EXAM CHEAT SHEET               ║
╠══════════════════════════════════════════════════════════════════════╣
║  SINGLE PARITY BIT                                                   ║
║  m data bits + 1 parity bit = m+1 transmitted                      ║
║  Least expensive error detection method                             ║
║  Even parity → total 1s must be EVEN                               ║
║  Odd parity  → total 1s must be ODD                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  SENDER: Add parity bit to make total 1s even/odd                  ║
║  RECEIVER: Count 1s → check parity rule                            ║
║    Pass → accept  |  Fail → error detected → retransmit            ║
╠══════════════════════════════════════════════════════════════════════╣
║  DETECTION CAPABILITY                                                ║
║  Detects:     odd number of bit flips (1, 3, 5, 7...)              ║
║  Misses:      even number of bit flips (2, 4, 6...)                ║
║  Cannot correct — only detect                                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  HAMMING DISTANCE                                                    ║
║  HD(A, B) = XOR(A, B) → count 1s in result                        ║
║  Measures number of positions where two code words differ           ║
╠══════════════════════════════════════════════════════════════════════╣
║  MINIMUM HAMMING DISTANCE (d_min)                                   ║
║  = smallest HD between ANY two valid code words in the set         ║
║  For single parity (even/odd) → d_min = 2                         ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY FORMULAS                                                        ║
║  Error DETECTION capability  = d_min − 1                           ║
║  Error CORRECTION capability = ⌊(d_min − 1) / 2⌋                  ║
║                                                                      ║
║  Single parity: d_min=2 → detects 1 → corrects 0                  ║
║  d_min=3 → detects 2 → corrects 1 (Hamming Code)                  ║
║  d_min=4 → detects 3 → corrects 1                                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  VALID CODE WORDS                                                    ║
║  4-bit data + 1 even parity bit → 16 valid out of 32 total        ║
║  Any flip of 1 bit → invalid code word → detected                 ║
║  Flip of 2 bits → may reach another VALID code word → MISSED      ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  Parity ≠ Security (attacker flips even bits → bypasses parity)    ║
║  ECC RAM uses Hamming code → auto-corrects single bit flips        ║
║  Non-ECC RAM → Rowhammer attack can flip bits → privilege escalate ║
║  Hamming Weight used in power side-channel attacks (SPA/DPA)       ║
║  Hamming Distance used in fuzzy hashing for malware detection      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Two-Dimensional Parity Check
- [ ] Checksum — calculation and verification
- [ ] CRC — Cyclic Redundancy Check (polynomial division)
- [ ] Hamming Code — error correction, bit positioning formula

---

_Notes compiled from: Networking Course Lecture 08 — Single Parity Bit & Hamming Distance_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
