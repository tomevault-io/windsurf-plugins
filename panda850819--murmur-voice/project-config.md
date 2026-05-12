---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Dev Commands

```bash
# Dev mode (hot reload frontend, Rust recompiles on change)
pnpm tauri dev

# Production build
pnpm tauri build

# Production build with CUDA (Windows only, requires CUDA toolkit)
pnpm tauri build --features cuda

# Rust-only commands (run from src-tauri/)
cd src-tauri
cargo check                                       # fast iteration
cargo clippy --all-targets -- -D warnings          # lint (zero warnings policy)
cargo test --lib                                   # 36 tests (llm, settings, state, whisper, model, audio)
cargo test test_valid_forward_transitions          # run a single test
```

## Architecture

Tauri 2 desktop app: Rust backend (core process) + vanilla HTML/JS/CSS frontend (webview). Cross-platform (macOS + Windows).

```
Frontend (src/)                    Backend (src-tauri/src/)
├── index.html      main window    ├── lib.rs        app setup, commands, tray
├── settings.html   preferences    ├── audio.rs      cpal mic capture → 16kHz mono
├── preview.html    transcription  ├── whisper.rs    whisper-rs (Metal/CUDA)
├── onboarding.html first-run      ├── hotkey.rs     cfg dispatcher → _macos.rs / _windows.rs
├── events.js       event consts   ├── frontapp.rs   cfg dispatcher → _macos.rs / _windows.rs
├── i18n.js         localization   ├── clipboard.rs  arboard + rdev (cfg gates for paste key)
├── *.js            invoke/listen  ├── model.rs      ModelConfig + HuggingFace download
└── *.css                          ├── settings.rs   JSON persistence + key mapping
                                   ├── state.rs      recording state machine
                                   └── llm.rs        TextEnhancer trait + multi-provider LLM
```

**IPC**: Frontend calls backend via `invoke("command")` (uses `window.__TAURI__.core.invoke` via `withGlobalTauri`), backend pushes to frontend via `app.emit("event", payload)`.

**Four windows**: `main` (420x48, always-on-top bottom bar, starts hidden), `preview` (420x280, transcription result), `settings` (460x700, on-demand), `onboarding` (560x480, first-run only). `main` and `preview` are defined in `tauri.conf.json`; `settings` and `onboarding` are created dynamically via `WebviewWindowBuilder` in `lib.rs`. All must be listed in `src-tauri/capabilities/default.json` to invoke Tauri commands.

**NSPanel overlay (macOS)**: `main` and `preview` windows are converted to NSPanel via `tauri-nspanel` (v2.1 branch, git dep) at startup. Panel config: `NonactivatingPanel` style + `CanJoinAllSpaces | Stationary | FullScreenAuxiliary` + level 1001. This enables overlay on fullscreen apps. The `panel!` macro lives in `mod overlay_panel` with required traits in scope.

**Tray menu**: Settings, Show/Hide toggle, Quit. Show/Hide controls `main_visible` and sets `manual_show` flag to suppress auto-hide after transcription.

**Frontend is static files** served directly from `src/` (no build step, no bundler). Plain HTML/JS/CSS.

## Cross-Platform Pattern

Platform-specific modules use a cfg dispatcher pattern:

```rust
// hotkey.rs, frontapp.rs — thin dispatchers
#[cfg(target_os = "macos")]
#[path = "hotkey_macos.rs"]
mod platform;

#[cfg(target_os = "windows")]
#[path = "hotkey_windows.rs"]
mod platform;

pub(crate) use platform::*;
```

Each platform file exports the same public API. `clipboard.rs` uses inline `#[cfg]` gates instead (simpler, only the paste key differs: MetaLeft vs ControlLeft).

Platform-conditional deps in `Cargo.toml`:
- macOS: `whisper-rs` with `metal` feature
- Windows: `whisper-rs` CPU-only by default; `cuda` crate feature enables GPU acceleration (requires CUDA toolkit at build time). `windows` crate for Win32 APIs

## Recording Flow

```
Hotkey press → platform hotkey listener → channel → do_start_recording()
  → AudioRecorder::start() → spawn live transcription thread (local engine only, peek every 2s)
Hotkey release → do_stop_recording()
  → audio.stop()
  → if engine=groq: llm::transcribe_groq() (cloud Whisper API)
    else: whisper.transcribe() (local GPU)
  → if llm_enabled: create_enhancer() → TextEnhancer::enhance() (Groq/Ollama/Custom)
  → clipboard.insert_text(paste simulation) → emit result → preview window
  → auto-hide main+preview after 3s (unless manual_show)
```

State machine: `Idle → Starting → Recording → Stopping → Transcribing → [Processing] → Idle`

Two recording modes: **Hold** (press=start, release=stop) and **Toggle** (press toggles, 5-min auto-stop).

**Window visibility**: Main window starts hidden. Auto-shows when recording starts, auto-hides ~3s after transcription completes. `preview_generation: AtomicU64` prevents stale hide timers from closing newer results. If user manually shows via tray (`manual_show` flag), auto-hide is suppressed until next recording cycle.

**ESC cancel**: Global ESC key cancels active recording. Emits `recording_cancelled` event to frontend. Resets state to Idle without transcribing.

## Transcription Engines

- **Local**: whisper-rs (Metal GPU on macOS). Live preview enabled. Model stored at app data dir `/models/`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panda850819/murmur-voice](https://github.com/panda850819/murmur-voice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
