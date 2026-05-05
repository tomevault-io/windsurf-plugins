---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**Prerequisites:** [Rust](https://rustup.rs/) (latest stable), [Bun](https://bun.sh/)

```bash
# Install dependencies
bun install

# Run in development mode
bun run tauri dev
# If cmake error on macOS:
CMAKE_POLICY_VERSION_MINIMUM=3.5 bun run tauri dev

# Build for production
bun run tauri build

# Linting and formatting (run before committing)
bun run lint              # ESLint for frontend
bun run lint:fix          # ESLint with auto-fix
bun run format            # Prettier + cargo fmt
bun run format:check      # Check formatting without changes
```

> **Note:** TTS models (Kokoro-82M) are downloaded at runtime by the user through the app UI — no bundled model setup is required for development.

## Architecture Overview

Parrot is a cross-platform desktop text-to-speech (TTS) app built with Tauri 2.x (Rust backend + React/TypeScript frontend).

### Backend Structure (`src-tauri/src/`)

**Core entry points:**

- `lib.rs` — Tauri setup, manager initialization, plugin wiring, command export via tauri-specta
- `main.rs` — Binary entry: CLI arg parsing, Linux GPU workaround, delegates to `lib.rs`
- `settings.rs` — `AppSettings` struct and persistence via `tauri-plugin-store` (`settings_store.json`)
- `cli.rs` — CLI argument definitions (clap derive macros)
- `signal_handle.rs` — Unix signal handlers (SIGUSR1/SIGUSR2) and shared `send_transcription_input()` helper

**Action pipeline:**

- `action_coordinator.rs` — Serializes shortcut lifecycle events through a single background thread; prevents concurrent TTS requests; debounces repeated keys (30ms)
- `actions.rs` — Action type definitions: `speak`, `cancel`, `play_pause`
- `audio_feedback.rs` — Plays start/stop audio cues via rodio
- `input.rs` — Enigo initialization for keyboard/mouse control (macOS requires accessibility permissions)

**Window management:**

- `overlay.rs` — Speaking overlay window lifecycle management
- `tray.rs` / `tray_i18n.rs` — System tray icon, context menu, and i18n-aware menu labels

**Utilities:**

- `shortcut.rs` — Global shortcut initialization and coordination
- `utils.rs` — Shared helpers (tray refresh, cancellation token management)

**Managers** (`managers/`):

- `tts.rs` — Core TTS engine: Kokoro model lifecycle, up to 2 parallel synthesis workers, crossfade between audio chunks (240 samples @ 24 kHz), audio playback via rodio/cpal, pause/resume, configurable model-unload timeout
- `model.rs` — Model catalog (Kokoro-82M), multi-component downloads with progress events, cancellation, extraction, and deletion
- `history.rs` — SQLite database (rusqlite) for TTS history: WAV file storage, configurable entry limit and retention policy

**Commands** (`commands/`):

- `mod.rs` — General: `get_app_settings`, `cancel_operation`, `toggle_tts_pause`, `preload_tts_model`, `get_model_status`, shortcut init/suspend/resume, permission helpers, `is_laptop`
- `audio.rs` — Audio devices: `get_available_output_devices`, `set_selected_output_device`, `play_test_sound`, `check_custom_sounds`
- `models.rs` — Models: `get_available_models`, `get_kokoro_voices`, `download_model`, `delete_model`, `set_active_model`, `cancel_download`
- `history.rs` — History: `get_history_entries`, `toggle_history_entry_saved`, `delete_history_entry`, `update_history_limit`, `update_history_retention_period`

**Audio Toolkit** (`audio_toolkit/`):

- `audio/device.rs` — Output device enumeration via cpal
- `audio/resampler.rs` — Audio resampling (rubato)
- `audio/utils.rs` — WAV file writing and audio utilities
- `constants.rs` — Shared audio constants

**Shortcut system** (`shortcut/`):

- Dual-backend architecture: **HandyKeys** (default on macOS) and **Tauri** via `tauri-plugin-global-shortcut` (default on Windows/Linux)
- Runtime switching via the `keyboard_implementation` setting; HandyKeys auto-falls back to Tauri with persistence on failure
- `handler.rs` — Routes shortcut events through ActionCoordinator

**Helpers** (`helpers/`):

- `clamshell.rs` — `is_laptop()`: detects laptop (clamshell) vs. desktop

### Frontend Structure (`src/`)

- `App.tsx` — Main shell: three-step onboarding (accessibility permissions → model selection → done), settings sidebar navigation, TTS error/no-selection toasts
- `bindings.ts` — **Manually maintained** Tauri command bindings; tauri-specta generates these only in debug builds — keep in sync by hand when adding/changing commands
- `stores/settingsStore.ts` — Zustand store for `AppSettings`; `settingUpdaters` map dispatches each settings key to the correct Rust command
- `stores/modelStore.ts` — Model download/selection state; listens to `download-progress` and `model-state-changed` backend events
- `hooks/useSettings.ts` — Thin wrapper around the settings store
- `components/settings/` — All settings UI, organized by feature area (40+ files)
- `components/model-selector/` — Model selection, download progress, and status UI
- `components/onboarding/` — First-run experience components
- `overlay/` — Separate Tauri window entry point for the speaking overlay shown during TTS playback

### Key Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rishiskhare/parrot](https://github.com/rishiskhare/parrot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
