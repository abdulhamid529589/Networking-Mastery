# 🔐 IPsec — Transport Mode vs Tunnel Mode

### Cybersecurity Student Notes | Networking Course — Network Layer Security

> **Source:** Gate Smashers — IPsec Transport Mode and Tunnel Mode
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Security+ · CEH
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [Quick Recap — What is IPsec?](#1-quick-recap--what-is-ipsec)
2. [The Journey of a Packet Through Network Layers](#2-the-journey-of-a-packet-through-network-layers)
3. [Transport Mode — Deep Dive](#3-transport-mode--deep-dive)
   - [3.1 What Transport Mode Does Step by Step](#31-what-transport-mode-does-step-by-step)
   - [3.2 Transport Mode Packet Structure](#32-transport-mode-packet-structure)
   - [3.3 Transport Mode — How Routers See It](#33-transport-mode--how-routers-see-it)
   - [3.4 When to Use Transport Mode](#34-when-to-use-transport-mode)
4. [Tunnel Mode — Deep Dive](#4-tunnel-mode--deep-dive)
   - [4.1 What Tunnel Mode Does Step by Step](#41-what-tunnel-mode-does-step-by-step)
   - [4.2 Tunnel Mode Packet Structure](#42-tunnel-mode-packet-structure)
   - [4.3 Why It Is Called "Tunnel" Mode](#43-why-it-is-called-tunnel-mode)
   - [4.4 When to Use Tunnel Mode](#44-when-to-use-tunnel-mode)
5. [Transport vs Tunnel — Full Comparison](#5-transport-vs-tunnel--full-comparison)
   - [5.1 Side-by-Side Packet Diagram](#51-side-by-side-packet-diagram)
   - [5.2 Comparison Table](#52-comparison-table)
6. [How Routers Handle Each Mode](#6-how-routers-handle-each-mode)
7. [Security Analysis — What Each Mode Hides](#7-security-analysis--what-each-mode-hides)
   - [7.1 Transport Mode Security Properties](#71-transport-mode-security-properties)
   - [7.2 Tunnel Mode Security Properties](#72-tunnel-mode-security-properties)
8. [Attacking Each Mode — Offensive Perspective](#8-attacking-each-mode--offensive-perspective)
   - [8.1 Attacks Against Transport Mode](#81-attacks-against-transport-mode)
   - [8.2 Attacks Against Tunnel Mode](#82-attacks-against-tunnel-mode)
9. [🧪 Practical Labs](#9--practical-labs)
   - [Lab 1 — Capture and Compare Both Modes in Wireshark](#lab-1--capture-and-compare-both-modes-in-wireshark)
   - [Lab 2 — Transport Mode Setup with strongSwan](#lab-2--transport-mode-setup-with-strongswan)
   - [Lab 3 — Tunnel Mode VPN Setup with strongSwan](#lab-3--tunnel-mode-vpn-setup-with-strongswan)
   - [Lab 4 — Simulate Both Modes with Scapy](#lab-4--simulate-both-modes-with-scapy)
   - [Lab 5 — Test Your Own Web Projects Behind IPsec Tunnel](#lab-5--test-your-own-web-projects-behind-ipsec-tunnel)
10. [Solved Examples](#10-solved-examples)
11. [Exam Cheat Sheet](#11-exam-cheat-sheet)

---

## 1. Quick Recap — What is IPsec?

Before diving into the two modes, recall the foundation:

```
IPsec = IP Security Protocol Suite

  Layer:    Network Layer (Layer 3) — same layer as IPv4, IPv6, ICMP
  Purpose:  Security for IP packets
  Type:     Suite / collection of protocols (NOT a single protocol)
  Defined:  IETF (Internet Engineering Task Force)

Three core protocols in the suite:
  ESP (50)  → Encryption + Authentication + Integrity
  AH  (51)  → Authentication + Integrity ONLY (no encryption)
  IKE       → Key Exchange (before data flows) — UDP port 500

Two security goals that drive the two modes:
  1. Confidentiality  → encrypt the data (ESP)
  2. Integrity        → verify data not changed + verify sender (AH or ESP HMAC)

IPsec can be used in TWO MODES:
  Mode 1 → TRANSPORT MODE
  Mode 2 → TUNNEL MODE
```

> **Why two modes exist:** The fundamental question is _how much_ of the original IP packet you want to protect — just the payload data, or the entire original packet including its IP header. The answer to this question determines which mode you use.

---

## 2. The Journey of a Packet Through Network Layers

To understand both modes, you must first understand how a packet is built going **down** the OSI layers. This is what happens BEFORE IPsec touches the packet:

```
OSI Model — Packet Building (Encapsulation going DOWN):

  Layer 7 — Application     [User Data]
                              ↓ passes down
  Layer 6 — Presentation    [User Data]
                              ↓ passes down
  Layer 5 — Session         [User Data]
                              ↓ passes down
  Layer 4 — Transport       [TCP/UDP Header | User Data]
                              ↓ passes down to Network Layer
  Layer 3 — Network         ← THIS IS WHERE IPsec OPERATES
```

```
What arrives at the Network Layer (Layer 3):

  From the Transport Layer above:
  ┌──────────────────────────────────────────────┐
  │  TCP or UDP Header  │  Application Data       │
  └──────────────────────────────────────────────┘
        ↑
        This entire block is called the "payload" from the Network Layer's perspective.
        Whether TCP or UDP — the Network Layer doesn't care.
        It just sees: "data that came from above".

The Network Layer's normal job (WITHOUT IPsec):
  → Add an IPv4 or IPv6 header to the front
  → The header tells the network: where to send it, how to route it
  → Result: a complete IP datagram/packet, ready to be sent

  ┌──────────────────────────────────────────────────────┐
  │  IPv4/IPv6 Header  │  TCP/UDP Header  │  App Data    │
  └──────────────────────────────────────────────────────┘
```

**IPsec steps in at the Network Layer** — BEFORE the IPv4/IPv6 header is added — and does additional processing depending on which mode is being used.

---

## 3. Transport Mode — Deep Dive

### 3.1 What Transport Mode Does Step by Step

```
TRANSPORT MODE — Step-by-Step Processing:

STEP 1: Packet arrives at Network Layer from Transport Layer above
  ┌────────────────────────────────────┐
  │  TCP/UDP Header  │  App Data       │
  └────────────────────────────────────┘
  (This is what came DOWN from Layer 4)

STEP 2: IPsec header and trailer are added — wrapping the payload
  ┌────────────────────────────────────────────────────────────────┐
  │  IPsec Header  │  TCP/UDP Header  │  App Data  │  IPsec Trailer│
  └────────────────────────────────────────────────────────────────┘
                   ↑──────────────────────────────↑
                   This section is ENCRYPTED (if using ESP)
                   and/or HASHED (for integrity with AH or ESP)

  Why add IPsec here?
    → To achieve CONFIDENTIALITY: encrypt the data payload
    → To achieve INTEGRITY: compute HMAC over the data
    → The payload is now protected

STEP 3: The packet is NOT yet complete — no IPv4/IPv6 header yet!
  At this point the packet cannot be routed anywhere.
  It still needs an IP header.

STEP 4: IPv4 or IPv6 header is added in front
  ┌──────────────────────────────────────────────────────────────────────────────┐
  │  IPv4/IPv6 Header  │  IPsec Header  │  Enc(TCP/UDP Header + App Data)  │ ... │
  └──────────────────────────────────────────────────────────────────────────────┘
  ↑──────────────────↑
  Added LAST — this is the ORIGINAL header (real source & dest IP addresses)
  This is what gets the packet from A to B across the network

STEP 5: Packet is sent down to Layer 2 (Data Link) and transmitted
```

```
Key insight of Transport Mode:

  The original IP header (with REAL source and destination addresses)
  is added OUTSIDE the IPsec protection zone.

  Routers on the path can SEE:
    → Source IP address: the actual sending host
    → Destination IP address: the actual receiving host
    → That IPsec/ESP is in use (Protocol field = 50)

  Routers CANNOT see:
    → TCP/UDP port numbers (hidden inside ESP encryption)
    → Application data (encrypted)
    → Whether it's HTTP, FTP, DNS — unidentifiable inside the ESP payload
```

---

### 3.2 Transport Mode Packet Structure

```
Transport Mode with ESP (most common):

  BEFORE IPsec (original packet structure):
  ┌─────────────────┬──────────────────────────────────────────┐
  │  IPv4/6 Header  │  TCP/UDP Header  │  Application Data      │
  └─────────────────┴──────────────────────────────────────────┘

  AFTER Transport Mode with ESP:
  ┌─────────────────┬─────────────┬══════════════════════════════════════════╗═══════════════┐
  │  IPv4/6 Header  │  ESP Header │ Encrypted(TCP/UDP Header + App Data)     ║  ESP Trailer  │
  │  (UNCHANGED)    │  SPI + Seq  │                                          ║  + ESP Auth   │
  └─────────────────┴─────────────┴══════════════════════════════════════════╝═══════════════┘
  ↑─────────────────↑             ↑───────────────────────────────────────────────────────────↑
  Visible to routers               This entire section is ENCRYPTED — nobody on the
  (real source & dest IP)          path can read what's inside

  AFTER Transport Mode with AH (no encryption):
  ┌─────────────────┬───────────────┬────────────────────────────────────────────┐
  │  IPv4/6 Header  │   AH Header   │  TCP/UDP Header  │  Application Data       │
  │  (UNCHANGED)    │  SPI+Seq+ICV  │  (VISIBLE!)      │  (VISIBLE!)             │
  └─────────────────┴───────────────┴────────────────────────────────────────────┘
  ↑──────────────────────────────────────────────────────────────────────────────↑
  Everything is VISIBLE — AH only authenticates, it does NOT encrypt
  ↑─────────────────────────────────────────────────────────────────────────────↑
  The AH ICV (HMAC) covers most of this — modification detected but content visible

What the numbers look like in the IPv4 header for each case:
  Transport Mode with ESP: Protocol field in IPv4 header = 50
  Transport Mode with AH:  Protocol field in IPv4 header = 51
```

```
What's inside the ESP Header (Transport Mode):

  ┌──────────────────────────────────────────────────────────┐
  │           SPI — Security Parameter Index (32 bits)        │
  ├──────────────────────────────────────────────────────────┤
  │                  Sequence Number (32 bits)                 │
  ├══════════════════════════════════════════════════════════╡
  │              ENCRYPTED PAYLOAD STARTS HERE               │
  │  TCP/UDP Header (hidden inside encryption)               │
  │  Application Data (HTTP, FTP, etc. — completely hidden)  │
  │              ENCRYPTED PAYLOAD ENDS HERE                 │
  ├──────────────────────────────────────────────────────────┤
  │  Padding (0–255 bytes) │ Pad Length │ Next Header (=6/17)│
  ├──────────────────────────────────────────────────────────┤
  │           ICV — Integrity Check Value (HMAC)             │
  └──────────────────────────────────────────────────────────┘

  SPI:         Tells receiver which key/algorithm to use
  Seq Number:  Anti-replay counter — every packet increments this
  Next Header: What's inside (6=TCP, 17=UDP) — only known after decryption
  ICV:         The HMAC — receiver recomputes and compares to detect tampering
```

---

### 3.3 Transport Mode — How Routers See It

```
Path diagram — Transport Mode:

  Host A ───────────────────[INTERNET]───────────────────→ Host B
  (10.0.1.5)         Router 1      Router 2               (10.0.2.8)

  Packet on the wire:
  ┌──────────────────────────────────────────────────────────────┐
  │  IPv4: src=10.0.1.5, dst=10.0.2.8  │  ESP  │  [ENCRYPTED]  │
  └──────────────────────────────────────────────────────────────┘

  What Router 1 sees:
    → Source IP:      10.0.1.5   ← VISIBLE (real host address)
    → Destination IP: 10.0.2.8   ← VISIBLE (real host address)
    → Protocol:       50 (ESP)   ← VISIBLE (knows it's IPsec)
    → Payload:        ENCRYPTED  ← CANNOT read

  What Router 1 does:
    → Routes based on Destination IP = 10.0.2.8
    → Doesn't need to know what's inside
    → Passes it to Router 2, then Router 2 to Host B

  What an ATTACKER on the network sees:
    → "10.0.1.5 is sending encrypted traffic to 10.0.2.8"
    → Attack surface: knows WHICH two hosts are communicating
    → Cannot read WHAT they are communicating
    → Can do traffic analysis: timing, volume, frequency

  PROBLEM: The actual host identities are EXPOSED in Transport Mode.
```

---

### 3.4 When to Use Transport Mode

```
Transport Mode is appropriate when:

  ✅ HOST-TO-HOST direct communication:
     Both endpoints are the actual communicating parties
     (NOT going through a VPN gateway)
     Example: Server A talks directly to Server B

  ✅ INTERNAL NETWORKS (trusted environment):
     Both hosts are in the same organisation / data centre
     The "who is talking to who" information is not sensitive
     You just want to protect the CONTENT of the communication

  ✅ LOW OVERHEAD IS PRIORITY:
     Only one IP header — smaller packet size
     Less processing required

  ✅ BOTH ENDS SUPPORT IPsec NATIVELY:
     Each host runs its own IPsec implementation
     No VPN gateways involved

  ❌ NOT suitable when:
     → You want to hide WHICH hosts are communicating
     → Traffic crosses the public internet and attacker shouldn't know
       the real source/destination IPs
     → You use VPN gateways (gateway doesn't know which internal host
       the packet came from with Transport Mode)
     → NAT is in the path (especially AH breaks with NAT)

Common real-world use cases:
  → Protecting DB server to app server communication in a data centre
  → Securing admin/management traffic between known internal hosts
  → Remote desktop or SSH replacement with IPsec-level protection
  → Host-based IPsec enforcement on internal subnets
```

---

## 4. Tunnel Mode — Deep Dive

### 4.1 What Tunnel Mode Does Step by Step

```
TUNNEL MODE — Step-by-Step Processing:

STEP 1: Packet arrives at Network Layer from Transport Layer above
  ┌────────────────────────────────────┐
  │  TCP/UDP Header  │  App Data       │
  └────────────────────────────────────┘

STEP 2: Network Layer FIRST adds the original IPv4/IPv6 header
  This step is DIFFERENT from Transport Mode!
  In Transport Mode, the IP header was added LAST (after IPsec).
  In Tunnel Mode, the IP header is added FIRST — before IPsec.

  ┌──────────────────────────────────────────────────────────┐
  │  IPv4/IPv6 Header  │  TCP/UDP Header  │  App Data        │
  └──────────────────────────────────────────────────────────┘
  ↑──────────────────↑
  ORIGINAL IP header — contains real source/destination IPs
  This is now a complete IP packet (IP datagram)

STEP 3: This ENTIRE IP packet (header + data) is treated as ONE BLOCK
  and encrypted/authenticated by IPsec.
  IPsec header and trailer are added around the WHOLE original packet.

  ┌─────────────────────────────────────────────────────────────────────┐
  │  IPsec Header  │  Encrypted(IPv4/IPv6 Header + TCP/UDP Hdr + Data)  │  IPsec Trailer │
  └─────────────────────────────────────────────────────────────────────┘
  ↑──────────────────────────────────────────────────────────────────────────────────────↑
  The ENTIRE original IP packet — including its header — is now HIDDEN inside

STEP 4: But now there's a problem!
  How will routers on the internet know where to send this packet?
  The only IP header that existed (the real one) is now ENCRYPTED inside.
  Routers cannot see inside ESP encryption.

STEP 5: A NEW outer IPv4/IPv6 header is added
  This new header contains:
  → Source IP:      VPN Gateway A's public IP (NOT the original host's IP)
  → Destination IP: VPN Gateway B's public IP (NOT the real destination host's IP)

  ┌────────────────────┬──────────────────────────────────────────────────────────────────────────────┐
  │ NEW Outer IP Hdr   │  IPsec Header  │  Encrypted(Original IP Header + TCP/UDP + App Data)  │ ...  │
  │  GW-A → GW-B       │  SPI + Seq     │                                                      │      │
  └────────────────────┴──────────────────────────────────────────────────────────────────────────────┘
  ↑──────────────────↑                   ↑──────────────────────────────────────────────────────────↑
  Routers use THIS                       EVERYTHING original is hidden in here — unreadable
  to route the packet

STEP 6: Packet is sent. It travels from Gateway A to Gateway B via the outer header.
STEP 7: At Gateway B, the outer header is stripped, IPsec decrypts the payload,
         the original IP packet is recovered and forwarded to the real destination host.
```

---

### 4.2 Tunnel Mode Packet Structure

```
Tunnel Mode with ESP — Full Structure:

  ORIGINAL PACKET (before any IPsec):
  ┌───────────────────┬─────────────────────────────────────────┐
  │  Original IP Hdr  │  TCP/UDP Header  │  Application Data    │
  │  src=10.0.1.5     │                  │  (HTTP, FTP, etc.)   │
  │  dst=10.0.2.8     │                  │                      │
  └───────────────────┴─────────────────────────────────────────┘

  AFTER Tunnel Mode with ESP:
  ┌──────────────────┬─────────────┬══════════════════════════════════════════════════════════════════╗══════════════════┐
  │  NEW Outer IP    │  ESP Header │ Encrypted(Original IP Hdr + TCP/UDP Hdr + App Data)             ║ ESP Trailer+Auth │
  │  src=GW-A        │  SPI + Seq  │ [10.0.1.5 → 10.0.2.8 │ TCP │ HTTP data — ALL hidden inside]   ║                  │
  │  dst=GW-B        │             │                                                                   ║                  │
  └──────────────────┴─────────────┴══════════════════════════════════════════════════════════════════╝══════════════════┘
  ↑────────────────↑               ↑─────────────────────────────────────────────────────────────────────────────────────↑
  Only this is          EVERYTHING original is inside here:
  visible to routers    ✅ Original IP header (real IPs) → HIDDEN
                        ✅ TCP/UDP header (port numbers) → HIDDEN
                        ✅ Application data               → HIDDEN
                        ✅ Internal network addresses     → HIDDEN

Two IP headers in Tunnel Mode:
  OUTER IP Header:   src = VPN Gateway A public IP, dst = VPN Gateway B public IP
  INNER IP Header:   src = Real source host IP,     dst = Real destination host IP
                     (completely hidden inside ESP encryption)

Protocol numbers:
  Outer IP header Protocol field = 50 (ESP)
  After ESP decryption, the inner packet's Protocol field = 6 (TCP) or 17 (UDP)
```

```
Tunnel Mode with AH (less common):

  ┌──────────────────┬──────────────────┬───────────────────┬───────────────────────────────────┐
  │  NEW Outer IP    │   AH Header      │  Original IP Hdr  │  TCP/UDP Header  │  App Data      │
  │  src=GW-A        │  SPI+Seq+ICV     │  src=10.0.1.5     │  (VISIBLE)       │  (VISIBLE)     │
  │  dst=GW-B        │                  │  dst=10.0.2.8     │                  │                │
  └──────────────────┴──────────────────┴───────────────────┴───────────────────────────────────┘
  ↑──────────────────────────────────────────────────────────────────────────────────────────────↑
  AH does NOT encrypt → everything is still visible
  BUT: AH ICV (HMAC) covers everything → tampering is detectable
  The original IP header IS inside the AH protection zone here

  Note: AH Tunnel Mode is rarely used in practice because:
  → AH still doesn't encrypt — content visible even though it's "tunnelled"
  → AH breaks with NAT (HMAC covers IP header fields, NAT changes them)
  → ESP Tunnel Mode gives you everything AH does PLUS encryption
```

---

### 4.3 Why It Is Called "Tunnel" Mode

```
The "Tunnel" analogy — why this name makes sense:

  Imagine digging a tunnel under a mountain:

  SURFACE ROAD (visible path):
    → Cars travel on the surface road
    → You can see every car: what type it is, where it came from, where it's going
    → Transport Mode = surface road — everything visible at IP layer

  TUNNEL (hidden path):
    → You enter the tunnel at Tunnel Entrance A (Gateway A)
    → While inside the tunnel: nobody can see you
    → What car you are, where you really came from, your actual destination
      — all HIDDEN while in the tunnel
    → You exit at Tunnel Exit B (Gateway B)
    → Only THEN does your actual destination become relevant again

  IPsec Tunnel Mode works exactly like this:
    → Gateway A = Tunnel Entrance (encapsulates the packet, hides the original)
    → Internet  = the dark tunnel (attackers see only outer header — Gateway to Gateway)
    → Gateway B = Tunnel Exit (decapsulates, recovers original packet, delivers it)

The original packet is hidden INSIDE the tunnel packet.
The outer packet carries the tunnel endpoints (gateways).
Only at the destination gateway is the original packet revealed.
```

```
Path diagram — Tunnel Mode:

  Internal          VPN                         VPN              Internal
  Host A            Gateway A     INTERNET       Gateway B        Host B
  (10.0.1.5)        (203.0.0.1)                 (198.51.0.1)     (10.0.2.8)

  ──→ Normal packet ──→
  [10.0.1.5→10.0.2.8|TCP|HTTP]
                     │
                     ↓ Gateway A encapsulates
  ┌─────────────────────────────────────────────────────────────────────┐
  │  Outer: 203.0.0.1→198.51.0.1  │  ESP  │  Enc[10.0.1.5→10.0.2.8   │
  │                                │       │  TCP│HTTP data]            │
  └─────────────────────────────────────────────────────────────────────┘
                     ──────────────────────────────→ (over internet)
                                                    │
                                                    ↓ Gateway B decapsulates
                                         [10.0.1.5→10.0.2.8|TCP|HTTP]
                                                    ──→ to Host B →

  What ANYONE on the internet sees between the gateways:
    Source IP:      203.0.0.1 (Gateway A)  ← NOT Host A's real IP
    Destination IP: 198.51.0.1 (Gateway B) ← NOT Host B's real IP
    Protocol:       ESP (50) — encrypted
    Content:        Encrypted — unreadable

  What they CANNOT see:
    → That 10.0.1.5 is talking to 10.0.2.8
    → What those internal IPs even are (10.x.x.x — could be anything)
    → What data is being transferred (HTTP, database queries, files — all hidden)
    → Even how many internal hosts are actively communicating
       (multiple internal hosts share the same gateway-to-gateway tunnel)
```

---

### 4.4 When to Use Tunnel Mode

```
Tunnel Mode is appropriate when:

  ✅ VPN (Virtual Private Network):
     The PRIMARY use case for Tunnel Mode.
     The whole point of a VPN is to create a private tunnel through the internet.
     Tunnel Mode IS the VPN tunnelling mechanism.

  ✅ SITE-TO-SITE VPN (Gateway to Gateway):
     Office A has VPN Gateway A.
     Office B has VPN Gateway B.
     All traffic between the two offices travels through the Tunnel Mode IPsec tunnel.
     Internal hosts don't need IPsec installed — the gateway handles it.

  ✅ REMOTE ACCESS VPN (Host to Gateway):
     A remote user's laptop runs IPsec client.
     Corporate VPN gateway is the other end.
     Tunnel Mode hides the user's real IP and protects all traffic.

  ✅ HIDING INTERNAL NETWORK TOPOLOGY:
     Internal IPs (RFC 1918: 10.x.x.x, 192.168.x.x, 172.16.x.x) should
     never be visible on the public internet.
     Tunnel Mode hides them entirely — attacker only sees gateway IPs.

  ✅ WHEN HOSTS DON'T SUPPORT IPsec:
     The VPN gateways handle everything.
     Old printers, IoT devices, legacy servers — they get IPsec protection
     automatically through the gateway without being modified.

  ✅ TRAVERSING NAT:
     Tunnel Mode outer IP header is brand new — NAT works fine on it.
     (AH + NAT = broken. ESP Tunnel Mode + NAT = works.)

  ❌ NOT ideal when:
     → Overhead must be minimal (two IP headers = more bytes per packet)
     → Direct host-to-host is preferred and gateways add unnecessary complexity
```

---

## 5. Transport vs Tunnel — Full Comparison

### 5.1 Side-by-Side Packet Diagram

```
TRANSPORT MODE — what the packet looks like on the wire:

  ┌──────────────────────┬──────────────┬═════════════════════════════════════╗═══════════════┐
  │  IPv4/IPv6 Header    │  ESP Header  │  Encrypted(TCP/UDP Header + Data)  ║  ESP Trailer  │
  │  src = real Host A   │  SPI + Seq   │  (ports, app data — all hidden)    ║  + Auth       │
  │  dst = real Host B   │              │                                     ║               │
  └──────────────────────┴──────────────┴═════════════════════════════════════╝═══════════════┘
  ↑─────────────────────↑
  VISIBLE to everyone on the path (routers, attackers, ISPs)

  One IP header: the original (real addresses)
  Protected zone: payload only
  Exposed: who is talking (source IP, destination IP)


TUNNEL MODE — what the packet looks like on the wire:

  ┌──────────────────────┬──────────────┬══════════════════════════════════════════════════════════════╗═══════════════┐
  │  NEW Outer IP Header │  ESP Header  │  Encrypted(Original IP Hdr + TCP/UDP Hdr + App Data)        ║  ESP Trailer  │
  │  src = VPN Gateway A │  SPI + Seq   │  src=Host A | dst=Host B | ports | data — ALL hidden        ║  + Auth       │
  │  dst = VPN Gateway B │              │                                                               ║               │
  └──────────────────────┴──────────────┴══════════════════════════════════════════════════════════════╝═══════════════┘
  ↑─────────────────────↑
  VISIBLE: only gateway addresses — NOT real host addresses

  Two IP headers: outer (gateways, visible) + inner (real hosts, hidden)
  Protected zone: entire original packet
  Exposed: only gateway-to-gateway communication exists — real hosts invisible
```

---

### 5.2 Comparison Table

| Property                        | Transport Mode                            | Tunnel Mode                                  |
| ------------------------------- | ----------------------------------------- | -------------------------------------------- |
| **What is protected**           | Payload only (TCP/UDP + data)             | Entire original IP packet (header + payload) |
| **Original IP header**          | ✅ Kept outside — visible to routers      | ✅ Encrypted and hidden inside ESP           |
| **New outer IP header**         | ❌ No — only one IP header                | ✅ Yes — new header with gateway IPs         |
| **Total IP headers in packet**  | 1 (the original)                          | 2 (outer = gateways, inner = real hosts)     |
| **Real source IP visible**      | ✅ YES — anyone can see Host A's IP       | ❌ NO — only Gateway A IP is visible         |
| **Real destination IP visible** | ✅ YES — anyone can see Host B's IP       | ❌ NO — only Gateway B IP is visible         |
| **TCP/UDP ports visible**       | ❌ Hidden inside ESP encryption           | ❌ Hidden inside ESP encryption              |
| **Application data visible**    | ❌ Encrypted by ESP                       | ❌ Encrypted by ESP                          |
| **Hides internal topology**     | ❌ NO — internal IPs exposed              | ✅ YES — internal addressing invisible       |
| **Packet overhead**             | Small (one extra IPsec header)            | Larger (two IP headers + IPsec headers)      |
| **Works through NAT**           | ❌ Difficult (AH breaks; ESP needs NAT-T) | ✅ Yes — outer IP header handles NAT         |
| **IPsec terminated at**         | The actual communicating hosts            | VPN gateways (on behalf of internal hosts)   |
| **Hosts need IPsec software**   | ✅ YES — each host runs IPsec             | ❌ NO — gateways handle it transparently     |
| **Traffic analysis risk**       | HIGH — attacker knows who talks to who    | LOW — attacker only sees gateway addresses   |
| **Primary use case**            | Host-to-host, data centre internal links  | VPN, site-to-site, remote access             |
| **Security level**              | Good (payload protected)                  | Better (entire original packet protected)    |
| **Typical users**               | System admins securing specific links     | VPN users, enterprise networking teams       |

```
Quick memory trick:

  TRANSPORT MODE:
  Think: "I (the host) TRANSPORT my data directly to you (the other host)"
  → Direct connection, payload protected, IPs visible

  TUNNEL MODE:
  Think: "My original packet goes INTO a TUNNEL at Gateway A,
          travels hidden, comes OUT at Gateway B"
  → Gateway to gateway, everything hidden inside, like a real tunnel
```

---

## 6. How Routers Handle Each Mode

```
The router's perspective — what it knows and what it does:

TRANSPORT MODE — Router's view:

  Packet arrives at Router:
  ┌────────────────────────────────┬────────────────────────────────────────┐
  │  IPv4: src=10.0.1.5, dst=10.0.2.8, Proto=50 (ESP)  │  ESP  │  [ENC]   │
  └──────────────────────────────────────────────────────┴────────┴──────────┘

  Router asks: "Where does this go?"
  Router reads: Destination IP = 10.0.2.8
  Router does:  Looks up 10.0.2.8 in routing table → forwards to next hop
  Router knows: The ACTUAL destination is 10.0.2.8 (Host B directly)
  Router does NOT: Inspect the payload (it's encrypted anyway)
  Role of router: Just ROUTE — pass the packet along. IPsec is transparent to it.


TUNNEL MODE — Router's view:

  Packet arrives at Router:
  ┌──────────────────────────────────────────────────────────┬────────┬──────────┐
  │  IPv4: src=203.0.0.1(GW-A), dst=198.51.0.1(GW-B), Proto=50│ ESP  │  [ENC]   │
  └──────────────────────────────────────────────────────────┴────────┴──────────┘

  Router asks: "Where does this go?"
  Router reads: Destination IP = 198.51.0.1 (Gateway B's public IP)
  Router does:  Routes toward 198.51.0.1 (the gateway, NOT the actual host)
  Router knows: Nothing about the real destination (10.0.2.8 is invisible)
  Router does NOT: Know that 10.0.1.5 and 10.0.2.8 are even communicating
  Role of router: Route to Gateway B. Let Gateway B handle decapsulation.


GATEWAY B — what happens when the tunnel packet arrives:

  Gateway B receives:
  ┌──────────────────────────────┬────────┬════════════════════════════════════════╗═══════┐
  │  Outer IP: GW-A → GW-B       │  ESP   │  Encrypted(Original IP Hdr + Data)    ║  ...  │
  └──────────────────────────────┴────────┴════════════════════════════════════════╝═══════┘

  Gateway B does:
    Step 1: See outer IP dst = 198.51.0.1 = "that's me!" → process it
    Step 2: Read SPI from ESP header → look up SA in SAD
    Step 3: Verify ICV (HMAC) → confirm packet integrity ✅
    Step 4: Check Sequence Number → not a replay ✅
    Step 5: Decrypt ESP payload using AES-256-GCM (or whatever algorithm)
    Step 6: Recover the ORIGINAL IP packet: [10.0.1.5→10.0.2.8|TCP|HTTP data]
    Step 7: Forward this ORIGINAL packet to 10.0.2.8 (Host B) on the internal network
    Step 8: Host B receives it — completely unaware IPsec was involved!
```

---

## 7. Security Analysis — What Each Mode Hides

### 7.1 Transport Mode Security Properties

```
What an attacker CAN learn from intercepted Transport Mode packets:

  ✅ (they CAN see this):
    → Source IP:        10.0.1.5   — the REAL Host A
    → Destination IP:   10.0.2.8   — the REAL Host B
    → Protocol:         ESP (50) or AH (51)
    → Packet sizes:     can infer application type from size patterns
    → Timing patterns:  when communication happens, how frequently
    → SPI value:        identifies the security association
    → Sequence numbers: how many packets have been sent

  ❌ (they CANNOT see this):
    → TCP/UDP port numbers (hidden inside ESP)
    → Application data (HTTP request, FTP upload, SQL query — all hidden)
    → What application is running (might be inferred from packet sizes)

What this means for an attacker:
  → "10.0.1.5 is communicating with 10.0.2.8" — KNOWN
  → "When? How much? How often?" — KNOWN
  → "What are they saying?" — UNKNOWN (encrypted)

  This is called TRAFFIC ANALYSIS — even without decryption,
  knowing WHO talks to WHO and HOW OFTEN can reveal a lot.
  Example: if 10.0.1.5 suddenly starts sending large amounts of traffic
  to 10.0.2.8 at 2am every night — suspicious, even without seeing content.
```

### 7.2 Tunnel Mode Security Properties

```
What an attacker CAN learn from intercepted Tunnel Mode packets:

  ✅ (they CAN see this):
    → Outer Source IP:      203.0.0.1  — Gateway A's public IP
    → Outer Destination IP: 198.51.0.1 — Gateway B's public IP
    → Protocol:             ESP (50)
    → Approximate packet sizes and timing (outer header visible)
    → SPI value

  ❌ (they CANNOT see this):
    → Real Host A IP (10.0.1.5) — HIDDEN inside ESP encryption
    → Real Host B IP (10.0.2.8) — HIDDEN inside ESP encryption
    → TCP/UDP port numbers         — HIDDEN
    → Application data             — HIDDEN
    → How many INTERNAL hosts are communicating
      (multiple hosts share the same tunnel — attacker sees only Gateway A ↔ Gateway B)
    → Internal network addressing  (10.0.x.x, 192.168.x.x — completely invisible)

What this means for an attacker:
  → "Someone at Gateway A (203.0.0.1) is communicating with Gateway B (198.51.0.1)"
  → "Could be 1 host or 1000 hosts behind those gateways — I don't know"
  → "What are they sending? Unknown. Who specifically? Unknown."
  → Traffic analysis is much harder — cannot link communication to specific hosts

This is why Tunnel Mode provides significantly STRONGER traffic privacy.
```

---

## 8. Attacking Each Mode — Offensive Perspective

> **Note:** All techniques below are for educational use only — test exclusively on systems you own or have explicit written permission to test (your own lab, Metasploitable2, your own MERN/PERN web projects).

### 8.1 Attacks Against Transport Mode

```bash
# ── Attack 1: Traffic Analysis in Transport Mode ────────────────────────────
# Even though data is encrypted, source/dest IPs are visible
# Goal: identify which internal hosts are communicating and build a map

# Capture Transport Mode ESP traffic:
sudo tcpdump -i eth0 -n "esp" -w /tmp/transport_capture.pcap

# Analyse in Wireshark — filter: esp
# You will see real source and destination IPs in every packet
# Build a communication graph: which hosts talk to which, when, how much

# Extract unique communication pairs:
tshark -r /tmp/transport_capture.pcap -T fields -e ip.src -e ip.dst -q | sort | uniq -c | sort -rn
# Output example:
#   47  10.0.1.5  10.0.2.8     ← these two hosts communicate a lot
#    3  10.0.1.9  10.0.2.8     ← occasional communication
# This alone reveals the network topology without decryption

# ── Attack 2: Timing Analysis Against Transport Mode ────────────────────────
# Analyse WHEN packets are sent to infer application behavior

# Extract timestamps and sizes:
tshark -r /tmp/transport_capture.pcap -T fields \
  -e frame.time_relative \
  -e ip.src \
  -e ip.dst \
  -e frame.len \
  -q > /tmp/timing_analysis.txt

# Look for:
# - Regular intervals → periodic sync, heartbeat, or backup process
# - Bursts → user activity (someone logged in and started working)
# - Large transfers at night → backup/exfiltration?
# Even without decryption, timing reveals a lot.

# ── Attack 3: Replay Attack Simulation Against Misconfigured IPsec ──────────
# IPsec replay protection depends on sequence numbers being checked
# A MISCONFIGURED implementation might not check sequence numbers properly

# Capture a legitimate Transport Mode packet:
sudo tcpdump -i eth0 -n "esp" -c 1 -w /tmp/single_packet.pcap

# Replay it with tcpreplay (against your own test machine):
sudo tcpreplay --intf1=eth0 /tmp/single_packet.pcap

# A correctly configured IPsec endpoint should:
# → Reject it: "Sequence number already seen — replay detected"
# Check logs:
sudo journalctl -u strongswan | grep -i "replay\|duplicate\|sequence"

# ── Attack 4: NAT-Based AH Bypass ───────────────────────────────────────────
# If Transport Mode uses AH (not ESP), and there's NAT in the path:
# NAT modifies IP header fields → AH HMAC covers those fields → HMAC fails
# This can cause a denial-of-service: AH packets rejected due to NAT modification

# Test: check if AH is in use (vulnerable to NAT break):
sudo ip xfrm state | grep "proto ah"
# If AH is found: the connection will break if NAT is involved
# Recommendation: use ESP instead of AH, especially for internet-facing tunnels
```

---

### 8.2 Attacks Against Tunnel Mode

```bash
# ── Attack 1: Outer Header Analysis in Tunnel Mode ──────────────────────────
# Tunnel Mode hides the inner IPs but the outer (gateway) IPs are visible
# Goal: identify VPN gateways and their communication patterns

# Capture Tunnel Mode traffic:
sudo tcpdump -i eth0 -n "esp" -w /tmp/tunnel_capture.pcap

# In Wireshark: you see only gateway IP pairs → cannot see internal hosts
# BUT: you can identify which gateways exist (which sites have VPN endpoints)

# Identify IKE endpoints (shows where VPN negotiation happens):
sudo tcpdump -i eth0 -n "udp port 500 or udp port 4500" -w /tmp/ike_capture.pcap

# Fingerprint the IKE implementation (identifies vendor → known CVEs):
sudo ike-scan 198.51.0.1   # Gateway B's IP
# Output tells you: IKEv1 or IKEv2, supported ciphers, vendor (Cisco? Juniper? strongSwan?)

# ── Attack 2: IKEv1 Aggressive Mode Against Tunnel Mode VPN ─────────────────
# IKEv1 Aggressive Mode is commonly used for remote-access VPNs (Tunnel Mode)
# It exposes the PSK hash during the handshake

# Check if target VPN gateway uses IKEv1 Aggressive Mode:
sudo ike-scan --aggressive --id=test 198.51.0.1

# If vulnerable — capture PSK hash:
sudo ike-scan --aggressive --id=test --pskcrack=/tmp/psk.txt 198.51.0.1

# Crack the hash offline (safe — only needs the captured file, not the network):
psk-crack /tmp/psk.txt
# OR use hashcat for GPU acceleration:
hashcat -m 5300 /tmp/psk.txt /usr/share/wordlists/rockyou.txt --force

# If cracked: you have the PSK → can establish a rogue tunnel, intercept traffic
# Defence: use IKEv2 + certificate authentication → no PSK hash exposed

# ── Attack 3: Split Tunnelling Abuse Against Tunnel Mode ─────────────────────
# Some VPN deployments use "split tunnelling":
# Only corporate traffic goes through the IPsec tunnel
# Regular internet traffic goes directly from the user's machine (NOT through VPN)

# This means:
# → The user thinks they are "protected by VPN" for everything
# → But non-corporate traffic (their bank website, personal email) is NOT in the tunnel
# → If an attacker is on the same network: they can sniff the non-tunnelled traffic

# Detect split tunnelling:
# On the VPN client machine — check routing table:
ip route show
# Look for:
#   0.0.0.0/0 dev tun0    → full tunnel (all traffic through VPN ✅)
#   10.0.0.0/8 dev tun0   → split tunnel (only 10.x.x.x through VPN — rest unprotected ⚠️)

# Exploiting split tunnelling:
# Stay on same network as VPN user
# Their non-corporate traffic bypasses the VPN
# Capture it with Wireshark — see plaintext HTTP, DNS queries, etc.
sudo tcpdump -i eth0 -n -A "tcp port 80 and not esp"
# Anything you see here BYPASSED the VPN tunnel

# ── Attack 4: Outer IP Spoofing Attempt Against Tunnel Mode ─────────────────
# The outer IP header in Tunnel Mode is brand new and NOT protected by IPsec
# (IPsec protects the INNER packet — the outer header is just for routing)
# Theoretically: can an attacker forge the outer source IP?

# The outer source IP is NOT covered by ESP ICV (that covers only the inner content)
# HOWEVER: IKE authentication ensures only authorised parties can decrypt
# Even if outer IP is spoofed: attacker cannot decrypt ESP payload
# The HMAC (ICV) will fail because attacker doesn't have the key

# In practice: outer IP spoofing against IPsec Tunnel Mode achieves little
# because:
# → Cannot decrypt content (no key)
# → Cannot forge a valid ICV (no key)
# → Can cause a DoS by sending malformed packets — but that's a separate issue
# → Defence: IKEv2 anti-DoS cookies protect the IKE handshake

# ── Attack 5: VPN Gateway Fingerprinting and CVE Mapping ────────────────────
# VPN gateways often run old firmware → may have known vulnerabilities

# Fingerprint the VPN gateway:
sudo ike-scan -v 198.51.0.1
# Look for "VID" (Vendor ID) in output:
#   VID = Cisco Systems    → search "Cisco IKE CVE 2024"
#   VID = Fortinet         → search "Fortinet VPN CVE"
#   VID = strongSwan x.x.x → check that version for known issues

# Nmap for additional service detection on the gateway:
sudo nmap -sU -p 500,4500 -sV 198.51.0.1
sudo nmap -sT -p 443,80,22,8080 198.51.0.1
# Gateways often run web-based management interfaces on port 443
# That management interface may have unpatched vulnerabilities
```

---

## 9. 🧪 Practical Labs

### Lab 1 — Capture and Compare Both Modes in Wireshark

```bash
# PURPOSE: Visually see the difference between Transport and Tunnel Mode packets
# REQUIREMENT: Two VMs — Parrot OS and Metasploitable2 on a host-only network

# PART A: Baseline — Capture unencrypted traffic (nothing to compare to later)
sudo tcpdump -i vboxnet0 -n -A "icmp" -c 10 -w /tmp/baseline_icmp.pcap
ping -c 5 192.168.56.102   # ping Metasploitable2 in another terminal

# Open baseline in Wireshark:
wireshark /tmp/baseline_icmp.pcap &
# You see: EVERYTHING — source IP, dest IP, ICMP payload, all clear

# PART B: Capture Transport Mode IPsec traffic
# (After completing Lab 2 to set up Transport Mode)
sudo tcpdump -i vboxnet0 -n "esp" -c 20 -w /tmp/transport_esp.pcap
ping -c 10 192.168.56.102   # traffic now goes through IPsec

# Open in Wireshark:
wireshark /tmp/transport_esp.pcap &
# Filter: esp
# You see:
#   → Source IP: 192.168.56.101 (real Parrot OS host — VISIBLE)
#   → Dest IP:   192.168.56.102 (real Metasploitable2 — VISIBLE)
#   → Protocol:  ESP
#   → Payload:   ENCRYPTED (shows as raw bytes — "Encrypted Data" in Wireshark)

# PART C: Capture Tunnel Mode IPsec traffic
# (After completing Lab 3 to set up Tunnel Mode)
sudo tcpdump -i eth0 -n "esp" -c 20 -w /tmp/tunnel_esp.pcap
# Generate traffic that goes through the tunnel

wireshark /tmp/tunnel_esp.pcap &
# Filter: esp
# You see:
#   → Source IP: Gateway A IP (NOT the real internal host)
#   → Dest IP:   Gateway B IP (NOT the real internal destination)
#   → Protocol:  ESP
#   → Payload:   ENCRYPTED (original packet hidden inside)

# COMPARISON:
echo "=== Transport Mode ==="
tshark -r /tmp/transport_esp.pcap -T fields -e ip.src -e ip.dst -e esp.spi -q | head -5

echo "=== Tunnel Mode ==="
tshark -r /tmp/tunnel_esp.pcap -T fields -e ip.src -e ip.dst -e esp.spi -q | head -5

# Transport Mode: you see real host IPs
# Tunnel Mode:    you see only gateway IPs — real hosts invisible
```

---

### Lab 2 — Transport Mode Setup with strongSwan

```bash
# PURPOSE: Set up IPsec Transport Mode between Parrot OS and Metasploitable2
# GOAL: Protect traffic between the two hosts while keeping IPs visible (Transport Mode)

# PREREQUISITES:
# Parrot OS:        192.168.56.101 (your attacker/lab machine)
# Metasploitable2:  192.168.56.102 (your target VM — you have access)
# Both on VirtualBox host-only network: vboxnet0

# ── On Parrot OS: ─────────────────────────────────────────────────────────

# Install strongSwan:
sudo apt update && sudo apt install strongswan strongswan-pki libcharon-extra-plugins -y

# Create /etc/ipsec.conf for Transport Mode:
sudo tee /etc/ipsec.conf << 'EOF'
# strongSwan IPsec Configuration — Transport Mode

config setup
    charondebug="ike 3, knl 3, cfg 3, net 1, esp 1, dmn 1, mgr 1"
    # High debug level so you can watch IKE negotiations in logs

conn transport-mode-lab
    type=transport                        # ← TRANSPORT MODE (not tunnel)
    left=192.168.56.101                   # This machine's IP (Parrot OS)
    leftid=@parrot                        # Identifier for this side
    right=192.168.56.102                  # Metasploitable2 IP
    rightid=@metasploitable               # Identifier for other side
    authby=psk                            # Pre-Shared Key authentication
    ike=aes256-sha256-modp2048!           # IKE cipher suite (! = exact match only)
    esp=aes256-sha256!                    # ESP cipher suite for data protection
    keyexchange=ikev2                     # Use IKEv2 (modern, more secure)
    auto=start                            # Automatically initiate on startup
    dpdaction=restart                     # Restart on Dead Peer Detection failure
    dpddelay=10s                          # Check peer every 10 seconds
EOF

# Create /etc/ipsec.secrets — the Pre-Shared Key:
sudo tee /etc/ipsec.secrets << 'EOF'
# Format: LEFT_ID RIGHT_ID : PSK "the-shared-secret"
@parrot @metasploitable : PSK "MySuperSecretTransportModeKey2024!"
# IMPORTANT: This same key must be configured on Metasploitable2
EOF

# Set correct permissions on secrets file:
sudo chmod 600 /etc/ipsec.secrets

# ── On Metasploitable2 (SSH into it): ─────────────────────────────────────
# ssh msfadmin@192.168.56.102

# Install strongSwan on Metasploitable2 (if not present):
# sudo apt-get update && sudo apt-get install strongswan -y

# /etc/ipsec.conf on Metasploitable2 (mirror configuration):
# sudo tee /etc/ipsec.conf << 'EOF'
# config setup
#     charondebug="ike 1, knl 1, cfg 0"
#
# conn transport-mode-lab
#     type=transport
#     left=192.168.56.102                # This side (Metasploitable2)
#     leftid=@metasploitable
#     right=192.168.56.101               # Parrot OS
#     rightid=@parrot
#     authby=psk
#     ike=aes256-sha256-modp2048!
#     esp=aes256-sha256!
#     keyexchange=ikev2
#     auto=add                           # Wait for Parrot to initiate
# EOF

# /etc/ipsec.secrets on Metasploitable2:
# @parrot @metasploitable : PSK "MySuperSecretTransportModeKey2024!"

# ── Back on Parrot OS: start and test ─────────────────────────────────────

# Start strongSwan:
sudo systemctl start strongswan
sudo systemctl enable strongswan

# Initiate the IPsec Transport Mode connection:
sudo ipsec up transport-mode-lab

# Verify it's working:
sudo ipsec status
# Should show: transport-mode-lab{1}: INSTALLED, TRANSPORT

# Check the Security Associations in the kernel:
sudo ip xfrm state
# You should see two SAs (one each direction):
#   src 192.168.56.101 dst 192.168.56.102
#   proto esp spi 0xXXXXXXXX reqid 1 mode transport    ← "mode transport"!
#   aead rfc4106(gcm(aes)) ... 256 bits

# Check the Security Policies:
sudo ip xfrm policy
# Shows which traffic is covered by IPsec

# Test: ping Metasploitable2 — should now travel as ESP:
ping 192.168.56.102 -c 5

# Verify in tcpdump — you should see ESP, NOT ICMP:
sudo tcpdump -i vboxnet0 -n "esp" -v -c 10
# If you see ESP packets: Transport Mode is working ✅
# You'll notice: source and dest IPs are the REAL host IPs (192.168.56.101, .102)

# Compare with what Wireshark would show WITHOUT IPsec:
# src/dst IPs: same
# Protocol: would be ICMP
# With IPsec Transport Mode:
# src/dst IPs: same (still visible)
# Protocol: ESP (ICMP is now encrypted INSIDE the ESP payload)
```

---

### Lab 3 — Tunnel Mode VPN Setup with strongSwan

```bash
# PURPOSE: Set up IPsec Tunnel Mode simulating a site-to-site VPN
# SCENARIO: "Network A" (Parrot's subnet) ↔ "Network B" (Metasploitable's subnet)
# Both sides are treated as VPN gateways for their respective networks

# Adjust addresses to match your VirtualBox setup:
# Parrot OS (Gateway A):        192.168.56.101
# Metasploitable2 (Gateway B):  192.168.56.102
# "Network A" behind Parrot:    10.0.1.0/24 (simulated with loopback)
# "Network B" behind Msf:       10.0.2.0/24 (simulated with loopback)

# ── Add loopback addresses to simulate internal networks ──────────────────

# On Parrot OS (simulate internal "Network A" hosts):
sudo ip addr add 10.0.1.1/24 dev lo
sudo ip link set lo up

# On Metasploitable2 (simulate internal "Network B" hosts):
# sudo ip addr add 10.0.2.1/24 dev lo
# sudo ip link set lo up

# ── Configure strongSwan for Tunnel Mode on Parrot OS ─────────────────────

sudo tee /etc/ipsec.conf << 'EOF'
config setup
    charondebug="ike 3, knl 3, cfg 3"
    uniqueids=yes

conn tunnel-mode-vpn
    type=tunnel                           # ← TUNNEL MODE
    left=192.168.56.101                   # Parrot OS — "Gateway A" public IP
    leftsubnet=10.0.1.0/24               # Traffic FROM "Network A" is tunnelled
    leftid=@gateway-a
    right=192.168.56.102                  # Metasploitable2 — "Gateway B" public IP
    rightsubnet=10.0.2.0/24              # Traffic TO "Network B" is tunnelled
    rightid=@gateway-b
    authby=psk
    ike=aes256-sha256-modp2048!
    esp=aes256gcm128!                     # AES-256-GCM — authenticated encryption
    keyexchange=ikev2
    auto=start
    mark=42                               # Kernel mark for policy matching
EOF

sudo tee /etc/ipsec.secrets << 'EOF'
@gateway-a @gateway-b : PSK "TunnelModeVPNSecretKey2024!RandomAndLong"
EOF
sudo chmod 600 /etc/ipsec.secrets

# Restart strongSwan with new config:
sudo systemctl restart strongswan

# Bring up the tunnel:
sudo ipsec up tunnel-mode-vpn

# Verify tunnel is established:
sudo ipsec status
# Should show: tunnel-mode-vpn{1}: INSTALLED, TUNNEL, reqid 1
#              tunnel-mode-vpn{1}: 10.0.1.0/24 === 10.0.2.0/24
#                                  ↑ traffic selector showing what's tunnelled

# Check the SA mode:
sudo ip xfrm state | grep "mode"
# Should show: mode tunnel   ← confirming Tunnel Mode

# Check the policies:
sudo ip xfrm policy
# Should show two policies:
#   src 10.0.1.0/24 dst 10.0.2.0/24 → action use (send through tunnel)
#   src 10.0.2.0/24 dst 10.0.1.0/24 → action use (receive from tunnel)

# Test tunnel traffic:
# From Parrot OS, try to reach Metasploitable2's "internal" network:
ping 10.0.2.1 -c 5
# This should travel:
#   Source: 10.0.1.1 (loopback on Parrot)
#   Tunnel Entry: 192.168.56.101 (Parrot as Gateway A)
#   Packet on wire: outer IP 192.168.56.101→192.168.56.102 | ESP | Enc(10.0.1.1→10.0.2.1|ICMP)
#   Tunnel Exit: 192.168.56.102 (Metasploitable2 as Gateway B)
#   Delivered to: 10.0.2.1 (loopback on Metasploitable2)

# Capture the tunnel traffic:
sudo tcpdump -i vboxnet0 -n "esp" -v -c 10 &

# In Wireshark: you should see:
#   src: 192.168.56.101 (Gateway A)  — NOT 10.0.1.1
#   dst: 192.168.56.102 (Gateway B)  — NOT 10.0.2.1
#   Protocol: ESP
#   The 10.0.x.x addresses are completely invisible in the captured packets
```

---

### Lab 4 — Simulate Both Modes with Scapy

```python
#!/usr/bin/env python3
# Save as: ipsec_modes_scapy_lab.py
# Run with: sudo python3 ipsec_modes_scapy_lab.py
# PURPOSE: Understand Transport and Tunnel Mode by crafting and analysing packets

from scapy.all import *

print("="*70)
print("IPsec TRANSPORT vs TUNNEL MODE — PACKET STRUCTURE LAB")
print("="*70)

# ─── SECTION 1: Transport Mode Packet Simulation ─────────────────────────────

print("\n" + "─"*70)
print("SECTION 1: TRANSPORT MODE PACKET STRUCTURE")
print("─"*70)

# The actual communicating hosts
real_host_a = "192.168.1.10"
real_host_b = "192.168.1.20"

# In Transport Mode:
# → Original IP header stays OUTSIDE
# → IPsec (ESP) protects the TCP/UDP + data payload
# → One IP header total

# Simulate BEFORE IPsec (what came from Transport Layer):
inner_data = Raw(load=b"GET /admin HTTP/1.1\r\nHost: mywebapp.local\r\n\r\n")
original_tcp = TCP(sport=54321, dport=80, flags="PA")

# Transport Mode packet structure:
transport_pkt = (
    IP(src=real_host_a, dst=real_host_b, proto=50) /   # Original IP header (Protocol=50=ESP)
    ESP(spi=0xAABBCCDD, seq=1) /                        # ESP header inserted
    Raw(load=b"\xDE\xAD\xBE\xEF" * 16)                # Simulated encrypted payload
    # In reality: TCP header + HTTP data are ENCRYPTED here
    # Nobody on the wire can read "GET /admin HTTP/1.1" — it's encrypted
)

print("\n[Transport Mode] Packet structure:")
transport_pkt.show()
print(f"\n  Outer Source IP:       {transport_pkt[IP].src}   ← REAL Host A (visible to everyone!)")
print(f"  Outer Dest IP:         {transport_pkt[IP].dst}   ← REAL Host B (visible to everyone!)")
print(f"  Protocol:              {transport_pkt[IP].proto} (ESP — tells router it's IPsec)")
print(f"  SPI:                   {hex(transport_pkt[ESP].spi)}")
print(f"  Payload:               ENCRYPTED (TCP header, HTTP data — all hidden inside)")
print(f"\n  ⚠️  ATTACKER SEES: '{real_host_a}' IS COMMUNICATING WITH '{real_host_b}'")
print(f"  ✅ ATTACKER CANNOT SEE: port numbers, HTTP content, application type")

# ─── SECTION 2: Tunnel Mode Packet Simulation ────────────────────────────────

print("\n" + "─"*70)
print("SECTION 2: TUNNEL MODE PACKET STRUCTURE")
print("─"*70)

# The VPN gateways
gateway_a_public_ip = "203.0.113.1"
gateway_b_public_ip = "198.51.100.1"

# The real internal hosts (behind gateways)
internal_host_a = "10.0.1.5"
internal_host_b = "10.0.2.8"

# Step 1: Build the INNER (original) IP packet
inner_ip_packet = (
    IP(src=internal_host_a, dst=internal_host_b) /
    TCP(sport=54321, dport=443) /
    Raw(load=b"SECRET HTTPS DATA — database password: hunter2 — salary: 85000 BDT")
)

# Step 2: This inner packet gets encrypted by ESP
# Step 3: New OUTER IP header is added (gateway addresses)
tunnel_pkt = (
    IP(src=gateway_a_public_ip, dst=gateway_b_public_ip, proto=50) /   # OUTER IP (gateways)
    ESP(spi=0x11223344, seq=1) /                                          # ESP header
    Raw(load=bytes(inner_ip_packet))  # Simulating encrypted inner packet as payload
    # In reality: the inner IP packet above would be ENCRYPTED here
)

print("\n[Tunnel Mode] Outer packet (what's on the wire):")
print(f"  Outer Source IP:       {tunnel_pkt[IP].src}  ← Gateway A (NOT Host A)")
print(f"  Outer Dest IP:         {tunnel_pkt[IP].dst} ← Gateway B (NOT Host B)")
print(f"  Protocol:              {tunnel_pkt[IP].proto} (ESP)")
print(f"  SPI:                   {hex(tunnel_pkt[ESP].spi)}")

print(f"\n[Tunnel Mode] Inner packet (hidden inside ESP — attacker CANNOT see this):")
print(f"  Inner Source IP:       {inner_ip_packet[IP].src}  ← Real Host A")
print(f"  Inner Dest IP:         {inner_ip_packet[IP].dst}    ← Real Host B")
print(f"  Inner Protocol:        TCP (port {inner_ip_packet[TCP].dport})")
print(f"  Sensitive Data:        {inner_ip_packet[Raw].load.decode()[:40]}...")

print(f"\n  ✅ ATTACKER SEES ONLY: '{gateway_a_public_ip}' communicating with '{gateway_b_public_ip}'")
print(f"  ❌ ATTACKER CANNOT SEE: '{internal_host_a}', '{internal_host_b}', port 443, or ANY data")
print(f"  ❌ ATTACKER CANNOT SEE: That the 10.0.x.x network even exists")

# ─── SECTION 3: Side-by-Side Comparison ─────────────────────────────────────

print("\n" + "─"*70)
print("SECTION 3: SIDE-BY-SIDE COMPARISON")
print("─"*70)

print(f"""
Property                Transport Mode              Tunnel Mode
──────────────────────  ──────────────────────────  ─────────────────────────────
Source IP visible       {real_host_a}    {gateway_a_public_ip}
Dest IP visible         {real_host_b}    {gateway_b_public_ip}
Real Host A hidden      NO                          YES ({internal_host_a})
Real Host B hidden      NO                          YES ({internal_host_b})
Port numbers visible    NO (encrypted by ESP)       NO (encrypted by ESP)
Application data        NO (encrypted)              NO (encrypted)
Number of IP headers    1                           2 (outer + inner)
Traffic analysis risk   HIGH (real IPs visible)     LOW (only gateway IPs visible)
""")

# ─── SECTION 4: Replay Attack Test on Both Modes ─────────────────────────────

print("\n" + "─"*70)
print("SECTION 4: SEQUENCE NUMBERS — SAME IN BOTH MODES")
print("─"*70)

print("\n[Sequence Number Test] Both Transport and Tunnel Mode use sequence numbers:")
for seq in [1, 2, 3, 4, 5]:
    pkt = IP(dst="127.0.0.1") / ESP(spi=0xAAAA, seq=seq) / Raw(load=b"\x00"*16)
    print(f"  Seq={seq}: Packet sent — receiver accepts (new sequence number)")

print(f"\n[Replay Attack] Attacker resends seq=3:")
replay = IP(dst="127.0.0.1") / ESP(spi=0xAAAA, seq=3) / Raw(load=b"\x00"*16)
print(f"  Seq=3: REPLAYED — receiver checks: 'Already saw seq=3' → DROPPED ✅")
print(f"  This protection works identically in BOTH Transport AND Tunnel Mode")
```

---

### Lab 5 — Test Your Own Web Projects Behind IPsec Tunnel

```bash
# PURPOSE: Add IPsec Tunnel Mode protection to your MERN/PERN web projects
# SCENARIO: Your web app server is behind a strongSwan VPN gateway
#           Testing whether your services are properly protected

# ── Step 1: Identify what your web projects expose ───────────────────────

# List all listening services:
sudo ss -tlnp
# Expected output for a MERN stack:
#   0.0.0.0:3000   → Node.js/Express backend (HTTP — UNENCRYPTED)
#   0.0.0.0:27017  → MongoDB (UNENCRYPTED by default)
#   0.0.0.0:6379   → Redis (UNENCRYPTED by default)
#   0.0.0.0:5173   → Vite/React dev server (HTTP — UNENCRYPTED)

# For PERN stack additionally:
#   0.0.0.0:5432   → PostgreSQL (check if SSL/TLS is enabled)

# ── Step 2: See what data is exposed before IPsec ────────────────────────

# Watch MongoDB traffic in plaintext (BEFORE IPsec):
sudo tcpdump -i lo -n -A "port 27017" -c 20
# You will see: collection names, query filters, document fields — ALL in plaintext!
# Example visible: { "username": "admin", "password": "$2b$..." }

# Watch PostgreSQL traffic (BEFORE IPsec):
sudo tcpdump -i lo -n -A "port 5432" -c 20
# SQL queries visible in plaintext: SELECT * FROM users WHERE ...

# Watch Redis traffic:
sudo tcpdump -i lo -n -A "port 6379" -c 10
# Redis commands visible: GET session:abc123 → returns session token in clear!

# ── Step 3: Check if your production traffic is protected ─────────────────

# If your web project is deployed (e.g., on a VPS):
# From your local machine, check what the connection looks like:
sudo tcpdump -i eth0 -n "esp" -c 10
# If you see ESP: traffic is protected by IPsec (good)
# If you do NOT see ESP: traffic is protected only if TLS is in use at app level

# Check TLS on your own web project:
openssl s_client -connect yourdomain.com:443 -brief
# Look for: "CONNECTION ESTABLISHED" with cipher and certificate info
# If you see: "Connection refused" or "SSL handshake failure" → no TLS!

# ── Step 4: Set up IPsec Transport Mode to protect DB connections ─────────

# Scenario: Your Node.js app server (192.168.56.101) talks to DB server (192.168.56.103)
# Problem: MongoDB/PostgreSQL traffic goes unencrypted over the network
# Solution: IPsec Transport Mode between app server and DB server

# On app server (Parrot OS / your dev machine):
sudo tee /etc/ipsec.d/app-to-db.conf << 'EOF'
conn app-to-db
    type=transport
    left=192.168.56.101          # App server IP
    right=192.168.56.103         # DB server IP
    authby=psk
    ike=aes256-sha256-modp2048!
    esp=aes256gcm128!
    keyexchange=ikev2
    auto=start
    # Note: leftsubnet/rightsubnet NOT specified for Transport Mode
    # This means ALL traffic between these two hosts will be protected
EOF

# Apply and bring up:
sudo ipsec reload
sudo ipsec up app-to-db

# Verify MongoDB traffic is now ESP (not plaintext):
sudo tcpdump -i vboxnet0 -n "esp" -c 20 &
# Run a MongoDB query from your app:
mongosh --host 192.168.56.103 --eval "db.users.find().limit(1)"
# You should now see: ESP packets instead of plaintext MongoDB wire protocol

# ── Step 5: Penetration test your own web projects via the tunnel ──────────

# From an external attacker's perspective (another VM or network):
# Test if your services are reachable outside the IPsec tunnel:

# Try connecting to MongoDB directly from "outside" the IPsec policy:
# (from a machine that is NOT part of the IPsec setup)
mongosh --host 192.168.56.103
# This should fail if the DB server enforces IPsec for all connections
# OR it will succeed if IPsec is not enforced — meaning the service is unprotected

# Test from outside attempting to connect to protected Node.js backend:
curl http://192.168.56.101:3000/api/users
# If this succeeds from a machine NOT in the IPsec policy: your API is unprotected!
# You should see: connection refused or timeout if IPsec is properly enforced

# Summary of what to fix in your web projects:
echo "Security checklist for your MERN/PERN projects:"
echo "  ✅ Use HTTPS (TLS) for all frontend-to-backend API calls"
echo "  ✅ Enable TLS in MongoDB: mongod --tlsMode requireTLS"
echo "  ✅ Enable SSL in PostgreSQL: ssl = on in postgresql.conf"
echo "  ✅ Enable TLS in Redis: redis-server --tls-port 6380"
echo "  ✅ Consider IPsec Transport Mode for server-to-server DB connections"
echo "  ✅ Never expose DB ports (27017, 5432, 6379) to public internet"
echo "  ✅ Use firewall rules (ufw/iptables) to restrict port access"
```

---

## 10. Solved Examples

### Example 1 — Identify the Mode from Packet Description

**Question:** A packet is captured on the internet. The packet shows `Source IP: 203.0.113.1`, `Destination IP: 198.51.100.1`, `Protocol: ESP (50)`. Inside the decrypted payload, you find another complete IP packet with `Source: 10.0.1.5`, `Destination: 10.0.2.8`. Which IPsec mode is this? Explain fully.

```
Answer: This is TUNNEL MODE. Evidence:

  1. TWO IP HEADERS present:
     Outer IP:  203.0.113.1 → 198.51.100.1  (these are VPN gateway addresses)
     Inner IP:  10.0.1.5    → 10.0.2.8      (these are the REAL communicating hosts)
     In Transport Mode, there would be only ONE IP header with the real host addresses.

  2. INNER IP HEADER is HIDDEN INSIDE the ESP payload:
     The original IP packet (with 10.0.1.5 and 10.0.2.8) was encapsulated
     INSIDE the ESP encrypted payload.
     In Transport Mode, the original IP header stays OUTSIDE and is NOT encrypted.

  3. RFC 1918 PRIVATE ADDRESSES inside:
     10.0.1.5 and 10.0.2.8 are private (RFC 1918) addresses.
     These would never appear as outer IP headers on the public internet.
     They are internal addresses, hidden inside the Tunnel Mode ESP payload.
     The outer header uses public IPs (203.0.113.1 and 198.51.100.1) —
     these are the VPN gateway public addresses.

  4. USE CASE MATCH:
     The 10.0.x.x addresses suggest two different internal networks.
     Tunnel Mode is the standard for site-to-site VPN connecting internal networks.

  Conclusion: Tunnel Mode, site-to-site VPN configuration.
  Gateway A (203.0.113.1) is tunnelling internal host (10.0.1.5)'s traffic
  to Gateway B (198.51.100.1), which decapsulates and delivers to (10.0.2.8).
```

---

### Example 2 — Choose the Right Mode

**Question:** A hospital has a central database server at its head office. Doctors at remote clinics need to access patient records over the internet. What IPsec mode should be used and why? Name all the components involved.

```
Answer: TUNNEL MODE should be used. Full justification:

  Scenario Analysis:
    → Doctors at remote locations (untrusted internet)
    → Central database server at head office (trusted internal network)
    → Patient data is highly sensitive (must be encrypted)
    → Multiple clinics, multiple doctors (needs to scale)
    → Traffic crosses the PUBLIC INTERNET (untrusted medium)

  Why Tunnel Mode:

    1. GATEWAY-TO-GATEWAY (Site-to-Site) OR HOST-TO-GATEWAY (Remote Access):
       Each clinic has a VPN gateway that tunnels ALL clinic traffic.
       OR each doctor's laptop runs a VPN client (host = gateway for itself).
       Either way: Tunnel Mode allows IPsec to be terminated at a gateway,
       so individual clinic workstations don't need IPsec installed.

    2. HIDING PATIENT DATA AND ADDRESSES:
       In Transport Mode: the real source IP (clinic computer) and real destination IP
       (database server) are visible on the internet.
       An attacker could know: "A doctor at IP X.X.X.X is accessing database at Y.Y.Y.Y"
       → This itself is a HIPAA/patient privacy violation.
       Tunnel Mode: only gateway IPs are visible — which clinics are connected, not who.

    3. ENCRYPTING PATIENT RECORDS:
       ESP encryption (AES-256-GCM) ensures patient data is encrypted in transit.
       Even if an attacker intercepts the packet: they cannot read patient records.

    4. NAT TRAVERSAL:
       Clinic internet connections likely use NAT (many devices share one public IP).
       Transport Mode (especially AH) breaks with NAT.
       Tunnel Mode works correctly through NAT.

  Components:

    Remote Clinic Doctor's Laptop
      → Runs IPsec/VPN client (IKEv2 + ESP)
      → Is a "mobile gateway" in Tunnel Mode

    Clinic VPN Gateway (if site-to-site)
      → Physical router/firewall with IPsec support
      → Handles tunnelling for all clinic traffic

    Internet
      → Untrusted medium
      → Only sees outer IP headers (gateway IPs) + ESP encrypted payload

    Hospital Head Office VPN Gateway
      → Terminates incoming IPsec tunnels
      → Decapsulates, forwards to internal database server

    Hospital Internal Network
      → Private addressing (10.x.x.x)
      → Database server lives here
      → Completely invisible to internet due to Tunnel Mode

    Database Server
      → Receives plain (decapsulated) patient record requests
      → Unaware of IPsec — operates normally

  IKEv2 negotiation: Certificate-based (not PSK) for hospital environment
  ESP cipher suite:  AES-256-GCM (authenticated encryption)
  PFS:               Enabled (DH Group 14 minimum)
```

---

### Example 3 — 5-Mark Exam: Explain the Difference

**Question:** With the help of packet diagrams, explain the difference between Transport Mode and Tunnel Mode in IPsec. State one use case for each.

```
Answer:

  BACKGROUND:
  IPsec operates at the Network Layer (Layer 3). It can be applied in
  two modes that differ in what portion of the original IP packet is protected
  and whether a new IP header is added.

  ──────────────────────────────────────────────────────────────────────
  TRANSPORT MODE:
  ──────────────────────────────────────────────────────────────────────

  Processing:
    1. Packet arrives from Transport Layer: [TCP/UDP Header | Data]
    2. IPsec ESP header and trailer WRAP the payload (TCP/UDP + data)
    3. THEN the original IP header is added OUTSIDE the IPsec protection

  Packet structure on the wire:
  ┌──────────────────┬──────────────┬═══════════════════════════════╗═══════════┐
  │  Original IP Hdr │  ESP Header  │ Enc(TCP/UDP Header + App Data)║ESP Trailer│
  │  src=A, dst=B    │  SPI + Seq   │                               ║+ ESP Auth │
  └──────────────────┴──────────────┴═══════════════════════════════╝═══════════┘
  ↑────────────────↑
  Visible to everyone — real source (A) and destination (B) exposed

  What is protected:   Payload (TCP/UDP + application data) — ENCRYPTED
  What is exposed:     Source IP, Destination IP — VISIBLE to all
  IP header count:     1 (the original)

  Use case: Host-to-host communication on an internal network
  Example:  App server (192.168.1.5) communicates securely with DB server (192.168.1.10)
            in the same data centre. Both are known endpoints. Hiding their IPs is
            unnecessary. We only need to encrypt the SQL queries being sent.

  ──────────────────────────────────────────────────────────────────────
  TUNNEL MODE:
  ──────────────────────────────────────────────────────────────────────

  Processing:
    1. Packet arrives from Transport Layer: [TCP/UDP Header | Data]
    2. Network Layer adds original IP header: [Orig IP Hdr | TCP/UDP | Data]
    3. This ENTIRE original IP packet is encrypted by IPsec ESP
    4. A NEW outer IP header is added (using gateway addresses, not host addresses)

  Packet structure on the wire:
  ┌──────────────────┬──────────────┬══════════════════════════════════════════════╗═══════════┐
  │  NEW Outer IP    │  ESP Header  │ Encrypted(Orig IP Hdr + TCP/UDP + App Data)  ║ESP Trailer│
  │  src=GW-A,dst=GW-B│ SPI + Seq   │  (ENTIRE original packet hidden inside)     ║+ ESP Auth │
  └──────────────────┴──────────────┴══════════════════════════════════════════════╝═══════════┘
  ↑────────────────↑                ↑────────────────────────────────────────────────────────────↑
  Visible: only gateway IPs         HIDDEN: original source IP, original dest IP,
  Not visible: real host IPs        TCP/UDP header, all application data

  What is protected:   ENTIRE original IP packet — header AND payload
  What is exposed:     Only gateway IP addresses
  IP header count:     2 (outer = gateways visible, inner = real hosts hidden)

  Use case: VPN / site-to-site connectivity over the internet
  Example:  Company Office A (Gateway: 203.0.113.1) connects to Company Office B
            (Gateway: 198.51.100.1) over the internet. All 10.0.1.x to 10.0.2.x
            traffic is tunnelled. Attackers on the internet see only the gateway
            IPs communicating — internal host IPs and all data are completely hidden.

  ──────────────────────────────────────────────────────────────────────
  KEY DIFFERENCE SUMMARY:
    Transport: payload protected, original IP header EXPOSED
    Tunnel:    entire original packet protected, original header HIDDEN
    Transport: 1 IP header     |  Tunnel: 2 IP headers
    Transport: host-to-host    |  Tunnel: gateway-to-gateway (VPN)
```

---

## 11. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════════╗
║       IPsec TRANSPORT vs TUNNEL MODE — EXAM CHEAT SHEET                  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  TRANSPORT MODE                                                            ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  Protects:     Payload ONLY (TCP/UDP + application data)                  ║
║  IP Headers:   1 (the original — with REAL source and dest IPs)          ║
║  Original Hdr: OUTSIDE IPsec protection — visible to everyone            ║
║  Source IP:    VISIBLE on the wire (real host addresses exposed)          ║
║  Dest IP:      VISIBLE on the wire (real host addresses exposed)          ║
║  Use case:     Host-to-host communication (direct, both ends run IPsec)   ║
║  Example:      Server A ↔ Server B, same data centre                     ║
║  Traffic risk: HIGH — attacker sees who is talking to who                ║
║  Overhead:     Low — one IP header, smaller packets                       ║
║  NAT support:  Limited (AH breaks with NAT)                              ║
╠══════════════════════════════════════════════════════════════════════════╣
║  TUNNEL MODE                                                               ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  Protects:     ENTIRE original IP packet (header + payload)               ║
║  IP Headers:   2 (outer = gateway addresses; inner = real hosts, hidden)  ║
║  Original Hdr: INSIDE ESP encryption — invisible to everyone on the path  ║
║  Source IP:    HIDDEN (original src IP encrypted inside ESP payload)      ║
║  Dest IP:      HIDDEN (original dst IP encrypted inside ESP payload)      ║
║  Use case:     VPN — site-to-site or remote access                       ║
║  Example:      Office A Gateway ↔ Office B Gateway over internet          ║
║  Traffic risk: LOW — attacker sees only gateway IPs                      ║
║  Overhead:     Higher — two IP headers, larger packets                    ║
║  NAT support:  ✅ Works — outer IP header handles NAT                    ║
╠══════════════════════════════════════════════════════════════════════════╣
║  PACKET STRUCTURE (quick visual)                                           ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  TRANSPORT:                                                                ║
║  [IP Hdr (visible)] [ESP Hdr] [ENCRYPTED: TCP/UDP + Data] [ESP Trail]    ║
║                                                                            ║
║  TUNNEL:                                                                   ║
║  [New Outer IP (visible)] [ESP Hdr] [ENCRYPTED: [Orig IP+TCP/UDP+Data]]  ║
║                                               ↑ entire original packet    ║
╠══════════════════════════════════════════════════════════════════════════╣
║  ONE-LINE DECISION RULE                                                    ║
║  Q: "Should I use Transport or Tunnel Mode?"                              ║
║                                                                            ║
║  → Are the endpoints HOSTS talking directly to each other?               ║
║    → Are you okay with the IPs being visible?                            ║
║    → Use TRANSPORT MODE                                                   ║
║                                                                            ║
║  → Are you using a VPN gateway?                                           ║
║  → Do you want to HIDE which hosts are communicating?                    ║
║  → Is this a site-to-site or remote access scenario?                     ║
║    → Use TUNNEL MODE                                                      ║
╠══════════════════════════════════════════════════════════════════════════╣
║  WHY IT'S CALLED "TUNNEL" MODE                                             ║
║  The original packet ENTERS a tunnel at Gateway A,                       ║
║  travels HIDDEN through the internet,                                     ║
║  and EXITS the tunnel at Gateway B — just like a real tunnel              ║
║  where you're invisible while inside it.                                  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  EXAM TRAPS SPECIFIC TO THIS TOPIC                                         ║
║  → Transport Mode does NOT add a new IP header (only 1 header)           ║
║  → Tunnel Mode DOES add a new outer IP header (2 headers total)          ║
║  → In Tunnel Mode: the INNER (original) IP header is ENCRYPTED           ║
║  → In Transport Mode: the IP header is NEVER encrypted (stays outside)   ║
║  → Both modes use ESP/AH — the mode is SEPARATE from the protocol        ║
║  → AH + NAT = BROKEN in BOTH modes (NAT changes IP header → HMAC fails) ║
║  → Tunnel Mode = used in VPN (this is the most common exam connection)   ║
║  → Transport Mode = used for direct host-to-host (NOT VPN gateways)     ║
║  → You can use AH or ESP with EITHER mode — 4 combinations possible:    ║
║      AH + Transport, AH + Tunnel, ESP + Transport, ESP + Tunnel          ║
╠══════════════════════════════════════════════════════════════════════════╣
║  SECURITY COMPARISON                                                       ║
║  Property              Transport           Tunnel                          ║
║  ─────────────────────────────────────────────────────────────────────── ║
║  Data encrypted        ✅ YES (ESP)        ✅ YES (ESP)                   ║
║  Real IPs visible      ✅ YES (exposed)    ❌ NO (hidden)                 ║
║  Internal IPs hidden   ❌ NO               ✅ YES                          ║
║  Traffic analysis      HIGH RISK           LOW RISK                        ║
║  NAT compatibility     DIFFICULT           EASY                            ║
║  Gateway support       ❌ Hosts only       ✅ Gateways on behalf of hosts  ║
╠══════════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS                                                              ║
║  TRANSPORT = hosts TRANSPORT data directly → payload protected, IPs show  ║
║  TUNNEL    = packet goes INTO a TUNNEL → everything hidden inside         ║
║  Transport = 1 IP header  (think: "one hop, direct")                     ║
║  Tunnel    = 2 IP headers (think: "two layers — outer gateway, inner real")║
╚══════════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] **IKEv2 — Deep Dive** (exchange messages, MOBIKE, EAP, certificate auth)
- [ ] **strongSwan Full Configuration** — site-to-site VPN with certificate-based auth
- [ ] **L2TP/IPsec** — how L2TP tunnels inside IPsec for remote access VPN
- [ ] **IPsec NAT Traversal (NAT-T)** — how UDP port 4500 wraps ESP for NAT
- [ ] **WireGuard vs IPsec vs OpenVPN** — modern comparison
- [ ] **IPsec Split Tunnelling** — configuration, risks, and bypass techniques
- [ ] **iptables / nftables rules for IPsec** — firewall configuration for AH, ESP, IKE
- [ ] **IPsec on AWS / Azure** — cloud VPN gateway configuration and security

---

_Notes compiled from: Networking Course — IPsec Transport Mode and Tunnel Mode (Gate Smashers)_
_Source: Gate Smashers YouTube_
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student_
