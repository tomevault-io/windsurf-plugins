---
trigger: always_on
description: Guidance for Claude Code (claude.ai/code) when working in this repository.
---

# CLAUDE.md

Guidance for Claude Code (claude.ai/code) when working in this repository.

> **Authority note.** `AGENTS.md` at the repo root remains the day-to-day operator handbook and takes precedence for command flags, session notes, and short-lived context. This file focuses on stable architectural facts and conventions.

---

## Project Overview

**SafeMask** is a Tauri v2 desktop application for industrial-grade local privacy masking, version **2.0.0**. All processing — rules, regex, AI NER, records — runs **100% offline**. There are no telemetry or remote reporting paths in the codebase.

- **Backend**: Rust 2024 edition, Tauri v2, ONNX Runtime (`ort` 2.0.0-rc.12), Rayon, mmap
- **Frontend**: React 19 + TypeScript + Zustand + Tailwind v3 + Vite 6 (no routing library — tab state in Zustand)
- **AI model**: `openai/privacy-filter` q4-quantized ONNX, auto-discovered under `src-tauri/models/privacy-filter/` (multiple search paths, see below)
- **Distribution**: Windows (MSI + portable ZIP), macOS (DMG, arm64), Linux (AppImage + deb) via `tauri-action` on `v*` tag push

The frontend stack is **not** Vue 3 / Pinia. Any older document that says so is out of date.

---

## Development Commands

```bash
# Install dependencies
npm install

# Full-stack dev (Vite + Tauri window)
npm run tauri dev

# Frontend-only dev server (127.0.0.1:18924, strictPort)
npm run dev

# Frontend typecheck + build (tsc, NOT vue-tsc)
npm run build

# Production bundle
npm run tauri build
```

**Rust commands — run from the repo root, not `src-tauri/`.** The repo is a Cargo workspace (resolver 2), and `src-tauri` is a member named `SafeMask`.

```bash
cargo check   -p SafeMask
cargo fmt     -p SafeMask
cargo clippy  -p SafeMask -- -D warnings
cargo test    -p SafeMask
cargo test    -p SafeMask test_name -- --nocapture
```

- Rust tests live **inline** in `#[cfg(test)]` modules — there is no `tests/` directory.
- `.cargo/config.toml` currently pins an HTTP proxy at `127.0.0.1:7890`. CI strips it (`rm -rf .cargo/config*`); if you don't run that proxy locally, either remove/comment the file or override with your own.

---

## Repository Layout

```
SafeMask/                        # Cargo workspace root
├── src/                         # React 19 frontend
│   ├── App.tsx                  # Root component (tab switch, event subscriptions, lazy pages)
│   ├── main.tsx                 # ReactDOM.createRoot entry, imports style.css
│   ├── components/
│   │   ├── dashboard/           # FileProcessor, StatCard
│   │   ├── feedback/            # MagicFeedback
│   │   ├── history/             # HistoryList, DocumentPreview (UTF-8 byte→char highlight)
│   │   ├── layout/              # Sidebar, Header
│   │   ├── overlay/             # ExitConfirm
│   │   ├── rules/               # RuleManager (regex sandbox, debounced)
│   │   ├── settings/            # SettingsPage, ModelDownloadCard
│   │   └── ui/                  # Atomic primitives (mostly under-used, see §Notes)
│   ├── hooks/
│   │   ├── useAppStore.ts       # Zustand store (bootstrap 2-phase load)
│   │   ├── useTauriEvents.ts    # Generic listen() wrapper with StrictMode-safe cleanup
│   │   ├── useAudioFeedback.ts  # Web Audio oscillator SFX (single AudioContext)
│   │   └── useModelDownloader.ts# Standalone Zustand store for AI model download
│   ├── services/api.ts          # Typed IPC wrappers (`MaskAPI`)
│   ├── lib/                     # utils, maskColors
│   ├── style.css                # Tailwind + custom keyframes (no framer-motion at runtime)
│   └── vite-env.d.ts            # __APP_VERSION__ global
├── src-tauri/                   # Rust backend (workspace member "SafeMask")
│   └── src/
│       ├── main.rs              # Binary entry — Tauri setup, plugins, invoke_handler
│       ├── lib.rs               # Library entry (safemask_lib) — staticlib/cdylib/rlib
│       ├── ai_downloader.rs     # Model download pipeline (zip fetch + verify + extract)
│       ├── api/                 # #[tauri::command] IPC handlers
│       │   ├── files.rs         # process_file_gui
│       │   ├── system.rs        # rules CRUD, history, settings, AI toggle, engine info
│       │   └── text.rs          # mask_text
│       ├── common/              # AppState, AppError (thiserror), event constants, EntitySpanBrief
│       ├── core/                # Pure business logic — zero Tauri imports, independently testable
│       │   ├── hybrid_engine.rs # Main engine (Registry + Resolver + MaskingEngine)
│       │   ├── engine.rs        # Legacy MaskEngine (superseded — candidate for removal)
│       │   ├── recognizer/      # aho_corasick / regex / ner / checksum / context_enhancer + registry
│       │   ├── resolver/        # Sub-span carving conflict resolution
│       │   ├── masking/         # 6 strategies: Replace/PartialMask/Hash/Redact/Token/Template
│       │   ├── orchestrator/    # SceneMode (Shadow/Sentry) business layer
│       │   ├── rules.rs         # Rule + RuleGroup types
│       │   ├── config.rs        # AppSettings
│       │   └── download_auth.rs # HMAC-SHA256 device-fingerprint download tokens
│       ├── infra/               # OS interactions
│       │   ├── ai/              # ModelManager (state), NerEngine (ort inference)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AiToByte/SafeMask](https://github.com/AiToByte/SafeMask) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
