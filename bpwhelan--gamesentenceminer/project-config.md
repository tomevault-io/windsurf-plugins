---
trigger: always_on
description: GSM is a hybrid Electron + Python desktop application for Japanese language learning that automates flashcard creation from games. The app captures text (via text hooks or OCR), records audio with Voice Activity Detection (VAD), takes screenshots, and sends everything to Anki.
---

# GameSentenceMiner (GSM) AI Coding Agent Instructions

## Overview
GSM is a hybrid Electron + Python desktop application for Japanese language learning that automates flashcard creation from games. The app captures text (via text hooks or OCR), records audio with Voice Activity Detection (VAD), takes screenshots, and sends everything to Anki.

**Core Architecture**: Electron frontend (TypeScript) ↔ Python backend (spawned process) ↔ Flask web server (for UI pages)

## Project Structure & Key Boundaries

```
GameSentenceMiner/
├── electron-src/main/          # Electron main process (TypeScript)
│   ├── main.ts                 # Entry point, process lifecycle, Python spawning
│   ├── communication/          # Python IPC via stdin/stdout (GSMMSG: protocol)
│   ├── ui/                     # IPC handlers for each feature (yuzu, vn, steam, ocr, obs)
│   └── python/                 # Python installer/version management
├── GameSentenceMiner/          # Python backend (package root)
│   ├── gsm.py                  # Python main entry, async event loop, Qt GUI integration
│   ├── gametext.py             # Text capture (clipboard, websockets from texthooker/Agent)
│   ├── vad.py                  # Voice Activity Detection (Silero/Whisper processors)
│   ├── anki.py                 # Anki-Connect integration
│   ├── ocr/                    # OCR engines (OneOCR, Google Lens fork)
│   ├── web/                    # Flask app for web UI (database browser, stats)
│   ├── ui/                     # PyQt6 dialogs (config, screenshot selector, Anki confirmation)
│   └── util/                   # Config (TOML), database (SQLite), logging
├── GSM_Overlay/                # Separate Electron app for transparent overlay (Yomitan integration)
└── texthooker/                 # Forked Svelte UI for game text hooking
```

## Communication Patterns

### Electron ↔ Python IPC
- **Protocol**: Stdout messages prefixed with `GSMMSG:` followed by JSON: `{"function": "event_name", "data": {...}}`
- **Implementation**: `electron-src/main/communication/pythonIPC.ts` (GSMStdoutManager) parses stdout
- **Python side**: `GameSentenceMiner/util/communication/electron_ipc.py` sends messages via `print(f"GSMMSG:{json.dumps(msg)}")`
- **Key events**: `cleanup_complete`, `start`, `anki_result`, `open_settings`, `obs_started`, etc.

### Python Internal Architecture
- **Main loop**: `gsm.py::async_main()` runs asyncio event loop + Qt on main thread
- **Text pipeline**: `gametext.py` monitors clipboard/websockets → `add_line_to_text_log()` → `util/text_log.py` → Flask web UI
- **Audio pipeline**: OBS records → VAD trims → stored in `temp/` → sent to Anki
- **Configuration**: Single source of truth in `GameSentenceMiner/util/configuration.py` (Config dataclass), saved as TOML

## Critical Developer Workflows

### Running the App (Development)
```powershell
# Quick start (recommended)
npm install
npm run start  # Builds TypeScript + launches Electron

# Alternative: Watch mode for TS changes
npm run dev    # Terminal 1: tsc --watch
npm run start  # Terminal 2: Run app

# Restart Python backend without restarting Electron
# Use "Restart Python App" from app menu (File → Restart Python)
```

### Building for Distribution
```powershell
npm run app:dist  # Creates installer in dist/ using electron-builder
```

### Python Development
- **NO manual Python setup needed**: Electron downloads/manages Python via `electron-src/main/python/python_downloader.ts`
- **Dependencies**: Managed by `uv` (bundled), installed automatically on first run
- **Local edits**: Changes to `GameSentenceMiner/` are picked up immediately (uses local files, not installed package)
- **Package management**: `pyproject.toml` defines deps, `uv sync` installs them

### Configuration Management
- **Master config**: `GameSentenceMiner/util/configuration.py::MasterConfig` + `Config` dataclass
- **Storage**: `~/.gsm/` (default) or app directory, saved as `master_config.toml`
- **Profile system**: Multiple game profiles (e.g., "Default", "Game1"), switched via `switch_profile_and_save()`
- **UI**: PyQt6 config window (`GameSentenceMiner/ui/config_gui_qt.py`), opened via hotkey or menu

## Project-Specific Conventions

### Python Patterns
1. **Logging**: Use `from GameSentenceMiner.util.logging_config import logger`, NOT `logging` module. The logging system uses loguru for enhanced functionality.
2. **Config access**: ALWAYS use `get_config()`, NEVER access global `config` directly
3. **Database**: `GameSentenceMiner/util/db.py::SQLiteDB` for thread-safe ops, `SentenceDatabase` for main DB
4. **Async/Sync mixing**: `gsm.py` runs asyncio loop, Qt on main thread. Use `run_new_thread()` for blocking ops
5. **Error handling**: Errors in `gsm.py::async_main()` call `handle_error_in_initialization()` to keep process alive for Electron

### TypeScript Patterns
1. **IPC registration**: Each feature has `register*IPC()` function in `electron-src/main/ui/*.ts` (called from `main.ts`)
2. **Process spawning**: Use `getSanitizedPythonEnv()` when spawning Python (strips virtual env vars)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bpwhelan/GameSentenceMiner](https://github.com/bpwhelan/GameSentenceMiner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
