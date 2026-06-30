---
trigger: always_on
description: **WebRTC-Demo** is a comprehensive, cross-platform WebRTC demonstration project that showcases real-time peer-to-peer communication capabilities across Web, Android, and iOS platforms. The project implements modern WebRTC APIs (currently using M125) to enable video calls, audio communication, screen sharing, and data channel messaging between multiple clients.
---

# AGENTS.md

## Project Overview

**WebRTC-Demo** is a comprehensive, cross-platform WebRTC demonstration project that showcases real-time peer-to-peer communication capabilities across Web, Android, and iOS platforms. The project implements modern WebRTC APIs (currently using M125) to enable video calls, audio communication, screen sharing, and data channel messaging between multiple clients.

## Architecture

### High-Level Architecture

The project follows a client-server architecture with the following components:

```
┌─────────────────────────────────────────────────────────────┐
│                       Clients                                │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │   Web    │    │  Android │    │   iOS    │              │
│  │ (Vue.js) │    │  (Native)│    │ (Native) │              │
│  └─────┬────┘    └─────┬────┘    └─────┬────┘              │
│        │               │               │                     │
│        └───────────────┴───────────────┘                     │
│                        │                                     │
│                        ▼                                     │
│              ┌─────────────────┐                            │
│              │  Signaling      │                            │
│              │  Server         │                            │
│              │  (Node.js +     │                            │
│              │   Socket.io)    │                            │
│              └─────────────────┘                            │
│                                                              │
│        P2P Connection (WebRTC - STUN/TURN)                  │
│        ════════════════════════════════════                  │
│        Client A ◄──────────────────► Client B               │
└─────────────────────────────────────────────────────────────┘
```

### Components

#### 1. Signaling Server (`signaling-server/`)

**Technology Stack:**
- Node.js
- Express.js
- Socket.io

**Purpose:**
The signaling server facilitates the initial peer discovery and exchange of connection information (SDP offers/answers and ICE candidates) between WebRTC clients. It does not handle the actual media streaming.

**Key Features:**
- Room-based architecture (max 2 participants per room)
- WebSocket-based real-time communication
- SDP offer/answer exchange
- ICE candidate relay
- Room management and participant tracking

**Protocol Flow:**
1. **Join Room**: Client connects and joins a room by ID
2. **Offer Exchange**: First client creates offer when second client joins
3. **Answer Exchange**: Second client responds with answer
4. **ICE Candidate Exchange**: Both peers exchange ICE candidates
5. **P2P Connection**: Direct peer-to-peer connection established

**Server Events:**
- `join room` - Client joins a specific room
- `offer` - Send WebRTC offer to peer
- `answer` - Send WebRTC answer to peer
- `new ice candidate` - Exchange ICE candidates
- `receive encryption key` - Exchange E2EE keys (web only)
- `send data channel message` - Relay data channel messages
- `leave room` - Leave the current room

#### 2. Web Client (`web/`)

**Technology Stack:**
- Vue.js 3 (Composition API)
- TypeScript
- Vite (build tool)
- Socket.io-client
- Native WebRTC API

**Architecture:**
```
App.vue
├── WebRTC PeerConnection Management
├── Socket.io Connection
├── Media Stream Management
│   ├── Local Camera/Microphone
│   ├── Screen Sharing
│   └── Remote Stream
├── Data Channel
└── E2EE (End-to-End Encryption)
    ├── Main Thread Implementation
    └── Web Worker Implementation
```

**Key Features:**

1. **Audio/Video Control**
   - Toggle microphone on/off
   - Toggle camera on/off
   - Switch between front/back camera (mobile)
   - Switch between microphone and speaker output

2. **Screen Sharing**
   - Desktop/tab/window sharing via `getDisplayMedia()`
   - Automatic track replacement in peer connection
   - Switch back to camera functionality

3. **Data Channel Messaging**
   - Real-time text messaging
   - Low-latency data transfer
   - Chat interface for peer-to-peer communication

4. **End-to-End Encryption (E2EE)**
   - **Current Status**: Only supported for Web-Web communication
   - **Algorithm**: AES-GCM with 256-bit keys
   - **Implementation Options**:
     - Main thread encryption/decryption
     - Web Worker-based encryption (offload processing)
   
   **E2EE Architecture**:
   ```
   Sender                           Receiver
   ──────                           ────────
   Media Frame
      │
      ▼
   Generate IV (12 bytes)
      │
      ▼
   AES-GCM Encrypt
      │
      ▼
   Append IV to encrypted data
      │
      ▼
   Send via WebRTC ──────────►  Extract IV
                                    │
                                    ▼
                                AES-GCM Decrypt
                                    │
                                    ▼
                                Render Frame
   ```

   **E2EE Process**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maitrungduc1410/WebRTC-Demo](https://github.com/maitrungduc1410/WebRTC-Demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
