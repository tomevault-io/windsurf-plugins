---
trigger: always_on
description: Museeks is a music player desktop app built on top of Tauri. It focuses on clean UX and simplicity — it does not compete with more full-featured players in terms of customization or niche features.
---

# AGENTS.md - AI Assistant Guide for Museeks

## Overview

Museeks is a music player desktop app built on top of Tauri. It focuses on clean UX and simplicity — it does not compete with more full-featured players in terms of customization or niche features.

## Stack

- **Frontend**
  - Types: TypeScript (strict)
  - UI Library: React.js 19
  - Routing: TanStack Router (file-based)
  - Accessibility: Base UI
  - Global State Management: Zustand
  - Styling: [StyleX](https://stylexjs.com/docs/learn)
  - Localization/i18n: [Lingui.js](https://lingui.dev/introduction)
  - Dev Infra (all the following is setup via [Vite Plus](https://viteplus.dev/guide/)) (see below for explanations)
    - Node.js
    - Linting: Oxlint
    - Formatting: Oxfmt
    - Unit tests: Vitest
    - E2E tests: [Vitest Browser Mode](https://vitest.dev/guide/browser/) (w/ Playwright)
    - Bundler: Vite

- **Backend**
  - Language: Rust · Desktop integration: Tauri v2
  - Database: SQLite (via `sqlx`)
  - Linting: Clippy
  - Formatting: Rustfmt

## Development Commands

```bash
vp install                    # Install dependencies
vp run tauri dev              # Start dev (Vite + Tauri with hot reload)
vp run tauri build            # Build distributable binaries for the current platform
vp build                      # Front-end code build only
vp check                      # Check formatting (Oxfmt) and Lint/Typecheck (Oxlint)

# Code Generation
vp run gen:types              # Generate TS types from Rust (ts-rs)
vp run gen:translations       # Extract i18n strings to .po files

# Rust
vp run clippy                 # Rust linting
```

## Project Structure

```
src/                     # React/TypeScript frontend
  components/            # Feature-level components (Header, Footer, PlayerControls, ...)
  elements/              # Reusable UI primitives (Button, Flexbox, Scrollable, ...)
  hooks/                 # Custom React hooks (usePlayer, useCover, useAllTracks, ...)
  lib/                   # Core logic, IPC bridges, utilities
    bridge-*.ts          # Tauri IPC helpers — add new IPC calls here, never in components
    player.ts            # Singleton EventEmitter-based audio player — side-effect-heavy, modify with care
  routes/                # File-based routes (TanStack Router)
  stores/                # Zustand stores (useLibraryStore, PlaylistsAPI, SettingsAPI, ...)
  translations/          # Lingui .po files (en, fr, ja, ru, zh-CN, zh-TW, es, ...)
  generated/             # Auto-generated — never edit manually, regenerate via scripts
  types/                 # TypeScript type definitions

src-tauri/               # Rust/Tauri backend
  src/
    libs/                # Core Rust libraries (database, track, playlist, events, ...)
    plugins/             # Custom Tauri plugins (db, config, cover, stream_server, ...)
    migrations/          # SQLite migrations — files are frozen, never edit existing ones
    main.rs
  capabilities/
    main.json            # Permissions for Tauri commands (native or custom)
  build.rs               # Rust build, including inlined custom Tauri commands
  Cargo.toml
  tauri.conf.json
```

## Conventions & Guidelines

- Optimize for readability and cognitive load, not LoC count or cleverness, unless it's performance-sensitive.
- Keep logic co-located: prefer one file per business concern over spreading it across many.
- Prefer atomic commits.
- Node.js is purely infra (dev, build, scripts) — there is no server-side JavaScript.

**Backend:**

- Desktop integrations (file associations, sleep blocker, etc.) and Tauri commands go in plugins (`src-tauri/src/plugins`), unless there is a documented exception.
- Default Rust code lives in `src-tauri/src/libs`.
- Migrations: prefer `DEFAULT` values over backfills when adding columns — users will rescan their library.
- Never edit existing migration `.sql` files — they are frozen.
- When adding new Tauri commands, don't forget to add them to `build.rs`, and allow them in `capabilities/main.json`.

**UI:**

- Never call `invoke` directly from React components — extend the `bridge-*.ts` helpers in `src/lib` (or re-use the same pattern).
- `src/elements` are business-agnostic UI primitives. `src/components` and `src/routes` hold business logic.
- Prefer URL state over Zustand global state unless persistence is required.
- TypeScript types generated from Rust live in `src/generated` and should be committed — regenerate via `gen:types`.

## Agent Operations Rules

- After UI edits: ensure `vp check` (covers formatting, linting + type-check) passes
- After Rust edits: ensure `cargo test` in `src-tauri` passes. Clippy is a bonus.
- After modifying a Rust struct exposed via `ts-rs`: run `vp run gen:types` and commit the output.
- After editing the configuration of a route, regenerating the route-tree must be done via the `vp build` script.
- Never manually edit `src/generated` — always regenerate.
- Don't fix pre-existing issues unrelated to the current task.
- Don't try to run `tsc` for checking typescript typings, it should be done via `vp check` instead.
- When uncertain, read nearby file patterns first, then ask.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [martpie/museeks](https://github.com/martpie/museeks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
