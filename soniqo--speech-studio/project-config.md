---
trigger: always_on
description: This file is for any AI coding agent working in this repo (Claude, Codex, Cursor, Aider, etc.).
---

# Agent Instructions

This file is for any AI coding agent working in this repo (Claude, Codex, Cursor, Aider, etc.).

## Project

**speech-studio** — Speech Studio, a Soniqo project. Open-source desktop app for content creators.

**MVP scope.** Voice cloning + adjusting a cloned voice over a video timeline + emotional markers (style / prosody tags on the synthesized speech). The first end-to-end story:

1. Drop a short reference clip → clone the speaker.
2. Drop a video → extract / line up the existing dialogue.
3. Rewrite or re-record lines in the cloned voice, with inline emotion markers (e.g. `<whisper>`, `<excited>`, `<sad>`).
4. Preview against the video; export muxed output.

Status: the clone → script → synthesize → export pipeline works end to end. macOS runs the MLX engines — `CosyVoice3` by default, with `VoxCPM2`, `Qwen3-TTS`, `Chatterbox`, `OmniVoice`, `Indic-Mio`, and `Fish Audio S2 Pro` selectable from the toolbar; Windows/Linux run `VoxCPM2` and `Indic-Mio` through speech-core's LiteRT backend.

## Stack

**Tauri** (Rust shell + web frontend) wrapping the Soniqo speech engines.

- **Rust process** — Tauri app, owns the window, menu, file pickers, IPC, model lifecycle, file I/O. Talks to a voice-cloning TTS backend through a **sidecar** chosen at compile time per OS:
  - **macOS (Apple Silicon)** — `speech-swift` (Swift / MLX) via the `swift-sidecar/` binary. `CosyVoice3` is the default cloning engine; `VoxCPM2`, `Qwen3-TTS`, `Chatterbox`, `OmniVoice`, `Indic-Mio`, and `Fish Audio S2 Pro` are also selectable (all MLX).
  - **Windows / Linux (x86_64)** — `speech-core` (C++) via the `core-sidecar/` binary, cloning + cloned-voice TTS with the `VoxCPM2` and `Indic-Mio` **LiteRT** models through the C ABIs in `include/speech_core/voxcpm2_c.h` and `include/speech_core/indic_mio_c.h`.
  - **v1+** — `speech-core`'s broader C ABI (`speech_core_c.h`) for STT (Parakeet), VAD (Silero), noise suppression (DeepFilterNet3), audio utilities.
- **Web frontend** — **React + Vite**, rendered into the OS WebView (WKWebView on macOS, WebView2 on Windows, WebKitGTK on Linux). Owns the video timeline, voice-clone manager, script editor with emotion markers, and waveform views. Talks to Rust via Tauri `invoke()` commands and events.
- **Bridge mechanism** — a stateful **sidecar binary** bundled with the app. Tauri spawns it; Rust talks to it over stdin/stdout using an NDJSON protocol (one JSON object per line each way). The sidecar loads the model once and keeps it resident across calls, so per-line synthesis after warmup is fast. The protocol (`ping` / `init_model` / per-engine `synthesize_*`) is implemented by both sidecars — the Swift sidecar handles every macOS engine (`synthesize_voxcpm2` / `_cosyvoice` / `_chatterbox` / `_icl` / `_indic_mio`), and the C++ sidecar handles `synthesize_voxcpm2` and `synthesize_indic_mio` — so `SidecarManager` (`src-tauri/src/lib.rs`) only varies the binary path per OS. `init_model` carries the selected engine. Code: `swift-sidecar/` (Swift package, macOS) and `core-sidecar/` (CMake C++, Windows/Linux).

**Target platforms.**

- **v0: macOS (Apple Silicon)** — MLX engines via `swift-sidecar`: `CosyVoice3` (default, cloning + emotional markers) plus selectable `VoxCPM2`, `Qwen3-TTS`, `Chatterbox`, `OmniVoice`, `Indic-Mio`, and `Fish Audio S2 Pro`.
- **v0: Windows / Linux (x86_64)** — `VoxCPM2` and `Indic-Mio` via speech-core's LiteRT backend (`core-sidecar`). Same clone + emotion-marker story without MLX. ASR-graded retry isn't wired here yet (see Notes); the first successful take is accepted.

Why Tauri (vs Electron): smaller binaries, native shell, easier C++ FFI from Rust, desktop-first distribution. Matches the "deploy-anywhere" positioning.

**No Chromium, no Node in the shipped app.** WKWebView is part of macOS; the only JS that ships is our built bundle. Node lives on dev machines as a build-time toolchain (like Cargo) — never in the `.app`.

## Sibling repos under `~/repos/`

- **speech-core** — C++ engine. **v0 dependency on Windows/Linux**: the `core-sidecar` links its `speech_core_models_litert` static lib + the `libLiteRt` runtime and drives `VoxCPM2` via `include/speech_core/voxcpm2_c.h` and `Indic-Mio` via `include/speech_core/indic_mio_c.h`. Also the v1+ source of truth for VAD / STT / non-cloned TTS / enhancement. Build it with `-DSPEECH_CORE_WITH_LITERT=ON -DLITERT_DIR=... -DSPEECH_CORE_WITH_HF_DOWNLOAD=ON` first (see its `AGENTS.md` for the C ABI and CMake targets).
- **speech-swift** — speech models runtime for Apple Silicon (MLX / CoreML). The macOS voice-cloning backend: `CosyVoiceTTS` (default), `VoxCPM2TTS`, `Qwen3TTS` (ICL clone API in `Qwen3TTS+ICL.swift`), and `ChatterboxTTS` (multilingual, `Sources/ChatterboxTTS/`).
- **speech-models** — model artifacts on Hugging Face (`aufklarer/`). Studio bundles or downloads from here on first use.

## Build

Common: Rust 1.95+ via `rustup`, Node 20+ and `pnpm` 11+.

**macOS (Apple Silicon)** — Swift sidecar:
- macOS 15+, Xcode 26+ (check with `xcode-select -p`)

**Windows / Linux (x86_64)** — C++ sidecar linking speech-core:
- A C++17 toolchain + CMake 3.16+ (MSVC + Visual Studio on Windows; gcc/clang on Linux)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [soniqo/speech-studio](https://github.com/soniqo/speech-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
