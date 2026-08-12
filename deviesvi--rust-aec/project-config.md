---
trigger: always_on
description: Real-time Acoustic Echo Cancellation (AEC) for Windows. Captures microphone + speaker loopback via WASAPI, runs WebRTC AEC3 (sonora), and outputs clean audio to a virtual audio cable so other apps (Discord, Zoom, Teams, etc.) can use it as their microphone input. Runs as a system tray application.
---

# Rust AEC — Agent Guidelines

## Project Overview

Real-time Acoustic Echo Cancellation (AEC) for Windows. Captures microphone + speaker loopback via WASAPI, runs WebRTC AEC3 (sonora), and outputs clean audio to a virtual audio cable so other apps (Discord, Zoom, Teams, etc.) can use it as their microphone input. Runs as a system tray application.

## Architecture

```
Main thread:       Win32 message pump + system tray icon (src/tray.rs)
Session monitor:   WASAPI session callbacks → Resume/Pause commands (src/audio/session_monitor.rs)
Engine thread:     AEC processing loop (src/engine.rs)
  loopback-capture:  WASAPI loopback → ref_ring (src/audio/loopback.rs)   ┐
  render:            out_ring → Virtual Cable (src/audio/render.rs)        ├─ ref pipeline stays warm across Pause
  mic-capture:       WASAPI capture → mic_ring (src/audio/capture.rs)     ┘   mic still stops on Pause
```

- **Main thread**: Runs Win32 message pump for the system tray icon. Sends `EngineCommand` to the engine thread via `crossbeam-channel`.
- **Engine thread**: Owns the AEC processor + audio threads + ring buffers. The reference pipeline (loopback-capture + render) and AEC state can stay warm across `EngineCommand::Pause`/`Resume`; only `MicCapture` is torn down on `Pause`, so the real microphone is released while long-lived allocations stay stable. While paused, loopback still drains WASAPI but discards data, render writes silence with `AUDCLNT_BUFFERFLAGS_SILENT`, and the engine thread blocks on the next command instead of polling. Resume latency for restarting mic capture is still ~50–100 ms.
- **Session monitor thread**: Registers `IAudioSessionNotification` only on capture endpoint(s) whose `ContainerID` matches the configured output cable. Each callback re-queries the OS for the live active session count; own-process sessions (rust-aec's MicCapture) are excluded by PID. `IAudioSessionEvents` keepers are deduplicated by session-instance ID and pruned on disconnect/expiry so repeated pause/resume or external session churn does not grow memory usage over time. Sends `Resume`/`Pause` to the engine only when state changes.
- **Inter-thread comms**: Lock-free SPSC ring buffers (`ringbuf` crate), 200ms capacity. Commands via `crossbeam-channel`.
- **Processing**: 10ms frames (480 samples @ 48kHz). AEC via `sonora` (pure-Rust WebRTC AEC3 port).
- **Audio API**: WASAPI directly via the `windows` crate (v0.58). Windows-only.

## Key Files

| File | Purpose |
|---|---|
| `src/main.rs` | CLI parsing, device selection (with cable filtering), tray + engine startup |
| `src/engine.rs` | `AudioEngine` — AEC processing loop, `RefPipeline`/`MicCapture` lifecycle, `EngineCommand` handling |
| `src/audio/session_monitor.rs` | COM callback session monitor — detects recording sessions via `IAudioSessionNotification`/`IAudioSessionEvents`, sends `Resume`/`Pause` to engine |
| `src/tray.rs` | Win32 system tray icon, context menus, `TrayState` shared with engine |
| `src/autostart.rs` | Registry-based Windows autostart (`HKCU\...\Run`) |
| `src/audio/device.rs` | WASAPI device enumeration, substring matching, virtual cable detection |
| `src/audio/capture.rs` | Mic capture thread (shared mode, event-driven, 10ms buffer) |
| `src/audio/loopback.rs` | Speaker loopback capture (`AUDCLNT_STREAMFLAGS_LOOPBACK`) |
| `src/audio/pcm.rs` | Reusable PCM scratch helpers for conversion/resampling/zero-fill |
| `src/audio/render.rs` | Writes clean audio to virtual cable render endpoint |
| `src/aec/mod.rs` | `AecProcessor` wrapping `sonora::AudioProcessing` |
| `src/sync/mod.rs` | `AudioRingBuf` — SPSC ring buffer wrapper |
| `build.rs` | Embeds `resources/app.ico` via `embed-resource` |
| `vendor/sonora-aec3/` | Local fork of `sonora-aec3` (v0.1.0) with off-by-one fix in `adaptive_fir_filter.rs::update_size` |

## CLI Usage

```
rust_aec.exe [--verbose] [mic_name] [speaker_name] [output_name]
```

- `--verbose` / `-v`: Open a dedicated console window for diagnostic output (device lists, buffer levels, peak levels every 2s). Ctrl+C in that window exits the app.
- All positional arguments are optional substring matches (case-insensitive).
- **mic_name**: Microphone device. Default: first real (non-virtual-cable) capture device.
- **speaker_name**: Speaker for loopback. Default: Windows default render device.
- **output_name**: Virtual cable output. Default: auto-detects device containing "cable".

## Virtual Audio Cable Setup (Required)

Install [VB-Audio Virtual Cable](https://vb-audio.com/Cable/) (free). It creates "CABLE Input" (render) and "CABLE Output" (capture) devices.

## Key Design Decisions

- **GUI subsystem (`#![windows_subsystem = "windows"]`)**: No console window on startup. With `--verbose`, allocates a dedicated console window via `AllocConsole()`. This gives reliable Ctrl+C support. Note: `AttachConsole(ATTACH_PARENT_PROCESS)` was attempted but GUI subsystem processes do not receive `CTRL_C_EVENT` even after attaching, so a dedicated window is the only reliable approach.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DeviesVi/rust-aec](https://github.com/DeviesVi/rust-aec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
