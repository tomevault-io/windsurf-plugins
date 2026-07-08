---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Rules (always apply — override everything else)

Project-specific rules live in `.Codex/rules/`. They are **mandatory** and take precedence over all other instructions, skills, and defaults. Read and follow every rule in that folder before doing any work.

| # | File | Summary |
|---|------|---------|
| 01 | `.Codex/rules/01-mask-credentials-on-output.md` | Never output secrets, keys, or passwords in plain text |
| 02 | `.Codex/rules/02-codebase-english.md` | All code, comments, identifiers, and commit messages must be in English |
| 03 | `.Codex/rules/03-readme-up-to-date.md` | Keep `README.md` current whenever build steps, features, or milestone status change |
| 04 | `.Codex/rules/04-feature-branch-per-milestone.md` | Each milestone/feature lives on its own branch (`ms/0X-*`); never commit directly to `main` |
| 05 | `.Codex/rules/05-feature-documentation.md` | Every feature must be documented in `docs/user/` and `docs/dev/` before the milestone is merged |
| 06 | `.Codex/rules/06-document-learnings.md` | Every bugfix or non-obvious insight is saved as an individual file in `~/.Codex/learnings/` |
| 07 | `.Codex/rules/07-code-quality-and-architecture.md` | Analyse before implementing; reuse over new code; clean, modular, type-safe, consistent architecture at all times |

## Project Overview

**LocalVoice** (working title: FlowDict) is an offline-first desktop voice dictation app built with **Tauri v2** (Rust + TypeScript/React). It records audio via global hotkey, transcribes locally using a whisper.cpp sidecar process, and outputs text via clipboard or direct insertion. The app is primarily a small floating "pill" UI — privacy-first, no cloud connectivity.

Full specification: `plan/flowdict_prd.md`. Milestone breakdown: `plan/milestones/flowdict_milestone_01.md` through `_10.md`.

## Status

Pre-implementation. The planning docs define the entire architecture, database schema, Tauri command API surface, and 10-milestone roadmap. **No source code exists yet.**

## Task Tracking

All work is tracked via a structured backlog in `plan/`:

```
plan/
  epics.md              Master index — all 10 epics with goals and acceptance criteria
  tasks/
    ms01_foundation.md  TASK-001–019
    ms02_recording.md   TASK-020–036
    ms03_transcription.md TASK-037–053
    ms04_output.md      TASK-054–067
    ms05_history.md     TASK-068–084
    ms06_dashboard.md   TASK-085–098
    ms07_models.md      TASK-099–116
    ms08_dictionary.md  TASK-117–132
    ms09_ambiguity.md   TASK-133–146
    ms10_polish.md      TASK-147–164
```

**Task workflow rules:**
- Before starting any implementation, read the relevant `plan/tasks/ms0X_*.md` file.
- Work milestones in order (MS-01 → MS-02 → …); MS-07 may start in parallel after MS-03.
- When a task is completed, mark its checkbox: `- [x] TASK-NNN: …`
- When a milestone is fully done, update its `**Status:**` line to `done` and update `plan/epics.md` table accordingly.
- Never skip tasks without noting why (add a comment below the checkbox if a task is intentionally skipped or superseded).
- If new work is identified during implementation that wasn't in the original plan, add it as a new `TASK-NNN` at the bottom of the relevant milestone file before starting it.

## Planned Tech Stack

| Layer | Technology |
|---|---|
| Desktop shell | Tauri v2 |
| Frontend | React + TypeScript, Zustand, Tailwind CSS, shadcn/ui |
| Backend (Rust) | cpal (audio), sqlx or rusqlite (SQLite), serde, tokio |
| Transcription | whisper.cpp as a **sidecar process** (not FFI) |
| Database | SQLite (local) |
| Charts | Recharts |
| Tables | TanStack Table |

## Build & Dev Commands (once scaffolded)

```bash
# Install frontend dependencies
npm install           # or pnpm install

# Run in development (hot-reload frontend + Rust watch)
npm run tauri dev

# Build for production
npm run tauri build

# Frontend only (Vite)
npm run dev

# Lint TypeScript
npm run lint

# Run Rust tests
cargo test

# Run a single Rust test
cargo test <test_name>
```

These commands will be in the root `package.json` once Milestone 1 scaffolding is complete.

## Architecture

```
src-tauri/           Rust backend (Tauri v2)
  src/
    commands/        Tauri #[tauri::command] handlers (recording, transcription, history, dictionary, models, settings, window)
    audio/           microphone capture via cpal, WAV/PCM encoding
    transcription/   whisper.cpp sidecar orchestration, segment parsing
    processing/      post-processing pipeline (dictionary rules, corrections)
    db/              SQLite access layer (sessions, dictionary, settings, models)
    state/           shared AppState (Arc<Mutex<...>>) injected into commands
  Cargo.toml
  tauri.conf.json    window config (pill + main window), sidecar allowlist, capabilities

src/                 React/TypeScript frontend
  components/
    pill/            default floating micro-UI (idle/listening/processing/success/error states)
    dashboard/       usage metrics, WPM chart, word count trends
    history/         session list, search, filters
    dictionary/      entries CRUD, correction rules

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iptoux/localvoice](https://github.com/iptoux/localvoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
