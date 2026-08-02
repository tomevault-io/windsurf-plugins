---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Sessionly is a cross-platform desktop app for browsing, monitoring, and exporting Claude Code CLI session history. It reads sessions directly from `~/.claude/projects/` and surfaces live session activity, search, and Markdown export.

**Tech Stack**: Tauri v2 (Rust backend), React 19, TypeScript 5, Vite 7, Redux Toolkit, Tailwind CSS, shadcn-style UI components.

> **History**: This project was originally an Electron + Drizzle/better-sqlite3 boilerplate and was migrated to Tauri v2 (Feb 2026). If you find references to Electron, IPC `window.electron`, Drizzle, or better-sqlite3 anywhere, they are stale — the current app uses Tauri `invoke` and reads the filesystem directly, with **no internal database**.

**Design Philosophy**: Minimalist black & white theme with the Inter font family, light/dark mode support.

## Repository Layout

The **only active project is the repository root.** Two sibling directories are local-only and **gitignored** (`.gitignore:44-45`) — do not treat them as part of the build:

- `sessionly-tauri/` — stale duplicate of an earlier Tauri scaffold (`version 0.1.0`). Safe to ignore/delete.
- `sessionly-macos/` — a separate, experimental **native macOS (Swift/SwiftPM)** implementation of the same product. Unrelated to the root build.

Other untracked root dirs are build output or scratch: `dist/`, `dist-electron/` (Electron-era leftover), `release/`, `debug/`, `promo-video/`, `node_modules/`.

## Development Commands

Everything runs from the repository root.

```bash
# Install
npm install              # JS deps; Rust/Cargo deps build on first `tauri` run

# Development
npm run tauri dev        # PRIMARY dev loop: Vite dev server (port 1420) + Rust app, hot reload
npm run dev              # Frontend-only in a browser (no native APIs / Tauri commands)

# Build
npm run build            # tsc + vite build (frontend bundle only)
npm run tauri build      # Full native app bundle (installers in src-tauri/target/release/bundle/)

# Quality gates (these mirror CI)
npm run typecheck        # tsc --noEmit
npm run lint             # eslint, --max-warnings 0
npm run format           # prettier --write
npm run format:check     # prettier --check
cd src-tauri && cargo clippy -- -D warnings
```

There is currently **no test runner configured** (the old Vitest setup was removed in the migration).

## Architecture

### Two-process model (Tauri)

1. **Frontend / WebView** (`src/`)
   - React 19 app bundled by Vite, rendered in the OS WebView.
   - Cannot access the filesystem or OS directly — it calls Rust via Tauri `invoke`.
   - Entry: `src/main.tsx` → `src/App.tsx`.

2. **Backend / Core** (`src-tauri/src/`)
   - Rust. Owns all privileged work: filesystem reads, session parsing, hook install, notifications.
   - Entry: `src-tauri/src/main.rs` → `src-tauri/src/lib.rs` (registers plugins + command handlers).

### Frontend ↔ Backend communication

The frontend talks to Rust through a single typed wrapper in `src/types/api.ts`:

```typescript
import { invoke } from '@tauri-apps/api/core'

export const api = {
  getVersion: () => invoke<string>('get_version'),
  sessionsGetAll: () => invoke<ProjectGroup[]>('get_projects'),
  sessionsGet: (sessionId, projectEncoded) =>
    invoke<Session>('get_session', { sessionId, projectEncoded }),
  // ...
}
```

**Always add new backend calls to `src/types/api.ts`** rather than calling `invoke` ad hoc — it keeps the surface typed and discoverable.

#### Adding a new backend command

1. **Implement** the command in `src-tauri/src/commands.rs` (or the relevant `*.rs` module):
   ```rust
   #[tauri::command]
   pub fn my_command(arg: String) -> Result<MyType, String> { ... }
   ```
2. **Register** it in the `generate_handler!` macro in `src-tauri/src/lib.rs`.
3. **Grant permission** if it needs a capability (see Capabilities below).
4. **Expose** it via a typed method in `src/types/api.ts`.
5. **Call** it from the frontend through `api.myMethod(...)`.

Note: command arguments are camelCase on the JS side and snake_case in Rust — Tauri converts automatically (`{ sessionId }` ↔ `session_id`).

### Backend modules (`src-tauri/src/`)

- `lib.rs` — app setup: registers plugins and command handlers.
- `commands.rs` — `#[tauri::command]` entry points exposed to the frontend.
- `session_store.rs` — reads `~/.claude/` and `~/.claude/projects/`; lists/parses sessions.
- `session_monitor.rs` — watches session files for live activity.
- `session_types.rs` — Rust data models (serde-serialized to the frontend).
- `markdown_export.rs` — session → Markdown export.
- `hooks.rs` — install/uninstall/status of Claude Code hooks.

Registered commands (`lib.rs`): `get_projects`, `get_session`, `get_version`, `get_native_theme`, `export_session_markdown`, `hooks_get_status`, `hooks_install`, `hooks_uninstall`, `hooks_is_installed`, `send_native_notification`.

Registered plugins: `opener`, `shell`, `dialog`, `fs`, `process`, `updater`, `notification`.

### Data model — there is no internal database


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sugarforever/sessionly](https://github.com/sugarforever/sessionly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
