# 🌐 Computer Networks — Fundamentals

### Cybersecurity Student Notes | Networking Course — Lecture 01

> **Target:** University Cybersecurity Dept. Student
> **Scope:** Theory + Practical Attack Surface + Ethical Hacking Context
> **Lab Environment:** Parrot OS (VirtualBox) · Metasploitable2 · Own Web Projects

---

## 📑 Table of Contents

1. [What is a Computer Network?](#1-what-is-a-computer-network)
2. [Sender, Receiver & the Communication Model](#2-sender-receiver--the-communication-model)
3. [Protocols — Why They Exist](#3-protocols--why-they-exist)
4. [Inter-Process Communication vs Computer Networks](#4-inter-process-communication-vs-computer-networks)
5. [Client-Server Model](#5-client-server-model)
6. [Network Functions — Mandatory vs Optional](#6-network-functions--mandatory-vs-optional)
7. [The OSI Model — Why It Was Needed](#7-the-osi-model--why-it-was-needed)
8. [OSI 7 Layers — Overview](#8-osi-7-layers--overview)
9. [🔴 Attack Surface Mapped to Each Concept](#9--attack-surface-mapped-to-each-concept)
10. [🧪 Practical Lab Guide — Your Own Systems](#10--practical-lab-guide--your-own-systems)
11. [Hacker Mindset — Black Hat / White Hat / Gray Hat](#11-hacker-mindset--black-hat--white-hat--gray-hat)
12. [Tools You Must Know at This Stage](#12-tools-you-must-know-at-this-stage)
13. [Quick Reference Cheat Sheet](#13-quick-reference-cheat-sheet)

---

## 1. What is a Computer Network?

### Definition

A **computer network** is a collection of computing devices (homogeneous or heterogeneous) interconnected so they can **share data**.

```
[Device A] ──── connection ──── [Device B]
              (wired / wireless)
```

- **Homogeneous devices** → same OS, same architecture (e.g., two Windows PCs)
- **Heterogeneous devices** → different OS/architecture (e.g., Android phone + Linux server)

### Core Purpose

> The single most important purpose: **Data Sharing**

Without networking, every piece of data stays siloed. Networking breaks those silos.

### Types of Connections

| Type         | Examples                  | Security Implication                                                             |
| ------------ | ------------------------- | -------------------------------------------------------------------------------- |
| **Wired**    | Ethernet (CAT5/6), Fiber  | Harder to intercept physically, but switch-level attacks possible (ARP spoofing) |
| **Wireless** | Wi-Fi (802.11), Bluetooth | Easier to intercept over-the-air (evil twin, deauth attacks)                     |

---

## 2. Sender, Receiver & the Communication Model

### Basic Model

```
[Sender / Client]  ──── packet/data ────►  [Receiver / Server]
         M  ────────────────────────────────►  M  (ideally)
         M  ──── noise / attack ────────────►  M' (corrupted)
```

- **Sender** → The machine/process generating and transmitting data
- **Receiver** → The machine/process consuming the received data
- **Message M** → The original data payload

### What Can Go Wrong (Security Lens)

| Threat                   | What Happens to M        | Attack Category        |
| ------------------------ | ------------------------ | ---------------------- |
| Network noise            | M becomes M' (garbled)   | Accidental corruption  |
| Man-in-the-Middle (MitM) | M is intercepted, read   | Confidentiality breach |
| MitM with tampering      | M is modified to M''     | Integrity breach       |
| Replay Attack            | Old M is resent later    | Authentication bypass  |
| DoS                      | M never reaches receiver | Availability breach    |

> 🔑 **CIA Triad** → **C**onfidentiality, **I**ntegrity, **A**vailability — every network attack targets one or more of these.

---

## 3. Protocols — Why They Exist

### The Language Analogy

> Imagine calling someone who speaks Telugu while you only speak Punjabi.
> Sound travels (connection works), but meaning is lost (protocol mismatch).

A **protocol** is a **standardized set of rules** that both sender and receiver agree to follow — ensuring the received data is understood, not just received.

### What a Protocol Defines

- **Syntax** → Format/structure of data (how bytes are arranged)
- **Semantics** → Meaning of each field (what a value represents)
- **Timing** → When to send, how fast, when to wait

### Common Protocols You'll Attack/Defend

| Protocol | Layer       | Use               | Known Vulnerabilities              |
| -------- | ----------- | ----------------- | ---------------------------------- |
| HTTP     | Application | Web browsing      | Cleartext, injection, CSRF         |
| HTTPS    | Application | Encrypted web     | SSL stripping, cert pinning bypass |
| FTP      | Application | File transfer     | Cleartext credentials              |
| SSH      | Application | Secure shell      | Brute force, weak keys             |
| DNS      | Application | Name resolution   | DNS spoofing, cache poisoning      |
| ARP      | Data Link   | IP→MAC mapping    | ARP spoofing, MitM                 |
| TCP      | Transport   | Reliable delivery | SYN flood, session hijacking       |
| UDP      | Transport   | Fast, unreliable  | UDP flood, amplification DDoS      |
| IP       | Network     | Routing packets   | IP spoofing                        |
| ICMP     | Network     | Ping/diagnostics  | Ping flood, Smurf attack           |

---

## 4. Inter-Process Communication vs Computer Networks

### IPC (Inter-Process Communication)

When processes communicate **within the same machine**:

```
[Keyboard Process] ──(IPC)──► [Monitor/Display Process]
         within ONE machine — managed by OS Kernel
```

- Handled by the **Operating System Kernel**
- Examples: Pipes, Shared Memory, Sockets (Unix domain), Signals
- **NOT part of computer networking**

### Where Computer Networking Begins

```
[Machine 1]  ════════════════════  [Machine 2]
  Process A  ◄── network stack ──► Process B
             physically separated
```

Computer networking begins the moment **client and server exist on different physical (or virtual) machines**.

### 🔴 Security Note — IPC Attacks

Even though IPC is OS-level, attackers exploit it:

- **Privilege escalation** via IPC (e.g., exploiting SUID programs)
- **Shared memory attacks** (reading another process's memory)
- **Unix socket hijacking**

In Metasploitable2, many local privilege escalation paths use IPC vulnerabilities.

---

## 5. Client-Server Model

### The Goal of Networking (Abstraction)

> The network should make **physically separated machines feel like they are the same machine** to the running processes.

When you open Facebook — you don't feel that data is coming from a server in the USA. It feels local. That seamless abstraction is what computer networks provide.

```
[Your Machine — Bangladesh]          [Facebook Server — USA]
      Client Process          ◄──────────────────────────►   Server Process
                              feels like same machine!
```

### Models of Communication

| Model                  | Description                          | Example            |
| ---------------------- | ------------------------------------ | ------------------ |
| **Client-Server**      | Client requests, server responds     | Web browsing, APIs |
| **Peer-to-Peer (P2P)** | Both sides can request/respond       | BitTorrent, WebRTC |
| **Publish-Subscribe**  | Publisher sends, subscribers receive | Kafka, MQTT        |

### 🔴 Attack Vectors in Client-Server

| Attack                  | Target                    | How                                 |
| ----------------------- | ------------------------- | ----------------------------------- |
| **SQL Injection**       | Server (DB)               | Malicious input from client         |
| **XSS**                 | Client (Browser)          | Malicious script from server        |
| **CSRF**                | Client session            | Forged client request               |
| **SSRF**                | Server's internal network | Tricking server to make requests    |
| **Directory Traversal** | Server filesystem         | Manipulating file paths in requests |
| **Brute Force**         | Server auth               | Repeated login attempts             |

---

## 6. Network Functions — Mandatory vs Optional

The transcript draws a crucial distinction. Not all network functions need to run on every connection.

### 6.1 Mandatory Functions

These always run when data is transmitted:

#### 🔧 Error Control

Ensures the received message is the same as the sent message.

```
Sender sends:   M = 10110101
Receiver gets:  M = 10110100  ← 1 bit flipped (noise/attack)
Error control:  "CRC mismatch! Request retransmission."
```

**Techniques:**

- **CRC (Cyclic Redundancy Check)** → Most common at data link layer
- **Checksum** → Used in TCP/UDP/IP headers
- **Parity bits** → Simple single-bit error detection
- **Hamming codes** → Error correction (not just detection)

**Security Context:**

- Attackers who flip specific bits carefully can bypass CRC (if they control the checksum too — this is relevant in MitM attacks)
- **Bit-flipping attacks** in CBC encryption mode exploit this

#### 🔧 Flow Control

Prevents the sender from overwhelming the receiver's buffer.

```
Sender: [████████████████] → sending 1Gbps
Receiver buffer: [████░░░░] → can handle 100Mbps
Without flow control: BUFFER OVERFLOW → packet loss
With flow control: receiver says "slow down!"
```

**Mechanisms:**

- **Stop-and-Wait** → Send one, wait for ACK
- **Sliding Window** → Send N frames, wait for ACK (TCP uses this)
- **Congestion control** → Network-wide flow management (TCP Tahoe, Reno, CUBIC)

**Security Context:**

- **Buffer overflow** attacks exploit missing bounds checking
- **SYN flood** exhausts server's connection buffer (TCP half-open connections)
- **Slowloris attack** → opens many connections slowly, exhausting server resources

#### 🔧 Multiplexing & Demultiplexing

Multiple processes share ONE network connection.

```
Machine has 3 processes:
  Chrome (port 54321) ──┐
  VS Code (port 54322) ──┼──► single NIC ──► Internet
  Postman (port 54323) ──┘

Incoming data gets demultiplexed to the right process by PORT NUMBER
```

**How it works:**

- Each process gets a unique **port number** (0–65535)
- **Ports 0–1023** = Well-known (HTTP:80, HTTPS:443, SSH:22, FTP:21)
- **Ports 1024–49151** = Registered
- **Ports 49152–65535** = Dynamic/Ephemeral (client-side)

**Security Context:**

- **Port scanning** (nmap) → Finding open ports = finding attack surface
- **Binding to privileged ports** (<1024 requires root) → Privilege escalation vector
- **Port knocking** → Security-through-obscurity technique
- **Firewall rules** → Block unnecessary ports

### 6.2 Optional Functions

Applied only when needed — adding them increases complexity and latency.

#### 🔐 Encryption / Decryption (Cryptography)

Data is transformed before transmission so interceptors can't read it.

```
Plaintext:  "password123"
Encrypted:  "aX9#kLm2$pQr..."
Intercepted by attacker: sees only "aX9#kLm2$pQr..." → useless

Receiver decrypts with key → gets "password123"
```

**When Required:**

- Banking applications ✅
- Login forms ✅
- Any sensitive data transfer ✅
- Public static content (images, CSS) ❌ (usually not needed)

**HTTP vs HTTPS:**
| | HTTP | HTTPS |
|--|------|-------|
| Encryption | ❌ None | ✅ TLS/SSL |
| Attack risk | Cleartext sniffable | Encrypted (but TLS attacks exist) |
| Port | 80 | 443 |
| Use today | Being deprecated | Standard |

**Types of Encryption:**
| Type | Examples | Key Sharing |
|------|----------|-------------|
| Symmetric | AES, DES, 3DES | Same key both sides |
| Asymmetric | RSA, ECC | Public + Private key pair |
| Hybrid | TLS handshake | Asymmetric for key exchange, then symmetric |
| Hashing | SHA-256, MD5 | One-way (not reversible) |

**Security Attack Context:**

- **SSL Stripping** → Downgrades HTTPS to HTTP (MitM)
- **BEAST, POODLE, HEARTBLEED** → Protocol-level TLS vulnerabilities
- **Weak cipher suites** → SSLv2/SSLv3, RC4 (broken)
- **Certificate spoofing** → Fake certs for MitM

#### 💾 Checkpointing (Resume Download)

Saves progress at intervals during large data transfers.

```
File: 500MB download
Checkpoint every: 100MB

Progress: [100MB✓][200MB✓][300MB✓][301MB... FAIL]
Resume:   Starts from 300MB checkpoint, not 0
```

**When Required:**

- Large file downloads ✅
- Video streaming ✅
- Small messages (WhatsApp text) ❌

**Security Context:**

- **Partial file injection** → If checkpoint data isn't validated, an attacker could inject content into a resumed download
- **Download resume abuse** → Used in some DoS scenarios

---

## 7. The OSI Model — Why It Was Needed

### The Problem: 70+ Network Functions

Before standardization, different vendors implemented networking differently. A Cisco device couldn't talk to a Juniper device without custom adapters.

**The solution:** Create a **standard model** that:

1. Lists all ~70 network functions
2. Groups them into logical **layers**
3. Every vendor/OS implements the same layers
4. Interoperability achieved

### OSI = Open Systems Interconnection

- Developed by **ISO (International Organization for Standardization)**
- Published: **1984**
- Purpose: Theoretical reference model — how data _should_ flow
- Actual internet runs on **TCP/IP model** (4 layers), but OSI is used for teaching/troubleshooting

### Conceptual Flow

```
Your message → passes DOWN through all 7 layers at sender
              → travels across the network
              → passes UP through all 7 layers at receiver
```

---

## 8. OSI 7 Layers — Overview

```
╔══════════════════════════════════════════════╗
║  LAYER 7 — APPLICATION                       ║  ← User-facing (HTTP, FTP, DNS, SMTP)
╠══════════════════════════════════════════════╣
║  LAYER 6 — PRESENTATION                      ║  ← Encryption, Encoding, Compression
╠══════════════════════════════════════════════╣
║  LAYER 5 — SESSION                           ║  ← Session management, checkpoints
╠══════════════════════════════════════════════╣
║  LAYER 4 — TRANSPORT                         ║  ← TCP/UDP, Ports, Flow/Error control
╠══════════════════════════════════════════════╣
║  LAYER 3 — NETWORK                           ║  ← IP addressing, Routing
╠══════════════════════════════════════════════╣
║  LAYER 2 — DATA LINK                         ║  ← MAC addresses, Frames, Switches
╠══════════════════════════════════════════════╣
║  LAYER 1 — PHYSICAL                          ║  ← Bits, Cables, Radio waves, NICs
╚══════════════════════════════════════════════╝
```

### Mnemonic (Top to Bottom)

**A**ll **P**eople **S**eem **T**o **N**eed **D**ata **P**rocessing

### Each Layer's Job + Protocol Data Unit (PDU)

| Layer | Name         | PDU     | Key Protocols              | Function                    |
| ----- | ------------ | ------- | -------------------------- | --------------------------- |
| 7     | Application  | Data    | HTTP, FTP, DNS, SMTP       | User-facing services        |
| 6     | Presentation | Data    | SSL/TLS, JPEG, ASCII       | Encrypt, encode, compress   |
| 5     | Session      | Data    | NetBIOS, RPC               | Start/end/manage sessions   |
| 4     | Transport    | Segment | TCP, UDP                   | End-to-end delivery, ports  |
| 3     | Network      | Packet  | IP, ICMP, OSPF             | Logical addressing, routing |
| 2     | Data Link    | Frame   | Ethernet, ARP, PPP         | Physical addressing (MAC)   |
| 1     | Physical     | Bit     | Ethernet cable, Wi-Fi, USB | Raw bit transmission        |

### Encapsulation & Decapsulation

```
SENDER (Encapsulation — adding headers layer by layer):
Application Data
  → [Transport Header][Data]              = Segment
  → [Network Header][Segment]             = Packet
  → [DL Header][Packet][DL Trailer]       = Frame
  → 0101010010110...                      = Bits

RECEIVER (Decapsulation — stripping headers layer by layer):
Bits → Frame → Packet → Segment → Data
```

> 🔴 **Security Note:** Each header layer is an attack surface. Headers can be forged, spoofed, or manipulated.

---

## 9. 🔴 Attack Surface Mapped to Each Concept

### By OSI Layer

| Layer                | Attacks                                                             | Tools                         |
| -------------------- | ------------------------------------------------------------------- | ----------------------------- |
| **7 - Application**  | SQLi, XSS, CSRF, SSRF, Command Injection, File Upload, RFI/LFI, XXE | Burp Suite, OWASP ZAP, sqlmap |
| **6 - Presentation** | SSL Stripping, BEAST, POODLE, HEARTBLEED, Downgrade attacks         | sslstrip, testssl.sh          |
| **5 - Session**      | Session Hijacking, Cookie theft, Replay attacks                     | Burp Suite, Wireshark         |
| **4 - Transport**    | SYN Flood, TCP Session Hijacking, Port Scanning, Slowloris          | nmap, hping3, Metasploit      |
| **3 - Network**      | IP Spoofing, ICMP Flood, Smurf Attack, Route poisoning              | Scapy, hping3                 |
| **2 - Data Link**    | ARP Spoofing, MAC Flooding, VLAN Hopping                            | Ettercap, arpspoof, macof     |
| **1 - Physical**     | Cable tapping, Signal jamming, Rogue AP (Wi-Fi)                     | Aircrack-ng, Kismet           |

### By Protocol Function

| Function        | Abuse by Attacker                           |
| --------------- | ------------------------------------------- |
| Error Control   | Bit-flipping attacks (CBC padding oracle)   |
| Flow Control    | SYN flood, Slowloris, connection exhaustion |
| Multiplexing    | Port scanning, service fingerprinting       |
| Encryption      | Downgrade attacks, weak cipher exploitation |
| Checkpointing   | Partial content injection                   |
| Protocol itself | Protocol fuzzing, malformed packets         |

---

## 10. 🧪 Practical Lab Guide — Your Own Systems

### Your Lab Setup

```
Host OS: Windows (HP ZBook Firefly 14 G7)
VM: VirtualBox
  ├── Parrot OS (attacker machine)
  └── Metasploitable2 (vulnerable target)
Your websites: MERN/PERN stack (React + Node.js + PostgreSQL/MongoDB)
```

### Lab 1 — Network Discovery (Passive + Active Recon)

**Goal:** Understand what devices/services exist on your network

```bash
# On Parrot OS — identify your IP range
ip addr show
# Typical: 192.168.x.x/24 on VirtualBox host-only network

# Ping sweep — find live hosts
nmap -sn 192.168.56.0/24

# Port scan Metasploitable2
nmap -sV -O 192.168.56.101
# -sV = service version detection
# -O  = OS detection

# Aggressive scan (slower but more info)
nmap -A 192.168.56.101
```

**What to observe:**

- Which ports are open on Metasploitable2?
- Which service runs on each port?
- What OS does nmap fingerprint?

This maps directly to **Layer 3/4 concepts** from the lecture.

---

### Lab 2 — Protocol Analysis with Wireshark

**Goal:** See actual packets flowing — understand sender/receiver/protocol in action

```bash
# Install Wireshark (already in Parrot OS)
wireshark &

# Or use CLI version
tshark -i eth0 -w capture.pcap

# Filter HTTP traffic
tshark -i eth0 -f "tcp port 80"

# Filter by host
tshark -i eth0 host 192.168.56.101
```

**Exercise:**

1. Open Wireshark, start capture on `eth0`
2. From Parrot OS, curl your Metasploitable2: `curl http://192.168.56.101`
3. Stop capture
4. Observe: TCP 3-way handshake → HTTP Request → HTTP Response
5. Find: Source IP, Destination IP, Port numbers, HTTP headers

**Directly maps to:** Sender/Receiver model, Protocol concept, Multiplexing

---

### Lab 3 — ARP Spoofing (Layer 2 MitM)

**Goal:** Intercept traffic between two machines — understand MitM practically

```bash
# Enable IP forwarding (so traffic passes through you)
echo 1 > /proc/sys/net/ipv4/ip_forward

# ARP spoof — tell Metasploitable2 that YOU are the gateway
arpspoof -i eth0 -t 192.168.56.101 192.168.56.1

# Run in another terminal — spoof the other direction
arpspoof -i eth0 -t 192.168.56.1 192.168.56.101

# Now capture traffic passing through you
wireshark -i eth0
```

**What this proves:** Even with a "connection", without encryption (protocol), data is readable by anyone in the middle. This is exactly the lecture's language analogy.

---

### Lab 4 — Flow Control Attack — SYN Flood

**Goal:** Understand how flow control (or lack of it) can be abused

```bash
# Install hping3
sudo apt install hping3

# SYN flood against Metasploitable2 port 80
# (ONLY on your own lab machine — never on real systems)
sudo hping3 -S --flood -V -p 80 192.168.56.101

# Monitor on Metasploitable2 side (SSH into it first)
netstat -an | grep SYN_RECV | wc -l
```

**What to observe:** SYN_RECV connections piling up. This is the flow control buffer being exhausted.

**Defense:** SYN cookies (enable with `sysctl -w net.ipv4.tcp_syncookies=1`)

---

### Lab 5 — Testing Your Own Web App (MERN/PERN)

**Goal:** Attack your own Node.js application to understand application-layer vulnerabilities

#### 5a. HTTP vs HTTPS Test

```bash
# Run your Node app on HTTP
node server.js  # port 3000

# Capture traffic from another terminal
sudo tcpdump -i lo -A port 3000 | grep -i "password\|token\|authorization"

# Log in through browser, watch tcpdump
# You'll see your password in cleartext if not encrypted
```

#### 5b. Check Your HTTP Headers (Mandatory Security Headers)

```bash
curl -I http://localhost:3000

# Look for missing security headers:
# X-Frame-Options: DENY
# X-Content-Type-Options: nosniff
# Content-Security-Policy: ...
# Strict-Transport-Security: max-age=31536000
```

Missing headers = attack surface for XSS, clickjacking, MIME sniffing.

#### 5c. Test Error Control in Your API

```bash
# Send malformed JSON — does your server crash or handle gracefully?
curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d '{"user": "test", "pass":}'  # malformed JSON

# Send oversized payload — test buffer limits
python3 -c "print('A'*100000)" | curl -X POST http://localhost:3000/api/login \
  -H "Content-Type: application/json" \
  -d @-
```

---

## 11. Hacker Mindset — Black Hat / White Hat / Gray Hat

### The Spectrum

```
Black Hat ──────────────── Gray Hat ──────────────── White Hat
Malicious intent          Ambiguous                  Authorized
No permission             Sometimes asks             Always has permission
Criminal                  Legally grey               Ethical / Legal
```

| Hat           | Who                                                           | What They Do                                         | Legal?             |
| ------------- | ------------------------------------------------------------- | ---------------------------------------------------- | ------------------ |
| **Black Hat** | Criminals, APT groups, Script kiddies                         | Unauthorized attacks for profit/damage               | ❌ Illegal         |
| **White Hat** | Penetration testers, Bug bounty hunters, Security researchers | Authorized testing to find and fix vulnerabilities   | ✅ Legal           |
| **Gray Hat**  | Some researchers, hacktivists                                 | Find vulnerabilities without permission, then report | ⚠️ Legal grey area |

### As a Cybersecurity Student — You are a White Hat

**Golden Rule:** Never test on systems you don't own or don't have **written permission** to test.

**Your Legal Lab:**

- ✅ Your own MERN/PERN web apps
- ✅ Metasploitable2 in your VirtualBox
- ✅ Deliberately vulnerable apps: DVWA, WebGoat, Juice Shop, VulnHub VMs
- ✅ HackTheBox, TryHackMe platforms (they have permission baked in)
- ❌ Any real website, even if you "find a bug"
- ❌ Anyone else's server without explicit written permission

---

## 12. Tools You Must Know at This Stage

### Recon & Scanning

| Tool      | Purpose               | Command Example      |
| --------- | --------------------- | -------------------- |
| `nmap`    | Port/service scanning | `nmap -sV -A target` |
| `whois`   | Domain info           | `whois example.com`  |
| `dig`     | DNS lookup            | `dig A example.com`  |
| `netstat` | Local connections     | `netstat -tulnp`     |
| `ss`      | Socket statistics     | `ss -tulnp`          |

### Traffic Analysis

| Tool          | Purpose                | Command Example               |
| ------------- | ---------------------- | ----------------------------- |
| `wireshark`   | GUI packet capture     | `wireshark`                   |
| `tcpdump`     | CLI packet capture     | `tcpdump -i eth0 -w out.pcap` |
| `tshark`      | CLI Wireshark          | `tshark -i eth0 -f "port 80"` |
| `netcat (nc)` | Raw TCP/UDP connection | `nc -lvnp 4444`               |

### Attack & Testing

| Tool         | Purpose             | Lab Use                      |
| ------------ | ------------------- | ---------------------------- |
| `metasploit` | Exploit framework   | Metasploitable2 exploitation |
| `burp suite` | Web proxy / scanner | Your MERN/PERN app testing   |
| `sqlmap`     | SQL injection       | Your PostgreSQL endpoints    |
| `arpspoof`   | ARP MitM            | Lab MitM simulation          |
| `hping3`     | Packet crafting     | SYN flood simulation         |
| `hydra`      | Brute force         | SSH/FTP brute force          |
| `curl`       | HTTP requests       | API testing                  |

### All pre-installed in Parrot OS ✅

---

## 13. Quick Reference Cheat Sheet

```
╔══════════════════════════════════════════════════════════╗
║           NETWORK FUNDAMENTALS — CHEAT SHEET            ║
╠══════════════════════════════════════════════════════════╣
║ KEY TERMS                                                ║
║  Sender/Client  → Initiates communication               ║
║  Receiver/Server→ Responds to communication             ║
║  Protocol       → Agreed rules for communication        ║
║  IPC            → Same machine process communication    ║
║  Network        → Different machine communication       ║
╠══════════════════════════════════════════════════════════╣
║ MANDATORY FUNCTIONS                                      ║
║  Error Control  → Detect/correct corrupted data         ║
║  Flow Control   → Prevent sender overwhelming receiver  ║
║  Mux/Demux      → Multiple processes share one link     ║
╠══════════════════════════════════════════════════════════╣
║ OPTIONAL FUNCTIONS                                       ║
║  Encryption     → Hide data from interceptors           ║
║  Checkpointing  → Resume large transfers                ║
╠══════════════════════════════════════════════════════════╣
║ OSI LAYERS (Top → Bottom)                               ║
║  7 Application  → HTTP, FTP, DNS, SMTP                  ║
║  6 Presentation → TLS/SSL, Encoding                     ║
║  5 Session      → Session management                    ║
║  4 Transport    → TCP, UDP, Ports                       ║
║  3 Network      → IP, Routing                           ║
║  2 Data Link    → MAC, Ethernet, ARP                   ║
║  1 Physical     → Cables, Bits, Wi-Fi                  ║
╠══════════════════════════════════════════════════════════╣
║ ATTACKS BY LAYER                                         ║
║  L7 → SQLi, XSS, CSRF, SSRF, RCE                       ║
║  L6 → SSL Strip, Downgrade, POODLE                     ║
║  L5 → Session Hijack, Replay                           ║
║  L4 → SYN Flood, Port Scan, Slowloris                  ║
║  L3 → IP Spoof, ICMP Flood, Smurf                      ║
║  L2 → ARP Spoof, MAC Flood, VLAN Hop                   ║
║  L1 → Jamming, Tap, Rogue AP                           ║
╠══════════════════════════════════════════════════════════╣
║ YOUR LAB TARGETS (Legal)                                 ║
║  Metasploitable2 IP: 192.168.56.101 (default)           ║
║  Your Node.js app:   localhost:3000 (or your port)      ║
║  DVWA, WebGoat, Juice Shop (install separately)         ║
╚══════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics (Upcoming Lectures)

Based on the lecture, next chapters will cover:

- [ ] OSI Layers in depth (one per lecture)
- [ ] TCP/IP Model vs OSI Model
- [ ] Physical Layer — Encoding, Media types
- [ ] Data Link Layer — MAC, ARP, Ethernet
- [ ] Network Layer — IP, Routing, ICMP
- [ ] Transport Layer — TCP 3-way handshake, UDP, Port numbers
- [ ] Application Layer — HTTP/HTTPS deep dive

---

## 🔖 References & Further Study

| Resource                          | Topic                            |
| --------------------------------- | -------------------------------- |
| TryHackMe — Pre-Security Path     | Networking basics + hands-on     |
| TryHackMe — Jr Penetration Tester | Web attacks on own systems       |
| PayloadsAllTheThings (GitHub)     | Attack payloads by category      |
| HackTricks.xyz                    | Layer-by-layer attack techniques |
| Wireshark Official Docs           | Protocol analysis                |
| nmap.org/book                     | Port scanning mastery            |
| OWASP Top 10                      | Web application attacks          |
| Metasploit Unleashed              | Metasploit framework guide       |

---

_Notes compiled from: Networking Course Lecture 01 — Computer Networks Fundamentals_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
