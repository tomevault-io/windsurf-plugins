---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Meetily** is a privacy-first AI meeting assistant that captures, transcribes, and summarizes meetings entirely on local infrastructure. The project consists of two main components:

1. **Frontend**: Tauri-based desktop application (Rust + Next.js + TypeScript)
2. **Backend**: FastAPI server for meeting storage and LLM-based summarization (Python)

### Key Technology Stack
- **Desktop App**: Tauri 2.x (Rust) + Next.js 14 + React 18
- **Audio Processing**: Rust (cpal, whisper-rs, professional audio mixing)
- **Transcription**: Whisper.cpp (local, GPU-accelerated)
- **Backend API**: FastAPI + SQLite (aiosqlite)
- **LLM Integration**: Ollama (local), Claude, Groq, OpenRouter

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

### Backend Development (FastAPI Server)

**Location**: `/backend`

```bash
# macOS
./build_whisper.sh small              # Build Whisper with 'small' model
./clean_start_backend.sh              # Start FastAPI server (port 5167)

# Windows
build_whisper.cmd small               # Build Whisper with model
start_with_output.ps1                 # Interactive setup and start
clean_start_backend.cmd               # Start server

# Docker (Cross-Platform)
./run-docker.sh start --interactive   # Interactive setup (macOS/Linux)
.\run-docker.ps1 start -Interactive   # Interactive setup (Windows)
./run-docker.sh logs --service app    # View logs
```

**Available Whisper Models**: `tiny`, `tiny.en`, `base`, `base.en`, `small`, `small.en`, `medium`, `medium.en`, `large-v1`, `large-v2`, `large-v3`, `large-v3-turbo`

### Service Endpoints
- **Whisper Server**: http://localhost:8178
- **Backend API**: http://localhost:5167
- **Backend Docs**: http://localhost:5167/docs
- **Frontend Dev**: http://localhost:3118

## High-Level Architecture

### Three-Tier System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Frontend (Tauri Desktop App)                  │
│  ┌──────────────────┐  ┌─────────────────┐  ┌────────────────┐ │
│  │   Next.js UI     │  │  Rust Backend   │  │ Whisper Engine │ │
│  │  (React/TS)      │←→│  (Audio + IPC)  │←→│  (Local STT)   │ │
│  └──────────────────┘  └─────────────────┘  └────────────────┘ │
│         ↑ Tauri Events           ↑ Audio Pipeline               │
└─────────┼────────────────────────┼─────────────────────────────┘
          │ HTTP/WebSocket         │
          ↓                        │
┌─────────────────────────────────┼─────────────────────────────┐
│              Backend (FastAPI)  │                              │
│  ┌────────────┐  ┌─────────────┴──────┐  ┌────────────────┐  │
│  │   SQLite   │←→│  Meeting Manager   │←→│  LLM Provider  │  │
│  │ (Meetings) │  │  (CRUD + Summary)  │  │ (Ollama/etc.)  │  │
│  └────────────┘  └────────────────────┘  └────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zackriya-Solutions/meetily](https://github.com/Zackriya-Solutions/meetily) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
