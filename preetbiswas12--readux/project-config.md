---
trigger: always_on
description: 100% Peer-to-Peer (P2P), End-to-End Encrypted (E2EE) communication suite with **$0 infrastructure cost** and **zero centralized servers**. Uses mesh networking where every client is a node.
---

# Claude's Work Memory - Session Log Phase 4.12

## Project Aegis - Serverless P2P Chat (Zero Infrastructure Cost)

### Core Concept
100% Peer-to-Peer (P2P), End-to-End Encrypted (E2EE) communication suite with **$0 infrastructure cost** and **zero centralized servers**. Uses mesh networking where every client is a node.

**STATUS (March 18, 2026 - UPDATED):** ✅ **3 CRITICAL FIXES IMPLEMENTED** | ✅ **96% complete** | ✅ **Android Setup COMPLETE** | ✅ **GitHub Actions CI/CD LIVE** | 0 TypeScript errors

---

## 1. Architecture Overview

**Model:** Mesh/P2P (not Client-Server)
- **Identity:** Cryptographic key pairs (Public/Private), not database entries
- **Transport:** Direct UDP/TCP tunnels between devices via WebRTC
- **Storage:** Source of truth is on user's physical hardware
- **Key Principle:** Local-first logic with decentralized synchronization

---

## 2. Technical Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Framework** | React Native (Expo) | Cross-platform mobile |
| **Discovery** | GunDB + SEA | Decentralized user lookup & key mgmt |
| **Tunneling** | WebRTC (react-native-webrtc) | P2P video, audio, data channels |
| **Signaling** | GunDB Relay | Handshake swap without custom server |
| **Local DB** | SQLite (expo-sqlite) | Chat history, timestamps, media paths |
| **Encryption** | libsignal / SEA | AES-256 & RSA for E2EE |
| **NAT Traversal** | Google Public STUN | Firewall bypass ($0) |
| **DB Encryption** | SQLCipher | Local database encryption |

---

## 3. Feature Execution

### A. Authentication & Search (DHT instead of Database)
1. **Signup:** Generate unique keypair (Pub/Priv), user picks alias (@preet)
2. **Publication:** Announce to Gun network: `alias @preet -> pubKey XYZ`
3. **Search:** Crawl decentralized graph to find pubKey + IP/signaling data

### B. Text & Read Receipts (WebRTC Data Channels)
- Message encrypted with recipient's public key
- Timestamps generated locally (Lamport Timestamps for clock skew)
- **Double Tick:** ACK packet when received
- **Blue Tick:** READ packet when chat UI opened

### C. Audio & Video Calling
1. Call request via GunDB
2. Both exchange ICE candidates (network addresses)
3. Direct encrypted stream established (no third-party relay)

### D. File, Photo & Video Transfer
1. **Chunking:** Break into 16KB chunks
2. **Streaming:** Send sequentially over data channel
3. **Reconstruction:** Receive chunks, write to local filesystem

---

## 4. Notification Workaround (Mobile Background)

**Android:** Persistent foreground service with WebSocket/UDP listener (permanent notification icon required)

**iOS:** Background fetch cycles or Firebase trigger to wake app for P2P network checks

---

## 5. Security (E2EE Protocol)

- **Key Exchange:** ECDH (Elliptic Curve Diffie-Hellman)
- **Perfect Forward Secrecy:** New keys per session
- **Local Encryption:** SQLCipher database keyed to login password

---

## 6. R&D Roadmap

### Phase 1: The Handshake (Week 1-2)
- [ ] Integrate GunDB with public relay discovery
- [ ] BIP-39 seed generation & key derivation
- [ ] User signup (generate keypair, pick alias @username)
- [ ] Login/logout from stored seed phrase
- [ ] SEA authentication sign/verify

### Phase 2: The Tunnel (Week 3-4)
- [ ] Integrate react-native-webrtc
- [ ] WebRTC data channel for text messages
- [ ] SQLite local storage for chat history
- [ ] Offline message queue (pending table)
- [ ] GunDB presence detection for sync trigger
- [ ] Basic sender ↔ receiver text flow

### Phase 3: The Media (Week 5-6)
- [ ] WebRTC audio/video streams
- [ ] Libsignal Double Ratchet integration
- [ ] Call request UI (via GunDB signaling)
- [ ] STUN latency testing
- [ ] ICE candidate exchange
- [ ] Video/audio permission handling

### Phase 4: Persistence & Scale (Week 7-8)
- [ ] Android Foreground Service (AlwaysOnline mode)
- [ ] Battery Saver mode (15-min check intervals)
- [ ] File chunking for 10MB+ transfers
- [ ] Message history peer-to-peer sync (new device)
- [ ] Group chat full-mesh architecture
- [ ] Local error logging + crash export
- [ ] TURN fallback for strict NAT

---

## 7. Architectural Decisions (Design Constraints)

### 1. GunDB Relay Strategy
- **Model:** Hybrid Public Relays + Peer-as-Relay
- **Discovery:** Public relays (gun.eco, dweb.link) only for initial key lookup
- **Transport:** Once connected via WebRTC, peers act as their own relays
- **Privacy:** Never store unencrypted chat on public relays—use them to "find the boat, not store the cargo"

### 2. Offline Message Queuing
- **Model:** Local Queue (Sender-Side Persistence)
- **Sync:** Purely synchronous tunnels; messages queued locally until recipient comes online
- **Trigger:** GunDB "presence" feature signals when peer is online → WebRTC handshake → flush queue
- **Guarantee:** No messages sit on public relays waiting

### 3. Group Chats
- **Architecture:** Full Mesh (each participant owns local copy)
- **Scale:** Up to 10-15 people per group
- **Protocol:** Sender loops through all group member pubKeys, sends individually
- **Guarantee:** No shared "conversation document" on any server

### 4. TURN Servers for Strict NAT

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/preetbiswas12) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
