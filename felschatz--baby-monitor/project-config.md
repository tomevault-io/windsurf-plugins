---
trigger: always_on
description: Real-time baby monitor web app using WebRTC for direct or server-relayed streaming between two phones.
---

# Baby Monitor - Development Context

## Project Overview

Real-time baby monitor web app using WebRTC for direct or server-relayed streaming between two phones.
- **Sender** (`/sender/{session}`) - Baby's phone with camera/mic
- **Receiver** (`/receiver/{session}`) - Parent's phone viewing stream

Sessions isolate multiple monitors on the same server. Session name acts as a shared secret.

## Tech Stack

- **Runtime**: Node.js 21.7.x
- **Server**: Pure Node.js http module with optional server-side WebRTC relay
- **Signaling**: Server-Sent Events (SSE) + HTTP POST (no WebSockets)
- **Streaming**: WebRTC (RTCPeerConnection)
- **Frontend**: Vanilla JS, no frameworks

## Key Files

### Server Modules (`server/`)

| Module | Purpose |
|--------|---------|
| `server.js` | Thin wrapper for backwards compatibility, starts the server |
| `server/index.js` | HTTP server, main router, env loading |
| `server/relay-manager.js` | Server-side WebRTC relay bridge + client ICE config |
| `server/session-manager.js` | Session state (Map), cleanup |
| `server/sse-manager.js` | SSE setup, broadcast, heartbeats |
| `server/signal-router.js` | WebRTC signaling message handlers |
| `server/music-api.js` | Playlist scanning, name.txt parsing |
| `server/static-server.js` | File serving, MIME types, path security |
| `server/utils.js` | parseJsonBody, sendJson, matchRoute, generateId |

### Frontend Modules (`public/js/`)

**Shared modules** (used by sender & receiver):

| Module | Purpose |
|--------|---------|
| `keep-awake.js` | Wake lock API, NoSleep video, auto-shutdown timer |
| `session.js` | URL parsing, localStorage, session prompt |
| `signaling.js` | SSE connection, sendSignal(), reconnection |
| `webrtc.js` | Runtime ICE config loading, peer connection utilities |

**Sender modules**:

| Module | Purpose |
|--------|---------|
| `screen-dimming.js` | Inactivity timer, dim overlay |
| `music-player.js` | Playlist loading, shuffle, timer, playback |
| `sender-offline-sw.js` | Sender-only service worker under `public/js/` for cached lullaby playback |
| `echo-cancellation.js` | FFT, spectral subtraction, fallback mode |
| `sender-webrtc.js` | Offer creation, stream handling, PTT receive |
| `sender-app.js` | Main orchestration, event wiring |

**Receiver modules**:

| Module | Purpose |
|--------|---------|
| `audio-analysis.js` | Volume detection, RMS calculation, alerts |
| `video-playback.js` | Autoplay handling, track monitoring |
| `ptt.js` | Push-to-talk, audio ducking |
| `receiver-webrtc.js` | Answer creation, offer handling |
| `receiver-app.js` | Main orchestration, event wiring |

### HTML/CSS Files

| File | Purpose |
|------|---------|
| `public/sender.html` | Sender page structure, loads `sender-app.js` module |
| `public/receiver.html` | Receiver page structure, loads `receiver-app.js` module |
| `public/index.html` | Landing page with session input |
| `public/*.css` | Separate stylesheets for sender/receiver |

## Architecture Notes

- SSE used instead of WebSockets for simpler hosting compatibility
- Session-based isolation: each session has its own sender and receivers
- Session names are server-side only, never broadcast to clients
- Single sender per session, multiple receivers supported
- Optional relay mode bridges media through the Node server with paired WebRTC peer connections
- PTT (Push-to-Talk) works via WebRTC `replaceTrack()` (no renegotiation)
- Relay mode preserves PTT by bridging a dedicated audio transceiver through the server
- Audio ducking reduces baby audio to 15% during PTT
- STUN servers: stunprotocol.org, nextcloud.com, sipgate.net
- FFT-based spectral subtraction for music echo reduction
- Auto-shutdown: Sender stops after timeout set by receiver (manual, no auto default)

## Visual States

| State | Sender | Receiver |
|-------|--------|----------|
| Connected | Green background | Green background |
| Disconnected | Red/black blink | Red/black overlay "CONNECTION LOST" |
| Loud sound | - | Red/black overlay "LOUD SOUND DETECTED" |
| PTT active | Blue pulsing "👂 Parent is speaking..." | - |
| Music playing | Purple pulsing "🎵 [track name]" | Track name + timer |
| Shutdown active | Orange "🌙 Shutdown in H:MM:SS" | Orange info strip + countdown in drawer |
| Screen dim | Black overlay after 5s | - |

## Implementation Details

- Wake Lock API keeps screens on (with auto-shutdown timer)
- Auto-shutdown configured by receiver (manual; supports minutes/hours/seconds, uses short options when ENABLE_DEBUG_TIMER=true)
- Relay mode is selected on the start page and preserved via `transport=relay` on sender URLs; receivers learn it automatically from the active session
- In relay mode the server keeps one sender-facing and one receiver-facing peer connection per receiver
- AudioContext analyzes volume for loud sound detection
- Sensitivity slider controls threshold (saved to localStorage)
- Volume control persisted to localStorage (boost >100% routes audio through Web Audio and may break Bluetooth)
- Screen dims on sender after 5s inactivity to save battery
- Sender caches fetched playlist metadata and tracks in Cache Storage so lullabies can keep playing offline after one online warm-up

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [felschatz/baby-monitor](https://github.com/felschatz/baby-monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
