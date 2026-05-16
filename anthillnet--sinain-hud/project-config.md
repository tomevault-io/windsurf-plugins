---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

SinainHUD is a privacy-first AI overlay system for macOS. It captures screen and audio context, analyzes it with LLMs via OpenRouter, and displays real-time advice in an invisible overlay (invisible to screen capture/recording). It escalates to an OpenClaw agent system when deeper analysis is needed.

## Architecture

Three main processes communicate over localhost:

- **sinain-core** (Node.js/TypeScript, port 9500) — Central hub. HTTP + WebSocket server, agent analysis loop, escalation orchestrator, ring buffers (feed: 100 items, sense: 30 events). ES modules (`"type": "module"`).
- **overlay** (Flutter/Dart, macOS) — Private overlay UI. NSPanel with `sharingType = .none` (invisible to screen capture). 4 display modes: Feed, Alert, Ticker, Hidden. Connects via WebSocket to sinain-core.
- **sense_client** (Python) — Reads screen frames from sck-capture IPC (`~/.sinain/capture/frame.jpg`), SSIM change detection, OCR via OpenRouter vision API, privacy stripping. POSTs to sinain-core `/sense`.
- **sck-capture** (Swift, `tools/sck-capture/`) — Unified ScreenCaptureKit binary. Single `SCStream` captures both system audio (raw PCM → stdout → sinain-core AudioPipeline) and screen frames (JPEG → IPC → sense_client). Replaces separate Python SCKCapture + old sck-audio.

Data flow: `sck-capture → stdout PCM → sinain-core AudioPipeline → VAD → transcription → feed buffer → WebSocket → overlay`. Screen: `sck-capture → IPC JPEG → sense_client → OCR → POST /sense → sinain-core`.

Escalation: Agent loop scores digests against patterns. If score >= threshold (or rich/focus mode), escalates to OpenClaw gateway via HTTP+WebSocket.

## Build & Run Commands

### sinain-core (from `sinain-core/`)
```bash
npm install                    # Install dependencies
npm run dev                    # Watch mode with tsx (development)
npm run build                  # Compile TypeScript to dist/
npm start                      # Run compiled dist/index.js
npm run eval                   # Evaluation harness (3 runs, reports to eval/reports/)
npm run eval:quick             # Quick evaluation (1 run, stdout)
npx tsc --noEmit               # Type-check only (used in CI)
```

### overlay
```bash
# Pre-built (users — no Flutter needed):
npx @geravant/sinain setup-overlay    # Downloads .app/.exe from GitHub Releases
npx @geravant/sinain setup-overlay --update  # Force re-download

# From source (developers, from overlay/):
flutter pub get                # Install dependencies
flutter run -d macos --debug   # Run in debug mode (macOS)
flutter run -d windows --debug # Run in debug mode (Windows)
flutter build macos            # Production build (macOS)
flutter build windows          # Production build (Windows)
flutter analyze                # Dart static analysis
flutter test                   # Run widget tests
npx @geravant/sinain setup-overlay --from-source  # Clone + build
```

### Windows overlay (`overlay/windows/runner/`)
- `window_control_plugin.cpp` — Platform channel `sinain_hud/window` (mirrors Swift WindowControlPlugin)
- `hotkey_handler.cpp` — Platform channel `sinain_hud/hotkeys` (mirrors Swift AppDelegate hotkeys)
- Private overlay via `SetWindowDisplayAffinity(WDA_EXCLUDEFROMCAPTURE)` — Windows 10 2004+
- Hotkeys use `Ctrl+Shift` (not `Cmd+Shift`)

### sense_client (from project root)
```bash
python -m sense_client         # Run screen capture pipeline
```

### Full system
```bash
./start.sh                     # Launch all services (core + sense + overlay)
./start.sh --no-sense          # Skip sense_client
./start.sh --no-overlay        # Skip overlay
```

## CI Pipeline (`.github/workflows/ci.yml`)

CI (`ci.yml`) — two jobs:
1. **sinain-core-typecheck** — Node 22, `npm ci` + `npx tsc --noEmit`
2. **overlay-analyze** — Flutter 3.27.x, `flutter pub get` + `flutter analyze` + `flutter test`

Release (`release-overlay.yml`) — triggered by `overlay-v*` tags:
- Builds Flutter macOS release on `macos-latest`, zips with `ditto`, uploads to GitHub Releases

## Key Source Locations

### sinain-core (`sinain-core/src/`)
- `index.ts` — Entry point, service wiring
- `config.ts` — All env var loading (see `.env.example` for full list)
- `types.ts` — Shared TypeScript interfaces (FeedMessage, SenseEvent, AgentResult, etc.)
- `server.ts` — HTTP endpoints + WebSocket setup
- `agent/loop.ts` — Event-driven agent analysis loop (debounced 3s, max 30s)
- `agent/analyzer.ts` — LLM prompt builder + OpenRouter API calls
- `agent/context-window.ts` — Context assembly with richness presets
- `escalation/escalator.ts` — Escalation orchestration (largest file ~28KB)
- `escalation/scorer.ts` — Pattern-based scoring for escalation decisions
- `buffers/feed-buffer.ts` — Ring buffer for feed items
- `buffers/sense-buffer.ts` — Ring buffer for screen events
- `eval/` — Evaluation framework with LLM-as-Judge, JSONL scenarios

### overlay (`overlay/lib/`)
- `main.dart` — App entry, service init
- `core/services/websocket_service.dart` — WebSocket bridge with auto-reconnect
- `core/services/window_service.dart` — Platform channel to Swift native code

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [anthillnet/sinain-hud](https://github.com/anthillnet/sinain-hud) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
