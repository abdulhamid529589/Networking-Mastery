# 🧩 Hamming Code — Error Detection & Correction

### Cybersecurity Student Notes | Networking Course — Lecture 30

> **Source:** Gate Smashers — Hamming Code (Error Detection & Correction)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is Hamming Code?](#1-what-is-hamming-code)
2. [Why Hamming Code is Powerful](#2-why-hamming-code-is-powerful)
3. [Key Terminology](#3-key-terminology)
4. [Step-by-Step Hamming Code Calculation](#4-step-by-step-hamming-code-calculation)
   - [4.1 Determine Total Bits & Parity Positions](#41-determine-total-bits--parity-positions)
   - [4.2 Place the Data Bits](#42-place-the-data-bits)
   - [4.3 Calculate Each Parity Bit (XOR)](#43-calculate-each-parity-bit-xor)
   - [4.4 Assemble the Final Code Word](#44-assemble-the-final-code-word)
5. [Receiver Side — Detection & Correction](#5-receiver-side--detection--correction)
6. [Efficiency Formula](#6-efficiency-formula)
7. [Quick Reference Rules](#7-quick-reference-rules)
8. [More Worked Examples](#8-more-worked-examples)
9. [Common Uses of Hamming-Family Codes](#9-common-uses-of-hamming-family-codes)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. What is Hamming Code?

**Hamming Code** is an error-control technique that can both **detect** and **correct** bit errors, by inserting multiple **parity (redundant) bits** at carefully chosen positions inside the data.

```
Sender:   Data bits → insert parity bits at positions 1,2,4,8,16,...
                     → calculate each parity bit via XOR of its group
          Transmit: [Full Hamming code word]

Receiver: Receives code word → recalculates every parity group
          If all groups match     → NO ERROR
          If some groups mismatch → combine mismatches (binary) = exact
                                     position of the corrupted bit
                                   → FLIP that bit → ERROR CORRECTED
```

### Where is Hamming Code Used?

| Application                                 | Notes                                                         |
| ------------------------------------------- | ------------------------------------------------------------- |
| **ECC RAM**                                 | Detects & corrects single-bit memory errors                   |
| **Satellite / deep-space comms**            | Retransmission is costly/impossible — correction is essential |
| **Embedded systems / flash storage**        | SSD controllers use Hamming-family SECDED codes               |
| **Older modem/telecom links**               | Basic FEC (Forward Error Correction)                          |
| **QR codes (Reed–Solomon, related family)** | Conceptually related error-correcting idea                    |

> Unlike a simple parity bit (detects only), and unlike CRC (detects only, doesn't locate), **Hamming Code can locate and repair a single-bit error without retransmission.**

---

## 2. Why Hamming Code is Powerful

| Error Type                     | Detected?                                      | Corrected?                    |
| ------------------------------ | ---------------------------------------------- | ----------------------------- |
| **Single-bit error**           | ✅ Always                                      | ✅ Always (basic Hamming)     |
| **Double-bit error**           | ✅ With extra parity (SECDED)                  | ❌ Cannot correct, only flags |
| **Burst errors**               | ❌ Not reliably                                | ❌ Not designed for this      |
| **Simple parity (comparison)** | Detects odd-count errors only, no correction   | —                             |
| **CRC (comparison)**           | Excellent burst detection, but zero correction | —                             |

> **Rule of thumb:** CRC is for _detecting_ errors over a communication line (and asking for retransmission). Hamming Code is for situations where you can't easily ask for a retransmission (e.g., RAM, deep-space links) and need to _fix_ the error immediately.

---

## 3. Key Terminology

| Term                    | Meaning                                                                                   |
| ----------------------- | ----------------------------------------------------------------------------------------- |
| **Data bits (d)**       | Original message bits                                                                     |
| **Parity bits (p / r)** | Redundant bits placed at power-of-2 positions, used for check groups                      |
| **Code word**           | Data bits + parity bits combined (what is actually transmitted)                           |
| **m**                   | Number of data bits                                                                       |
| **r**                   | Number of parity bits                                                                     |
| **m + r**               | Total bits transmitted                                                                    |
| **Even / Odd parity**   | Convention deciding whether each group's total 1-count should be even or odd              |
| **Syndrome**            | The binary number formed by combining all parity check results, giving the error position |

---

## 4. Step-by-Step Hamming Code Calculation

### Worked Example from Lecture

```
Data (m):        1010          (4 bits)
Total code:      7 bits        (Hamming(7,4))
Parity convention: Even parity
```

---

### 4.1 Determine Total Bits & Parity Positions

**Rule:** Number bit positions starting from **1**. Any position that is a **power of 2** (`2^n` for n = 0, 1, 2, 3, …) is a **parity bit**. Every other position is a **data bit**.

```
2^0 = 1   → Position 1  = Parity (P0)
2^1 = 2   → Position 2  = Parity (P1)
2^2 = 4   → Position 4  = Parity (P2)
2^3 = 8   → Position 8  = Parity (P3), if it exists in the code length
2^4 = 16  → Position 16 = Parity (P4), if it exists
...
```

**Formula for minimum parity bits needed:**

```
2^r ≥ m + r + 1

For m = 4 data bits:
  r = 3 → 2^3 = 8 ≥ 4 + 3 + 1 = 8   ✓ (satisfied exactly)
  So r = 3 parity bits → total code length = 4 + 3 = 7 bits
```

#### Position map for a 7-bit code:

| Position | 1   | 2   | 3   | 4   | 5   | 6   | 7   |
| -------- | --- | --- | --- | --- | --- | --- | --- |
| Type     | P0  | P1  | D3  | P2  | D2  | D1  | D0  |

#### Position map for an 11-bit code (7 data bits):

| Position | 1   | 2   | 3   | 4   | 5   | 6   | 7   | 8   | 9   | 10  | 11  |
| -------- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Type     | P0  | P1  | D   | P2  | D   | D   | D   | P3  | D   | D   | D   |

> Position 16 (2⁴) doesn't exist in an 11-bit code, so only 4 parity bits (P0, P1, P2, P3) are needed.

---

### 4.2 Place the Data Bits

Data bits are written **MSB → LSB** into all the **non-power-of-2** positions.

```
Data:  1 0 1 0
       ↓ ↓ ↓ ↓
      D3 D2 D1 D0
       1  0  1  0
```

Placed into the 7-bit frame:

| Position | 1 (P0) | 2 (P1) | 3 (D3) | 4 (P2) | 5 (D2) | 6 (D1) | 7 (D0) |
| -------- | ------ | ------ | ------ | ------ | ------ | ------ | ------ |
| Value    | ?      | ?      | 1      | ?      | 0      | 1      | 0      |

---

### 4.3 Calculate Each Parity Bit (XOR)

Each parity bit checks a specific subset of positions using a **"pick N, skip N"** pattern based on its own position's power of 2.

| Parity Bit | Position | Pick / Skip Pattern                            |
| ---------- | -------- | ---------------------------------------------- |
| **P0**     | 1        | Pick 1, skip 1 → 1, 3, 5, 7, 9, 11, …          |
| **P1**     | 2        | Pick 2, skip 2 → 2, 3, 6, 7, 10, 11, …         |
| **P2**     | 4        | Pick 4, skip 4 → 4, 5, 6, 7, 12, 13, 14, 15, … |
| **P3**     | 8        | Pick 8, skip 8 → 8–15, 24–31, …                |

**Calculating P0** (covers positions 1, 3, 5, 7 → i.e. D3, D1, D0, excluding itself):

```
D3 = 1, D1 = 1, D0 = 0
XOR:  1 ⊕ 1 ⊕ 0 = 0
P0 = 0
```

**Calculating P1** (covers positions 2, 3, 6, 7 → i.e. D3, D2, D0):

```
D3 = 1, D2 = 0, D0 = 0
XOR:  1 ⊕ 0 ⊕ 0 = 1
P1 = 1
```

**Calculating P2** (covers positions 4, 5, 6, 7 → i.e. D2, D1, D0):

```
D2 = 0, D1 = 1, D0 = 0
XOR:  0 ⊕ 1 ⊕ 0 = 1
P2 = 1
```

---

### 4.4 Assemble the Final Code Word

```
Position:  1    2    3    4    5    6    7
Bit:       P0   P1   D3   P2   D2   D1   D0
Value:     0    1    1    1    0    1    0
```

**Final transmitted code word: `0111010`**

This is what gets sent across the network / stored in memory.

---

## 5. Receiver Side — Detection & Correction

### Worked Example from Lecture (11-bit code)

```
Data word:  1 0 0 1 1 0 1 1   (8 bits)
Code type:  Hamming(11,7) — 4 parity bits: r1 (P0), r2 (P1), r4 (P2), r8 (P3)
```

### 5.1 Sender Side — Calculate All Parity Bits

| Parity | Position | Covers             |
| ------ | -------- | ------------------ |
| r1     | 1        | 1, 3, 5, 7, 9, 11  |
| r2     | 2        | 2, 3, 6, 7, 10, 11 |
| r4     | 4        | 4, 5, 6, 7         |
| r8     | 8        | 8, 9, 10, 11       |

Each parity bit = XOR of all data bits in its group. These are inserted, producing the final 11-bit code word, which is transmitted.

### 5.2 Simulated Transmission Error

Bit position **9** is flipped (0 → 1) during transmission — simulating channel noise/corruption.

### 5.3 Receiver Detects the Error

The receiver recalculates each parity group (including the parity bit itself this time) and checks whether it satisfies the agreed parity convention (here, even parity).

**Recalculating r1** (positions 1, 3, 5, 7, 9, 11):

```
Bits: 1, 1, 0, 1, 1(flipped), 1
Count of 1s = 5 → ODD → violates even-parity rule
→ MISMATCH → error confirmed somewhere in r1's group
```

### 5.4 Locate the Exact Error Position

Check every parity group independently:

| Parity Check | Result       | Reason                          |
| ------------ | ------------ | ------------------------------- |
| r1           | **1 (fail)** | Position 9 is in this group     |
| r2           | **0 (pass)** | Position 9 is NOT in this group |
| r4           | **0 (pass)** | Position 9 is NOT in this group |
| r8           | **1 (fail)** | Position 9 is in this group     |

Combine results in order **r8 r4 r2 r1**:

```
r8 r4 r2 r1
 1  0  0  1   = binary 1001 = decimal 9

→ Error is exactly at bit position 9
```

> This is the elegant core of Hamming Code: the pattern of passing/failing parity checks _is itself_ the binary address of the corrupted bit. No bit-by-bit search needed.

### 5.5 Correct the Error

```
Bit at position 9 received as: 1
Flip it: 1 → 0
Original correct data restored — no retransmission required.
```

---

## 6. Efficiency Formula

```
Efficiency = (m / (m + r)) × 100%

Where:
  m = number of data bits
  r = number of parity bits

Example from lecture:
  m = 4, r = 3  → Efficiency = (4/7) × 100 = 57.1%   (Hamming(7,4))
  m = 7, r = 4  → Efficiency = (7/11) × 100 = 63.6%  (Hamming(11,7))
```

| m   | r   | Efficiency |
| --- | --- | ---------- |
| 4   | 3   | 57.1%      |
| 7   | 4   | 63.6%      |
| 11  | 4   | 73.3%      |
| 26  | 5   | 83.9%      |

> As message length grows, fewer parity bits are needed relative to data (parity grows logarithmically) → efficiency improves with larger m.

---

## 7. Quick Reference Rules

```
┌─────────────────────────────────────────────────────────────────┐
│  RULE 1: Parity bit positions                                   │
│    Always at powers of 2: 1, 2, 4, 8, 16, 32, ...               │
│    All other positions → data bits                              │
│                                                                   │
│  RULE 2: Minimum parity bits needed                              │
│    2^r ≥ m + r + 1   (solve for smallest r)                      │
│                                                                   │
│  RULE 3: Which bits each parity checks                           │
│    Parity at position 2^n → pick 2^n bits, skip 2^n bits, repeat│
│                                                                   │
│  RULE 4: Calculating a parity bit                                │
│    XOR together all bits (data + other, per convention) in      │
│    its group                                                     │
│                                                                   │
│  RULE 5: Receiver check                                          │
│    Recalculate every parity group                                │
│    All match  → no error                                         │
│    Mismatches → combine as binary (highest position first)      │
│                 → gives exact error bit position → flip it       │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. More Worked Examples

### Example 2 — Encoding `1101` into Hamming(7,4)

```
Data: 1 1 0 1  →  D3=1, D2=1, D1=0, D0=1

Position map:  1(P0) 2(P1) 3(D3) 4(P2) 5(D2) 6(D1) 7(D0)
Data placed:    ?     ?     1     ?     1     0     1

P0 (positions 1,3,5,7 → D3,D1,D0): 1 ⊕ 0 ⊕ 1 = 0
P1 (positions 2,3,6,7 → D3,D2,D0): 1 ⊕ 1 ⊕ 1 = 1
P2 (positions 4,5,6,7 → D2,D1,D0): 1 ⊕ 0 ⊕ 1 = 0

Final code word:  0 1 1 0 1 0 1
```

### Example 3 — Detecting a Different Single-Bit Error

```
Take the code word from Example 2: 0 1 1 0 1 0 1
Flip bit at position 6 (D1): 0 → becomes 0 1 1 0 1 1 1  (bit 6 now 1)

Receiver recalculates:
  r1 (1,3,5,7): 0⊕1⊕1⊕1 = 1  → mismatch (should be 0 with parity bit included → odd count)
  r2 (2,3,6,7): 1⊕1⊕1⊕1 = 0  → mismatch
  r4 (4,5,6,7): 0⊕1⊕1⊕1 = 1  → mismatch

Combine (r4 r2 r1): 1 1 1 = binary 110 → decimal 6
→ Error located at position 6 → flip it back → corrected
```

---

## 9. Common Uses of Hamming-Family Codes

| Standard / System                | Code Family                         | Purpose                                                      |
| -------------------------------- | ----------------------------------- | ------------------------------------------------------------ |
| **ECC DIMM (server RAM)**        | Hamming SECDED (72,64)              | Single-error correction, double-error detection              |
| **NAND flash / SSD controllers** | BCH (Hamming-related)               | Multi-bit correction for flash wear errors                   |
| **Deep space / satellite links** | Reed–Muller, Hamming, convolutional | Forward error correction where retransmission is impractical |
| **Older telecom / modem FEC**    | Hamming, BCH                        | Basic forward error correction                               |
| **QR Codes**                     | Reed–Solomon (related family)       | Correction of smudged/damaged codes                          |

---

## 10. 🔴 Security Context

### Hamming Code is NOT a Security Mechanism

```
Hamming Code protects against ACCIDENTAL bit errors (noise, hardware faults).
Hamming Code does NOT protect against INTENTIONAL tampering.

Why?
  Hamming Code is a linear code, just like CRC.
  An attacker who can modify data in transit or in memory can also
  recompute valid parity bits for the modified data.
  There is no secret key involved anywhere in the scheme —
  the position/grouping rules are entirely public and deterministic.
```

### Conceptual Forgery Example

```python
# Conceptual demonstration (attacker's perspective) — NOT a real attack tool,
# just illustrating why error-correcting codes ≠ integrity/security

original_data = "1001"
codeword = hamming_encode(original_data)     # sender computes parity normally

# Attacker with write-access to memory/transit intercepts and changes data:
modified_data = "1101"
forged_codeword = hamming_encode(modified_data)  # attacker recomputes parity too

# Receiver's parity check on forged_codeword passes completely —
# because the parity bits were correctly recalculated for the new data.
# Hamming Code gives ZERO protection against a deliberate, informed attacker.
```

### What Actually Provides Data Integrity/Security

| Mechanism             | Purpose                           | Secure Against a Deliberate Attacker? |
| --------------------- | --------------------------------- | ------------------------------------- |
| Hamming Code          | Random single-bit hardware errors | ❌ No                                 |
| CRC-32                | Random transmission errors        | ❌ No                                 |
| MD5 (hash)            | Historic integrity check          | ❌ Broken (collision attacks)         |
| SHA-256 (hash alone)  | Integrity only, no key            | ⚠️ No authentication without a key    |
| **HMAC-SHA256**       | Authenticated integrity           | ✅ Yes (needs a shared secret key)    |
| **Digital Signature** | Integrity + non-repudiation       | ✅ Yes (needs a private key)          |
| **AES-GCM**           | Confidentiality + integrity       | ✅ Yes                                |

> **Rule:** Hamming Code / CRC = for correcting and detecting _random noise_. HMAC / signatures / AEAD ciphers = for defending against a _deliberate adversary_. Never rely on an error-correcting code as your only integrity check on a security-sensitive channel.

### Why This Matters for a Security Engineer

- **RAM-based fault attacks (Rowhammer-adjacent research):** ECC memory using Hamming-family SECDED codes can correct random single-bit flips, but researchers have shown that _known, structured_ multi-bit flips can sometimes bypass single-error-correcting schemes — which is why security-critical systems layer on additional cryptographic integrity checks rather than relying on ECC RAM alone.
- **Storage systems:** SSD/flash controllers use Hamming-family ECC to keep data readable as cells wear out. This is a **reliability** feature, not a way to detect that a file was maliciously altered — that job belongs to hashes/signatures.
- **Takeaway for exams and practice:** If a question asks "does Hamming Code prevent tampering by an attacker," the correct answer is **no** — it is a reliability mechanism, not an authentication mechanism.

---

## 11. 🧪 Practical Labs

### Lab 1 — Implement Hamming(7,4) Encode/Decode from Scratch in Python

```python
# Save as hamming74_full.py — run: python3 hamming74_full.py

def encode_hamming74(data: str) -> str:
    """Encode 4 data bits into a 7-bit Hamming code word (even parity)."""
    assert len(data) == 4, "Data must be 4 bits"
    d3, d2, d1, d0 = data[0], data[1], data[2], data[3]

    # Positions: 1=P0 2=P1 3=D3 4=P2 5=D2 6=D1 7=D0
    p0 = str(int(d3) ^ int(d1) ^ int(d0))   # covers 1,3,5,7
    p1 = str(int(d3) ^ int(d2) ^ int(d0))   # covers 2,3,6,7
    p2 = str(int(d2) ^ int(d1) ^ int(d0))   # covers 4,5,6,7

    code = p0 + p1 + d3 + p2 + d2 + d1 + d0
    return code

def decode_hamming74(code: str) -> dict:
    """Detect and correct a single-bit error in a 7-bit Hamming code word."""
    bits = ['_'] + list(code)  # 1-indexed for clarity (bits[1]..bits[7])

    c1 = int(bits[1]) ^ int(bits[3]) ^ int(bits[5]) ^ int(bits[7])  # r1 group incl. parity
    c2 = int(bits[2]) ^ int(bits[3]) ^ int(bits[6]) ^ int(bits[7])  # r2 group incl. parity
    c4 = int(bits[4]) ^ int(bits[5]) ^ int(bits[6]) ^ int(bits[7])  # r4 group incl. parity

    error_pos = c4 * 4 + c2 * 2 + c1 * 1   # syndrome → decimal position

    corrected = bits[:]
    if error_pos != 0:
        corrected[error_pos] = '1' if corrected[error_pos] == '0' else '0'

    corrected_code = ''.join(corrected[1:])
    data_bits = corrected_code[2] + corrected_code[4] + corrected_code[5] + corrected_code[6]

    return {
        'received': code,
        'error_position': error_pos,
        'corrected_code': corrected_code,
        'recovered_data': data_bits
    }

# ─── Test: Lecture example ───
print("=" * 55)
print("ENCODE EXAMPLE")
print("=" * 55)
data = "1010"
code = encode_hamming74(data)
print(f"Data:        {data}")
print(f"Code word:   {code}")

print()
print("=" * 55)
print("DECODE — NO ERROR")
print("=" * 55)
result = decode_hamming74(code)
print(result)

print()
print("=" * 55)
print("DECODE — SINGLE BIT ERROR INJECTED")
print("=" * 55)
corrupted = list(code)
corrupted[4] = '1' if corrupted[4] == '0' else '0'   # flip bit at position 5
corrupted = ''.join(corrupted)
print(f"Corrupted code word: {corrupted}")
result = decode_hamming74(corrupted)
print(result)
assert result['recovered_data'] == data, "Correction failed!"
print("Correction verified — recovered original data ✓")
```

### Lab 2 — Hamming(11,7) Encode/Decode (Matches Lecture's Second Example)

```python
# Save as hamming11_7.py — run: python3 hamming11_7.py

def positions_for_parity(p_pos: int, total_len: int):
    """Return list of positions (1-indexed) covered by a parity bit at p_pos."""
    positions = []
    for pos in range(1, total_len + 1):
        if pos & p_pos:
            positions.append(pos)
    return positions

def encode_hamming(data: str) -> str:
    m = len(data)
    r = 0
    while (2 ** r) < (m + r + 1):
        r += 1
    total_len = m + r

    code = ['0'] * (total_len + 1)  # 1-indexed
    data_iter = iter(data)
    parity_positions = [2 ** i for i in range(r)]

    for pos in range(1, total_len + 1):
        if pos not in parity_positions:
            code[pos] = next(data_iter)

    for p in parity_positions:
        covered = positions_for_parity(p, total_len)
        value = 0
        for pos in covered:
            if pos != p:
                value ^= int(code[pos])
        code[p] = str(value)

    return ''.join(code[1:])

def decode_hamming(code: str) -> dict:
    total_len = len(code)
    bits = ['0'] + list(code)  # 1-indexed
    r = 0
    while (2 ** r) <= total_len:
        r += 1

    syndrome = 0
    for i in range(r):
        p = 2 ** i
        if p > total_len:
            continue
        covered = positions_for_parity(p, total_len)
        value = 0
        for pos in covered:
            value ^= int(bits[pos])
        if value != 0:
            syndrome += p

    corrected = bits[:]
    if syndrome != 0:
        corrected[syndrome] = '1' if corrected[syndrome] == '0' else '0'

    parity_positions = {2 ** i for i in range(r)}
    data_bits = ''.join(
        corrected[pos] for pos in range(1, total_len + 1) if pos not in parity_positions
    )

    return {
        'received': code,
        'error_position': syndrome,
        'corrected_code': ''.join(corrected[1:]),
        'recovered_data': data_bits
    }

# ─── Test: Lecture's 8-bit data example ───
print("=" * 55)
print("ENCODE — 8-bit data word")
print("=" * 55)
data = "10011011"
code = encode_hamming(data)
print(f"Data:      {data}")
print(f"Code word: {code}  (length {len(code)})")

print()
print("=" * 55)
print("SIMULATE ERROR AT POSITION 9 (as in the lecture)")
print("=" * 55)
corrupted = list(code)
corrupted[8] = '1' if corrupted[8] == '0' else '0'   # index 8 = position 9 (0-indexed list)
corrupted = ''.join(corrupted)
print(f"Corrupted: {corrupted}")

result = decode_hamming(corrupted)
print(f"Detected error at position: {result['error_position']}")
print(f"Corrected code:  {result['corrected_code']}")
print(f"Recovered data:  {result['recovered_data']}")
assert result['recovered_data'] == data, "Correction failed!"
print("Correction verified — matches lecture's worked example ✓")
```

### Lab 3 — Compare Hamming Code vs. Simple Parity vs. CRC (Detection Power Demo)

```python
# Save as compare_error_control.py — run: python3 compare_error_control.py
# Demonstrates WHY Hamming Code is preferred when correction (not just
# detection) is required — useful for building intuition before exams.

import random

def simple_parity_check(data: str) -> str:
    return data + str(data.count('1') % 2)  # even parity bit

def flip_random_bit(bits: str) -> str:
    bits = list(bits)
    i = random.randint(0, len(bits) - 1)
    bits[i] = '1' if bits[i] == '0' else '0'
    return ''.join(bits), i

data = "1010"

print("SIMPLE PARITY — can only tell you AN error occurred, not where:")
encoded = simple_parity_check(data)
corrupted, idx = flip_random_bit(encoded)
detected = corrupted.count('1') % 2 != 0
print(f"  Encoded: {encoded} | Corrupted (bit {idx} flipped): {corrupted}")
print(f"  Detected error: {detected} | Can locate exact bit? NO\n")

print("HAMMING CODE — tells you an error occurred AND exactly where:")
print("  (see Lab 1/2 above — syndrome directly gives the bit position)\n")

print("CRC — excellent at detecting (esp. burst errors) but cannot correct")
print("  or locate a single bit; it can only say 'resend the whole message'.")
```

### Lab 4 — Simulate ECC RAM Behavior (Conceptual, for Learning Only)

```bash
# This is a conceptual/educational exercise, not an actual RAM fault injector.
# Real ECC RAM testing requires specialized hardware tools (e.g., memtest86+
# with ECC reporting) which is outside the scope of a software-only lab.

python3 -c "
from hamming11_7 import encode_hamming, decode_hamming

# Simulate a 'memory word' being stored and later read back with a
# random single-bit flip, similar to how ECC DIMMs behave in real servers.
import random

memory_word = '1100101'
stored = encode_hamming(memory_word)
print(f'Original memory word: {memory_word}')
print(f'Stored (with ECC):    {stored}')

# Simulate a cosmic-ray / voltage-noise style single-bit flip
bits = list(stored)
flip_at = random.randint(0, len(bits) - 1)
bits[flip_at] = '1' if bits[flip_at] == '0' else '0'
read_back = ''.join(bits)
print(f'Read back (bit {flip_at} flipped): {read_back}')

result = decode_hamming(read_back)
print(f'ECC detected/corrected at position: {result[\"error_position\"]}')
print(f'Recovered word: {result[\"recovered_data\"]}')
print(f'Match original: {result[\"recovered_data\"] == memory_word}')
"
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              HAMMING CODE — EXAM CHEAT SHEET                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHAT IS HAMMING CODE?                                               ║
║  Error detection AND correction via redundant parity bits at        ║
║  power-of-2 positions                                                ║
║  Used in: ECC RAM, SSD/flash controllers, satellite/deep-space FEC  ║
╠══════════════════════════════════════════════════════════════════════╣
║  HAMMING CODE CAN                                                     ║
║  ✓ Detect a single-bit error                                        ║
║  ✓ Correct a single-bit error (locate exact position, then flip)    ║
║  ✓ Detect (not correct) a double-bit error, with an extra parity   ║
║    bit (SECDED variant)                                              ║
║  ✗ Not designed for burst errors (use CRC for that instead)         ║
╠══════════════════════════════════════════════════════════════════════╣
║  ENCODING STEPS                                                       ║
║  1. Find minimum r such that 2^r ≥ m + r + 1                        ║
║  2. Number positions from 1; positions that are powers of 2 are     ║
║     parity bits, all others are data bits                           ║
║  3. Place data bits (MSB→LSB) into the non-parity positions         ║
║  4. For each parity bit at position 2^n: XOR together all bits in   ║
║     its "pick 2^n, skip 2^n" group                                   ║
║  5. Insert the calculated parity values → final code word            ║
╠══════════════════════════════════════════════════════════════════════╣
║  DECODING / CORRECTION STEPS                                         ║
║  1. Recalculate each parity group (including the parity bit itself) ║
║  2. Record 0 (match) or 1 (mismatch) for every group                 ║
║  3. Combine mismatches in order (highest position first, e.g.       ║
║     r8 r4 r2 r1) as a binary number                                  ║
║  4. That number = exact position of the flipped bit                  ║
║  5. Flip the bit at that position → error corrected                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  EFFICIENCY FORMULA                                                   ║
║  Efficiency = m / (m + r) × 100%                                     ║
║  m = data bits, r = parity bits                                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  COMMON CODE SIZES                                                    ║
║  Hamming(7,4)  → 4 data bits, 3 parity bits, 7 total                ║
║  Hamming(11,7) → 7 data bits, 4 parity bits, 11 total               ║
║  Rule: parity bits sit at 1, 2, 4, 8, 16, ... (powers of 2)         ║
╠══════════════════════════════════════════════════════════════════════╣
║  HAMMING CODE vs CRC vs SIMPLE PARITY                                 ║
║  Simple parity → detect only, cannot locate                          ║
║  CRC            → excellent burst-error detection, cannot correct    ║
║  Hamming Code    → best for single-bit correction w/o retransmit     ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                       ║
║  Hamming Code ≠ Security (attacker can recompute valid parity        ║
║  bits after tampering with data — it's a linear, keyless scheme)     ║
║  Hamming Code protects against RANDOM hardware/noise errors only     ║
║  For tamper resistance: use HMAC-SHA256, digital signatures,         ║
║  or AES-GCM instead                                                   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Checksum — TCP/UDP/IP header checksum
- [ ] Flow Control — Stop-and-Wait, Sliding Window
- [ ] MAC protocols — CSMA/CD, CSMA/CA
- [ ] Network Layer — IP Addressing, Subnetting
- [ ] Error Control vs Flow Control — comparison for exams

---

_Notes compiled from: Networking Course Lecture 30 — Hamming Code (Error Detection & Correction)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
