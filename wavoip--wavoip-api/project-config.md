---
trigger: always_on
description: TypeScript library that integrates audio calls via Wavoip devices into web projects.
---

# Project Overview
TypeScript library that integrates audio calls via Wavoip devices into web projects.
Communicates with devices via WebSockets (Socket.IO) and standard Web APIs (WebRTC, AudioContext).

# Tech Stack

| Component | Technology |
|---|---|
| Language | TypeScript |
| Build / Test | Vite, Vitest |
| WebSocket | Socket.IO |
| HTTP | Axios |
| Media transport | WebRTC (official calls), WebSocket binary (unofficial calls) |
| Audio transcoding | LibSamplerateJs (via AudioWorklet) |
| Audio encoding | PCMU / µ-law G.711 |


# Folder Structure
```
src/
├── modules/
│   ├── call/           Call facades and internal event bus
│   ├── device/         Device connection, state and call construction
│   ├── media/          Audio transport (WebRTC / WebSocket) and MediaManager
│   ├── shared/         Shared primitives (EventEmitter)
│   └── worklets/       AudioWorklet processors (excluded from main TS build)
├── test/              Projects tests (Follow the same structure as modules)
│   ├── ...
├── Wavoip.ts           Public entry point — facade over all modules
└── index.ts            Public type/class exports
```


# Architecture Overview

## Layer diagram
```
Consumer
   │
   ▼
Wavoip              ← top-level facade; holds devices + MediaManager
   │
   ▼
DeviceConnection    ← one per device token; owns socket, builds call objects
   │  creates
   ├──▶ Offer          (incoming call, awaiting accept/reject)
   ├──▶ CallOutgoing   (outgoing call, awaiting peer answer)
   └──▶ CallActive     (call in progress, bidirectional audio)
          │  uses
          └──▶ ITransport  (WebRTCTransport | WebsocketTransport)
                    │  uses
                    └──▶ MediaManager  (mic, speaker, AudioContext)
```

## Key design decisions
- **Facades** — `Offer`, `CallOutgoing`, `CallActive` are the only objects returned to consumers.
  They encapsulate all internal state; consumers cannot mutate `Call`, `Device`, or transport directly.
- **CallBus** — internal normalized event bus (`src/modules/call/CallBus.ts`).
  Aggregates raw socket events (`DeviceSocket`) and transport events (`ITransport`) into a single
  typed stream. Facades subscribe to `CallBus` only — no direct socket/transport listener wiring.
- **No CallManager** — `DeviceConnection` is responsible for constructing all call objects
  (`Offer`, `CallOutgoing`, `CallActive`). It creates the `Call` model, `CallBus`, and the
  appropriate transport, then returns the facade to `Wavoip`.
- **Single MediaManager** — one `MediaManager` instance is shared across all `DeviceConnection`s
  and all active transports. It owns the `AudioContext` and all mic/speaker state.
- **EventEmitter-based event API** — All consumer-facing objects (`Wavoip`, `Device`/`DeviceConnection`,
  `Offer`, `CallOutgoing`, `CallActive`) expose a typed `on(event, callback)` method for subscribing
  to events. This allows multiple listeners per event and returns an `Unsubscribe` function.
  Legacy `on*()` convenience methods (e.g. `onStatus`, `onEnd`, `onPeerMute`) are **deprecated** —
  they limit to a single listener per event and silently overwrite the previous one on subsequent calls.
  Facades use internal `EventEmitter` composition: `CallBus` events are wired to a facade-scoped
  emitter so consumers never interact with bus internals directly.


# Modules

## `modules/call/`
Pure call-side facades and supporting types. No socket or transport code — receives everything via `CallBus`.

| File | Role |
|---|---|
| `CallBus.ts` | Internal event bus; normalizes socket + transport events for one call |
| `Offer.ts` | Facade for an incoming call awaiting accept/reject. Exports `OfferEvents` |
| `CallOutgoing.ts` | Facade for an outgoing call awaiting peer answer. Exports `CallOutgoingEvents` |
| `CallActive.ts` | Facade for an in-progress call. Exports `CallActiveEvents` |
| `Peer.ts` | `CallPeer` type (phone, displayName, profilePicture, muted) |
| `Stats.ts` | `CallStats` type (RTT, tx/rx packet counts and loss) |

## `modules/device/`
Device connection, state machine, and call object construction.

| File | Role |
|---|---|
| `DeviceConnection.ts` | Socket.IO connection per device; builds Offer/CallOutgoing/CallActive. Exports `DeviceEvents` |
| `Device.ts` | Holds device state (status, qrCode, contact); `canCall()` business rules |
| `Call.ts` | Call model and status state machine (RINGING → ACTIVE → ENDED etc.) |
| `WebSocket.ts` | Socket.IO factory + full `ServerEvents` / `ClientEvents` type definitions |

## `modules/media/`
Audio I/O and transport implementations.

| File | Role |
|---|---|
| `MediaManager.ts` | Mic/speaker enumeration, stream lifecycle, mute, hot-swap, AudioContext |
| `ITransport.ts` | Interface + event types shared by both transports |
| `WebRTC.ts` | `WebRTCTransport` — RTCPeerConnection, SDP, stats, mute detection via FFT |
| `WebSocket.ts` | `WebsocketTransport` — binary WebSocket, Int16 PCM decode, AudioWorklet pipeline, auto-reconnect |

## `modules/shared/`
| File | Role |
|---|---|
| `EventEmitter.ts` | Generic typed event emitter used throughout the project |

## `modules/worklets/` *(excluded from main TS build)*
| File | Role |
|---|---|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wavoip/wavoip-api](https://github.com/wavoip/wavoip-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
