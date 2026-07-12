---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Meetily-Local** is a privacy-first AI meeting assistant that captures, transcribes, and summarizes meetings entirely on local infrastructure. It's a single self-contained Tauri desktop application — no separate backend server.

### Key Technology Stack
- **Desktop shell**: Tauri 2.11 (Rust) + Next.js 16 + React 19 + Tailwind 4
- **Audio Processing**: Rust (cpal, whisper-rs, professional audio mixing)
- **Transcription**: Whisper.cpp (local, GPU-accelerated, in-process via whisper-rs)
- **Persistence**: SQLite via sqlx in the Tauri Rust process
- **LLM Integration**: built-in llama.cpp sidecar (`llama-helper` crate), or remote Ollama / Claude / Groq / OpenRouter / OpenAI-compatible endpoint

## Essential Development Commands

### Frontend Development (Tauri Desktop App)

**Location**: `/frontend`

```bash
# macOS Development
./clean_run.sh              # Clean build and run with info logging
./clean_run.sh debug        # Run with debug logging
./clean_build.sh            # Production build

# Windows Development
clean_run_windows.bat       # Clean build and run
clean_build_windows.bat     # Production build

# Manual Commands
pnpm install                # Install dependencies
pnpm run dev                # Next.js dev server (port 3118)
pnpm run tauri:dev          # Full Tauri development mode
pnpm run tauri:build        # Production build

# GPU-Specific Builds (for testing acceleration)
pnpm run tauri:dev:metal    # macOS Metal GPU
pnpm run tauri:dev:cuda     # NVIDIA CUDA
pnpm run tauri:dev:vulkan   # AMD/Intel Vulkan
pnpm run tauri:dev:cpu      # CPU-only (no GPU)
```

### Service Endpoint
- **Frontend Dev**: http://localhost:3118 (Next.js Turbopack with HMR)

The Tauri Rust side has no HTTP listener — frontend ↔ Rust communication is
all in-process via `invoke()` commands and emitted events.

## High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Tauri Desktop App (single process)           │
│  ┌──────────────────┐    ┌──────────────────────────────────┐  │
│  │ Next.js UI       │    │ Rust core                        │  │
│  │ (React/TS)       │←──→│   • Audio capture + mixing + VAD │  │
│  │                  │    │   • whisper-rs / parakeet        │  │
│  └──────────────────┘    │   • SQLite via sqlx              │  │
│         ↑ Tauri events   │   • Summary engine               │  │
│         ↓ invoke()       │   • llama-helper sidecar         │  │
│                          └──────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
                ↓ optional outbound LLM calls
   Ollama (local or remote) / Claude / Groq / OpenRouter / custom OpenAI
```

### Audio Processing Pipeline (Critical Understanding)

The audio system has **two parallel paths** with different purposes:

```
Raw Audio (Mic + System)
         ↓
┌────────────────────────────────────────────────────────────┐
│              Audio Pipeline Manager                         │
│  (frontend/src-tauri/src/audio/pipeline.rs)                │
└─────────────┬──────────────────────────┬───────────────────┘
              ↓                          ↓
    ┌─────────────────┐        ┌─────────────────────┐
    │ Recording Path  │        │ Transcription Path  │
    │ (Pre-mixed)     │        │ (VAD-filtered)      │
    └─────────────────┘        └─────────────────────┘
              ↓                          ↓
    RecordingSaver.save()      WhisperEngine.transcribe()
```

**Key Insight**: The pipeline performs **professional audio mixing** (RMS-based ducking, clipping prevention) for recording, while simultaneously applying **Voice Activity Detection (VAD)** to send only speech segments to Whisper for transcription.

### Audio Device Modularization (Recently Completed)

**Context**: The audio system was refactored from a monolithic 1028-line `core.rs` file into focused modules. See [AUDIO_MODULARIZATION_PLAN.md](AUDIO_MODULARIZATION_PLAN.md) for details.

```
audio/
├── devices/                    # Device discovery and configuration
│   ├── discovery.rs           # list_audio_devices, trigger_audio_permission
│   ├── microphone.rs          # default_input_device
│   ├── speakers.rs            # default_output_device
│   ├── configuration.rs       # AudioDevice types, parsing
│   └── platform/              # Platform-specific implementations
│       ├── windows.rs         # WASAPI logic (~200 lines)
│       ├── macos.rs           # ScreenCaptureKit logic
│       └── linux.rs           # ALSA/PulseAudio logic
├── capture/                   # Audio stream capture
│   ├── microphone.rs          # Microphone capture stream
│   ├── system.rs              # System audio capture stream
│   └── core_audio.rs          # macOS ScreenCaptureKit integration
├── pipeline.rs                # Audio mixing and VAD processing
├── recording_manager.rs       # High-level recording coordination
├── recording_commands.rs      # Tauri command interface
└── recording_saver.rs         # Audio file writing
```

**When working on audio features**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Hankanman/Meetily-Local](https://github.com/Hankanman/Meetily-Local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
