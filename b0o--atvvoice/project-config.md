---
trigger: always_on
description: **atvvoice** is a Rust userspace daemon that captures voice audio from BLE TV remotes using the Google Voice over BLE (ATVV) protocol and exposes it as a PipeWire virtual microphone source on Linux.
---

# AGENTS.md - atvvoice

## Project Overview

**atvvoice** is a Rust userspace daemon that captures voice audio from BLE TV remotes using the Google Voice over BLE (ATVV) protocol and exposes it as a PipeWire virtual microphone source on Linux.

Target remotes: G20S Pro and any ATVV-compatible remote following the Google Reference Design. The daemon is generic - not tied to a specific remote model.

## Architecture

```
BLE Remote <--[BlueZ/D-Bus/GATT]--> atvvoice daemon --[PipeWire virtual source]--> Apps
```

Ten modules:

| Module            | File                     | Responsibility                                                         |
| ----------------- | ------------------------ | ---------------------------------------------------------------------- |
| BLE Discovery     | `src/ble.rs`             | Find ATVV devices, resolve GATT characteristics, AcquireNotify         |
| Protocol Types    | `src/protocol/types.rs`  | Strongly typed wire types: opcodes, codecs, stream IDs, reasons, etc.  |
| Protocol Trait    | `src/protocol/mod.rs`    | `Protocol` trait, `create_protocol()`, `get_caps_cmd()`, `parse_caps_resp()` |
| Protocol v0.4     | `src/protocol/v04.rs`    | v0.4 command encoding, CTL parsing, headered frame decoding            |
| Protocol v1.0     | `src/protocol/v10.rs`    | v1.0 command encoding, CTL parsing, headerless frame decoding, PTT/HTT |
| Session Loop      | `src/atvv.rs`            | Generic session state machine over `dyn Protocol`, `BleStreams` struct  |
| ADPCM Decoder     | `src/adpcm.rs`           | Stateful `AdpcmDecoder` struct + post-processing (declip, lowpass, gain) |
| PipeWire Source   | `src/pw.rs`              | Virtual audio source node (own thread, not async)                      |
| D-Bus Control     | `src/dbus.rs`            | Session bus interface for external mic control (optional feature)       |
| Consumer Events   | `src/consumer.rs`        | `ConsumerEvent` type for PipeWire consumer presence notifications      |
| CLI / Main        | `src/main.rs`            | CLI parsing, `negotiate()`, tokio runtime, reconnect loop, signal handling |

## Tech Stack

- **Language:** Rust (2021 edition)
- **Async runtime:** tokio (multi-threaded)
- **BLE:** `bluer` 0.17 (async BlueZ D-Bus bindings, feature `bluetoothd`)
- **Audio:** `pipewire` 0.9 (`pipewire-rs` bindings)
- **CLI:** `clap` 4 (derive)
- **Protocol types:** `num_enum` 0.7 (TryFromPrimitive/IntoPrimitive), `bitflags` 2 (codec bitmask)
- **Logging:** `tracing` + `tracing-subscriber` (env-filter)
- **Async utilities:** `futures` 0.3
- **Build:** Nix flake (crane or naersk), also buildable with plain `cargo`
- **Platform:** Linux only (NixOS primary target)

## ATVV Protocol Reference

### GATT Service

Service UUID: `AB5E0001-5A21-4F05-BC7D-AF01F617B664`

| Characteristic | UUID suffix | Direction     | Purpose                           |
| -------------- | ----------- | ------------- | --------------------------------- |
| TX             | `AB5E0002`  | Host → Remote | Commands (write without response) |
| RX             | `AB5E0003`  | Remote → Host | Audio data (notify)               |
| CTL            | `AB5E0004`  | Remote → Host | Control signals (notify)          |

### Commands (Host → Remote, written to TX)

| Command   | Bytes                      | Notes                                                        |
| --------- | -------------------------- | ------------------------------------------------------------ |
| GET_CAPS   | `0x0A 0x01 0x00 0x00 0x03 0x03` | Always v1.0: version, reserved, codecs (8k+16k), models (PTT+HTT) |
| MIC_OPEN   | `0x0C 0x00 0x01`           | **Big-endian** codec. `0x01 0x00` is WRONG and gets rejected |
| MIC_CLOSE  | `0x0D`                     |                                                              |
| MIC_EXTEND | `0x0E 0x00`                | Reset audio transfer timeout. stream_id=0x00 for MIC_OPEN-initiated streams. No response expected. |

### Control Signals (Remote → Host, received on CTL)

| Signal        | First byte | Action                             |
| ------------- | ---------- | ---------------------------------- |
| AUDIO_STOP    | `0x00`     | Stop streaming, send MIC_CLOSE     |
| AUDIO_START   | `0x04`     | Begin streaming                    |
| START_SEARCH  | `0x08`     | Mic button pressed - send MIC_OPEN |
| GET_CAPS_RESP | `0x0B`     | Capabilities response (log only)   |

### Session Flow

**Negotiation phase** (in `main.rs::negotiate()`):
1. Subscribe to CTL, RX, and device event streams
2. Send GET_CAPS (always v1.0) → receive GET_CAPS_RESP
3. Parse CAPS_RESP to determine remote's protocol version and capabilities
4. Create version-specific Protocol from negotiated capabilities

**Session phase** (in `atvv::run_session()`):
5. User presses mic → START_SEARCH
6. Send MIC_OPEN → AUDIO_START
7. Audio frames stream on RX (~30.8 fps, 134 bytes each)
8. User releases mic → AUDIO_STOP (or second START_SEARCH)
9. Send MIC_CLOSE


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [b0o/ATVVoice](https://github.com/b0o/ATVVoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
