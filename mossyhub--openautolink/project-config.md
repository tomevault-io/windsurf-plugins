---
trigger: always_on
description: Wireless Android Auto for AAOS head units. No external hardware required.
---

# OpenAutoLink — Project Guidelines

## What This Is

Wireless Android Auto for AAOS head units. No external hardware required.

```
Android Phone ──WiFi (phone hotspot)──▶ Car AAOS Head Unit
                                          ├── Companion App (Nearby Connections) → stream pipe
                                          ├── aasdk v1.6 (C++ JNI) → AA wire protocol
                                          ├── MediaCodec → video rendering
                                          ├── AudioTrack → 5-purpose audio
                                          └── VHAL/GNSS/IMU → sensor forwarding
```

Two components:
1. **App** (`app/`) — Kotlin/Compose AAOS app. Embeds aasdk C++ via JNI for AA protocol. Renders video/audio, forwards touch/GNSS/vehicle data
2. **Companion** (`companion/`) — Phone-side app. Uses Google Nearby Connections to create a stream pipe between phone AA and car app

> **Historical**: Bridge mode (SBC hardware) is preserved on the `bridge-mode` branch. Bridge code in `bridge/` is reference only on this branch.

## Performance Priorities

**Video, audio, and touch performance are the #1 priority.** Every design decision must optimize for:

1. **Fast initial render**: Connection → first video frame → audio playing must be as fast as possible. No unnecessary handshake delays, no lazy initialization on the hot path
2. **Stable streaming**: Zero dropped audio, minimal dropped video frames, immediate touch response. The car experience must feel native, not remote
3. **Seamless reconnection**: This is critical due to how cars work:
   - When the car "turns off", the AAOS head unit enters sleep/suspend — the app remains in memory at its last state
   - When the car turns back on, the app wakes up having lost its WiFi/Nearby connections
   - The app must retry patiently with a clean UI state (no error spam, no crash)
   - Once the phone is reachable (phone hotspot reconnects), reconnect automatically with no user interaction
   - First frame after reconnect must be clean — no black frames, no decoder artifacts, no partial/grainy frames. Wait for IDR before rendering
   - Audio must resume without pops, clicks, or stale buffer playback
   - The user experience should be: car on → brief "Connecting..." → projection appears. Indistinguishable from a fresh start

> **Design test**: If a feature adds latency to connection, first-frame, or reconnection — it needs exceptional justification.

## Cross-Component Rule: Always Reference the Other Side

When modifying **app Kotlin** code that interacts with the JNI layer, **read the C++ JNI code first** (`app/src/main/cpp/`). When modifying **JNI C++** code, read the Kotlin side first. When touching aasdk channel handling, **read the corresponding aasdk header** (`external/opencardev-aasdk/include/aasdk/Channel/`). Don't trust assumptions — verify what the code actually sends/receives.

## Architecture

### App (`app/`) — Component Islands

| Island | Responsibility | Test Anchor |
|--------|---------------|-------------|
| `transport/aasdk/` | aasdk JNI session (C++ pipeline), Nearby transport, AA protocol | Integration: companion app + phone |
| `transport/direct/` | Nearby Connections manager, legacy Kotlin AA protocol (being replaced) | Unit: Nearby mocks |
| `video/` | MediaCodec lifecycle, Surface rendering, codec detection | Unit: frame header parsing. Integration: decode test streams |
| `audio/` | Multi-purpose AudioTrack (5 slots), mic capture, ring buffer | Unit: purpose routing, ring buffer. Integration: PCM playback |
| `input/` | Touch forwarding, GNSS, vehicle data (VHAL), IMU sensors (accel/gyro/compass) | Unit: coordinate scaling, NMEA formatting. Integration: VHAL mock |
| `ui/` | Compose screens — projection surface, settings, diagnostics | Unit: ViewModel state. Integration: Compose test rules |
| `navigation/` | Nav state from aasdk, maneuver icons, cluster service | Unit: maneuver mapping. Integration: cluster IPC |
| `session/` | Session orchestrator — connects islands, manages lifecycle | Integration: full session with mock bridge |

- **Min SDK 32**, target SDK 36, Kotlin, Jetpack Compose, DataStore preferences
- **MVVM** with `StateFlow` — ViewModels own UI state, repositories own data
- Uses aasdk v1.6 AA protocol via JNI (C++ native library)
- **No USB adapter support** — WiFi/Nearby only

### C++ JNI Layer (`app/src/main/cpp/`)

| File | Purpose |
|------|---------|
| `aasdk_jni.cpp` | JNI entry point — native method registration |
| `jni_session.{h,cpp}` | aasdk pipeline: SSLWrapper → Cryptor → Messenger → channels. Control + video handler |
| `jni_channel_handlers.{h,cpp}` | Separate handler classes for audio, sensor, input, nav, mic, media, phone, BT |
| `jni_transport.{h,cpp}` | ITransport backed by Nearby streams via JNI readBytes/writeBytes |
| `stubs/` | libusb stub (USB not used on Android) |

### External Dependencies (`external/`)

| Submodule | Fork | Branch | Purpose |
|-----------|------|--------|---------|
| `external/opencardev-aasdk/` | [mossyhub/aasdk](https://github.com/mossyhub/aasdk) | `openautolink` | aasdk v1.6 with our NavigationStatus extensions |
| `external/opencardev-openauto/` | upstream `opencardev/openauto` | `main` | Reference only — not modified |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mossyhub/openautolink](https://github.com/mossyhub/openautolink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
