---
trigger: always_on
description: **Generated:** 2026-05-05
---

# NOCTURNED - PROJECT KNOWLEDGE BASE

**Generated:** 2026-05-05
**Commit:** d09ad40
**Branch:** main
**Related repos** (separate sibling checkouts; this repo is just the daemon source): `iap2-rs` (consumed via Cargo git dep), `nocturne-ui` (talks to this daemon over WS), `nocturne-app` (mobile companion, talks over BT), `nocturne-image` (Buildroot firmware that bakes this daemon in), `nocturne-ota` (update server), `nocturne-connector` (Pi-side Wi-Fi gateway).

## OVERVIEW

Rust daemon (`nocturned`) running on the Spotify Car Thing (armv7). Talks to iPhone over iAP2/Bluetooth RFCOMM (via the `iap2-rs` library), to the Car Thing UI over WebSocket port 5000, and to the mobile companion app over BT (iAP2 EA on iOS / RFCOMM SPP on Android). This repo is **private** and contains only the daemon source.

## BUILD

Cannot run on host — requires Car Thing hardware (`/dev/apple_mfi`, ALSA `hw:0,0`). Use `cargo check` for validation.

```bash
cargo check                  # Validate (cannot run on host)
just lint                    # cargo clippy --fix --allow-dirty && cargo fmt
just build                   # cross build --target=armv7-unknown-linux-gnueabihf --release
just copy                    # Build + scp binary to Car Thing at 172.16.42.2
```

### Cross-Compilation
- Target: `armv7-unknown-linux-gnueabihf` (Car Thing: arm64 kernel, armv7l userspace)
- Local: `just build` → `cross build --target=armv7-unknown-linux-gnueabihf --release`
- CI: `houseabsolute/actions-rust-cross` with SSH deploy key for private iap2-rs repo
- Pre-build deps: `libdbus-1-dev`, `libopus-dev` (installed via Cross.toml)

## STRUCTURE

```
nocturned-private/
├── src/                    # Rust daemon source (17 modules) — see src/AGENTS.md
├── Cargo.toml              # Binary crate manifest (binary: nocturned). iap2-rs pulled from GitHub SSH.
├── Cargo.lock
├── Cross.toml              # ARM cross-compilation config (libdbus-1-dev, libopus-dev pre-installed)
├── Justfile                # Build/lint/deploy commands
├── MFi.md                  # MFi authentication deep-dive (IOCTLs, cert format, challenge-response)
├── resources.zip           # Reverse-engineering artifacts (packet dumps, decompiled stock daemon, MFi spec)
└── target/                 # Cargo build output (gitignored)
```

**Wire boundaries** (consumers/producers of this daemon's APIs — separate repos, NOT subdirs):

- `iap2-rs` — iAP2 protocol library; consumed via Cargo git dependency on GitHub SSH (`ssh://git@github.com/usenocturne/iap2-rs.git`). To test against a local checkout, add a `[patch]` override in `Cargo.toml`.
- `nocturne-ui` — Car Thing web frontend. Connects to this daemon over WebSocket port 5000.
- `nocturne-app` — iOS/Android companion. Connects over Bluetooth (iAP2 EA on iOS, RFCOMM SPP on Android), both speaking MsgPack RPC handled by `src/app/msgpack.rs`.
- `nocturne-image` — Buildroot firmware. Bakes this daemon into the Car Thing rootfs at build time.
- `nocturne-ota` — OTA server. `src/app/msgpack.rs::download_ota_chunks_task` fetches signed SWU packages from there. Server URL configured in `/etc/nocturne/config.json` (loaded by `src/config.rs`).

## ARCHITECTURE

The daemon follows a layered protocol architecture:

```
main.rs
├── bluetooth.rs          RFCOMM listener & connection management
├── websocket.rs          WebSocket server for UI communication (port 5000)
├── audio.rs              Audio capture, Opus encoding & broadcast
├── wakeword.rs           ONNX wake word detection → triggers audio recording
├── app/                  Application layer
│   ├── mod.rs            App communication manager & message types
│   ├── msgpack.rs        MsgPack RPC handler (chunking, CRC32, EA commands)
│   └── websocket_handler.rs  WebSocket→iPhone command routing
├── iap2_wrapper.rs       Bridge to iap2-rs crate (config, events, EA session routing)
├── mfi.rs + mfi_impl.rs  MFi hardware chip interface (/dev/apple_mfi)
└── iap2-rs/ (external)   Protocol implementation
    ├── link.rs           Link layer: packet framing, SYN/ACK, sequence numbers
    ├── packet.rs         Binary packet encode/decode, CRC-8 checksums
    ├── auth.rs           MFi certificate authentication
    └── session/          Control, EA, file transfer, now playing, HID sessions
```

### Key Patterns

1. **Async Connection Handling**: Each iPhone connection spawns a separate async task progressing through link negotiation → MFi auth → identification → EA sessions
2. **Stateful Protocol Layers**: Link state machine (Idle → DetectSent → SynSent → Established), auth flow, session management — all in iap2-rs
3. **MsgPack Wire Protocol**: EA sessions use MsgPack RPC with 2000-byte chunking and CRC32 checksums
4. **Dual Transport**: Messages flow over both iAP2 (iOS) and SPP (Android) paths

### Subproject Relationships

```
nocturned (daemon)  ←── iAP2/BT/MsgPack ──→  nocturne-app (mobile, via EA/SPP)
       ↕ WebSocket (port 5000)
nocturne-ui (Car Thing display, served via Chromium kiosk)
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Daemon code | `src/` | Binary crate, 17 modules — see `src/AGENTS.md` |
| iAP2 protocol internals | `iap2-rs` repo | link, packet, session, auth layers (Cargo git dep, not in this repo) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usenocturne/nocturned](https://github.com/usenocturne/nocturned) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
