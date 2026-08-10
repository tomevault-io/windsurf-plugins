---
trigger: always_on
description: This file provides guidance to Codex when working with code in this
---

# AGENTS.md

This file provides guidance to Codex when working with code in this
repository. It mirrors `CLAUDE.md` — keep the two in sync when either
changes.

## What this is

**WorkScreen (Gilbreth)** — a desktop app that records the user's actions
via accessibility APIs. Cargo workspace + Tauri 2. Both macOS
(CGEventTap + AX) and Windows (UI Automation + event hooks) have real
capture backends; Linux is out of scope.

Beyond raw capture the app records meetings (detected from mic
activity), transcribes them on-device with whisper.cpp, and can show
real-time suggestions during a live call (`docs/assist.md`). Pattern
mining over the captured stream — the therbligs the name refers to —
is the next layer and is not implemented; `apps/gilb-analyzer` is the
first step toward it.

## Conventions

- **Commit messages: English** (subject + body). Terse subject ≤72
  chars, body wraps ~72 cols. Match the style of recent commits.
- **User-visible strings: English** — HTML, TypeScript messages,
  Rust dialog/error text, `Info.plist` usage descriptions, READMEs.
  Frontend strings live in `apps/gilb-app-tauri/src/i18n.ts` (en + ru
  dictionaries; static markup carries `data-i18n` attributes). The
  locale and product name are baked in per build via `VITE_LOCALE` /
  `VITE_BRAND_NAME` (default: en / WorkScreen) so differently-branded builds
  reuse this frontend without forking. New user-facing strings go
  through `t()` / `data-i18n`, with both dictionary entries filled.
- **AGENTS.md, CLAUDE.md and any other docs read by an agent as
  instructions: English.**
- **UI/UX: follow `docs/ui-design.md`.** Single main window with in-app
  modal overlays (not popup windows) for app screens, explicit
  Save/Cancel, the green=active / red=stop color language, etc. Read it
  before adding or changing any frontend UI.

## Commands

Build and run go through the root Cargo workspace plus `npm` inside
`apps/gilb-app-tauri`.

```sh
# Cargo workspace (Rust). Run from the repo root.
cargo build                                # whole workspace
cargo build -p gilb-a11y                   # one crate
cargo test                                 # all tests
cargo test -p gilb-db                      # one crate's tests
cargo test -p gilb-a11y text_buffer        # name-filtered tests
cargo clippy --workspace --all-targets     # lint
cargo fmt --all                            # format

# MCP server over the recorded DB (stdio). Spawned by Claude Code,
# but handy to run manually:
cargo run -p gilb-mcp

# Tauri (frontend + Rust shell). Run from apps/gilb-app-tauri.
cd apps/gilb-app-tauri
npm install                                # once
npm run tauri dev                          # dev shell with hot-reload
npm run tauri build                        # release .dmg/.exe; signing per RELEASING.md
```

Capture defaults are controlled by env vars consumed by
`RecordingSettings::from_env`: `CAPTURE_EVENTS`, `CAPTURE_MOUSE_MOVE`,
`CAPTURE_CLIPBOARD`, `CAPTURE_TREE_SNAPSHOTS`. Logging: `RUST_LOG=...`
(defaults: `info,gilb=debug` in the Tauri shell, `info` in the CLI).

Everything the app writes lives in one **visible** folder —
`~/Documents/Gilb` on macOS, `%USERPROFILE%\Documents\Gilb` on Windows,
resolved through the OS's Documents known-folder rather than assembled
from `$HOME` (on Windows it may be redirected, e.g. into OneDrive). See
`gilb_config::data_dir`:

```
<Documents>/Gilb/
├── db.sqlite            actions, sessions, meetings, transcripts
├── meetings/<stamp>/    video.mp4 + audio.wav per recorded call
├── models/              the downloaded whisper model (~570 MB)
├── prompts/             realtime_assist.md — the suggestions prompt
├── logs/                daily-rotated app log
├── prefs.json           UI preferences
└── credentials.json     analyzer credentials, when configured
```

Installs from before the move keep their data in `~/.gilb`;
`gilb_config::migrate_legacy_data_dir` renames it into place at startup,
before the logger touches the new directory. A product embedding these
crates calls `set_data_dir` first and is never migrated.

## Architecture

### Crates and dependencies

```
gilb-core ──► (types: Action, ActionKind, AppInfo, ElementContext, SessionId)
gilb-config ─► (RecordingSettings, Preferences, data_dir / db_path /
               prompts_dir)
gilb-events ─► (EventBus: broadcast HealthEvent + RecordingEvent)

gilb-db ─────► gilb-core, gilb-config
              (SqlitePool + migrations under migrations/, sessions / actions modules)

gilb-a11y ───► gilb-core, gilb-config, gilb-events, gilb-db
              (trait CapturePlatform; cfg-gated implementations;
               text_buffer, tree/, password_masking;
               bin gilb-a11y-cli)

gilb-engine ─► all crates above
              (Engine — long-lived process-wide object; owns the DB pool,
               EventBus, current CaptureSession; spawns the writer task)

gilb-meeting ► (standalone: MeetingDetector trait + MeetingEvent enum;
               macOS unified-log and Windows WASAPI detectors, plus an
               in-memory MockDetector)

gilb-record ─► gilb-events, gilb-db
              (screen + audio capture: ScreenCaptureKit/AVAssetWriter on
               macOS, Windows.Graphics.Capture/Media Foundation on Windows;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [farol-team/workscreen](https://github.com/farol-team/workscreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
