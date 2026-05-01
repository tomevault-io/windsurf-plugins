---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

VoiceFlow is a cross-platform voice-to-text paste utility built with Pyloid (Python desktop framework using PySide6/Qt WebEngine) and React. Users hold a hotkey to record audio, release to transcribe using faster-whisper, and the text is automatically pasted at the cursor. Supports Windows, Linux (Wayland/X11), and macOS.

## Commands

```bash
# Initial setup (installs both Node and Python dependencies)
pnpm run setup

# Development mode (runs Vite frontend + Pyloid backend concurrently)
pnpm run dev

# Development with hot-reload for Python changes
pnpm run dev:watch

# Build desktop application
pnpm run build

# Build platform installers (run on each target OS)
pnpm run build:installer          # Windows (.exe via Inno Setup)
pnpm run build:installer:linux    # Linux (.tar.gz + .AppImage)
pnpm run build:installer:macos    # macOS (.dmg)

# Run Python tests
cd VoiceFlow && uv run -p .venv pytest src-pyloid/tests/

# Run single test file
uv run -p .venv pytest src-pyloid/tests/test_transcription.py -v

# Run frontend only (for UI development)
pnpm run vite

# Lint frontend
pnpm run lint
```

## Architecture

### Backend (src-pyloid/)

Python backend using Pyloid framework with PySide6:

- **main.py** - Application entry point. Creates Pyloid app, tray icon, main dashboard window, and recording popup window. Sets up UI callbacks connecting backend events to popup state changes.
- **server.py** - RPC server using `PyloidRPC`. Exposes methods (`get_settings`, `update_settings`, `get_history`, etc.) that frontend calls via `pyloid-js` RPC.
- **app_controller.py** - Singleton controller orchestrating all services. Handles hotkey activate/deactivate flow: start recording -> stop recording -> transcribe -> paste at cursor -> save to history.

**Services (src-pyloid/services/):**
- `audio.py` - Microphone recording using sounddevice, streams amplitude for visualizer
- `transcription.py` - faster-whisper model loading and transcription
- `hotkey.py` - Global hotkey listener using keyboard library
- `clipboard.py` - Clipboard operations and paste-at-cursor using pyautogui
- `settings.py` - Settings management with defaults
- `database.py` - SQLite database for settings and history (stored at ~/.VoiceFlow/VoiceFlow.db)
- `logger.py` - Domain-based logging with hybrid format `[timestamp] [LEVEL] [domain] message | {json}`. Supports domains: model, audio, hotkey, settings, database, clipboard, window. Configured with 100MB log rotation.
- `model_manager.py` - Whisper model download/cache management using huggingface_hub. Provides download progress tracking (percent, speed, ETA), cancellation via CancelToken, daemon thread execution, and `clear_cache()` to delete only VoiceFlow's faster-whisper models.

### Frontend (src/)

React 18 + TypeScript + Vite frontend:

- **App.tsx** - Hash-based routing between `/popup`, `/onboarding`, and `/dashboard`. Checks model cache on startup and shows recovery modal if model is missing.
- **lib/api.ts** - RPC wrapper using `pyloid-js` to call Python backend methods. Includes model management APIs (`getModelInfo`, `startModelDownload`, `cancelModelDownload`).
- **lib/types.ts** - TypeScript interfaces for Settings, HistoryEntry, Stats, Options, ModelInfo, DownloadProgress
- **pages/** - Popup (recording indicator), Onboarding (includes model download step), Dashboard
- **components/** - Feature components plus shadcn/ui components in `components/ui/`
  - `ModelDownloadProgress.tsx` - Download progress UI with progress bar, speed, ETA, and retry support
  - `ModelDownloadModal.tsx` - Dialog wrapper for model downloads triggered from settings
  - `ModelRecoveryModal.tsx` - Startup modal for missing model recovery

### Frontend-Backend Communication

The frontend uses `pyloid-js` RPC to call Python methods:
```typescript
import { rpc } from "pyloid-js";
const settings = await rpc.call("get_settings");
```

Backend sends events to popup window via:
```python
popup_window.invoke('popup-state', {'state': 'recording'})
```

### Recording Flow

1. User holds hotkey (configurable, default Ctrl+Win)
2. `HotkeyService.on_activate` -> `AppController._handle_hotkey_activate` -> `AudioService.start_recording`
3. Popup transitions to "recording" state, shows amplitude visualizer
4. User releases hotkey
5. `AudioService.stop_recording` returns audio numpy array
6. `TranscriptionService.transcribe` runs faster-whisper
7. `ClipboardService.paste_at_cursor` pastes text
8. History saved to database
9. Popup returns to "idle" state

### Qt Threading Pattern

The `keyboard` library runs hotkey callbacks in a separate thread, but Qt requires UI operations on the main thread. The solution uses Qt signals/slots:

1. `ThreadSafeSignals` class in `main.py` defines signals (recording_started, recording_stopped, etc.)
2. Callback functions from `AppController` emit signals instead of directly updating UI
3. Signals connect to slot functions with `Qt.QueuedConnection` to ensure they run on the main thread
4. Slot functions safely update popup state and window properties

### Popup Window Transparency


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [infiniV/VoiceFlow](https://github.com/infiniV/VoiceFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
