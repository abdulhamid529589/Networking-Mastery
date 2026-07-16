# 🔄 CRC — Cyclic Redundancy Check

### Cybersecurity Student Notes | Networking Course — Lecture 29

> **Source:** Gate Smashers — CRC (Cyclic Redundancy Check)
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is CRC?](#1-what-is-crc)
2. [Why CRC is Powerful](#2-why-crc-is-powerful)
3. [Key Terminology](#3-key-terminology)
4. [Step-by-Step CRC Calculation](#4-step-by-step-crc-calculation)
   - [4.1 Convert Polynomial → Binary](#41-convert-polynomial--binary)
   - [4.2 Append Zeros to Message](#42-append-zeros-to-message)
   - [4.3 Binary Division (XOR)](#43-binary-division-xor)
   - [4.4 Replace Zeros with Remainder](#44-replace-zeros-with-remainder)
5. [Receiver Side — Verification](#5-receiver-side--verification)
6. [Efficiency Formula](#6-efficiency-formula)
7. [Quick Reference Rules](#7-quick-reference-rules)
8. [More Worked Examples](#8-more-worked-examples)
9. [Common CRC Standards](#9-common-crc-standards)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. What is CRC?

**CRC (Cyclic Redundancy Check)** is an error detection method based on polynomial division.

```
Sender:   Message → divide by generator polynomial → get remainder (CRC bits)
          Transmit: [Message + CRC bits]

Receiver: Receives [Message + CRC bits] → divide by same generator
          If remainder = 0 → NO ERROR
          If remainder ≠ 0 → ERROR DETECTED
```

### Where is CRC Used?

| Application          | CRC Standard                 |
| -------------------- | ---------------------------- |
| **Ethernet**         | CRC-32 (FCS field in frames) |
| **Wi-Fi (802.11)**   | CRC-32                       |
| **USB**              | CRC-16                       |
| **ZIP / GZIP files** | CRC-32                       |
| **Hard drives**      | CRC-32                       |
| **Bluetooth**        | CRC-16                       |
| **HDLC / PPP**       | CRC-16                       |

> Most widely used error detection method in real systems.

---

## 2. Why CRC is Powerful

CRC can detect:

| Error Type                                        | Detected?                     |
| ------------------------------------------------- | ----------------------------- |
| All **single bit** errors                         | ✅ Always                     |
| All **double bit** errors                         | ✅ Always                     |
| All **odd number** of errors                      | ✅ Always                     |
| **Burst errors** of length ≤ degree of polynomial | ✅ Always                     |
| Burst errors of length = degree + 1               | ✅ With high probability      |
| Burst errors of length > degree + 1               | ✅ With probability (1 - 2⁻ʳ) |

> Compare to parity (detects only odd-count errors) — CRC is far superior.

---

## 3. Key Terminology

| Term                         | Meaning                                                         |
| ---------------------------- | --------------------------------------------------------------- |
| **Message (M)**              | Original data bits to be sent                                   |
| **Generator (G)**            | Divisor polynomial — agreed between sender & receiver           |
| **CRC bits / Remainder (R)** | Redundant bits computed from division                           |
| **Code word**                | Message + CRC bits (what is actually transmitted)               |
| **m**                        | Number of message bits                                          |
| **r**                        | Number of CRC (redundant) bits = degree of generator polynomial |
| **m + r**                    | Total bits transmitted                                          |

---

## 4. Step-by-Step CRC Calculation

### Worked Example from Lecture

```
Message (M):   1001 1100 10   (10 bits)
Generator (G): x⁴ + x³ + 1   (polynomial form)
```

---

### 4.1 Convert Polynomial → Binary

Map each term of the polynomial to a bit position:

```
Generator: x⁴ + x³ + x² + x¹ + x⁰
                ↓
Coefficients: 1  1  0  0  1
              x⁴ x³ x² x¹ x⁰

Generator polynomial: x⁴ + x³ + 1
  x⁴ → coefficient 1
  x³ → coefficient 1
  x² → coefficient 0 (not present)
  x¹ → coefficient 0 (not present)
  x⁰ → coefficient 1

Binary representation: 1 1 0 0 1
```

**General rule:**

- Present term → write **1**
- Absent term → write **0**
- Always start from highest degree down to x⁰

**More examples:**

| Polynomial       | Binary      |
| ---------------- | ----------- |
| x⁴ + x³ + 1      | `11001`     |
| x³ + x + 1       | `1011`      |
| x⁴ + x + 1       | `10011`     |
| x⁸ + x² + x + 1  | `100000111` |
| x⁵ + x⁴ + x² + 1 | `110101`    |

---

### 4.2 Append Zeros to Message

**Rule:** Append **r zeros** to the message, where **r = degree of generator polynomial**.

```
Generator degree = 4  (highest power of x is 4)
Append 4 zeros to message.

Message:            1 0 0 1 1 1 0 0 1 0
Appended message:   1 0 0 1 1 1 0 0 1 0 | 0 0 0 0
                                           ↑ ↑ ↑ ↑
                                        4 zeros appended

These 4 zero positions will be REPLACED by the CRC remainder later.
```

**If generator is given in binary (not polynomial):**

```
Binary generator has n bits
Zeros to append = n - 1

Example: Generator = 11001 (5 bits) → append 5-1 = 4 zeros
```

---

### 4.3 Binary Division (XOR)

Perform binary long division using **XOR** (not arithmetic subtraction).

**XOR rules:**

```
0 XOR 0 = 0
1 XOR 1 = 0
0 XOR 1 = 1
1 XOR 0 = 1
Same → 0,  Different → 1
```

**Division rules:**

- Always start dividing from the **leading 1** (first 1 in current dividend)
- If leading bit is 0 → bring down next bit (cannot divide yet)
- We only care about the **remainder**, not the quotient

**Performing the division:**

```
Dividend: 1 0 0 1 1 1 0 0 1 0 0 0 0 0   (message + 4 zeros)
Divisor:  1 1 0 0 1

Step 1: Take first 5 bits: 10011
        XOR with divisor:  11001
        ─────────────────────────
        Result:             01010   → leading 0, bring down next bit

Step 2: Current: 10101 (dropped leading 0, brought down 0→10100, bring 1)
        Wait — result was 01010, bring down next bit (1):
        Current: 10101
        XOR:     11001
        ─────────────────────────
        Result:  01100   → bring down next bit (0)

Step 3: Current: 11000
        XOR:     11001
        ─────────────────────────
        Result:  00001   → bring down next bit (0)

Step 4: Current: 00010 → leading 0s, bring down: 00100 → still 0, bring down: 01000
        Current: 01000 → leading 0, bring: 10000...
        → Bring down bits one at a time until you have 5 bits starting with 1

[Following the full XOR division from lecture:]

Full division:
  10011 10 0000
  11001
  ─────
  01010 ← bring down 1
  10101
  11001
  ─────
  01100 ← bring down 0
  11000
  11001
  ─────
  00001 ← bring down 0
  00010 ← bring down 0
  00100 ← bring down 0
  01000 ← bring down 0 (no more bits)

Remainder = last 4 bits (r=4): 0010
```

---

### 4.4 Replace Zeros with Remainder

```
Appended message:  1 0 0 1 1 1 0 0 1 0 | 0 0 0 0
                                          ↑ ↑ ↑ ↑
                                          replace with remainder

Remainder: 0 0 1 0

Final code word:   1 0 0 1 1 1 0 0 1 0 | 0 0 1 0
                   ←── message (10 bits) ──→ ←CRC→
                   ←────── transmitted (14 bits) ──────→
```

> **Important:** If remainder has fewer bits than r → **pad with leading zeros** on the left.

---

## 5. Receiver Side — Verification

Receiver divides the **entire received code word** (message + CRC) by the **same generator**.

```
Received:  1 0 0 1 1 1 0 0 1 0 0 0 1 0   (14 bits, no error case)
Divisor:   1 1 0 0 1

Receiver performs same XOR division...

If NO error occurred:
  Remainder = 0 0 0 0  →  "No error, data is valid" ✓

If error occurred (any bit flipped):
  Remainder ≠ 0  →  "Error detected! Request retransmission" ✗
```

### Why Remainder = 0 When No Error?

```
Sender transmits: Message + Remainder
This = Message × x^r + Remainder
     = exactly divisible by Generator (by construction of CRC)

So: (Message × x^r + Remainder) / Generator = 0 remainder

Receiver divides same value by same generator → 0 remainder
Any bit change → division no longer exact → nonzero remainder
```

---

## 6. Efficiency Formula

```
Efficiency = (m / (m + r)) × 100%

Where:
  m = number of message bits
  r = number of CRC (redundant) bits

Example from lecture:
  m = 10 bits
  r = 4 bits
  Efficiency = (10 / 14) × 100 = 71.4%
```

| m   | r   | Efficiency |
| --- | --- | ---------- |
| 10  | 4   | 71.4%      |
| 100 | 4   | 96.2%      |
| 7   | 3   | 70.0%      |
| 8   | 4   | 66.7%      |

> Higher m (longer messages) relative to r → better efficiency.

---

## 7. Quick Reference Rules

```
┌─────────────────────────────────────────────────────────────────┐
│  RULE 1: Zeros to append                                        │
│    Generator given as POLYNOMIAL → append (degree) zeros       │
│    Generator given as BINARY     → append (bits - 1) zeros     │
│                                                                  │
│  RULE 2: Division always starts from leading 1                 │
│    If current chunk starts with 0 → bring down next bit        │
│    Never divide if leading bit is 0                             │
│                                                                  │
│  RULE 3: Remainder extraction                                   │
│    Take LAST r bits of remainder (LSB side)                    │
│    If remainder < r bits → pad with leading zeros              │
│                                                                  │
│  RULE 4: Receiver check                                         │
│    Divide full code word by generator                           │
│    Remainder = 0 → no error                                     │
│    Remainder ≠ 0 → error detected                               │
└─────────────────────────────────────────────────────────────────┘
```

---

## 8. More Worked Examples

### Example 2 — Generator in Binary Form

```
Message:    1 1 0 1 0 1 1   (7 bits)
Generator:  1 0 1 1         (4 bits in binary)

Zeros to append: 4 - 1 = 3 zeros
Appended message: 1 1 0 1 0 1 1 | 0 0 0

XOR Division with 1011:

1101011000
1011
────
 110011000  → leading 0, bring 1 → 1100
 1011
 ────
  111 → bring down 1 → 1111
  1011
  ────
   100 → bring down 0 → 1000
   1011
   ────
    011 → bring → 0110 → bring → 1100
    1011
    ────
     111 → bring → no bits

Remainder (last 3 bits): 1 1 1
Code word: 1 1 0 1 0 1 1 | 1 1 1   (10 bits total)
```

### Example 3 — Quick Verification

```
Transmitted: 1 1 0 1 0 1 1 1 1 1   (from example 2)
Generator:   1 0 1 1

Divide 1101011111 by 1011:
[perform XOR division same way]

Remainder = 0 0 0  →  No error ✓
```

---

## 9. Common CRC Standards

| Standard      | Generator Polynomial | Bits | Used In                       |
| ------------- | -------------------- | ---- | ----------------------------- |
| **CRC-8**     | x⁸+x²+x+1            | 8    | ATM, USB                      |
| **CRC-16**    | x¹⁶+x¹⁵+x²+1         | 16   | USB, HDLC, Bluetooth          |
| **CRC-32**    | x³²+x²⁶+x²³+...      | 32   | **Ethernet, Wi-Fi, ZIP, PNG** |
| **CRC-CCITT** | x¹⁶+x¹²+x⁵+1         | 16   | HDLC, X.25, Bluetooth         |

### CRC-32 Generator (Ethernet FCS)

```
x³² + x²⁶ + x²³ + x²² + x¹⁶ + x¹² + x¹¹ + x¹⁰ + x⁸ + x⁷ + x⁵ + x⁴ + x² + x + 1

Binary: 0x04C11DB7 (standard form)
        0xEDB88320 (reflected/reversed — used in most implementations)

This is appended as the 4-byte FCS at the end of every Ethernet frame.
```

---

## 10. 🔴 Security Context

### CRC is NOT Cryptographically Secure

```
CRC detects ACCIDENTAL errors.
CRC does NOT protect against INTENTIONAL tampering.

Why?
  CRC is linear → attacker can forge valid CRC for modified data
  Attacker knows the generator polynomial (it's public/standard)
  Attacker modifies data → recalculates CRC → sends modified data + new CRC
  Receiver's check passes → attack undetected

This is a fundamental property of linear codes.
```

### CRC Forgery Attack

```python
# Conceptual demonstration (attacker's perspective)
original_data = "transfer $100"
crc_original = compute_crc32(original_data)
# → sends: original_data + crc_original

# Attacker intercepts and modifies:
modified_data = "transfer $900"
crc_forged = compute_crc32(modified_data)  # attacker recalculates!
# → forwards: modified_data + crc_forged

# Receiver checks: crc_forged matches modified_data → PASSES
# Attack successful — CRC provides ZERO security
```

### What Actually Provides Integrity Security?

| Mechanism             | Purpose                     | Secure Against Attacker?          |
| --------------------- | --------------------------- | --------------------------------- |
| CRC-32                | Accidental errors           | ❌ No                             |
| MD5 (hash)            | Was used for integrity      | ❌ Broken (collisions)            |
| SHA-256 (hash)        | Integrity                   | ⚠️ Hash alone = no authentication |
| **HMAC-SHA256**       | Authenticated integrity     | ✅ Yes (requires secret key)      |
| **Digital Signature** | Non-repudiation + integrity | ✅ Yes (requires private key)     |
| **AES-GCM**           | Encrypted + authenticated   | ✅ Yes                            |

> **Rule:** CRC for error detection. HMAC/signatures for security.

### CRC in Real Protocol Attacks

#### Ethernet Frame Injection

```
Attacker crafts a fake Ethernet frame:
[Dst MAC][Src MAC (spoofed)][EtherType][Payload][Recalculated CRC-32]

Switch receives frame → checks CRC-32 → passes (attacker recalculated it)
Switch forwards frame based on dst MAC
Frame injection successful

Tools: Scapy (can auto-compute correct CRC-32)
```

#### ZIP/File Integrity Bypass

```
Old archive software relied only on CRC-32 for integrity.
Attacker modifies zip contents → updates CRC-32 in zip header
Software sees matching CRC → accepts modified file
Used in: malware distribution, file tampering

Modern approach: PGP/GPG signatures on archives
```

#### CRC Oracle Attack (Advanced)

```
In some protocols, CRC failure generates a different response than success.
Attacker uses this oracle to:
  1. Guess plaintext one byte at a time
  2. Verify guesses by observing CRC-pass vs CRC-fail responses

This is analogous to padding oracle attacks on encryption.
```

---

## 11. 🧪 Practical Labs

### Lab 1 — Implement CRC from Scratch in Python

```python
# Save as crc_full.py — run: python3 crc_full.py

def poly_to_binary(terms: dict, degree: int) -> str:
    """Convert polynomial coefficients to binary string"""
    binary = ''
    for i in range(degree, -1, -1):
        binary += '1' if i in terms else '0'
    return binary

def xor_divide(dividend: str, divisor: str) -> str:
    """Perform CRC XOR division, return remainder"""
    dividend = list(dividend)
    div_len = len(divisor)

    for i in range(len(dividend) - div_len + 1):
        if dividend[i] == '1':
            for j in range(div_len):
                dividend[i+j] = '0' if dividend[i+j] == divisor[j] else '1'

    remainder = ''.join(dividend[-(div_len-1):])
    return remainder

def compute_crc(message: str, generator: str) -> dict:
    """Full CRC computation: returns CRC bits and code word"""
    r = len(generator) - 1
    appended = message + '0' * r
    remainder = xor_divide(appended, generator)

    # Pad remainder to r bits if shorter
    remainder = remainder.zfill(r)

    # Replace appended zeros with remainder
    code_word = message + remainder

    return {
        'message': message,
        'generator': generator,
        'r': r,
        'appended_message': appended,
        'remainder': remainder,
        'code_word': code_word,
        'efficiency': f"{len(message)}/{len(code_word)} = {len(message)/len(code_word)*100:.1f}%"
    }

def verify_crc(received: str, generator: str) -> bool:
    """Receiver: check if received code word is error-free"""
    remainder = xor_divide(received, generator)
    return all(b == '0' for b in remainder)

# ─── Test: Lecture example ───
print("=" * 55)
print("LECTURE EXAMPLE")
print("=" * 55)
message   = "1001110010"
generator = "11001"       # x^4 + x^3 + 1

result = compute_crc(message, generator)
print(f"Message:          {result['message']}")
print(f"Generator:        {result['generator']}")
print(f"Redundant bits:   {result['r']}")
print(f"Appended message: {result['appended_message']}")
print(f"CRC remainder:    {result['remainder']}")
print(f"Code word sent:   {result['code_word']}")
print(f"Efficiency:       {result['efficiency']}")
print()

# ─── Receiver verification ───
print("RECEIVER SIDE:")
print(f"No error:  {verify_crc(result['code_word'], generator)} → " +
      ("NO ERROR ✓" if verify_crc(result['code_word'], generator) else "ERROR!"))

# Introduce single bit error
corrupted = list(result['code_word'])
corrupted[3] = '1' if corrupted[3] == '0' else '0'
corrupted_str = ''.join(corrupted)
print(f"1-bit error: {not verify_crc(corrupted_str, generator)} → " +
      ("ERROR DETECTED ✓" if not verify_crc(corrupted_str, generator) else "MISSED ✗"))

# Burst error (bits 3-6)
burst = list(result['code_word'])
for i in range(3, 7):
    burst[i] = '1' if burst[i] == '0' else '0'
burst_str = ''.join(burst)
print(f"Burst error: {not verify_crc(burst_str, generator)} → " +
      ("ERROR DETECTED ✓" if not verify_crc(burst_str, generator) else "MISSED ✗"))

# ─── Test: Polynomial conversion ───
print()
print("=" * 55)
print("POLYNOMIAL → BINARY CONVERSION EXAMPLES")
print("=" * 55)
polynomials = {
    "x^4 + x^3 + 1":           ({4,3,0}, 4),
    "x^3 + x + 1":             ({3,1,0}, 3),
    "x^4 + x + 1":             ({4,1,0}, 4),
    "x^5 + x^4 + x^2 + 1":    ({5,4,2,0}, 5),
}
for name, (terms, deg) in polynomials.items():
    binary = poly_to_binary(terms, deg)
    print(f"  {name:30s} → {binary}")
```

### Lab 2 — Real CRC-32 (Ethernet Standard)

```python
# Save as crc32_ethernet.py — run: python3 crc32_ethernet.py
import binascii
import struct

def ethernet_crc32(data: bytes) -> int:
    """Compute CRC-32 as used in Ethernet FCS"""
    return binascii.crc32(data) & 0xFFFFFFFF

# Simulate Ethernet frame payload
payload = b"Hello from Parrot OS to Metasploitable2"
crc = ethernet_crc32(payload)
print(f"Payload:  {payload}")
print(f"CRC-32:   0x{crc:08X} ({crc})")
print(f"FCS bytes: {struct.pack('<I', crc).hex()}")

# Verify: CRC of (data + CRC bytes) should be a constant (0xDEBB20E3)
full_frame = payload + struct.pack('<I', crc)
check = ethernet_crc32(full_frame)
print(f"\nFull frame CRC check: 0x{check:08X}")
print(f"Magic constant expected: 0xDEBB20E3")
print(f"Valid frame: {'YES ✓' if check == 0xDEBB20E3 else 'NO ✗'}")

# Demonstrate tampering detection
print("\n--- Tampering Test ---")
tampered = bytearray(payload)
tampered[0] = ord('X')           # change first byte
tampered_crc = ethernet_crc32(bytes(tampered))
tampered_frame = bytes(tampered) + struct.pack('<I', crc)  # old CRC
check2 = ethernet_crc32(tampered_frame)
print(f"Tampered payload (old CRC): 0x{check2:08X}")
print(f"Tampering detected: {'YES ✓' if check2 != 0xDEBB20E3 else 'NO ✗'}")

# Attacker recalculates CRC (forgery)
forged_crc = ethernet_crc32(bytes(tampered))
forged_frame = bytes(tampered) + struct.pack('<I', forged_crc)
check3 = ethernet_crc32(forged_frame)
print(f"\nForged frame (attacker recalculated CRC): 0x{check3:08X}")
print(f"Forgery detected by CRC: {'YES' if check3 != 0xDEBB20E3 else 'NO ✗ ← CRC BYPASSED! Use HMAC instead.'}")
```

### Lab 3 — See CRC-32 in Wireshark (Ethernet FCS)

```bash
# Enable FCS validation in Wireshark
# Edit → Preferences → Protocols → Ethernet
# ✓ "Validate the Ethernet checksum if possible"

# Generate traffic
ping -c 5 192.168.56.101

# In Wireshark display filter:
# eth.fcs_bad == 1   ← shows frames with bad CRC

# View FCS field in frame:
# Expand "Ethernet II" → look for "Frame check sequence: 0x..."

# Capture and inspect with tcpdump
sudo tcpdump -i eth0 -XX -c 10 host 192.168.56.101 2>/dev/null | head -80
# -XX shows raw hex bytes including FCS
```

### Lab 4 — Forge an Ethernet Frame (Security Demo)

```bash
# Demonstrate CRC-32 forgery with Scapy (your lab only)
sudo python3 - << 'EOF'
from scapy.all import *

# Scapy auto-computes correct CRC-32 for crafted frames
# This means an attacker can always create valid-CRC frames

# Craft a frame with spoofed source MAC
frame = Ether(src="00:de:ad:be:ef:00", dst="ff:ff:ff:ff:ff:ff") / \
        IP(src="192.168.56.200", dst="192.168.56.101") / \
        ICMP()

# Scapy automatically computes correct Ethernet FCS
print("Crafted frame:")
frame.show()
print(f"\nEthernet FCS (CRC-32): auto-computed by Scapy")
print("This proves CRC alone cannot detect forged frames!")
print("Attacker can always compute valid CRC for any payload.")

# To actually send: sendp(frame, iface="eth0")
# Commented out — just showing the concept
EOF
```

### Lab 5 — CRC in File Integrity Checking

```bash
# CRC-32 is used in many file formats for integrity
# PNG, ZIP, GZIP, etc. use CRC-32

# Create a test file
echo "This is test data from Parrot OS lab" > /tmp/test_data.txt

# Compute CRC-32 of file (using python)
python3 -c "
import binascii
with open('/tmp/test_data.txt', 'rb') as f:
    data = f.read()
crc = binascii.crc32(data) & 0xFFFFFFFF
print(f'File: test_data.txt')
print(f'CRC-32: 0x{crc:08X}')
print(f'This is what ZIP/PNG stores for integrity check')
"

# Compare with cksum command (uses CRC)
cksum /tmp/test_data.txt

# Tamper with file
echo "TAMPERED" >> /tmp/test_data.txt

# CRC changes completely (avalanche effect)
python3 -c "
import binascii
with open('/tmp/test_data.txt', 'rb') as f:
    data = f.read()
crc = binascii.crc32(data) & 0xFFFFFFFF
print(f'Tampered CRC-32: 0x{crc:08X}')
print('CRC changed → tampering detected by receiver')
"

# Clean up
rm /tmp/test_data.txt
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              CRC — EXAM CHEAT SHEET                                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHAT IS CRC?                                                        ║
║  Error detection via polynomial division (XOR-based)               ║
║  Most powerful & most widely used detection method                 ║
║  Used in: Ethernet(CRC-32), Wi-Fi, USB, ZIP, Hard drives           ║
╠══════════════════════════════════════════════════════════════════════╣
║  CRC DETECTS                                                         ║
║  ✓ All single bit errors                                           ║
║  ✓ All double bit errors                                           ║
║  ✓ All odd number of bit errors                                    ║
║  ✓ Burst errors of length ≤ degree of polynomial                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  SENDER STEPS                                                        ║
║  1. Convert generator polynomial → binary (coefficients)           ║
║  2. Append r zeros to message (r = degree of polynomial)           ║
║     OR if binary given: append (bits-1) zeros                     ║
║  3. XOR divide appended message by generator                       ║
║  4. Take last r bits of remainder (pad with 0s if needed)         ║
║  5. Replace the r appended zeros with remainder                    ║
║  6. Transmit: [message + remainder] = code word                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  RECEIVER STEPS                                                      ║
║  1. Divide received code word by same generator                    ║
║  2. Remainder = 0 → NO ERROR                                       ║
║  3. Remainder ≠ 0 → ERROR DETECTED → request retransmit           ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY RULES FOR DIVISION                                              ║
║  Always start dividing from leading 1                              ║
║  If current chunk starts with 0 → bring down next bit             ║
║  Use XOR (same bits→0, different bits→1)                          ║
║  We only need REMAINDER, not quotient                              ║
╠══════════════════════════════════════════════════════════════════════╣
║  EFFICIENCY FORMULA                                                  ║
║  Efficiency = m / (m + r) × 100%                                   ║
║  m = message bits, r = CRC bits                                    ║
╠══════════════════════════════════════════════════════════════════════╣
║  POLYNOMIAL → BINARY CONVERSION                                      ║
║  x⁴+x³+1  → 11001   (degree 4, 5 bits total)                      ║
║  x³+x+1   → 1011    (degree 3, 4 bits total)                      ║
║  Rule: degree d → d+1 bits in binary representation               ║
╠══════════════════════════════════════════════════════════════════════╣
║  ZEROS TO APPEND                                                     ║
║  Polynomial given → append (degree) zeros                         ║
║  Binary given     → append (num_bits - 1) zeros                   ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  CRC ≠ Security (attacker can recalculate CRC after tampering)     ║
║  CRC detects ACCIDENTAL errors only                                ║
║  For security: use HMAC-SHA256, AES-GCM, or digital signatures    ║
║  CRC forgery: Scapy auto-computes valid CRC for crafted frames    ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Hamming Code — error correction
- [ ] Checksum — TCP/UDP/IP header checksum
- [ ] Flow Control — Stop-and-Wait, Sliding Window
- [ ] MAC protocols — CSMA/CD, CSMA/CA
- [ ] Network Layer — IP Addressing, Subnetting

---

_Notes compiled from: Networking Course Lecture 09 — CRC (Cyclic Redundancy Check)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
