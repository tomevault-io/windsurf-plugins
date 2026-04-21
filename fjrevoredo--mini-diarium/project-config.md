---
trigger: always_on
description: **Mini Diarium** is an encrypted, local-first desktop journaling app built with SolidJS, Rust/Tauri v2, and SQLite. Diary entries are AES-256-GCM encrypted at rest; plaintext must never be written to disk.
---

# AGENTS.md — Mini Diarium

**Mini Diarium** is an encrypted, local-first desktop journaling app built with SolidJS, Rust/Tauri v2, and SQLite. Diary entries are AES-256-GCM encrypted at rest; plaintext must never be written to disk.

**Core principles:** privacy-first, no network access, incremental development, clean architecture, TypeScript strict mode, and Rust type safety.

**Platforms:** Windows 10/11, macOS 10.15+, Linux (Ubuntu 20.04+, Fedora, Arch).

**Status:** See `docs/OPEN_TASKS.md` for structured roadmap items and `docs/TODO.md` for the working backlog.

## Domain Guides

For subsystem-specific conventions, use these as the detailed references:

- [Frontend (src/)](src/CLAUDE.md) — SolidJS state, i18n, TipTap, themes, testing, `data-testid` inventory
- [Backend (src-tauri/)](src-tauri/CLAUDE.md) — Tauri commands, auth architecture, plugins, search hook points, Rust patterns
- [E2E (e2e/)](e2e/CLAUDE.md) — WebdriverIO, tauri-driver, viewport rules, E2E mode contracts
- [Benchmarks (benchmarks/)](benchmarks/CLAUDE.md) — Criterion, Vitest bench, CI tracking

This file is the concise, cross-cutting guide for agents. The domain guides above carry most implementation detail.

## Architecture

**Visual diagrams:**

- [System context](docs/diagrams/context.mmd) - High-level local-only data flow
- [Unlock flow](docs/diagrams/unlock.mmd) - Password/key-file unlock through DB open, backup rotation, and unlocked session
- [Save-entry flow](docs/diagrams/save-entry.mmd) - Multi-entry editor persistence flow
- [Layered architecture](docs/diagrams/architecture.svg) - Presentation/state/backend/data layers

**Regenerate diagrams from this shell:**

```bash
cmd.exe /c bun run diagrams
```

Quick reference:

```text
Presentation (SolidJS components)
  -> State signals
  -> Tauri invoke()/listen()
  -> Rust backend commands/business logic
  -> Local files only: diary.db, config.json, backups/, plugins/
```

**Key relationships:**

- Entries are encrypted in SQLite. The `entries` table uses `id INTEGER PRIMARY KEY AUTOINCREMENT`, so multiple entries per date are supported.
- Search is intentionally stubbed. The old plaintext FTS table was removed; the interface remains in place for future secure search.
- Journals are tracked in `{app_data_dir}/config.json`. Each journal points to its own directory containing `diary.db`.
- `DiaryState` holds a single active connection. Switching journals updates `db_path` and `backups_dir`, then auto-locks the active session.
- Preferences are stored in `localStorage`, not via a Tauri store plugin.

## Execution Environment

This repo is commonly worked on from a WSL shell over a Windows checkout (`/mnt/d/Repos/mini-diarium` <-> `D:\Repos\mini-diarium`). In that setup, the reliable path is the Windows toolchain, not the WSL one.

**Operational rule for agents in this environment:**

- Prefer `cmd.exe /c ...` for project commands unless you have explicitly verified a Linux-native setup.
- Do not start with bare `bun`, `cargo`, `vite`, `vitest`, or `tauri` from WSL in this repo.
- For Rust commands under `src-tauri`, switch drives explicitly:
  - `cmd.exe /c "cd /d D:\Repos\mini-diarium\src-tauri && cargo test"`
- Use repo-local Tauri CLI through `bun run tauri ...`; do not assume `cargo tauri` is globally installed.
- Treat generic shell snippets in docs as human-oriented unless they already say "Run from this Codex shell".

**Commands verified to work from this shell via Windows:**

- `cmd.exe /c bun run type-check`
- `cmd.exe /c bun run lint`
- `cmd.exe /c bun run test:run`
- `cmd.exe /c bun run build`
- `cmd.exe /c "cd /d D:\Repos\mini-diarium\src-tauri && cargo test"`
- `cmd.exe /c bun run test:e2e`
- `cmd.exe /c bun run tauri info`
- `cmd.exe /c bun run diagrams:check`

**Commands with side effects:**

- `cmd.exe /c bun run website:build-static` rewrites generated files under `website/`
- `cmd.exe /c bun run diagrams` regenerates SVG outputs

## Quick Reference

Most common agent tasks:

- Add a Tauri command: implement it in `src-tauri/src/commands/`, register it in `src-tauri/src/lib.rs`, add a typed wrapper in `src/lib/tauri.ts`
- Change frontend behavior: check `src/CLAUDE.md` first for SolidJS, i18n, and testing patterns
- Change backend behavior: check `src-tauri/CLAUDE.md` first for command patterns and security constraints
- Work on E2E or UI visibility defaults: audit `e2e/specs/` because the suite runs below the `lg` breakpoint
- Update diagrams: use `cmd.exe /c bun run diagrams:check` for verification-only, `cmd.exe /c bun run diagrams` to regenerate

## File Structure

High-level layout:

```text
website/     marketing site
src/         SolidJS frontend
e2e/         WebdriverIO + tauri-driver end-to-end tests
src-tauri/   Rust/Tauri backend
docs/        product, release, privacy, translation, diagram, and plugin docs
```

Notes:

- `website/` is plain HTML/CSS/JS and deploys via `website/docker-compose.yml`
- `src/state/session.ts` resets entries, search, and UI state on lock/session boundary
- `e2e/specs/` currently contains `diary-workflow.spec.ts` and `multi-entry.spec.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fjrevoredo/mini-diarium](https://github.com/fjrevoredo/mini-diarium) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
