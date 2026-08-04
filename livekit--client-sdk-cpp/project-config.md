---
trigger: always_on
description: Follow `cpp-tools/AGENTS.md` for shared C++ rules and this file for SDK-specific
---

# AGENTS.md — LiveKit C++ Client SDK

## Shared C++ Baseline

Follow `cpp-tools/AGENTS.md` for shared C++ rules and this file for SDK-specific
guidance.

Before C++ work, verify the shared guidance and root `.clang-format` /
`.clang-tidy` symlinks are present. If not, flag it and recommend
`git submodule update --init cpp-tools` or `./cpp-tools/install.sh` as
appropriate. Never use `--force` without approval or claim tooling verification
while these prerequisites are missing. Project-specific commands are documented
in `docs/tools.md`.

## Project Overview

This is **client-sdk-cpp**, the official LiveKit C++ client SDK. It wraps a Rust core (`client-sdk-rust/`) via a protobuf-based FFI bridge. All WebRTC, networking, and media logic lives in Rust; the C++ layer provides an ergonomic API for C++ consumers.

## Architecture

### Core Principle
Rust owns as much of the business logic as possible. The C++ layer should be a thin wrapper around the Rust core. If a feature may be used by another SDK it should be implemented in Rust. Since this is an SDK,
ensure backwards compatibility is maintained when possible. Do not update auto-generated code.

### Platform Support
The SDK must be supported on the following platforms:
- windows x64
- linux x86_64
- linux arm64
- macOS x86_64
- macOS arm64

### FFI Patterns
SDK features follow one of two FFI patterns:

**Synchronous calls:**
1. Serialize a protobuf `FfiRequest` message.
2. Send it to Rust via `FfiClient::instance().sendRequest(req)`.
3. Receive a `FfiResponse` with the result.
4. Expose the result through the C++ API.

**Asynchronous calls:**
1. Set up an async handler that listens for an event keyed by a `request_async_id`.
2. Serialize a protobuf `FfiRequest` message containing the `request_async_id`.
3. Send it to Rust via `FfiClient::instance().sendRequest(req)`.
4. Receive a synchronous `FfiResponse` (acknowledgement) and, later, an asynchronous `FfiEvent` callback with the actual result.
5. Expose the result through the C++ API.

When making larger scale changes, check with the developer before committing to architecture changes involving changes to the `client-sdk-rust/` submodule.

### Threading Model

The SDK has three categories of threads:

**FFI callback thread** — The Rust FFI layer calls `ffiEventCallback` from a Rust-managed thread (typically a Tokio runtime thread). This single entry point deserializes the `FfiEvent` and calls `FfiClient::pushEvent`, which:
1. Completes any pending async `std::promise` matched by `async_id`.
2. Invokes all registered `FfiClient` listeners (including `Room::onEvent`).

All `RoomDelegate` callbacks and stream handler callbacks (e.g., `registerTextStreamHandler`) are invoked on this FFI callback thread. **Handlers must not block**; spawn a background thread if synchronous work is needed.

**Per-subscription reader threads** — `SubscriptionThreadDispatcher` creates a dedicated `std::thread` for each active audio, video, or data track subscription. These threads block on `AudioStream::read()`, `VideoStream::read()`, or `DataTrackStream::read()` and invoke the registered `AudioFrameCallback`, `VideoFrameCallback`, or `DataFrameCallback` on that reader thread — not on the FFI callback thread. A hard limit of 20 concurrent reader threads is enforced.

**Application threads** — The calling thread for public API methods such as `Room::connect`, `LocalParticipant::publishTrack`, `AudioSource::captureFrame`, etc. These may block while waiting for FFI responses or future completion.

#### Thread-safety guarantees

| Component | Thread-safe? | Notes |
|-----------|-------------|-------|
| `FfiClient::sendRequest` | Yes (C++ side) | No C++ mutex; relies on the Rust FFI being safe for concurrent calls. Multiple threads may call concurrently. |
| `FfiClient` listener/async registration | Yes | Protected by internal `std::mutex`. |
| `Room` | Yes | Internal `std::mutex` protects all mutable state. `RoomDelegate` is called outside the lock. |
| `SubscriptionThreadDispatcher` | Yes | Internal `std::mutex` protects registrations and active readers. Thread joins happen outside the lock. |
| `AudioStream` / `VideoStream` / `DataTrackStream` | Yes | Internal `std::mutex` + `condition_variable` coordinate the FFI producer thread and the consumer reader thread. |
| `AudioSource::captureFrame` | No | Not safe to call concurrently from multiple threads. |
| `PlatformAudio` / `PlatformAudioSource` | Yes | Thin `sendRequest` wrappers over immutable FFI handle state; destruction and move operations must be externally synchronized. |
| `VideoSource::captureFrame` | No | Not safe to call concurrently from multiple threads. |
| `LocalAudioTrack` / `LocalVideoTrack` | No | Thin `sendRequest` wrappers with no internal synchronization. |
| `LocalDataTrack::tryPush` | No | Thin `sendRequest` wrapper with no internal synchronization. |
| `TextStreamWriter` / `ByteStreamWriter` | Serialized | `write()` is serialized by an internal `write_mutex_`. |

### Directory Layout
Be sure to update the directory layout in this file if the directory layout changes.

| Path | Description |
|------|-------------|
| `include/livekit/` | Public API headers (what SDK consumers include) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [livekit/client-sdk-cpp](https://github.com/livekit/client-sdk-cpp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
