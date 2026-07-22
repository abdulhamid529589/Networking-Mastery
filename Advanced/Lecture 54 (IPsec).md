# 🔐 IPsec — IP Security Protocol

### " "Networking Course — Network Layer Security

> **Source:** Gate Smashers — IPsec Protocol
> **Exam Relevance:** GATE · UGC NET · University Exams · Interviews · CompTIA Security+ · CEH
> **Lab Environment:** Parrot OS + Metasploitable2 + Own MERN/PERN Projects

---

## 📑 Table of Contents

1. [What is IPsec?](#1-what-is-ipsec)
2. [Where IPsec Lives — Network Layer](#2-where-ipsec-lives--network-layer)
3. [IPsec — Collection of Protocols](#3-ipsec--collection-of-protocols)
   - [3.1 ESP — Encapsulating Security Payload](#31-esp--encapsulating-security-payload)
   - [3.2 AH — Authentication Header Protocol](#32-ah--authentication-header-protocol)
   - [3.3 IKE — Internet Key Exchange](#33-ike--internet-key-exchange)
4. [What IPsec Provides — Security Goals](#4-what-ipsec-provides--security-goals)
   - [4.1 Confidentiality (Privacy)](#41-confidentiality-privacy)
   - [4.2 Authentication and Integrity](#42-authentication-and-integrity)
   - [4.3 Replay Attack Protection](#43-replay-attack-protection)
5. [IPsec Modes of Operation](#5-ipsec-modes-of-operation)
   - [5.1 Transport Mode](#51-transport-mode)
   - [5.2 Tunnel Mode](#52-tunnel-mode)
   - [5.3 Transport vs Tunnel — Comparison](#53-transport-vs-tunnel--comparison)
6. [IPsec in IPv4 vs IPv6](#6-ipsec-in-ipv4-vs-ipv6)
7. [Security Associations (SA)](#7-security-associations-sa)
8. [How IPsec Works End-to-End — Full Flow](#8-how-ipsec-works-end-to-end--full-flow)
9. [Attack Types IPsec Defends Against](#9-attack-types-ipsec-defends-against)
10. [Attacking IPsec — Offensive Perspective](#10-attacking-ipsec--offensive-perspective)
11. [🧪 Practical Labs](#11--practical-labs)
12. [Solved Examples](#12-solved-examples)
13. [Exam Cheat Sheet](#13-exam-cheat-sheet)

---

## 1. What is IPsec?

IPsec (IP Security) is **not a single protocol** — it is a **standard set (collection) of protocols** defined by the **IETF (Internet Engineering Task Force)** that work together to provide security at the network layer.

```
Full name:  IP Security Protocol
Short:      IPsec
Defined by: IETF (Internet Engineering Task Force)
Layer:      Network Layer (Layer 3 in OSI — same layer as IPv4, IPv6, ICMP, IGMP)
Type:       Collection / Suite of protocols (not one single protocol)

Why a "suite"?
  → Different security goals require different mechanisms
  → Confidentiality alone → use ESP
  → Authentication alone  → use AH
  → Key exchange needed   → use IKE
  → Mix and match as needed — modular design
```

**The analogy:** Think of IPsec like a security toolkit. You don't use every tool every time — you pick the right tool for the job. Need to lock a door? Use the lock (ESP). Need to verify who's knocking? Use the peephole (AH). Need to safely hand over a key? Use a secure key handoff protocol (IKE).

---

## 2. Where IPsec Lives — Network Layer

```
OSI Model (Bottom to Top):

  Layer 7 — Application
  Layer 6 — Presentation
  Layer 5 — Session
  Layer 4 — Transport     (TCP, UDP live here)
  Layer 3 — Network  ◄──── IPsec lives HERE (same as IPv4, IPv6, ICMP, IGMP)
  Layer 2 — Data Link
  Layer 1 — Physical

Why Network Layer matters for security:
  → Network layer handles addressing and routing
  → Every packet passing between hosts goes through this layer
  → Securing it here protects ALL traffic regardless of the application
  → Application doesn't need to be modified — security is transparent
  → One layer to secure → all applications above get protection automatically
```

```
Comparison — where security is applied:

  Layer 7 (Application): HTTPS, SSH, PGP/GPG — application-specific
  Layer 4 (Transport):   TLS/SSL — secures the TCP connection
  Layer 3 (Network):     IPsec  — secures the IP packet itself

  IPsec advantage: secures everything at the packet level
  → Even protocols that have NO built-in security (like raw UDP) get protected
  → Works for ALL traffic: HTTP, FTP, DNS, custom protocols — anything
```

> **Security relevance:** Because IPsec operates at Layer 3, it can protect traffic **before** it even reaches TCP/UDP. This is why VPNs predominantly use IPsec — it wraps and secures the entire IP packet, making it transparent to all layers above.

---

## 3. IPsec — Collection of Protocols

IPsec consists of three core protocols. Each has a distinct role:

```
IPsec Protocol Suite:

  ┌─────────────────────────────────────────────────────────┐
  │                     IPsec Suite                          │
  │                                                          │
  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
  │  │     ESP      │  │      AH      │  │     IKE      │  │
  │  │ Encapsulating│  │Authentication│  │  Internet    │  │
  │  │  Security    │  │   Header     │  │    Key       │  │
  │  │  Payload     │  │  Protocol    │  │  Exchange    │  │
  │  │              │  │              │  │              │  │
  │  │ Encryption ✅│  │ Auth only ✅ │  │ Key mgmt ✅  │  │
  │  │ Auth ✅      │  │ No Encrypt ❌│  │              │  │
  └──┴──────────────┴──┴──────────────┴──┴──────────────┴──┘
```

---

### 3.1 ESP — Encapsulating Security Payload

```
Full name:  Encapsulating Security Payload
Purpose:    Encryption + Authentication + Integrity
Next Header value (in IPv6): 50

Primary role: CONFIDENTIALITY
  → Encrypts the payload of the IP packet
  → Even if an attacker intercepts the packet — they cannot read the content
  → The data is scrambled — unreadable without the key

What ESP provides:
  ✅ Confidentiality  — payload is ENCRYPTED
  ✅ Authentication   — sender identity verified
  ✅ Integrity        — tampering detected
  ✅ Replay protection — sequence numbers prevent replay attacks

ESP is the most commonly used IPsec protocol because it provides
the MOST complete set of security services in one component.
```

```
ESP Header and Trailer structure:

  ┌───────────────────────────────────────────────────────────────┐
  │               SPI — Security Parameter Index (32 bits)        │  ← ESP Header
  ├───────────────────────────────────────────────────────────────┤
  │                    Sequence Number (32 bits)                   │
  ├═══════════════════════════════════════════════════════════════╡
  │                   *** ENCRYPTED PAYLOAD ***                    │  ← Encrypted
  │     (Original data — could be TCP segment, UDP, anything)     │
  │     (Even TCP/UDP port numbers are hidden inside here)        │
  ├───────────────────────────────────────────────────────────────┤
  │                    Padding (0–255 bytes)                       │  ← ESP Trailer
  ├───────────────────────────────────────────────────────────────┤
  │         Pad Length (8 bits) │  Next Header (8 bits)            │
  ├───────────────────────────────────────────────────────────────┤
  │              Integrity Check Value — ICV (variable)            │  ← ESP Auth
  └───────────────────────────────────────────────────────────────┘

  SPI:          Identifies which Security Association (key+algorithm) to use
  Seq Number:   Monotonically increasing — catches replay attacks
  Encrypted:    Everything in here is unreadable without the key
  Padding:      Aligns payload to block cipher boundaries
  ICV:          HMAC digest — verifies authenticity and integrity
```

> **Security relevance — ESP as a double-edged sword:**
>
> - **Defensive:** Nobody on the path (ISP, attacker, surveillance) can read the payload
> - **Offensive:** Your own IDS/IPS/DLP tools are also blind to ESP-encrypted content
> - **Attacker use case:** Malware can use ESP tunnels to **exfiltrate data** or maintain **C2 (Command & Control) channels** that bypass all network monitoring
> - **Forensics:** To decrypt captured ESP traffic you need the session keys from the endpoints — raw `.pcap` files are useless without them

---

### 3.2 AH — Authentication Header Protocol

```
Full name:  Authentication Header Protocol
Purpose:    Authentication + Integrity (NO encryption)
Next Header value (in IPv6): 51

Primary role: VERIFYING THE SENDER AND DATA INTEGRITY
  → Confirms the packet really came from who it claims to come from
  → Confirms the packet was NOT modified in transit
  → Does NOT hide the data — payload is still readable

What AH provides:
  ✅ Authentication   — sender identity verified via HMAC
  ✅ Integrity        — any modification of the packet is detected
  ❌ Confidentiality  — payload is NOT encrypted (visible in plaintext)
  ✅ Replay protection — sequence numbers prevent replay attacks
```

```
How AH works (conceptual):

  SENDER SIDE:
    1. Compute HMAC over:
       - Selected IPv6/IPv4 header fields (that don't change in transit)
       - AH header itself (with ICV field zeroed out during computation)
       - Full payload data
    2. Insert the HMAC result as ICV into the AH header
    3. Send the packet

  RECEIVER SIDE:
    1. Receive the packet
    2. Extract the ICV from AH header
    3. Recompute HMAC independently using the shared key
    4. Compare computed HMAC vs received ICV:
       → MATCH:    packet is authentic and unmodified ✅ → accept
       → NO MATCH: packet was tampered with OR spoofed ❌ → DISCARD
```

```
AH Header structure:

  ┌────────────────┬──────────────┬──────────────────────────────┐
  │ Next Header(8) │ Payload Len  │         Reserved (16)         │
  ├────────────────┴──────────────┴──────────────────────────────┤
  │                    SPI — Security Parameter Index (32)        │
  ├───────────────────────────────────────────────────────────────┤
  │                    Sequence Number (32)                        │
  ├───────────────────────────────────────────────────────────────┤
  │              Integrity Check Value — ICV (variable length)     │
  │                    (the actual HMAC digest)                    │
  └───────────────────────────────────────────────────────────────┘

  Next Header:    What comes after the AH header (TCP=6, UDP=17, ESP=50, etc.)
  Payload Len:    Length of the AH header itself
  SPI:            Which Security Association (key/algorithm) to use
  Sequence Num:   Anti-replay counter — unique, always increasing
  ICV:            The HMAC authentication digest (HMAC-SHA256, HMAC-SHA512, etc.)
```

```
Important AH limitation — MUTABLE FIELDS:
  AH must skip certain fields when computing the HMAC because
  those fields are ALLOWED to change in transit (routers modify them):

  Fields that change in transit (NOT covered by AH):
    → Hop Limit / TTL     (decremented by every router)
    → DSCP / Traffic Class (may be remarked by QoS policies)
    → Flow Label

  Fields covered by AH (don't change):
    → Source/Destination address
    → Protocol / Next Header
    → Payload data

  This means AH provides PARTIAL header protection, not full.
```

> **When to use AH vs ESP:**
>
> ```
> Use AH when:
>   → You need to verify the packet sender and detect tampering
>   → Privacy is NOT required (payload can be seen)
>   → Example: internal network where you trust the pipe, but need to verify identity
>
> Use ESP when:
>   → You need confidentiality (no one should read the payload)
>   → Plus authentication and integrity
>   → Example: VPN over the internet — need everything
>
> Use both AH + ESP together when:
>   → You need maximum security — header authentication (AH) AND payload encryption (ESP)
>   → AH goes first (outer), ESP encrypts the payload
>   → Provides integrity over the full IP header + encryption of the data
> ```

---

### 3.3 IKE — Internet Key Exchange

```
Full name:  Internet Key Exchange
Purpose:    Securely exchanging cryptographic keys between two parties
            before IPsec communication begins

The problem it solves:
  IPsec uses symmetric encryption (both sides need the SAME key)
  OR uses public-key cryptography.
  Either way — the key must be SHARED SAFELY before any IPsec data flows.

  Question: How do you share the key safely over an unsafe network?
  Answer:   You use IKE!

IKE versions:
  IKEv1: Original version — more complex, two-phase handshake
  IKEv2: Current standard — simpler, faster, more secure, better NAT traversal
```

```
IKE operates in two phases:

  Phase 1 — Establish a secure channel (IKE SA):
    → Negotiate encryption algorithm (AES, 3DES...)
    → Negotiate integrity algorithm (SHA-256, SHA-512...)
    → Negotiate Diffie-Hellman group (for key agreement)
    → Authenticate each other (pre-shared key or certificates)
    → Result: a secure, encrypted, authenticated channel

  Phase 2 — Negotiate IPsec SA over that secure channel:
    → Negotiate which IPsec protocol: AH? ESP? Both?
    → Negotiate encryption algorithm for the actual data
    → Exchange session keys for IPsec data protection
    → Result: IPsec Security Associations established → data can flow

  IKEv2 (modern) streamlines this into fewer round-trips.
```

```
Key exchange methods supported by IKE:

  Pre-Shared Key (PSK):
    → Both sides already have the same secret configured manually
    → Simple but doesn't scale — every pair needs a configured PSK
    → Common in small VPN setups, home routers

  Public Key Infrastructure (PKI / Certificates):
    → Each side has a certificate signed by a trusted CA
    → IKE uses RSA or ECDSA to authenticate
    → Scales well — large enterprise VPNs use this
    → More complex to set up

  Diffie-Hellman (DH) Key Agreement:
    → Even if PSK or certificates are used for authentication,
      DH is used to derive the ACTUAL session encryption keys
    → Provides Perfect Forward Secrecy (PFS):
      even if the long-term key is later compromised,
      past session keys cannot be derived from it
```

> **Security relevance — IKE attacks:**
>
> ```
> IKEv1 weaknesses (reasons IKEv2 was created):
>   → Aggressive Mode PSK vulnerability: the PSK hash is sent in cleartext
>     in Aggressive Mode → can be captured and brute-forced offline
>   → Known IKEv1 implementation bugs: Cisco, Juniper IKEv1 CVEs
>
> IKEv2 improvements:
>   → Always mutual authentication before any sensitive data exchanged
>   → Built-in NAT traversal (NAT-T) — works through firewalls better
>   → MOBIKE: handles IP address changes (mobile devices switching networks)
>   → Fewer message round-trips → faster, harder to exploit timing
>
> Tool for IKE fingerprinting / testing:
>   ike-scan — identifies IKE implementations and configurations
>   sudo ike-scan TARGET_IP
>   → Reveals: IKEv1 vs IKEv2, supported transforms, vendor ID (fingerprints the device)
> ```

---

## 4. What IPsec Provides — Security Goals

IPsec provides three core security services. You can use **all of them** together, or **just one** depending on your need:

```
  ┌─────────────────────────────────────────────────────────────┐
  │              IPsec Security Services                         │
  │                                                              │
  │  1. CONFIDENTIALITY (Privacy)                                │
  │     → No unauthorised person can READ the data               │
  │     → Provided by: ESP (encryption)                          │
  │                                                              │
  │  2. AUTHENTICATION + INTEGRITY                               │
  │     → Verify WHO sent it (source authentication)             │
  │     → Verify data was NOT CHANGED (message integrity)        │
  │     → Provided by: AH or ESP (HMAC)                         │
  │                                                              │
  │  3. REPLAY ATTACK PROTECTION                                 │
  │     → Old captured packets CANNOT be re-sent                 │
  │     → Provided by: AH and ESP (sequence numbers)            │
  └─────────────────────────────────────────────────────────────┘
```

---

### 4.1 Confidentiality (Privacy)

```
The Problem:
  The internet is a shared, untrusted medium. Between sender A and receiver B,
  there are many intermediate nodes: routers, ISP equipment, switches.
  Any of these — or any attacker who gains access to them — can potentially
  READ every packet that passes through.

  A ──────────────[INTERNET]──────────────→ B
                 ↑
             Eve is here, reading everything

  Without IPsec:
    → Eve reads the HTTP request containing your password
    → Eve reads the email you thought was private
    → Eve sees the file you uploaded
    → Full visibility = full compromise

The IPsec Solution:
  → ESP encrypts the payload
  → Even if Eve captures every packet, she sees only ciphertext
  → Without the key: decryption is computationally infeasible
  → Data remains private even over an untrusted network
```

```
Encryption algorithms used in IPsec ESP:

  AES-256-GCM      → Current standard — fast, authenticated encryption
  AES-128-GCM      → Slightly faster, still very strong
  AES-CBC-256      → Older mode, still common — requires separate HMAC
  3DES-CBC         → Legacy — avoid in new deployments (slow, weaker)
  ChaCha20-Poly1305→ Modern alternative, especially for mobile devices

  Recommendation for your lab:
    Use AES-256-GCM wherever possible — provides confidentiality + integrity
    in a single pass (authenticated encryption), faster than CBC+HMAC
```

> **Practical test — can you see VPN traffic?**
>
> ```bash
> # On Parrot OS, capture traffic to/from a VPN-connected host
> sudo tcpdump -i eth0 -n "esp" -v
>
> # You will see ESP packets — source IP, destination IP, SPI value
> # But the PAYLOAD is completely unreadable — just encrypted bytes
>
> # This demonstrates confidentiality working correctly
> # Compare with unencrypted HTTP:
> sudo tcpdump -i eth0 -n -A "tcp port 80"
> # You see full HTTP headers and body in plaintext
> ```

---

### 4.2 Authentication and Integrity

Authentication and integrity are two related but distinct concepts:

```
TWO TYPES OF AUTHENTICATION IN IPsec:

  Type 1 — MESSAGE INTEGRITY (Data Authentication):
    ─────────────────────────────────────────────────
    Problem: A sends a message to B. An attacker (Mallory) intercepts
             the packet in transit, modifies the content, and forwards it.
             B receives a modified message but doesn't know it was changed.

    A ──────→ [Mallory modifies "transfer $100" to "transfer $10,000"] ──→ B

    Without integrity check:
      → B accepts the modified message as legitimate
      → $10,000 is transferred
      → A had no idea

    With IPsec AH/ESP integrity:
      → A computes HMAC over the original message
      → Mallory modifies the payload
      → HMAC now doesn't match
      → B computes HMAC → mismatch detected → packet DISCARDED
      → Attack fails ✅

  Type 2 — SOURCE AUTHENTICATION (Identity Authentication):
    ─────────────────────────────────────────────────────────
    Problem: A sends a message to B. Mallory pretends to BE A —
             spoofs A's IP address and sends a fake message to B.
             B thinks it came from A.

    Mallory (using A's IP address) ──────→ B thinks it's from A

    Without source authentication:
      → B believes the fake message
      → Mallory can impersonate A completely

    With IPsec AH/ESP source authentication:
      → Only A knows the shared secret key (or private key)
      → Mallory cannot compute a valid HMAC without the key
      → B verifies HMAC → Mallory's packet fails verification → DISCARDED
      → Impersonation fails ✅
```

```
Hashing algorithms used for HMAC in IPsec:

  HMAC-SHA-256      → Current standard — strong, fast
  HMAC-SHA-512      → Stronger, slightly slower
  HMAC-SHA-1        → Legacy — still used but weakening, avoid in new setups
  AES-XCBC-96       → AES-based MAC — used in some hardware implementations
  AES-GCM           → Combined encryption + authentication in one (ESP)

  MD5 and SHA-1 for HMAC: technically still valid in IKEv1 configurations
  but AVOID in new deployments — use SHA-256 minimum.
```

---

### 4.3 Replay Attack Protection

```
What is a Replay Attack?

  Step 1: A sends a legitimate message to B:
          "Please transfer 1,000 BDT to account X"

  Step 2: Mallory is on the network, captures this packet.
          Mallory doesn't need to read it or modify it.

  Step 3: B receives the message, verifies it (authenticated, unmodified)
          and transfers 1,000 BDT to account X. All good.

  Step 4 (the attack): Mallory re-sends the SAME captured packet to B.
          10 minutes later: "Please transfer 1,000 BDT to account X" (again)
          B receives it. It still has a VALID HMAC — it wasn't modified.
          B transfers another 1,000 BDT!

  Step 5: Mallory keeps replaying. Again. Again. Again.
          1,000 + 1,000 + 1,000 + 1,000 + ... → unlimited money

  This is a REPLAY ATTACK — reusing a valid captured packet.
```

```
How IPsec prevents Replay Attacks — Sequence Numbers:

  How it works:
    → Every IPsec packet has a SEQUENCE NUMBER in the AH/ESP header
    → Sender starts at 1, increments by 1 for every packet
    → Receiver maintains a SLIDING WINDOW of accepted sequence numbers
    → Receiver tracks which sequence numbers it has already accepted

  When Mallory replays a packet:
    → The replayed packet has sequence number, say, 47
    → B's receiver has already processed sequence number 47
    → B checks: "I already received #47 → this is a replay" → DISCARD ✅

  The sliding window:
    → Receiver accepts packets within a window (e.g., last 64 sequence numbers)
    → Out-of-window packets are rejected (too old or too far ahead)
    → Already-seen sequence numbers within window are also rejected
    → New, unique, in-window sequence numbers are accepted
```

```
Sequence Number behaviour:
  → 32-bit counter in AH/ESP: 0 to 4,294,967,295
  → If the counter wraps around to 0, a NEW Security Association must be
    negotiated (new keys, new SPI) — counter rollover is not allowed
    (a rollover would let Mallory replay very old packets)
  → Extended Sequence Number (ESN): 64-bit counter for high-speed links
    where 32 bits would exhaust too quickly
```

> **Security relevance — Real-world replay attacks:**
>
> ```
> Replay attacks are used in:
>   → Authentication bypass: replay a captured login packet
>   → Session hijacking: replay a session token packet
>   → Financial fraud: replay payment/transfer packets (as above)
>   → Protocol downgrade: replay old handshake packets to force weaker crypto
>
> Tools to simulate replay attacks in your lab:
>   tcpreplay — replay captured pcap files on the network
>   Scapy — craft and resend specific captured packets
>
>   # Basic replay with tcpreplay (against your own Metasploitable2):
>   sudo tcpreplay --intf1=eth0 /tmp/captured.pcap
>
>   # Verify IPsec rejects the replayed packets by watching logs:
>   sudo journalctl -f | grep -i "replay\|ipsec"
> ```

---

## 5. IPsec Modes of Operation

IPsec operates in **two distinct modes**. The mode determines **how much of the original IP packet is protected** and **what the resulting packet looks like**.

---

### 5.1 Transport Mode

```
Transport Mode — Protects PAYLOAD only

What it does:
  → The ORIGINAL IP header is KEPT as-is (not encrypted, not hidden)
  → Only the payload (data portion) is encrypted/authenticated
  → IPsec header (AH or ESP) is inserted BETWEEN the IP header and payload

Original packet:
  [ IP Header | Data (TCP/UDP/...) ]

After IPsec in Transport Mode with ESP:
  [ IP Header | ESP Header | Encrypted(Data) | ESP Trailer | ESP Auth ]
              ↑             ↑─────────────────────────────────────────↑
              inserted      this whole section is encrypted/protected

After IPsec in Transport Mode with AH:
  [ IP Header | AH Header | Data ]
              ↑            ↑
              inserted      data is still visible, but authenticated
```

```
Properties of Transport Mode:

  ✅ Lightweight — smaller packet overhead (original IP header unchanged)
  ✅ Source/Destination IP addresses are visible to routers
  ✅ Good for end-to-end communication (host to host directly)
  ❌ IP addresses are exposed — attacker can see WHO is talking to WHO
  ❌ Traffic analysis possible — attacker can track communication patterns
  ❌ Cannot pass through NAT easily (especially AH — NAT breaks AH integrity)

Typical use cases:
  → Host-to-host communication on a trusted internal network
  → Protecting traffic between two specific servers in the same data centre
  → Remote desktop sessions between known endpoints
```

```
Transport Mode diagram:

  Host A                                              Host B
  (192.168.1.10)                                (192.168.1.20)

  [ IP: 10 → 20 | ESP Header | Enc(TCP:443, HTTP data) | ESP Auth ]
  ──────────────────────────────────────────────────────────────────→

  Router sees:    Source: 192.168.1.10, Destination: 192.168.1.20
  Router sees:    ESP (Next Header = 50) — encrypted payload
  Router cannot see: what's inside (TCP port, HTTP content — all encrypted)
  Router CAN see:    that 192.168.1.10 is talking to 192.168.1.20
```

---

### 5.2 Tunnel Mode

```
Tunnel Mode — Protects ENTIRE ORIGINAL PACKET

What it does:
  → The ENTIRE original IP packet (header + payload) is treated as payload
  → A BRAND NEW outer IP header is added
  → Original IP header is HIDDEN inside (encrypted/authenticated)
  → This creates a "tunnel" — original packet travels inside a new packet

Original packet:
  [ Original IP Header | Data ]

After IPsec in Tunnel Mode with ESP:
  [ New IP Header | ESP Header | Encrypted(Original IP Header + Data) | ESP Trailer | ESP Auth ]
  ↑─────────────↑              ↑──────────────────────────────────────────────────────────────↑
  New header added              ENTIRE original packet is encrypted inside here
```

```
Properties of Tunnel Mode:

  ✅ Maximum security — original IP addresses COMPLETELY HIDDEN
  ✅ Source and destination in the new header are the VPN gateways,
     not the actual communicating hosts
  ✅ Works through NAT — outer IP header is fresh, NAT works normally
  ✅ Can hide network topology — attackers can't see internal addressing
  ❌ Larger packet overhead — two IP headers instead of one
  ❌ Slightly more processing at VPN gateways

Typical use cases:
  → VPN (Virtual Private Networks) — the PRIMARY use case
  → Site-to-site VPN: gateway A ←→ gateway B, tunnelling all traffic
  → Remote access VPN: user's device ←→ corporate VPN gateway
  → Any scenario where you want to HIDE that two specific hosts are communicating
```

```
Tunnel Mode diagram — Site-to-Site VPN:

  Internal Host A            VPN Gateway A       VPN Gateway B       Internal Host B
  (10.0.1.5)                 (203.0.113.1)       (198.51.100.1)      (10.0.2.8)

  Sends normal packet:
  [ 10.0.1.5 → 10.0.2.8 | HTTP data ]

  Gateway A encapsulates into IPsec tunnel:
  [ 203.0.113.1 → 198.51.100.1 | ESP Header | Enc(10.0.1.5→10.0.2.8 | HTTP data) | ESP Auth ]
   ↑──────── Outer IP ─────────↑              ↑───────── Inner (hidden) ─────────────────────↑

  Over the internet, attackers see:
    Source: 203.0.113.1 (Gateway A)
    Dest:   198.51.100.1 (Gateway B)
    Content: ESP encrypted — unreadable

  They CANNOT see:
    → 10.0.1.5 is communicating with 10.0.2.8
    → What's inside (HTTP, FTP, database queries — all hidden)
    → Internal network addressing (10.0.x.x networks — hidden)

  Gateway B decapsulates, forwards the original packet to Host B.
```

---

### 5.3 Transport vs Tunnel — Comparison

| Property                    | Transport Mode                     | Tunnel Mode                          |
| --------------------------- | ---------------------------------- | ------------------------------------ |
| **What's protected**        | Payload only                       | Entire original packet               |
| **Original IP header**      | Preserved (visible)                | Encrypted and hidden                 |
| **New IP header added**     | ❌ No                              | ✅ Yes (outer header)                |
| **Packet size overhead**    | Small (one extra header)           | Larger (two IP headers)              |
| **Hides source/dest IPs**   | ❌ No — IPs visible to attackers   | ✅ Yes — original IPs hidden         |
| **Works through NAT**       | Difficult (especially with AH)     | ✅ Yes — outer header handles NAT    |
| **Typical use case**        | Host-to-host, internal network     | VPN, site-to-site, remote access     |
| **Hides internal topology** | ❌ No                              | ✅ Yes                               |
| **Processing overhead**     | Lower                              | Higher (two header sets)             |
| **Who terminates IPsec**    | The communicating hosts themselves | VPN gateway (can be separate device) |

```
Quick memory trick:
  TRANSPORT = the actual hosts TRANSPORT-ing to each other directly
             → host-to-host, payload protected
  TUNNEL     = a TUNNEL is dug through the internet hiding everything
             → gateway-to-gateway, entire packet hidden
```

---

## 6. IPsec in IPv4 vs IPv6

```
A common exam trap: students assume IPsec is IPv6-only. It is NOT.

  IPsec with IPv4:
    → IPsec was ORIGINALLY designed for IPv4
    → OPTIONAL in IPv4 — you choose whether to use it or not
    → Adding IPsec to IPv4 is done via protocol fields in the header
    → AH: Protocol field in IPv4 header set to 51
    → ESP: Protocol field in IPv4 header set to 50
    → Widely used in IPv4 VPNs today (Cisco, Juniper, OpenVPN, etc.)

  IPsec with IPv6:
    → IPsec is MANDATORY in IPv6 specification
    → Every IPv6 implementation MUST support IPsec
    → AH and ESP are built-in extension headers (Type 51 and Type 50)
    → IKEv2 is the required key exchange for IPv6 IPsec
    → In practice, whether IPsec is actively USED on IPv6 is configurable
      (mandatory support ≠ mandatory use)

  Bottom line:
    → IPsec works with BOTH IPv4 and IPv6
    → More deeply integrated in IPv6 (extension headers)
    → More bolted-on in IPv4 (uses Protocol field)
```

```
How IPsec headers appear in IPv4 vs IPv6:

  IPv4 + IPsec (AH):
  [ IPv4 Header (Protocol=51) | AH Header | Data ]

  IPv4 + IPsec (ESP):
  [ IPv4 Header (Protocol=50) | ESP Header | Encrypted Data | ESP Trailer | ESP Auth ]

  IPv6 + IPsec (AH):
  [ IPv6 Base Header (Next=51) | AH Extension Header | Data ]

  IPv6 + IPsec (ESP):
  [ IPv6 Base Header (Next=50) | ESP Extension Header | Encrypted Data | ESP Trailer | ESP Auth ]

  IPv6 + Both AH + ESP (maximum security):
  [ IPv6 Base Header (Next=51) | AH Header (Next=50) | ESP Header | Encrypted Data | ESP Trailer | ESP Auth ]
```

---

## 7. Security Associations (SA)

Before two parties can use IPsec, they must agree on the parameters — this agreement is called a **Security Association (SA)**:

```
Security Association (SA) — what it defines:

  A Security Association is a ONE-WAY agreement between two parties containing:

  1. SPI (Security Parameter Index):
     → A 32-bit number identifying this SA
     → Included in every AH/ESP packet header
     → Receiver uses SPI to look up which key/algorithm to use for THIS packet

  2. Destination IP address:
     → Who this SA applies to (the receiving end)

  3. IPsec protocol:
     → AH (51) or ESP (50) — which one is being used

  4. Encryption algorithm:
     → Which cipher: AES-256-GCM, AES-128-CBC, ChaCha20, etc.

  5. Authentication algorithm:
     → Which HMAC: HMAC-SHA256, HMAC-SHA512, etc.

  6. Encryption key:
     → The actual key used for encryption/decryption

  7. Authentication key:
     → The actual key used for HMAC computation

  8. Key lifetime:
     → When does this SA expire? (time-based or byte-count-based)
     → After expiry: re-negotiate new SA with fresh keys

  9. Mode:
     → Transport or Tunnel mode
```

```
Why SA is ONE-WAY:

  A ←→ B communication requires TWO Security Associations:
    SA1: A → B  (for traffic going FROM A TO B)
    SA2: B → A  (for traffic going FROM B TO A)

  Each direction uses different keys and may use different algorithms.
  Both SAs stored in the SAD (Security Association Database) at each endpoint.

  Related concept — SPD (Security Policy Database):
    → Defines WHAT traffic should be protected by IPsec
    → Rules like: "all traffic from 10.0.1.0/24 to 10.0.2.0/24 → use ESP tunnel mode"
    → Traffic not matching any SPD rule passes through unprotected (or is dropped)
```

---

## 8. How IPsec Works End-to-End — Full Flow

```
Complete IPsec connection flow (using IKEv2 + ESP Tunnel Mode):

PHASE 0 — Policy Check:
  Host A wants to send data to Host B.
  Host A checks SPD: "Is there a policy requiring IPsec for this traffic?"
  → YES: proceed with IKE negotiation
  → NO:  send without IPsec

PHASE 1 — IKE_SA_INIT (IKEv2 Phase 1):
  A → B: "I want to establish an IKE SA. Here are my supported algorithms,
          my DH public value, and a nonce."
  B → A: "OK. Here are my chosen algorithms, my DH public value, and nonce."
  Both sides: compute the same shared DH secret independently
  Both sides: derive IKE SA keys from the DH secret + nonces
  Result: SECURE CHANNEL established for Phase 2 negotiation

PHASE 2 — IKE_AUTH (IKEv2 Phase 2) — runs INSIDE Phase 1 secure channel:
  A → B: "Authenticate me (PSK / certificate). Propose IPsec SA parameters:
          ESP, AES-256-GCM, SHA-256, Tunnel Mode. Traffic selector: 10.0.1.0/24 ↔ 10.0.2.0/24"
  B → A: "Authenticated. Accepting ESP, AES-256-GCM. Here are SPI values."
  Both sides: derive IPsec SA keys from DH material
  Both sides: install SA in SAD (Security Association Database)
  Result: IPsec SAs established → data can flow

PHASE 3 — DATA FLOW:
  Host A → Gateway A: normal IP packet (10.0.1.5 → 10.0.2.8 | HTTP data)
  Gateway A:
    → Looks up SPD: this traffic needs ESP Tunnel Mode
    → Looks up SAD: finds the SA with SPI, key, algorithm
    → Encrypts the original packet with AES-256-GCM
    → Adds new outer IP header (203.0.113.1 → 198.51.100.1)
    → Adds ESP header with SPI and sequence number
    → Sends encrypted tunnel packet

  Internet: attacker sees only outer IP header + encrypted ESP payload

  Gateway B:
    → Receives packet, reads outer IP header
    → Sees Next Header = 50 (ESP)
    → Reads SPI from ESP header → looks up SA in SAD
    → Verifies ICV (integrity) → passes
    → Checks sequence number → not a replay → passes
    → Decrypts the payload with AES-256-GCM
    → Recovers original packet (10.0.1.5 → 10.0.2.8 | HTTP data)
    → Forwards to Host B (10.0.2.8) on the internal network

  Host B: receives the original HTTP request — completely unaware of IPsec
```

---

## 9. Attack Types IPsec Defends Against

```
Attack                What it does                          IPsec defence
──────────────────────────────────────────────────────────────────────────
Eavesdropping         Attacker reads packets in transit     ESP encryption
(Sniffing)            e.g., Wireshark on a shared network   → payload unreadable

Man-in-the-Middle     Attacker intercepts + modifies        AH/ESP HMAC
(MitM)                packets between A and B               → modification detected

IP Spoofing           Attacker uses A's IP to send          AH/ESP authentication
(Impersonation)       fake packets to B                     → HMAC fails without key

Replay Attack         Attacker re-sends captured packets    Sequence numbers
                      to repeat an action (e.g., transfer)  → already-seen seq rejected

Session Hijacking     Attacker takes over an established    AH/ESP authentication
                      session mid-communication             → injected packets rejected

Traffic Analysis      Attacker infers communication         ESP Tunnel Mode
                      patterns from IP addresses            → original IPs hidden

Denial of Service     Partially — IPsec itself doesn't      IKE rate limiting,
(DoS)                 stop DoS but can prevent              anti-DoS cookies (IKEv2)
                      packet injection that amplifies DoS
```

---

## 10. Attacking IPsec — Offensive Perspective

> **Note:** All techniques below are for educational purposes and legal testing on systems you own or have explicit written permission to test.

### 10.1 IKE Reconnaissance

```bash
# ike-scan: discover and fingerprint IKE services
# Available in Parrot OS / Kali Linux

# Basic IKE scan (IKEv1):
sudo ike-scan TARGET_IP

# IKEv2 scan:
sudo ike-scan --ikev2 TARGET_IP

# Aggressive Mode scan (reveals PSK hash):
sudo ike-scan --aggressive TARGET_IP

# What ike-scan tells you:
#   → Is IKE running? (port UDP/500)
#   → IKEv1 or IKEv2?
#   → Supported encryption transforms
#   → Vendor ID (identifies the software/vendor → known CVEs)
#   → In Aggressive Mode: the PSK hash (can be brute-forced!)
```

### 10.2 IKEv1 Aggressive Mode PSK Cracking

```bash
# IKEv1 Aggressive Mode sends PSK hash in cleartext during handshake
# This hash can be captured and cracked offline

# Step 1: Capture the Aggressive Mode handshake with ike-scan
sudo ike-scan --aggressive --pskcrack=/tmp/psk_hash.txt TARGET_IP

# Step 2: Crack the hash with psk-crack (part of ike-scan tools)
psk-crack /tmp/psk_hash.txt

# Step 3: Or use hashcat for GPU-accelerated cracking
# The hash format is HMAC-MD5 or HMAC-SHA1 depending on the negotiated transform
hashcat -m 5300 /tmp/psk_hash.txt /usr/share/wordlists/rockyou.txt

# Defence:
#   → Disable IKEv1 Aggressive Mode entirely — use IKEv2 only
#   → Use certificate-based authentication instead of PSK
#   → If PSK required: use a long, random PSK (32+ characters)
```

### 10.3 ESP Traffic Analysis (Even Without Decryption)

```bash
# Even when you can't decrypt ESP, metadata reveals a lot:

# Capture IPsec ESP traffic:
sudo tcpdump -i eth0 -n "esp" -w /tmp/ipsec_capture.pcap

# Analyse in Wireshark:
# Filter: esp
# Observe:
#   → Packet sizes (reveal type of traffic: VoIP packets are tiny+regular;
#     web browsing has variable sizes)
#   → Timing patterns (reveals communication patterns)
#   → Source/Dest IPs (in transport mode — reveals who is talking to who)
#   → SPI values (identify specific security associations)

# Traffic analysis attack steps:
# 1. Collect timing data: when packets are sent, how many, what size
# 2. Correlate with known application patterns
# 3. Even encrypted VoIP can reveal a conversation is happening
# 4. Websites can be fingerprinted by the pattern of encrypted packet sizes
```

### 10.4 Testing Your Own IPsec Configuration for Weaknesses

```bash
# Check what IPsec is running on your system (Parrot OS / Kali):
sudo ip xfrm policy   # Show IPsec policies (SPD)
sudo ip xfrm state    # Show IPsec SAs (SAD) — shows algorithms in use!

# Verify IKE daemon is running:
sudo systemctl status strongswan   # strongSwan IKE daemon (most common on Linux)
sudo systemctl status xl2tpd       # L2TP daemon (often paired with IPsec)

# Check your strongSwan configuration for weak settings:
cat /etc/strongswan.conf
cat /etc/ipsec.conf

# Common weaknesses to look for in config:
#   → DES or 3DES encryption (weak — use AES-256 instead)
#   → MD5 or SHA-1 HMAC (weak — use SHA-256 or SHA-512)
#   → IKEv1 Aggressive Mode enabled
#   → PSK shorter than 20 characters
#   → Key lifetime too long (should be hours, not days)
#   → PFS (Perfect Forward Secrecy) disabled

# Test your VPN endpoint with strongSwan (against your own lab):
sudo strongswan initiate vpn-connection-name

# Watch IKE negotiation in real time:
sudo journalctl -f -u strongswan
```

### 10.5 IPsec Bypass Techniques

```bash
# IPsec can sometimes be bypassed if not configured to cover ALL traffic:

# Scenario: IPsec policy only covers TCP traffic
# Attacker sends UDP traffic — it bypasses IPsec entirely
# Check for this with:
sudo ip xfrm policy
# Look at "selector" fields — what traffic is actually covered?

# Test: send traffic that might not match the IPsec selector
# From attacker machine: send ICMP to a host that should be IPsec-protected
ping TARGET_IP
# If ICMP replies come back WITHOUT going through IPsec → misconfiguration!

# Fragmentation bypass:
# Some IPsec implementations only apply policy to the FIRST fragment
# Send a fragmented packet where the actual content is in fragment 2
# This can sometimes slip through IPsec inspection

# IPv6 vs IPv4 bypass:
# IPsec policy may only cover IPv4
# Try reaching the same host via IPv6 — may bypass IPsec entirely
ping6 TARGET_IPv6_ADDRESS
```

---

## 11. 🧪 Practical Labs

### Lab 1 — Capture and Inspect IPsec Traffic (Wireshark)

```bash
# On Parrot OS — capture all IPsec-related traffic

# Step 1: Capture IKE negotiation traffic (UDP port 500 and 4500)
sudo tcpdump -i eth0 -n "(udp port 500 or udp port 4500)" -w /tmp/ike_capture.pcap

# Step 2: Capture ESP traffic
sudo tcpdump -i eth0 -n "esp" -w /tmp/esp_capture.pcap

# Step 3: Capture AH traffic
sudo tcpdump -i eth0 -n "ah" -w /tmp/ah_capture.pcap

# Step 4: Open captures in Wireshark
wireshark /tmp/ike_capture.pcap &

# In Wireshark — useful filters:
# isakmp          → IKE packets (both v1 and v2)
# esp             → ESP packets
# ah              → AH packets
# isakmp.version.major == 2   → IKEv2 only
# esp.spi == 0xYOUR_SPI       → specific Security Association

# What to look for:
#   IKEv2: IKE_SA_INIT → IKE_AUTH → CREATE_CHILD_SA exchanges
#   ESP: src IP, dst IP, SPI value — encrypted payload shown as raw bytes
#   AH: src IP, dst IP, SPI, sequence number — payload visible in plaintext
```

### Lab 2 — Set Up IPsec with strongSwan on Linux (Transport Mode)

```bash
# Install strongSwan on Parrot OS:
sudo apt update && sudo apt install strongswan strongswan-pki -y

# For a simple host-to-host transport mode test between:
#   Parrot OS host: 192.168.56.101 (your machine)
#   Metasploitable2: 192.168.56.102 (your VM)

# On Parrot OS — create /etc/ipsec.conf:
sudo tee /etc/ipsec.conf << 'EOF'
config setup
    charondebug="ike 2, knl 2, cfg 2"

conn host-to-host
    type=transport
    left=192.168.56.101          # your Parrot IP
    right=192.168.56.102         # Metasploitable2 IP
    authby=psk
    ike=aes256-sha256-modp2048!
    esp=aes256-sha256!
    keyexchange=ikev2
    auto=start
EOF

# Create /etc/ipsec.secrets (PSK):
sudo tee /etc/ipsec.secrets << 'EOF'
192.168.56.101 192.168.56.102 : PSK "YourStrongRandomPSKHere123456789!"
EOF

# Start strongSwan:
sudo systemctl start strongswan
sudo systemctl enable strongswan

# Initiate the connection:
sudo ipsec up host-to-host

# Verify IPsec is active:
sudo ipsec status
sudo ip xfrm state    # shows active SAs with algorithm details
sudo ip xfrm policy   # shows SPD policies

# Test: ping Metasploitable2 — traffic should now be ESP-protected
ping 192.168.56.102

# Capture and verify in Wireshark:
sudo tcpdump -i vboxnet0 -n "esp" -v
# You should see ESP packets — not ICMP — proving IPsec is working
```

### Lab 3 — Craft IPsec Packets with Scapy

```python
# Save as: ipsec_scapy_lab.py
# Run: sudo python3 ipsec_scapy_lab.py
# Purpose: Understand IPsec headers by crafting packets from scratch

from scapy.all import *

print("="*60)
print("IPsec PACKET CRAFTING LAB")
print("="*60)

TARGET = "127.0.0.1"  # loopback — safe, tests on your own machine

# ── Lab 3a: Craft an AH packet ──────────────────────────────────
# Note: Scapy's AH layer handles the structure but NOT the actual HMAC
# (that requires the key — this just shows the structure)

pkt_ah = (
    IP(src="192.168.1.1", dst="192.168.1.2") /
    AH(spi=0x1234, seq=1) /
    TCP(dport=80, sport=12345) /
    Raw(load=b"GET / HTTP/1.1\r\nHost: example.com\r\n\r\n")
)

print("\n[Lab 3a] AH Packet Structure:")
pkt_ah.show()
print(f"  Protocol number in IP header: {pkt_ah[IP].proto} (should be 51 = AH)")
print(f"  SPI: {hex(pkt_ah[AH].spi)}")
print(f"  Sequence Number: {pkt_ah[AH].seq}")
print(f"  Note: Payload is VISIBLE — AH does NOT encrypt!")

# ── Lab 3b: Craft an ESP packet ─────────────────────────────────
# Note: Without actual encryption keys, we simulate the structure
pkt_esp = (
    IP(src="192.168.1.1", dst="192.168.1.2") /
    ESP(spi=0x5678, seq=1) /
    Raw(load=b"\x00" * 48)  # Simulating encrypted payload (all zeros)
)

print("\n[Lab 3b] ESP Packet Structure:")
pkt_esp.show()
print(f"  Protocol number in IP header: {pkt_esp[IP].proto} (should be 50 = ESP)")
print(f"  SPI: {hex(pkt_esp[ESP].spi)}")
print(f"  Sequence Number: {pkt_esp[ESP].seq}")
print(f"  Payload: encrypted bytes — unreadable without the key")

# ── Lab 3c: Sequence number variation (replay attack simulation) ─
print("\n[Lab 3c] Replay Attack Simulation — Sequence Numbers:")
print("  Legitimate packets (increasing sequence numbers):")
for seq in [1, 2, 3, 4, 5]:
    pkt = IP(dst=TARGET) / ESP(spi=0xAAAA, seq=seq) / Raw(load=b"\x00" * 16)
    print(f"    Seq={seq}: {pkt.summary()}")

print("\n  Replay attack — resending seq=3 again:")
replay_pkt = IP(dst=TARGET) / ESP(spi=0xAAAA, seq=3) / Raw(load=b"\x00" * 16)
print(f"    Replayed Seq=3: {replay_pkt.summary()}")
print(f"    IPsec receiver would CHECK: 'I already saw seq=3 → DROP!'")

# ── Lab 3d: Tunnel Mode simulation ──────────────────────────────
print("\n[Lab 3d] Tunnel Mode Simulation:")
# Inner packet (what's being protected)
inner_pkt = IP(src="10.0.1.5", dst="10.0.2.8") / TCP(dport=443) / Raw(load=b"HTTPS data")
# Outer packet (what goes over the internet)
outer_pkt = IP(src="203.0.113.1", dst="198.51.100.1") / ESP(spi=0x9999, seq=1) / Raw(load=bytes(inner_pkt))

print(f"  Inner packet: {inner_pkt.summary()}")
print(f"  Outer packet: {outer_pkt.summary()}")
print(f"  What attacker sees: {outer_pkt[IP].src} → {outer_pkt[IP].dst} (ESP encrypted)")
print(f"  Attacker CANNOT see: 10.0.1.5 → 10.0.2.8 — hidden inside ESP payload")
```

### Lab 4 — IPsec Security Audit on Your Own Web Projects

```bash
# Test if your MERN/PERN web projects are running services
# that should be protected by IPsec or at least TLS

# Step 1: Check what's running on your dev machine
sudo ss -tlnp    # all listening TCP services
sudo ss -ulnp    # all listening UDP services

# Step 2: Check if any sensitive services are exposed without encryption
# Look for services on these ports:
#   22  — SSH (encrypted, but check if IPsec is also required)
#   3000 — Node.js dev server (HTTP — NOT encrypted)
#   5432 — PostgreSQL (check if TLS is enabled)
#   27017 — MongoDB (check if TLS is enabled)
#   6379 — Redis (usually no encryption by default!)

# Step 3: Capture traffic to your own MongoDB/Redis from localhost
# to see what goes over the wire unprotected:
sudo tcpdump -i lo -n -A "port 27017"  # MongoDB
sudo tcpdump -i lo -n -A "port 6379"  # Redis
# You may see database queries/responses in plaintext!

# Step 4: Verify your production-facing services use TLS
# Test with OpenSSL:
openssl s_client -connect yourwebsite.com:443
# Should show certificate chain and TLS handshake

# Step 5: Test if IPsec is protecting traffic to your VPS/server
# Check for ESP traffic when connecting:
sudo tcpdump -i eth0 -n "esp" -v
# If no ESP seen: traffic is unprotected at the IP layer (relies on TLS only)

# For database servers: consider IPsec transport mode
# to protect all traffic regardless of whether the app uses TLS
```

### Lab 5 — Identify and Fingerprint IPsec in the Wild

```bash
# Using ike-scan to safely probe lab targets (only targets you own)

# Install ike-scan:
sudo apt install ike-scan -y

# Basic IKEv1 probe:
sudo ike-scan 192.168.56.102   # your Metasploitable2

# Verbose output — see full negotiation:
sudo ike-scan -v 192.168.56.102

# Probe for IKEv2:
sudo ike-scan --ikev2 192.168.56.102

# Try multiple encryption transforms to find what's supported:
sudo ike-scan --trans=5,2,1,2 192.168.56.102
# 5,2,1,2 = 3DES, SHA1, pre-shared key, 768-bit DH (IKEv1 transform)

# Aggressive Mode — may reveal PSK hash:
sudo ike-scan --aggressive --id=test 192.168.56.102

# Scan results tell you:
#   "SA=(Enc=3DES Hash=SHA1 Auth=PSK Group=2:modp1024 LifeType=Seconds LifeDuration=28800)"
#   → Encryption: 3DES (WEAK — should be AES)
#   → Hash: SHA1 (WEAK — should be SHA-256)
#   → Auth: PSK (check if it's a weak PSK)
#   → DH Group 2 (768-bit, WEAK — should be Group 14+ = 2048-bit)

# Translate these findings into recommendations:
echo "Weak configuration found! Recommend:"
echo "  Encryption: upgrade to AES-256"
echo "  Hash: upgrade to SHA-256 or SHA-512"
echo "  DH Group: upgrade to Group 14 (modp2048) minimum"
echo "  Consider migrating to IKEv2"
```

---

## 12. Solved Examples

### Example 1 — Protocol Identification

**Question:** A packet arrives at a router. The IP header shows Protocol = 50. What does this mean and what is the packet doing?

```
Answer:
  Protocol number 50 = ESP (Encapsulating Security Payload)
  This is part of the IPsec protocol suite.

  The packet contains an ESP header, which means:
    → The payload is ENCRYPTED (confidentiality provided)
    → The sender's identity has been authenticated (HMAC in ICV field)
    → Data integrity is protected (HMAC detects tampering)
    → Replay attack protection is active (sequence number present)

  The router can:
    → Read: source IP, destination IP, SPI value from ESP header
    → NOT read: anything inside the encrypted payload
                (TCP/UDP ports, application data — all hidden)

  If the packet uses Tunnel Mode:
    → The original IP header (with real source/dest) is also hidden inside
    → The outer IP header shows VPN gateway addresses, not actual hosts
```

---

### Example 2 — Replay Attack Scenario

**Question:** A bank's server sends message "Transfer 5,000 BDT to account Y" protected by IPsec AH. An attacker captures this packet and resends it 10 times. Does the transfer happen 10 times? Explain.

```
Answer:
  NO — the transfer does NOT happen 10 times. Here is why:

  The original packet from the bank server:
    → Contains an AH header with Sequence Number = 47
    → ICV (HMAC) is computed over the message including seq=47
    → Bank server increments its sequence counter to 48 for the next packet

  When the attacker replays the captured packet (seq=47) 10 times:

  Replay 1: Receiver has already seen seq=47 → DISCARD
  Replay 2: seq=47 again → already in "received" window → DISCARD
  ...
  Replay 10: seq=47 still → DISCARD

  Why? The receiver maintains a sliding window of received sequence numbers.
  Once a sequence number has been accepted, any packet arriving with the
  same sequence number is automatically discarded as a replay.

  The attacker would need to forge a valid seq=48, 49, etc. with a valid
  HMAC — which requires the shared secret key. Without the key:
  → Cannot generate valid HMAC for new sequence numbers → attack fails ✅

  Conclusion: IPsec sequence numbers effectively prevent replay attacks.
```

---

### Example 3 — Transport vs Tunnel Mode

**Question:** A company has two offices — Office A and Office B. They want to connect them securely over the internet so employees at Office A can access resources at Office B. Should they use Transport Mode or Tunnel Mode? Justify.

```
Answer:
  They should use TUNNEL MODE. Justification:

  Scenario characteristics:
    → Two separate network sites connected over the internet
    → VPN gateways at each site (routers/firewalls)
    → Internal employees at each office need to communicate
    → Traffic traverses the PUBLIC internet (untrusted)

  Why Tunnel Mode:
    1. GATEWAY-TO-GATEWAY:
       The IPsec is terminated at the gateways, not individual hosts.
       Tunnel mode allows gateways to encrypt/decrypt on behalf of hosts.
       Hosts don't even need to know IPsec is happening.

    2. HIDES INTERNAL TOPOLOGY:
       In Transport Mode: original IP headers visible → attacker knows
       10.0.1.5 in Office A is talking to 10.0.2.8 in Office B.
       In Tunnel Mode: only gateway IPs are visible → internal addresses hidden.

    3. NAT TRAVERSAL:
       Internet routers use NAT. Transport Mode (especially AH) breaks under NAT.
       Tunnel Mode's outer IP header handles NAT correctly.

    4. SCALABILITY:
       Any host in Office A can reach any host in Office B through the tunnel.
       No need to configure IPsec on each individual workstation.

  Configuration summary:
    Gateway A (203.0.113.1) ←── IPsec ESP Tunnel Mode ──→ Gateway B (198.51.100.1)
    Traffic selector: 10.0.1.0/24 ↔ 10.0.2.0/24
    IKEv2, AES-256-GCM, SHA-256, DH Group 14 (modp2048), PFS enabled
```

---

### Example 4 — Security Exam (5-mark)

**Question:** Compare AH and ESP in IPsec. When would you choose AH, when ESP, and when both?

```
                AH (Authentication Header)    ESP (Encapsulating Security Payload)
Feature         Protocol 51                   Protocol 50
──────────────────────────────────────────────────────────────────────────────────
Confidentiality ❌ NO encryption               ✅ YES — payload fully encrypted
Authentication  ✅ YES — HMAC over packet      ✅ YES — HMAC in ICV field
Integrity       ✅ YES — detects tampering     ✅ YES — detects tampering
Replay protect  ✅ YES — sequence numbers      ✅ YES — sequence numbers
Header coverage Covers IP header fields too    Only covers payload (not IP header)
                (except mutable fields)
NAT compatible  ❌ POOR — NAT changes fields   ✅ GOOD — ESP works through NAT
                AH HMAC then fails
Overhead        Smaller (AH header only)       Larger (ESP header + trailer + pad)

──────────────────────────────────────────────────────────────────────────────────
USE AH WHEN:
  → You need to verify data integrity + source authentication
  → Privacy is NOT required (content can be seen)
  → No NAT in the path (AH breaks with NAT)
  → You want to authenticate even the IP header fields
  → Example: internal corporate network, no NAT, just need to verify packets
             haven't been tampered with or spoofed

USE ESP WHEN:
  → You need confidentiality (most common case)
  → Traffic crosses the internet (must encrypt against eavesdropping)
  → NAT is in the path (ESP works with NAT)
  → You need privacy + authentication + integrity all together
  → Example: VPN, remote access, any internet-facing IPsec deployment

USE BOTH AH + ESP WHEN:
  → Maximum security is required
  → ESP encrypts the payload; AH authenticates the outer IP header too
  → AH goes first (outer), then ESP header, then encrypted payload
  → Provides full header authentication (AH) + payload encryption (ESP)
  → Example: highly sensitive government or financial sector communications
  → Note: rare in practice due to NAT incompatibility of AH and overhead
  → In most real deployments: ESP alone is sufficient (AES-GCM handles both)
```

---

## 13. Exam Cheat Sheet

```
╔══════════════════════════════════════════════════════════════════════╗
║              IPsec — EXAM CHEAT SHEET                                ║
╠══════════════════════════════════════════════════════════════════════╣
║  WHAT IS IPsec?                                                       ║
║  → Collection / suite of protocols (NOT a single protocol)           ║
║  → Defined by: IETF (Internet Engineering Task Force)                ║
║  → Layer: Network Layer (Layer 3 — same as IPv4, IPv6, ICMP)        ║
║  → Works with: BOTH IPv4 (optional) and IPv6 (mandatory)            ║
╠══════════════════════════════════════════════════════════════════════╣
║  THREE PROTOCOLS IN IPsec SUITE                                       ║
║  ─────────────────────────────────────────────────────────          ║
║  ESP (50)  Encapsulating Security Payload                            ║
║            → Encryption ✅ Auth ✅ Integrity ✅ Replay ✅            ║
║            → MOST COMMONLY USED — provides everything                ║
║                                                                        ║
║  AH (51)   Authentication Header                                     ║
║            → Encryption ❌ Auth ✅ Integrity ✅ Replay ✅            ║
║            → Verify only — does NOT hide data                        ║
║            → BREAKS with NAT — use ESP instead for internet VPNs    ║
║                                                                        ║
║  IKE       Internet Key Exchange (IKEv1 / IKEv2)                    ║
║            → Safely negotiate and exchange keys BEFORE data flows    ║
║            → IKEv2 is current standard — faster, more secure        ║
║            → Runs on UDP port 500 (and 4500 for NAT traversal)      ║
╠══════════════════════════════════════════════════════════════════════╣
║  THREE SECURITY SERVICES IPsec PROVIDES                               ║
║  ─────────────────────────────────────────────────────────          ║
║  1. CONFIDENTIALITY   → no one reads data in transit (ESP)          ║
║  2. AUTHENTICATION    → verify sender + verify data not changed      ║
║     + INTEGRITY         (AH or ESP — uses HMAC)                     ║
║  3. REPLAY PROTECTION → sequence numbers prevent packet reuse        ║
║                                                                        ║
║  Mix and match: use all 3, or just 1 — modular design               ║
╠══════════════════════════════════════════════════════════════════════╣
║  TWO MODES OF OPERATION                                               ║
║  ─────────────────────────────────────────────────────────          ║
║  TRANSPORT MODE:                                                      ║
║    → Protects PAYLOAD only (original IP header kept visible)         ║
║    → Used for: host-to-host communication                            ║
║    → IP addresses visible — traffic analysis possible                ║
║    → Smaller overhead                                                 ║
║                                                                        ║
║  TUNNEL MODE:                                                         ║
║    → Protects ENTIRE original packet (new outer IP header added)    ║
║    → Used for: VPN, site-to-site, remote access                     ║
║    → Original IPs HIDDEN — maximum privacy                           ║
║    → Works through NAT ✅                                            ║
║    → Slightly larger overhead (two IP headers)                       ║
╠══════════════════════════════════════════════════════════════════════╣
║  AH vs ESP — KEY EXAM DIFFERENTIATOR                                  ║
║  AH:  Auth ✅  Integrity ✅  Encrypt ❌  NAT ❌  Proto=51           ║
║  ESP: Auth ✅  Integrity ✅  Encrypt ✅  NAT ✅  Proto=50           ║
║  Both: Replay protection via Sequence Numbers                        ║
╠══════════════════════════════════════════════════════════════════════╣
║  PROTOCOL NUMBERS (memorise)                                          ║
║  AH  = Protocol/Next Header 51                                       ║
║  ESP = Protocol/Next Header 50                                       ║
║  IKE = UDP port 500 (NAT traversal: UDP port 4500)                  ║
╠══════════════════════════════════════════════════════════════════════╣
║  SECURITY ASSOCIATION (SA)                                            ║
║  → Agreement on: algorithm, key, SPI, mode, lifetime                ║
║  → ONE-WAY: A→B needs one SA, B→A needs another SA                  ║
║  → Stored in: SAD (Security Association Database)                    ║
║  → Policies in: SPD (Security Policy Database)                      ║
║  → SPI identifies which SA to use for incoming packets              ║
╠══════════════════════════════════════════════════════════════════════╣
║  EXAM TRAPS                                                            ║
║  → IPsec is NOT IPv6-only — works with IPv4 too                     ║
║  → IPsec is a SUITE of protocols, not one single protocol            ║
║  → AH does NOT encrypt — it only authenticates                       ║
║  → AH BREAKS through NAT (NAT changes IP fields → HMAC fails)      ║
║  → Only SOURCE can fragment in IPv6 (routers cannot)                ║
║  → ESP hides TCP/UDP port numbers too (even from firewalls)          ║
║  → IKEv1 Aggressive Mode is VULNERABLE — PSK hash captured          ║
╠══════════════════════════════════════════════════════════════════════╣
║  ATTACKS AND DEFENCES                                                  ║
║  Eavesdropping → ESP encryption                                      ║
║  IP Spoofing   → AH/ESP HMAC authentication                         ║
║  MitM          → AH/ESP integrity check (HMAC mismatch = drop)      ║
║  Replay Attack → Sequence numbers (already-seen seq = drop)         ║
║  Traffic Analy.→ ESP Tunnel Mode (original IPs hidden)              ║
╠══════════════════════════════════════════════════════════════════════╣
║  MEMORY TRICKS                                                         ║
║  ESP → "Encrypt, Secure, Protect" → does the most, use by default   ║
║  AH  → "Authenticate Header" → auth only, no encrypt                ║
║  IKE → "I Keep the Keys" → manages key exchange before data flows   ║
║  Transport → "direct host-TO-host" → payload protected only         ║
║  Tunnel → "hides everything in a TUNNEL" → whole packet encrypted   ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## 📚 Next Topics

- [ ] **IPsec Transport Mode — Deep Dive** (packet diagrams, Wireshark analysis)
- [ ] **IPsec Tunnel Mode — Deep Dive** (VPN setup, gateway configuration)
- [ ] **IKEv2 in Detail** — exchange messages, MOBIKE, EAP authentication
- [ ] **strongSwan Configuration** — full site-to-site VPN lab on Parrot OS
- [ ] **L2TP/IPsec** — how L2TP tunnels inside IPsec for remote access VPN
- [ ] **OpenVPN vs IPsec vs WireGuard** — comparison for security deployments
- [ ] **IPsec in AWS / Azure** — cloud VPN gateway configuration
- [ ] **Firewall rules for IPsec** — ip6tables / iptables for AH, ESP, IKE

---

_Notes compiled from: Networking Course — IPsec Protocol (Gate Smashers)_
" "
_Environment: Parrot OS + Metasploitable2 + MERN/PERN Web Projects_
_Author: Cybersecurity Dept. Student_
