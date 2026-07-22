# 🌐 Socket Address — IP Address + Port Number

### Cybersecurity Student Notes | Networking Course — Lecture 59

> **Source:** Gate Smashers — What is Socket Address?
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Network+/Security+
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is a Socket Address?](#1-what-is-a-socket-address)
2. [What is a Port Number?](#2-what-is-a-port-number)
3. [Port Number Ranges](#3-port-number-ranges)
4. [Part 1 — Why Port Number Alone is Not Sufficient](#4-part-1--why-port-number-alone-is-not-sufficient)
5. [Part 2 — Why IP Address Alone is Not Sufficient](#5-part-2--why-ip-address-alone-is-not-sufficient)
6. [Why the Combination Works — Socket Address](#6-why-the-combination-works--socket-address)
7. [TCP Connection and Socket Address](#7-tcp-connection-and-socket-address)
8. [Socket Address in IPv4 vs IPv6](#8-socket-address-in-ipv4-vs-ipv6)
9. [All Addressing Concepts Side-by-Side](#9-all-addressing-concepts-side-by-side)
10. [🔴 Security Context](#10--security-context)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Exam Cheat Sheet](#12-exam-cheat-sheet)

---

## 1. What is a Socket Address?

### Definition

```
Socket Address = IP Address + Port Number

In IPv4:
  Socket Address = 32-bit IP Address + 16-bit Port Number
                 = 48 bits total

Purpose:
  To UNIQUELY IDENTIFY a specific connection (TCP session)
  between two machines on a network.

Why "socket"?
  A socket is the software endpoint of a communication link.
  It is identified by IP + Port together.
  Two sockets (one at each end) form a TCP connection.
```

### One-Line Exam Definition

```
Socket Address = IP Address (32 bits) + Port Number (16 bits)
               = Used to uniquely identify any network connection
```

### Analogy

> An IP address is like a **building address** (identifies the building).
> A port number is like a **flat/room number** (identifies who inside).
> Together they form the **Socket Address** — the complete delivery address.
> You need BOTH to deliver a letter to the right person in the right flat.

---

## 2. What is a Port Number?

### Definition

```
A port number is a 16-bit number that identifies a specific
PROCESS or APPLICATION running on a machine.

While IP identifies the machine,
Port identifies WHICH PROCESS on that machine should receive the data.

Size:   16 bits
Range:  0 to 2¹⁶ - 1  =  0 to 65,535
```

### Why Port Numbers Exist

```
A single machine runs MULTIPLE PROCESSES simultaneously:
  → Web browser (HTTP)
  → Email client (SMTP)
  → File transfer (FTP)
  → SSH terminal

All of these communicate over the SAME IP address.
The operating system uses port numbers to direct incoming
data to the correct process.

Without port numbers:
  OS would not know which process should receive which packet.
```

---

## 3. Port Number Ranges

### Three Categories

```
16-bit range: 0 to 65,535

┌─────────────────────────────────────────────────────────┐
│ Range         │ Name                │ Assigned To        │
├───────────────┼─────────────────────┼────────────────────┤
│ 0 – 1023      │ Well-Known Ports    │ Standard protocols │
│ 1024 – 49151  │ Registered Ports    │ Vendor/app use     │
│ 49152 – 65535 │ Dynamic/Ephemeral   │ OS assigns to      │
│               │ Ports               │ client connections │
└─────────────────────────────────────────────────────────┘
```

### Well-Known Ports (0–1023) — Must Memorize

| Port | Protocol | Service           |
| ---- | -------- | ----------------- |
| 20   | TCP      | FTP Data          |
| 21   | TCP      | FTP Control       |
| 22   | TCP      | SSH               |
| 23   | TCP      | Telnet            |
| 25   | TCP      | SMTP (Email send) |
| 53   | UDP/TCP  | DNS               |
| 67   | UDP      | DHCP Server       |
| 68   | UDP      | DHCP Client       |
| 80   | TCP      | HTTP              |
| 110  | TCP      | POP3              |
| 143  | TCP      | IMAP              |
| 443  | TCP      | HTTPS             |

### Registered Ports (1024–49151)

```
Used by specific applications and vendors.
Examples:
  3306  → MySQL database
  5432  → PostgreSQL database
  8080  → HTTP alternate / proxy
  27017 → MongoDB
  6379  → Redis
  3000  → Node.js / React dev server (your MERN/PERN apps!)
  5000  → Flask (Python web apps)
```

### Dynamic / Ephemeral Ports (49152–65535)

```
Assigned AUTOMATICALLY by the Operating System
when YOUR machine initiates a connection.

Example:
  You open browser and connect to Facebook.
  OS picks a random port from this range, e.g., 52,341
  Your socket: 192.168.1.5 : 52341
  Facebook socket: 157.240.1.35 : 443

You never choose this number — OS handles it.
Lecture uses "P" (general symbol) for this range.
```

---

## 4. Part 1 — Why Port Number Alone is Not Sufficient

### Scenario Setup

```
My machine (M): connects to Facebook server (S1)
My port number: P (some ephemeral port, say 52341)

M creates connection:
  M's socket: [My IP] : P
  S1 records: "Connection from port P"
```

### The Problem — Another Machine Gets the Same Port

```
Another machine (G) from a DIFFERENT network:
  G also wants to connect to Facebook (S1)
  OS assigns G the SAME port number P (very likely!)
  Port range is only 0–65535 — with millions of users,
  duplicate port numbers across different machines are certain.

G creates connection:
  G's socket: [G's IP] : P
  S1 records: "Connection from port P" ← SAME AS M!

Now S1 has TWO connections, BOTH labeled "port P".

When S1 sends a reply to "port P":
  Which machine should it go to? M or G?
  S1 cannot tell! → CONFLICT / AMBIGUITY!

Conclusion: Port number alone is NOT sufficient
            to uniquely identify a connection.
```

### Visual

```
[Machine M] ──P:52341──► [Facebook S1]
                              ↑
[Machine G] ──P:52341──► [Facebook S1]

S1 receives data from "port 52341"
S1 cannot distinguish M from G!
Port number alone FAILS to uniquely identify the connection.
```

---

## 5. Part 2 — Why IP Address Alone is Not Sufficient

### Scenario Setup

```
Machine M has IP address: IP1
M creates ONE connection to Facebook S1.
S1 records: "Connection from IP1"
```

### The Problem — Same Machine, Multiple Processes

```
Modern OS supports MULTITASKING.
Multiple processes on the SAME machine can connect to the same server.

Example on Machine M (IP = IP1):
  Process 1: Chrome browser tab → opens connection to Facebook
  Process 2: Another Chrome tab → also opens connection to Facebook
  Process 3: A mobile app → also opens connection to Facebook

All three connections come from:
  IP1 : [different ports]

But if we only track IP:
  Connection 1: from IP1
  Connection 2: from IP1  ← SAME!
  Connection 3: from IP1  ← SAME!

S1 cannot distinguish which data belongs to which process!
Replies go to "IP1" — but which PROCESS on that machine?

Conclusion: IP address alone is NOT sufficient
            to uniquely identify a connection.
```

### Visual

```
Machine M (IP = IP1):
  Process 1 (Chrome Tab 1)  ──IP1──► S1
  Process 2 (Chrome Tab 2)  ──IP1──► S1
  Process 3 (Mobile App)    ──IP1──► S1

S1 receives data from "IP1" three times.
S1 cannot tell which process to send replies to!
IP address alone FAILS.
```

---

## 6. Why the Combination Works — Socket Address

### Both Problems Solved Together

```
Case 1: Two different machines, same port number

  Machine M: IP1 : P     → Socket = (IP1, P)
  Machine G: IP2 : P     → Socket = (IP2, P)

  IP1 ≠ IP2  →  (IP1, P) ≠ (IP2, P)  → UNIQUE ✓

Case 2: Same machine, multiple processes

  Machine M (IP1):
    Process 1: IP1 : P1  → Socket = (IP1, P1)
    Process 2: IP1 : P2  → Socket = (IP1, P2)

  P1 ≠ P2  →  (IP1, P1) ≠ (IP1, P2)  → UNIQUE ✓

Conclusion:
  IP Address alone  → NOT unique (same machine, multiple processes)
  Port Number alone → NOT unique (different machines, same port possible)
  IP + Port together → ALWAYS UNIQUE → Socket Address ✓
```

### Why It Works — The Math

```
Total possible sockets per machine:
  1 IP address × 65,536 port numbers = 65,536 unique sockets per machine

Total possible sockets across all IPv4 machines:
  2³² IP addresses × 2¹⁶ port numbers = 2⁴⁸ unique socket combinations

This is more than enough to uniquely identify any connection
anywhere in the world simultaneously.
```

### Visual — Socket Address in Action

```
[Machine M]              [Machine G]
IP1 : P1  ────────────►  [Facebook S1]  ◄──────────  IP2 : P1
IP1 : P2  ────────────►  [Facebook S1]              (different IP)

S1 maintains connection table:
  (IP1, P1) → Process 1 on M
  (IP1, P2) → Process 2 on M
  (IP2, P1) → Process on G

Each entry is UNIQUE → no confusion possible!
```

---

## 7. TCP Connection and Socket Address

### TCP is Connection-Oriented

```
TCP = Connection-Oriented protocol
Before sending data: TCP performs 3-way handshake
(SYN → SYN-ACK → ACK)

Connection setup = RESERVING RESOURCES:
  → Buffer memory allocated
  → CPU resources reserved
  → State maintained at both ends

When a TCP connection is established, it is uniquely identified by:
  4-tuple: (Source IP, Source Port, Destination IP, Destination Port)

This 4-tuple is sometimes called a "connection identifier"
Socket Address = one half of this (source or destination end).
```

### Full TCP Connection Identifier

```
TCP Connection = Source Socket + Destination Socket
              = (Src IP : Src Port) + (Dst IP : Dst Port)

Example:
  Your machine:  192.168.1.5  : 52341  (your socket)
  Facebook:      157.240.1.35 : 443    (Facebook's socket)

  Connection = (192.168.1.5, 52341, 157.240.1.35, 443)
  This 4-tuple uniquely identifies THIS connection globally.

Even if you open TWO tabs to Facebook:
  Tab 1: (192.168.1.5, 52341, 157.240.1.35, 443)
  Tab 2: (192.168.1.5, 52899, 157.240.1.35, 443)
                        ↑ different ephemeral port
  Both are UNIQUE connections.
```

---

## 8. Socket Address in IPv4 vs IPv6

### Comparison

```
IPv4 Socket Address:
  IP  = 32 bits
  Port = 16 bits
  Total = 48 bits

  Written as: 192.168.1.5:8080
  (IP address colon port number)

IPv6 Socket Address:
  IP  = 128 bits
  Port = 16 bits
  Total = 144 bits

  Written as: [2001:db8::1]:8080
  (IPv6 in square brackets, then colon, then port)
  Brackets needed because IPv6 address itself contains colons.
```

### Why IPv6 Uses Brackets

```
IPv4: 192.168.1.5:443
      Easy to separate — last colon separates IP from port.

IPv6: 2001:0db8:85a3::8a2e:0370:7334:443
      Which colon separates IP from port? AMBIGUOUS!

Solution: Use brackets around IPv6 address:
      [2001:0db8:85a3::8a2e:0370:7334]:443
       ←────────── IP ──────────────→ port
```

---

## 9. All Addressing Concepts Side-by-Side

| Concept            | Layer         | Size               | Identifies                       | Example                          |
| ------------------ | ------------- | ------------------ | -------------------------------- | -------------------------------- |
| MAC Address        | Data Link     | 48 bits            | Device hardware (NIC)            | `AA:BB:CC:DD:EE:FF`              |
| IP Address         | Network       | 32 bits (IPv4)     | Machine on network               | `192.168.1.5`                    |
| Port Number        | Transport     | 16 bits            | Process on machine               | `443`, `8080`                    |
| **Socket Address** | **Transport** | **48 bits (IPv4)** | **Specific connection endpoint** | **`192.168.1.5:443`**            |
| TCP 4-tuple        | Transport     | 96 bits (IPv4)     | Complete connection              | `(srcIP:srcPort, dstIP:dstPort)` |

### Why Each Alone is Insufficient

| Identifier    | Problem                                               |
| ------------- | ----------------------------------------------------- |
| MAC only      | Changes at every hop; not routable across internet    |
| IP only       | Cannot distinguish multiple processes on same machine |
| Port only     | Cannot distinguish same port on different machines    |
| **IP + Port** | **Unique across all machines AND all processes ✓**    |

---

## 10. 🔴 Security Context

### Port Scanning — The First Recon Step

```
Since each open port = a running service,
attackers SCAN ports to find attack surface.

nmap -sV 192.168.56.101    → find open ports + services on Metasploitable2
nmap -p- 192.168.56.101    → scan ALL 65,535 ports

Metasploitable2 open ports (from your labs):
  21   → FTP  (vsftpd 2.3.4 — backdoor!)
  22   → SSH  (weak config)
  80   → HTTP (vulnerable web apps)
  3306 → MySQL (root, no password!)
  5432 → PostgreSQL
  6667 → IRC  (Unreal IRCd backdoor!)

Each open port = potential entry point.
Knowing socket address concept → understanding WHY port scan matters.
```

### Attack — Port Hijacking / Confusion

```
Attacker exploits the fact that port numbers are reused:

Scenario:
  Client has connection: 192.168.1.5:52341 → Server:443
  Attacker injects forged packet:
    Source: 192.168.1.5:52341  (spoofed!)
    Dest:   Server:443
    Contains: RST flag

  Server sees RST from the correct socket address → closes connection!
  Client's connection is killed without attacking either endpoint directly.

This is a TCP RST injection attack.
Protection: TCP sequence number randomization (modern OS does this).
```

### Attack — Port Scanning Evasion Techniques

```
Attackers try to scan ports without being detected:

1. SYN scan (half-open):
   nmap -sS target   → sends SYN, never completes handshake
   → Less likely to be logged (connection never established)

2. Slow scan:
   nmap -T1 target   → one probe every 15 seconds
   → Evades rate-based IDS rules

3. Decoy scan:
   nmap -D RND:10 target → appears to come from 10 different IPs
   → Hard to identify real attacker IP

4. Random port order:
   nmap --randomize-hosts target
   → Avoids sequential scan detection patterns

As a defender on your MERN/PERN apps:
  → Close all unnecessary ports
  → Use fail2ban to block repeated port scan sources
  → Monitor with: netstat -tlnp (see all listening sockets)
```

### Attack — Ephemeral Port Prediction

```
When your machine connects to a server:
  OS assigns an ephemeral port from 49152–65535

If an attacker can PREDICT which ephemeral port you'll use:
  They can craft packets that appear to come from your socket
  (IP spoofing + port prediction = session hijacking)

Old OSes used SEQUENTIAL ephemeral ports:
  Connection 1: port 49152
  Connection 2: port 49153  ← attacker predicts next!

Modern OSes use RANDOM ephemeral ports to prevent this.

Check your system's ephemeral port range:
  cat /proc/sys/net/ipv4/ip_local_port_range
  # Typical: 32768  60999
```

### Defense — Firewall Rules Based on Socket Address

```bash
# Block specific port (deny service)
sudo iptables -A INPUT -p tcp --dport 23 -j DROP      # Block Telnet

# Allow only specific socket addresses (source IP + port)
sudo iptables -A INPUT -s 192.168.56.0/24 -p tcp --dport 22 -j ACCEPT
sudo iptables -A INPUT -p tcp --dport 22 -j DROP      # Block all others on SSH

# Allow only your app's port
sudo iptables -A INPUT -p tcp --dport 3000 -j ACCEPT  # Your Node.js app
sudo iptables -A INPUT -p tcp --dport 5432 -j DROP    # Block PostgreSQL from outside

# View all current listening sockets on your machine
sudo ss -tlnp
# or
sudo netstat -tlnp
# Output columns: Proto, Local Address (IP:Port), PID/Process
# This shows ALL socket addresses currently open on your system
```

---

## 11. 🧪 Practical Labs

### Lab 1 — Socket Address Analyzer (Python)

```python
# Save as socket_analyzer.py — run: python3 socket_analyzer.py
import socket
import struct

def explain_socket_address(ip: str, port: int):
    """Analyze and explain a socket address"""
    # Validate IP
    try:
        packed = socket.inet_aton(ip)
        ip_int = struct.unpack("!I", packed)[0]
    except socket.error:
        print(f"Invalid IP: {ip}")
        return

    # Determine port category
    if port < 0 or port > 65535:
        print(f"Invalid port: {port} (must be 0–65535)")
        return
    elif port <= 1023:
        category = "Well-Known Port (reserved for standard protocols)"
    elif port <= 49151:
        category = "Registered Port (vendor/application use)"
    else:
        category = "Dynamic/Ephemeral Port (OS-assigned for client connections)"

    # Well-known port lookup
    well_known = {
        20: "FTP Data", 21: "FTP Control", 22: "SSH",
        23: "Telnet", 25: "SMTP", 53: "DNS", 67: "DHCP Server",
        68: "DHCP Client", 80: "HTTP", 110: "POP3",
        143: "IMAP", 443: "HTTPS", 3306: "MySQL",
        5432: "PostgreSQL", 6379: "Redis", 8080: "HTTP Alt",
        27017: "MongoDB", 3000: "Node.js/React Dev",
    }
    service = well_known.get(port, "Unknown/Custom Service")

    print(f"\n{'='*55}")
    print(f"  Socket Address:    {ip}:{port}")
    print(f"  ─────────────────────────────────────────")
    print(f"  IP Address:        {ip}")
    print(f"    → Size:          32 bits (IPv4)")
    print(f"    → Identifies:    The machine")
    print(f"    → Binary:        {ip_int:032b}")
    print(f"  Port Number:       {port}")
    print(f"    → Size:          16 bits")
    print(f"    → Category:      {category}")
    print(f"    → Service:       {service}")
    print(f"    → Identifies:    The process on the machine")
    print(f"  ─────────────────────────────────────────")
    print(f"  Together (Socket): Uniquely identifies this")
    print(f"                     specific connection endpoint")
    print(f"  Total bits:        32 + 16 = 48 bits")

# Examples from the lecture
examples = [
    ("192.168.1.5",   52341),   # Client ephemeral port
    ("157.240.1.35",  443),     # Facebook HTTPS
    ("192.168.56.101", 21),     # Metasploitable FTP
    ("192.168.56.101", 3306),   # Metasploitable MySQL
    ("127.0.0.1",     3000),    # Your Node.js dev server
    ("127.0.0.1",     5432),    # Your PostgreSQL
]

print("SOCKET ADDRESS ANALYSIS")
for ip, port in examples:
    explain_socket_address(ip, port)
```

### Lab 2 — View All Active Socket Addresses on Your Machine

```bash
# See all currently open socket addresses (listening + established)

echo "=== ALL LISTENING SOCKETS (servers waiting for connections) ==="
sudo ss -tlnp
# Columns: State, Recv-Q, Send-Q, Local Address:Port, Peer Address:Port, Process

echo ""
echo "=== ALL ESTABLISHED TCP CONNECTIONS (active sessions) ==="
sudo ss -tnp state established
# Shows: local socket ↔ remote socket for each active TCP connection

echo ""
echo "=== HUMAN-READABLE SUMMARY ==="
sudo netstat -tlnp 2>/dev/null || sudo ss -tlnp

# Filter for specific services
echo ""
echo "=== YOUR APP PORTS (3000, 5432, 27017, 6379) ==="
sudo ss -tlnp | grep -E "3000|5432|27017|6379|8080|5000"

# Show socket addresses for web browsers (connections to port 80/443)
echo ""
echo "=== YOUR BROWSER CONNECTIONS TO WEB SERVERS ==="
sudo ss -tnp | grep -E ":80|:443"
```

### Lab 3 — Python Socket: Create a Real Socket Address

```python
# Save as socket_demo.py — run: python3 socket_demo.py
# Demonstrates creating actual socket addresses in Python

import socket
import threading
import time

def run_server(host='127.0.0.1', port=9999):
    """Simple TCP server — creates a socket address"""
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
    server.bind((host, port))
    server.listen(5)

    print(f"\nSERVER socket address: {host}:{port}")
    print(f"  IP:   {host} (32-bit address)")
    print(f"  Port: {port} (16-bit port)")
    print(f"  Together: uniquely identifies THIS server endpoint")
    print(f"\nWaiting for connections...")

    conn, addr = server.accept()
    client_socket = f"{addr[0]}:{addr[1]}"
    print(f"\nClient connected!")
    print(f"  Client socket address: {client_socket}")
    print(f"  Full TCP 4-tuple:")
    print(f"    Source: {addr[0]}:{addr[1]}")
    print(f"    Dest:   {host}:{port}")
    print(f"  This 4-tuple UNIQUELY identifies this connection globally.")

    data = conn.recv(1024)
    print(f"\nReceived from client: {data.decode()}")
    conn.send(b"Hello from server!")
    conn.close()
    server.close()

def run_client(host='127.0.0.1', port=9999):
    """Simple TCP client — OS assigns ephemeral port"""
    time.sleep(0.5)  # wait for server
    client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client.connect((host, port))

    # See what ephemeral port OS assigned to this client
    local_ip, local_port = client.getsockname()
    print(f"\nCLIENT socket address: {local_ip}:{local_port}")
    print(f"  OS automatically chose ephemeral port: {local_port}")
    print(f"  Range 49152–65535: {'YES ✓' if local_port >= 49152 else 'NO (system assigned)'}")

    client.send(b"Hello from client!")
    data = client.recv(1024)
    print(f"Received from server: {data.decode()}")
    client.close()

# Run server and client in threads
server_thread = threading.Thread(target=run_server)
client_thread = threading.Thread(target=run_client)

server_thread.start()
client_thread.start()

server_thread.join()
client_thread.join()

print("\nConversation complete!")
print("This is exactly how your MERN/PERN app communicates:")
print("  Your Node.js server: 0.0.0.0:3000 (listens on all IPs, port 3000)")
print("  Browser connects:    127.0.0.1:XXXXX → 127.0.0.1:3000")
```

### Lab 4 — Port Scan to Find All Socket Addresses on Metasploitable2

```bash
# Find all open socket addresses on your lab target

TARGET="192.168.56.101"

echo "=== Scanning all socket addresses on Metasploitable2 ==="
echo "Target: $TARGET"
echo ""

# Quick top-1000 port scan
nmap -sV -T4 $TARGET

echo ""
echo "=== Full port scan (all 65535 ports) ==="
nmap -p- -T4 $TARGET --open

echo ""
echo "=== Check specific dangerous sockets ==="
python3 - << 'EOF'
dangerous_sockets = [
    ("192.168.56.101", 21,   "FTP — vsftpd 2.3.4 backdoor!"),
    ("192.168.56.101", 23,   "Telnet — credentials in plaintext!"),
    ("192.168.56.101", 3306, "MySQL — root with no password!"),
    ("192.168.56.101", 5900, "VNC — weak authentication!"),
    ("192.168.56.101", 6667, "IRC — Unreal IRCd backdoor!"),
    ("192.168.56.101", 5432, "PostgreSQL — check default creds"),
]

import socket
print(f"{'Socket Address':<30} {'Status':<12} {'Risk'}")
print("-"*75)
for ip, port, risk in dangerous_sockets:
    try:
        s = socket.socket()
        s.settimeout(1)
        result = s.connect_ex((ip, port))
        status = "OPEN ⚠️" if result == 0 else "closed"
        s.close()
    except:
        status = "error"
    print(f"{ip+':'+str(port):<30} {status:<12} {risk}")
EOF
```

### Lab 5 — Monitor Socket Addresses in Real Time

```bash
# Watch socket connections open and close in real time
# Useful for observing your MERN/PERN app connections

# Method 1: Watch ss output refresh every second
watch -n 1 'ss -tnp | head -30'

# Method 2: Log all new connections (requires tcpdump)
# Capture only SYN packets (new connection attempts) = new socket addresses forming
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0 and tcp[tcpflags] & tcp-ack == 0' \
  -n -l | awk '{print "New connection:", $3, "→", $5}'

# Method 3: Monitor your Node.js app specifically
# Start your app first, then:
APP_PID=$(pgrep -f "node")
echo "Monitoring connections for Node.js PID: $APP_PID"
sudo ss -tnp | grep "pid=$APP_PID"

# Method 4: Count active connections per destination port
echo "=== Connection count by port ==="
sudo ss -tn | grep ESTAB | awk '{print $5}' | cut -d: -f2 | sort | uniq -c | sort -rn
```

---

## 12. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              SOCKET ADDRESS — EXAM CHEAT SHEET                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  DEFINITION                                                          ║
║  Socket Address = IP Address + Port Number                          ║
║  IPv4: 32-bit IP + 16-bit Port = 48 bits total                     ║
║  Written as: 192.168.1.5:443  (IP:Port)                            ║
║  Purpose: Uniquely identify a specific connection endpoint          ║
╠══════════════════════════════════════════════════════════════════════╣
║  PORT NUMBER RANGES                                                  ║
║  0–1023       Well-Known Ports   (HTTP=80, HTTPS=443, SSH=22)      ║
║  1024–49151   Registered Ports   (MySQL=3306, Node.js=3000)        ║
║  49152–65535  Dynamic/Ephemeral  (OS-assigned to client sessions)  ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY PORT ALONE IS NOT ENOUGH                                        ║
║  Machine M: port P → connects to S1                                ║
║  Machine G: port P → also connects to S1 (same port, different IP!) ║
║  S1 sees two connections both labeled "port P" → AMBIGUOUS ✗       ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY IP ALONE IS NOT ENOUGH                                          ║
║  Machine M (IP1): Process 1 → connects to S1                       ║
║  Machine M (IP1): Process 2 → also connects to S1 (same IP!)       ║
║  S1 sees two connections both labeled "IP1" → AMBIGUOUS ✗          ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHY IP + PORT WORKS                                                 ║
║  Different machines, same port: (IP1:P) ≠ (IP2:P) ✓               ║
║  Same machine, different processes: (IP1:P1) ≠ (IP1:P2) ✓         ║
║  IP + Port = always unique = Socket Address ✓                      ║
╠══════════════════════════════════════════════════════════════════════╣
║  TCP CONNECTION = SOCKET PAIR (4-tuple)                             ║
║  (Src IP : Src Port , Dst IP : Dst Port)                           ║
║  Example: (192.168.1.5:52341, 157.240.1.35:443)                   ║
║  This 4-tuple uniquely identifies the COMPLETE connection           ║
╠══════════════════════════════════════════════════════════════════════╣
║  IPv4 vs IPv6 SOCKET NOTATION                                        ║
║  IPv4: 192.168.1.5:443                                             ║
║  IPv6: [2001:db8::1]:443  (brackets needed around IPv6 address)    ║
╠══════════════════════════════════════════════════════════════════════╣
║  ADDRESS COMPARISON                                                  ║
║  MAC   → identifies hardware NIC (Data Link Layer)                 ║
║  IP    → identifies machine on network (Network Layer)             ║
║  Port  → identifies process on machine (Transport Layer)           ║
║  Socket → identifies connection endpoint (IP + Port)               ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY NOTES                                                      ║
║  Open socket = open attack surface → close unused ports            ║
║  Port scan reveals all socket addresses: nmap -sV <target>         ║
║  TCP RST injection: forged socket address kills connection          ║
║  Ephemeral ports randomized by modern OS to prevent prediction      ║
║  Check your sockets: ss -tlnp or netstat -tlnp                     ║
╠══════════════════════════════════════════════════════════════════════╣
║  YOUR MERN/PERN APP SOCKETS                                          ║
║  Node.js backend:  0.0.0.0:3000  or  127.0.0.1:3000               ║
║  React frontend:   127.0.0.1:5173 (Vite) or :3000                 ║
║  PostgreSQL:       127.0.0.1:5432                                  ║
║  MongoDB:          127.0.0.1:27017                                 ║
║  Close all to outside world in production using firewall rules!    ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] TCP — Three-way handshake, connection establishment using socket addresses
- [ ] UDP — Connectionless — socket address still used but no handshake
- [ ] DNS — How domain names map to IP addresses (completing the socket)
- [ ] NAT — How private socket addresses are translated for internet access
- [ ] Multiplexing & Demultiplexing — How ports enable multi-process communication

---

_Notes compiled from: Networking Course Lecture 59 — What is Socket Address?_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student — Bangladesh University_
