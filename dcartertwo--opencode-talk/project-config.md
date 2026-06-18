---
trigger: always_on
description: This document helps AI agents understand and contribute to the OpenCode Talk codebase.
---

# AGENTS.md

This document helps AI agents understand and contribute to the OpenCode Talk codebase.

## Project Overview

**OpenCode Talk** is a macOS desktop app that provides a voice interface for [OpenCode](https://opencode.ai). Users speak naturally, the app sends requests to OpenCode, and responses stream back as both text and speech.

### Why Voice?

- **Hands-free interaction** - Talk while sketching, pacing, or keeping hands on the keyboard
- **Lower friction** - Speak intent naturally instead of crafting prompts
- **Real-time conversation** - Streaming text + voice creates natural dialogue
- **Privacy by default** - Local TTS keeps code conversations on your machine
- **High-quality voice** - Natural speech you want to listen to

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Framework** | Tauri 2 (Rust backend + React frontend) |
| **Frontend** | React 19, TypeScript, Zustand, Tailwind CSS 4 |
| **Backend** | Rust (async with Tokio) |
| **TTS** | Kokoro (primary), Piper (fallback), macOS `say` (fallback) |
| **STT** | SuperWhisper + Macrowhisper |

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         OpenCode Talk                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                   React Frontend                         │    │
│  │  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐   │    │
│  │  │ voice-      │  │ sentence-    │  │ stores/       │   │    │
│  │  │ bridge.ts   │  │ buffer.ts    │  │ *.ts          │   │    │
│  │  └──────┬──────┘  └──────┬───────┘  └───────────────┘   │    │
│  └─────────┼────────────────┼──────────────────────────────┘    │
│            │                │                                    │
│            │ invoke()       │                                    │
│            ▼                ▼                                    │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │                    Tauri Rust                            │    │
│  │  ┌─────────────┐  ┌──────────────┐  ┌───────────────┐   │    │
│  │  │ lib.rs      │  │ tts.rs       │  │ transcription │   │    │
│  │  │ (commands)  │  │ (audio)      │  │ _server.rs    │   │    │
│  │  └─────────────┘  └──────┬───────┘  └───────┬───────┘   │    │
│  └──────────────────────────┼──────────────────┼───────────┘    │
│                             │                  │                 │
└─────────────────────────────┼──────────────────┼─────────────────┘
                              │                  │
                              ▼                  ▼
                    ┌──────────────┐    ┌───────────────┐
                    │ Kokoro       │    │ Macrowhisper  │
                    │ Server :7892 │    │ POST :7891    │
                    └──────────────┘    └───────┬───────┘
                                                │
                                                ▼
                                       ┌───────────────┐
                                       │ SuperWhisper  │
                                       │ (system-wide) │
                                       └───────────────┘

External:
┌─────────────────┐
│ OpenCode Server │ ◄── SSE streaming ──► voice-bridge.ts
│ :4096           │
└─────────────────┘
```

### Voice Input Flow

```
SuperWhisper (system-wide STT)
    │
    ▼ (sends transcription via Macrowhisper)
Macrowhisper
    │
    ▼ HTTP POST :7891
transcription_server.rs
    │
    ▼ Tauri event
voice-bridge.ts → processVoiceInput()
    │
    ▼
sendMessage() → OpenCode
```

### Voice Output Flow

```
OpenCode Server
    │
    ▼ SSE (Server-Sent Events)
voice-bridge.ts (sendMessageStreaming)
    │
    ▼ text deltas
sentence-buffer.ts
    │
    ▼ complete sentences
invoke('speak_sentence')
    │
    ▼ Rust tts.rs (generation task)
Kokoro server :7892 (or Piper fallback)
    │
    ▼ audio file path
Audio thread (rodio Sink)
    │
    ▼ seamless playback
```

### Kokoro Server (Subcomponent)

The Kokoro TTS server keeps the model loaded in memory for fast responses.

| Property | Value |
|----------|-------|
| **Location** | `src-tauri/scripts/kokoro_server.py` |
| **Port** | 7892 |
| **Cold start** | ~5 seconds (model loading) |
| **Warm response** | ~0.3 seconds per sentence |
| **API** | `POST /tts` (JSON: text, voice, speed) |
| **Health** | `GET /health` |
| **Fallback** | Piper TTS if server unavailable |

Started automatically by `lib.rs` on app launch.

## Key Files Reference

### Frontend (`src/`)

| File | Purpose |
|------|---------|
| `lib/voice-bridge.ts` | Main integration - SSE streaming, sendMessage(), connects to OpenCode |
| `lib/sentence-buffer.ts` | Buffers streaming text, emits complete sentences for TTS |
| `lib/response-formatter.ts` | Formats OpenCode responses for speech |
| `lib/confirmation.ts` | Handles confirmation dialogs for dangerous actions |
| `stores/conversation.ts` | Messages, streaming state, voice state (Zustand) |
| `stores/settings.ts` | TTS engine, voice, speed, server URL (persisted) |

### Backend (`src-tauri/`)

| File | Purpose |
|------|---------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dcartertwo/opencode-talk](https://github.com/dcartertwo/opencode-talk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
