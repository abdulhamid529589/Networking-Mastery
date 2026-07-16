# 🌐 TCP/IP Protocol Suite & TCP/IP vs OSI Model

### Cybersecurity Student Notes | Networking Course — Lecture 03

> **Source:** Gate Smashers — TCP/IP Protocol Suite
> **Exam Relevance:** GATE · University Exams · Interviews · CompTIA ITF+ / Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is TCP/IP Protocol Suite?](#1-what-is-tcpip-protocol-suite)
2. [4-Layer vs 5-Layer TCP/IP Architecture](#2-4-layer-vs-5-layer-tcpip-architecture)
3. [TCP/IP vs OSI — Full Comparison](#3-tcpip-vs-osi--full-comparison)
4. [Each TCP/IP Layer — Deep Dive](#4-each-tcpip-layer--deep-dive)
5. [Protocol Stack — How Data Flows](#5-protocol-stack--how-data-flows)
6. [PDU Names by Layer](#6-pdu-names-by-layer)
7. [Client-Server vs Peer-to-Peer](#7-client-server-vs-peer-to-peer)
8. [🔴 Attack Surface Mapped to TCP/IP Layers](#8--attack-surface-mapped-to-tcpip-layers)
9. [🧪 Practical Labs — Your Setup](#9--practical-labs--your-setup)
10. [Exam & Interview Cheat Sheet](#10-exam--interview-cheat-sheet)

---

## 1. What is TCP/IP Protocol Suite?

### Definition

TCP/IP (Transmission Control Protocol / Internet Protocol) is a **complete, implementable** set of protocols that defines how data is transmitted over the internet.

- Also called: **Internet Protocol Suite**
- It is NOT just two protocols — it is an entire **family/suite** of protocols
- The internet runs on TCP/IP — every packet you send uses it

### Who Created It?

| Entity                   | Role                                                                  |
| ------------------------ | --------------------------------------------------------------------- |
| **DARPA**                | Defense Advanced Research Projects Agency (USA) — funded the research |
| **ARPANET**              | Advanced Research Projects Agency Network — implemented it            |
| **Vint Cerf & Bob Kahn** | Designed the core TCP/IP protocols (1970s)                            |

> ARPANET was the predecessor to the internet — a military/academic network. TCP/IP was born from it.

---

## 2. 4-Layer vs 5-Layer TCP/IP Architecture

### The Debate

Different textbooks use different versions:

| Version     | Layers                                                | Books That Use It    |
| ----------- | ----------------------------------------------------- | -------------------- |
| **4-Layer** | Network Access, Internet, Transport, Application      | Forouzan's textbook  |
| **5-Layer** | Physical, Data Link, Internet, Transport, Application | Tanenbaum's textbook |

**The story/concept is the same** — only whether Physical and Data Link are combined or separate differs.

### 4-Layer Architecture

```
┌─────────────────────────────┐
│     APPLICATION LAYER       │  ← (Application + Presentation + Session of OSI)
├─────────────────────────────┤
│      TRANSPORT LAYER        │  ← (Transport of OSI) — also called Host-to-Host
├─────────────────────────────┤
│      INTERNET LAYER         │  ← (Network of OSI) — also called Network Layer
├─────────────────────────────┤
│   NETWORK ACCESS LAYER      │  ← (Physical + Data Link of OSI combined)
└─────────────────────────────┘
```

### 5-Layer Architecture

```
┌─────────────────────────────┐
│     APPLICATION LAYER       │  ← (Application + Presentation + Session of OSI)
├─────────────────────────────┤
│      TRANSPORT LAYER        │
├─────────────────────────────┤
│      INTERNET LAYER         │
├─────────────────────────────┤
│      DATA LINK LAYER        │  ← Separated from Physical
├─────────────────────────────┤
│      PHYSICAL LAYER         │  ← Separated from Data Link
└─────────────────────────────┘
```

> **Exam tip:** Most GATE/competitive exams use 4-layer. Check which book your college follows.

---

## 3. TCP/IP vs OSI — Full Comparison

### Side-by-Side Layer Mapping

```
OSI Model (7 layers)          TCP/IP 4-Layer         TCP/IP 5-Layer
─────────────────────         ──────────────         ──────────────
┌─────────────────┐           ┌─────────────┐        ┌─────────────┐
│   Application   │           │             │        │             │
├─────────────────┤           │ Application │        │ Application │
│   Presentation  │    ──►    │   Layer     │        │   Layer     │
├─────────────────┤           │             │        │             │
│     Session     │           └─────────────┘        └─────────────┘
├─────────────────┤           ┌─────────────┐        ┌─────────────┐
│    Transport    │    ──►    │  Transport  │        │  Transport  │
├─────────────────┤           └─────────────┘        └─────────────┘
│     Network     │    ──►    ┌─────────────┐        ┌─────────────┐
├─────────────────┤           │  Internet   │        │  Internet   │
│   Data Link     │           └─────────────┘        └─────────────┘
├─────────────────┤           ┌─────────────┐        ┌─────────────┐
│    Physical     │    ──►    │Network Acc. │        │  Data Link  │
└─────────────────┘           └─────────────┘        ├─────────────┤
                                                      │  Physical   │
                                                      └─────────────┘
```

### Key Differences Table

| Parameter                 | OSI Model                                                | TCP/IP Model                                      |
| ------------------------- | -------------------------------------------------------- | ------------------------------------------------- |
| **Full Form**             | Open Systems Interconnection                             | Transmission Control Protocol / Internet Protocol |
| **Developed by**          | **ISO** (International Organization for Standardization) | **ARPANET** (funded by DARPA)                     |
| **Type**                  | **Theoretical / Reference Model**                        | **Practical / Implementable Model**               |
| **Layers**                | **7 layers**                                             | **4 layers** (or 5)                               |
| **Protocol independence** | Protocol-independent (generic)                           | Protocol-specific (built around TCP & IP)         |
| **Usage**                 | Teaching, troubleshooting reference                      | Actual Internet communication                     |
| **Architecture**          | Client-Server                                            | Both Client-Server AND Peer-to-Peer               |
| **Reliability**           | Defined at multiple layers                               | Primarily at Transport layer (TCP)                |
| **Development approach**  | Model first, then protocols                              | Protocols first, then model described             |

### Simple Way to Remember

```
OSI  = Blueprint / Theory / Documentation
TCP/IP = Actual Building / Practice / Implementation

Like:
  OSI  → Architectural plan on paper
  TCP/IP → The actual constructed building
```

---

## 4. Each TCP/IP Layer — Deep Dive

### Layer 4 — Application Layer

**OSI Equivalent:** Application + Presentation + Session layers combined

**Responsibilities:**

- **Process-to-process delivery** (same as OSI Application layer)
- **Encoding/Decoding** (OSI Presentation — handling different encoding systems)
- **Encryption/Decryption** (OSI Presentation — TLS/SSL lives here conceptually)
- **Session creation and management** (OSI Session)
- **Data generation** — where the user interacts

**Key Protocols:**
| Protocol | Purpose | Port | Security Risk |
|----------|---------|------|---------------|
| **HTTP** | Web browsing | 80 | Cleartext — sniffable |
| **HTTPS** | Encrypted web | 443 | TLS attacks, SSL strip |
| **SMTP** | Send email | 25, 587 | Open relay, spam |
| **FTP** | File transfer | 20, 21 | Cleartext credentials |
| **SSH** | Secure remote login | 22 | Brute force, weak keys |
| **Telnet** | Insecure remote login | 23 | Everything in cleartext |
| **DNS** | Domain name resolution | 53 | Cache poisoning, spoofing |
| **DHCP** | IP address assignment | 67, 68 | Starvation, rogue server |
| **SNMP** | Network device management | 161 | Community string abuse |
| **POP3/IMAP** | Receive email | 110/143 | Credential theft |

---

### Layer 3 — Transport Layer

**Also called:** Host-to-Host Layer

**Responsibilities:**

- End-to-end communication between **hosts**
- **Port-based multiplexing** (which process gets which data)
- **Connection management** (TCP only)
- **Reliability** — retransmission, ordering (TCP only)
- **Flow control** and **error control**

**Key Protocols:**
| Protocol | Type | Reliability | Use Case |
|----------|------|-------------|----------|
| **TCP** | Connection-oriented | Reliable (ACK, retransmit) | Web, Email, SSH, FTP |
| **UDP** | Connectionless | Unreliable (fire and forget) | DNS, VoIP, Video stream, DHCP |
| **SCTP** | Connection-oriented | Reliable | Telecom signaling |

#### TCP 3-Way Handshake

```
Client                    Server
  │                         │
  │──── SYN ───────────────►│   Step 1: Client initiates
  │                         │
  │◄─── SYN-ACK ────────────│   Step 2: Server acknowledges + sends its SYN
  │                         │
  │──── ACK ───────────────►│   Step 3: Client acknowledges
  │                         │
  │  ═══ Connection Established ═══
```

#### TCP vs UDP Quick Compare

```
TCP                          UDP
───────────────────          ───────────────────
✓ Connection-oriented        ✗ Connectionless
✓ Reliable delivery          ✗ No guarantee
✓ Ordered packets            ✗ May arrive out of order
✓ Flow control               ✗ No flow control
✓ Error control              ✗ Minimal error check only
✗ Slower (overhead)          ✓ Faster (low overhead)
Use: HTTP, SSH, FTP          Use: DNS, VoIP, Gaming
```

---

### Layer 2 — Internet Layer

**Also called:** Network Layer (maps to OSI Network Layer)

**Responsibilities:**

- **Source-to-destination delivery** (end-to-end across entire internet)
- **IP addressing** — adds source & destination IP to packet header
- **Routing** — finding path from source to destination
- **Logical addressing** (vs MAC = physical addressing)

**Key Protocols:**
| Protocol | Full Form | Purpose |
|----------|-----------|---------|
| **IPv4** | Internet Protocol v4 | 32-bit logical addressing |
| **IPv6** | Internet Protocol v6 | 128-bit logical addressing |
| **ICMP** | Internet Control Message Protocol | Error reporting, ping |
| **IGMP** | Internet Group Management Protocol | Multicast group management |
| **ARP** | Address Resolution Protocol | IP → MAC resolution |
| **OSPF** | Open Shortest Path First | Routing protocol |
| **BGP** | Border Gateway Protocol | Internet routing between ISPs |

#### IPv4 vs IPv6

| Parameter       | IPv4                | IPv6             |
| --------------- | ------------------- | ---------------- |
| Address size    | 32-bit              | 128-bit          |
| Example         | 192.168.1.1         | 2001:0db8::1     |
| Total addresses | ~4.3 billion        | ~340 undecillion |
| Header size     | 20 bytes (variable) | 40 bytes (fixed) |
| Security        | Optional (IPSec)    | Built-in IPSec   |

---

### Layer 1 — Network Access Layer (4-layer) / Physical + Data Link (5-layer)

**Responsibilities:**

- **Node-to-node delivery** (within one network segment — one hop)
- **MAC addressing** (physical hardware addresses)
- **Error control within local network**
- **Flow control within local network**
- **Frame creation** — encapsulates packets into frames
- **Physical transmission** — converting bits to electrical/optical/radio signals

**Key Protocols:**
| Protocol | Purpose |
|----------|---------|
| **Ethernet (IEEE 802.3)** | Wired LAN framing |
| **Wi-Fi (IEEE 802.11)** | Wireless LAN |
| **ARP** | Resolve IP → MAC address |
| **PPP** | Point-to-Point Protocol (WAN links) |
| **MAC protocols** | CSMA/CD (wired), CSMA/CA (wireless) |

---

## 5. Protocol Stack — How Data Flows

### At the Sender (Encapsulation)

```
User generates data: "GET /index.html HTTP/1.1"
          │
          ▼
┌─────────────────────────────────────────────┐
│ APPLICATION LAYER                           │
│ Adds: HTTP header, TLS if HTTPS             │
│ Result: [App Header][Data]                  │
└───────────────────┬─────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────┐
│ TRANSPORT LAYER                             │
│ Adds: TCP header (src port, dst port, seq#) │
│ Result: [TCP Header][App Header][Data]      │
└───────────────────┬─────────────────────────┘  ← Segment
                    │
                    ▼
┌─────────────────────────────────────────────┐
│ INTERNET LAYER                              │
│ Adds: IP header (src IP, dst IP, TTL)       │
│ Result: [IP][TCP][App][Data]                │
└───────────────────┬─────────────────────────┘  ← Packet
                    │
                    ▼
┌─────────────────────────────────────────────┐
│ NETWORK ACCESS LAYER                        │
│ Adds: Frame header (src MAC, dst MAC), FCS  │
│ Result: [Frame Hdr][IP][TCP][App][Data][FCS]│
└───────────────────┬─────────────────────────┘  ← Frame
                    │
                    ▼
              010110100101...  → Physical transmission (Bits)
```

### At a Router (Intermediate Node)

```
Router has ONLY: Network Access Layer + Internet Layer

Incoming frame:
  Strip frame header → read IP header → routing decision → new frame header → forward

Router does NOT look at TCP or Application headers
(Stateful firewalls / Deep Packet Inspection are exceptions)
```

### At the Receiver (Decapsulation)

```
Bits arrive
  → Frame checked (MAC address matches? FCS correct?)
  → IP header checked (destination IP = me?)
  → TCP header checked (which port? reassemble segments?)
  → Application gets the data
```

---

## 6. PDU Names by Layer

| TCP/IP Layer   | OSI Equivalent       | PDU Name                               | Contains                  |
| -------------- | -------------------- | -------------------------------------- | ------------------------- |
| Application    | App + Pres + Session | **Data / Message**                     | Raw application data      |
| Transport      | Transport            | **Segment** (TCP) / **Datagram** (UDP) | TCP/UDP header + data     |
| Internet       | Network              | **Packet**                             | IP header + segment       |
| Network Access | Data Link + Physical | **Frame** → **Bits**                   | MAC header + packet + FCS |

> **Memory trick:** **D**o **S**ome **P**eople **F**ear **B**irds?
> **D**ata → **S**egment → **P**acket → **F**rame → **B**its

---

## 7. Client-Server vs Peer-to-Peer

TCP/IP supports both architectures (unlike OSI which is primarily client-server):

### Client-Server

```
        [Server — Centralized]
       /         |          \
[Client1]   [Client2]   [Client3]

- Server has the data/service
- Clients send requests
- Server responds
- Example: Web browsing (you → Facebook servers)
```

**Your MERN/PERN app = Client-Server:**

- React frontend = Client
- Node.js backend = Server
- PostgreSQL/MongoDB = Database Server

### Peer-to-Peer (P2P)

```
[Node A] ←──────────────► [Node B]
    ▲                          ▲
    │                          │
    ▼                          ▼
[Node C] ←──────────────► [Node D]

- No central authority
- Every node can be both client and server
- Data distributed across all nodes
- Example: BitTorrent, Blockchain, WebRTC video calls
```

### Security Implications

| Model         | Attack Surface          | Specific Risks                            |
| ------------- | ----------------------- | ----------------------------------------- |
| Client-Server | Server is single target | DDoS the server → all clients down        |
| Client-Server | Centralized data        | Breach server → all user data leaked      |
| P2P           | Distributed             | Sybil attack (fake nodes), Eclipse attack |
| P2P           | No central auth         | Poisoning (serving malicious files)       |

---

## 8. 🔴 Attack Surface Mapped to TCP/IP Layers

### Application Layer Attacks

| Attack                    | Target Protocol | Tool         | Description                   |
| ------------------------- | --------------- | ------------ | ----------------------------- |
| **SQL Injection**         | HTTP/HTTPS      | sqlmap, Burp | Inject SQL in web requests    |
| **XSS**                   | HTTP/HTTPS      | Burp Suite   | Inject scripts via web app    |
| **CSRF**                  | HTTP/HTTPS      | Burp Suite   | Forge authenticated requests  |
| **DNS Spoofing**          | DNS (UDP 53)    | dnsspoof     | Fake DNS responses            |
| **DNS Cache Poisoning**   | DNS             | Scapy        | Corrupt resolver cache        |
| **SMTP Open Relay**       | SMTP (25)       | Telnet       | Send spam through mail server |
| **FTP Bounce**            | FTP (21)        | nmap -b      | Port scan via FTP server      |
| **Telnet Sniffing**       | Telnet (23)     | Wireshark    | Everything in cleartext       |
| **SSL Stripping**         | HTTPS→HTTP      | sslstrip     | Downgrade to cleartext        |
| **SNMP Community String** | SNMP (161)      | snmpwalk     | Read device config            |

### Transport Layer Attacks

| Attack                     | Target              | Tool                | Description                     |
| -------------------------- | ------------------- | ------------------- | ------------------------------- |
| **SYN Flood**              | TCP 3-way handshake | hping3              | Exhaust server's SYN backlog    |
| **TCP Session Hijacking**  | TCP sequence number | Scapy               | Inject into established session |
| **UDP Flood**              | UDP                 | hping3, UDP unicorn | Flood with UDP datagrams        |
| **Port Scanning**          | TCP/UDP ports       | nmap                | Find open services              |
| **Slowloris**              | HTTP over TCP       | slowloris.py        | Keep connections half-open      |
| **UDP Amplification DDoS** | DNS/NTP over UDP    | Various             | Small request → large response  |

### Internet Layer Attacks

| Attack                      | Target           | Tool          | Description                 |
| --------------------------- | ---------------- | ------------- | --------------------------- |
| **IP Spoofing**             | IPv4 src address | Scapy, hping3 | Forge source IP in packets  |
| **ICMP Flood (Ping Flood)** | ICMP             | hping3        | Overwhelm with ping packets |
| **Smurf Attack**            | ICMP + broadcast | Legacy        | Amplified ICMP flood        |
| **Fragmentation Attack**    | IP fragmentation | Scapy         | Send crafted fragments      |
| **TTL Manipulation**        | IP TTL field     | Scapy         | Evade IDS/firewalls         |
| **BGP Hijacking**           | BGP routing      | Router config | Reroute internet traffic    |

### Network Access Layer Attacks

| Attack                     | Target            | Tool               | Description                   |
| -------------------------- | ----------------- | ------------------ | ----------------------------- |
| **ARP Spoofing**           | ARP (IP→MAC)      | arpspoof, Ettercap | Become MitM on LAN            |
| **MAC Flooding**           | Switch CAM table  | macof              | Force switch to broadcast all |
| **VLAN Hopping**           | 802.1Q            | Scapy              | Bypass VLAN isolation         |
| **DHCP Starvation**        | DHCP              | Yersinia           | Exhaust IP pool               |
| **Rogue DHCP Server**      | DHCP              | Yersinia, dnsmasq  | Assign fake gateway           |
| **Wi-Fi Deauth**           | 802.11 management | aireplay-ng        | Disconnect Wi-Fi clients      |
| **Evil Twin AP**           | 802.11            | hostapd            | Fake access point             |
| **WPA2 Handshake Capture** | 802.11 auth       | airodump-ng        | Capture for offline crack     |

---

## 9. 🧪 Practical Labs — Your Setup

### Lab 1 — See TCP/IP in Action with Wireshark

```bash
# Start Wireshark on Parrot OS
wireshark &

# Filter for a full TCP connection to Metasploitable2
# In Wireshark filter bar:
tcp and ip.addr == 192.168.56.101

# From terminal — make an HTTP request
curl http://192.168.56.101

# In Wireshark — look for:
# 1. ARP request (Network Access — IP to MAC)
# 2. TCP SYN → SYN-ACK → ACK (Transport — 3-way handshake)
# 3. HTTP GET request (Application)
# 4. HTTP 200 OK response (Application)
# 5. TCP FIN → FIN-ACK (connection teardown)
```

**What each packet maps to:**

```
ARP          → Network Access Layer
TCP SYN/ACK  → Transport Layer (TCP)
IP header    → Internet Layer
HTTP headers → Application Layer
```

---

### Lab 2 — See Every Layer Header in Wireshark

```bash
# Capture and analyze one packet in detail

# Step 1: Capture HTTP traffic
sudo tcpdump -i eth0 -w /tmp/http_capture.pcap host 192.168.56.101

# Step 2: Make request (new terminal)
curl http://192.168.56.101

# Step 3: Stop capture (Ctrl+C), open in Wireshark
wireshark /tmp/http_capture.pcap

# Step 4: Click any HTTP packet → Expand layers:
# ► Frame (Physical + Data Link info)
# ► Ethernet II (Data Link — src/dst MAC)
# ► Internet Protocol (Internet Layer — src/dst IP)
# ► Transmission Control Protocol (Transport — ports, seq#)
# ► Hypertext Transfer Protocol (Application — HTTP data)
```

---

### Lab 3 — TCP SYN Flood (Transport Layer DoS)

```bash
# Flood Metasploitable2 port 80 with SYN packets
sudo hping3 -S --flood -V -p 80 192.168.56.101
# -S = SYN flag
# --flood = send as fast as possible
# -p 80 = target port 80

# Monitor from Metasploitable2 (in another terminal via SSH):
ssh msfadmin@192.168.56.101
watch -n1 'netstat -an | grep SYN_RECV | wc -l'

# See SYN_RECV count climbing → transport layer buffer filling up
# This is flow control being bypassed

# Defense (on Metasploitable2):
sudo sysctl -w net.ipv4.tcp_syncookies=1
```

---

### Lab 4 — ARP Spoofing MitM (Network Access Layer)

```bash
# Enable IP forwarding so you don't break connectivity
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward

# Start ARP spoofing (become MitM between Metasploitable2 and gateway)
sudo arpspoof -i eth0 -t 192.168.56.101 192.168.56.1 &
sudo arpspoof -i eth0 -t 192.168.56.1 192.168.56.101 &

# Now capture traffic passing through you
sudo wireshark -i eth0

# You will see HTTP traffic from Metasploitable2 in cleartext
# This is WHY encryption (Application Layer) matters!

# Stop after testing:
sudo pkill arpspoof
echo 0 | sudo tee /proc/sys/net/ipv4/ip_forward
```

---

### Lab 5 — Test Your Node.js App at Each Layer

```bash
# INTERNET LAYER — ping your app server
ping 192.168.56.1    # or your host machine IP

# TRANSPORT LAYER — check if port is open
nmap -p 3000 192.168.56.1
nc -zv 192.168.56.1 3000

# APPLICATION LAYER — make HTTP request
curl -v http://192.168.56.1:3000/api/health
# -v shows all headers (application layer data)

# Check if your app is vulnerable to cleartext sniffing:
sudo tcpdump -i lo -A port 3000 | grep -i "password\|token\|authorization\|cookie"
# Then log in through browser
# If you see credentials → you need HTTPS (encryption at Application Layer)
```

---

### Lab 6 — DNS (Application Layer) Attack Simulation

```bash
# On Parrot OS — test DNS resolution (normal)
dig google.com
nslookup facebook.com

# DNS spoofing concept — using /etc/hosts (local DNS override)
echo "1.2.3.4 facebook.com" | sudo tee -a /etc/hosts
curl http://facebook.com  # will go to 1.2.3.4 instead
# Remove it after: sudo sed -i '/1.2.3.4 facebook.com/d' /etc/hosts

# DNS traffic analysis
sudo tcpdump -i eth0 port 53 -v
# Make some DNS queries → see UDP packets at port 53
# This is Application Layer protocol over UDP (Transport)
```

---

## 10. Exam & Interview Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║          TCP/IP vs OSI — EXAM CHEAT SHEET                           ║
╠══════════════════════════════════════════════════════════════════════╣
║  OSI MODEL (7)           TCP/IP 4-LAYER         TCP/IP 5-LAYER      ║
║  ─────────────           ─────────────────       ─────────────────  ║
║  7. Application  ─┐      4. Application   ─┐     5. Application     ║
║  6. Presentation ─┼──►      (combined)    ─┘     4. Transport       ║
║  5. Session      ─┘      3. Transport            3. Internet        ║
║  4. Transport    ─────►  3. Transport             2. Data Link       ║
║  3. Network      ─────►  2. Internet              1. Physical        ║
║  2. Data Link    ─┐                               ──────────────     ║
║  1. Physical     ─┼──►  1. Network Access                           ║
║                  ─┘        (combined)                               ║
╠══════════════════════════════════════════════════════════════════════╣
║  KEY DIFFERENCES                                                     ║
║  OSI    → ISO developed → Theoretical → 7 layers                    ║
║  TCP/IP → ARPANET/DARPA → Practical  → 4 layers (or 5)             ║
║  OSI    → Pure reference model (blueprint)                          ║
║  TCP/IP → Actual internet communication (the building)              ║
╠══════════════════════════════════════════════════════════════════════╣
║  PDU NAMES (Bottom → Top)                                           ║
║  Bits → Frames → Packets → Segments → Data                         ║
║  D o   S ome   P eople  F ear      B irds  (mnemonic top→bottom)   ║
╠══════════════════════════════════════════════════════════════════════╣
║  LAYER RESPONSIBILITIES                                              ║
║  Application → Process-to-Process (HTTP, FTP, DNS, SMTP)           ║
║  Transport   → Host-to-Host (TCP, UDP) + Ports                     ║
║  Internet    → Source-to-Destination (IP, ICMP, ARP)              ║
║  Net Access  → Node-to-Node (Ethernet, Wi-Fi, MAC)                ║
╠══════════════════════════════════════════════════════════════════════╣
║  ARCHITECTURES TCP/IP SUPPORTS                                       ║
║  Client-Server → Centralized server, clients connect               ║
║  Peer-to-Peer  → No central authority, distributed                 ║
╠══════════════════════════════════════════════════════════════════════╣
║  ATTACKS BY LAYER                                                    ║
║  App   → SQLi, XSS, CSRF, DNS spoof, SSL strip                    ║
║  Trans → SYN flood, session hijack, port scan, Slowloris           ║
║  Net   → IP spoof, ICMP flood, BGP hijack                         ║
║  L2/L1 → ARP spoof, MAC flood, Evil twin, DHCP starve             ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] Physical Layer — Transmission media, encoding, bandwidth
- [ ] Data Link Layer — Framing, MAC, error detection (CRC)
- [ ] Network Layer — IP addressing, subnetting, routing algorithms
- [ ] Transport Layer — TCP in depth, UDP, socket programming
- [ ] Application Layer — HTTP deep dive, DNS internals, SMTP

---

_Notes compiled from: Networking Course Lecture 03 — TCP/IP Protocol Suite & TCP/IP vs OSI_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
