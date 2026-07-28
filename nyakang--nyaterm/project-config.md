---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development commands

### Root app
- `pnpm install` — install JS dependencies
- `pnpm dev` — run the Vite frontend only
- `pnpm tauri dev` — run the full desktop app in Tauri dev mode
- `pnpm build` — run `tsc` and build the frontend with Vite
- `pnpm tauri build` — build the production desktop bundle
- `pnpm lint` — run Biome checks for `src/**/*.ts` and `src/**/*.tsx`
- `pnpm format` — apply Biome formatting to `src/**/*.ts` and `src/**/*.tsx`
- `pnpm format:check` — check Biome formatting without writing changes
- `pnpm i18n:check` — check locale JSON formatting
- `pnpm i18n:fix` — rewrite locale JSON formatting
- `pnpm version-sync` — sync version numbers across app files
- `pnpm release` — version sync + frontend build + Tauri build

### Rust / Tauri backend
- `cargo fmt --manifest-path src-tauri/Cargo.toml` — format Rust code
- `cargo clippy --manifest-path src-tauri/Cargo.toml --all-targets` — lint Rust code
- `cargo test --manifest-path src-tauri/Cargo.toml` — run backend Rust tests
- `cargo test --manifest-path src-tauri/Cargo.toml <test_name>` — run a single backend Rust test
- `cargo test --manifest-path src-tauri/crates/otp/Cargo.toml` — run OTP crate tests
- `cargo test --manifest-path src-tauri/crates/otp/Cargo.toml <test_name>` — run a single OTP crate test

Example single-test command:
- `cargo test --manifest-path src-tauri/Cargo.toml normalizes_trailing_slashes_without_breaking_roots`

### Docs site
- `pnpm --dir docs-site start` — build and serve the docs site locally for all locales (`/` and `/en/`)
- `pnpm --dir docs-site start:zh` — run the zh-CN docs dev server with hot reload
- `pnpm --dir docs-site start:en` — run the English docs dev server with hot reload
- `pnpm --dir docs-site start:ko` — run the Korean docs dev server with hot reload
- `pnpm --dir docs-site build` — build the docs site

## Big-picture architecture

- This is a Tauri 2 desktop app: React/TypeScript frontend in `src/`, Rust backend in `src-tauri/src/`, and IPC between them via Tauri commands/events.
- The frontend should call Rust through the typed wrapper in `src/lib/invoke.ts`, not raw scattered `invoke()` calls where a shared wrapper already exists.
- Tauri commands are registered centrally in `src-tauri/src/lib.rs` and grouped by concern under `src-tauri/src/cmd/` (`session`, `sftp`, `connection`, `credential`, `settings`, `watcher`, `translate`, `stats`, `tunnel`, `proxy`, `otp`, `importer`, plus `app`, `backup`, `clipboard`, `cloud_sync`, `log`, and `ai`).

### Window model
- `src/main.tsx` decides between two boot paths:
  - main window: `AppProvider` + `App.tsx`
  - child windows: `ChildAppProvider` + `ChildWindowRouter`
- Child windows are opened from `src/lib/windowManager.ts` using `?window=` query params. Current child-window flows include settings, new-session, quick-command, and per-file auto-upload dialogs.
- Modal child-window focus/enable state is managed in `windowManager.ts` plus `src/ChildWindowRouter.tsx`; changes here affect cross-window UX.

### Frontend state model
- `src/context/AppContext.tsx` is the main state container. It owns:
  - tab/session workspace state
  - persisted UI/app settings
  - saved connections/groups loading and refresh
  - startup session restoration from persisted `ui.open_tabs`
- `src/context/ChildAppProvider.tsx` is a lightweight provider for child windows. It only loads/saves settings and emits cross-window events; it does not manage the full tab/session workspace.
- `src/context/TransferContext.tsx` separately tracks file transfer progress from backend `transfer-event` notifications.

### Workspace / terminal model
- The terminal workspace has two distinct layers that are easy to confuse:
  - `src/lib/workspaceTabs.ts` manages the persistent logical tab model. Each tab owns a recursive pane tree (`leaf` session panes and `split` panes). This is what gets serialized into `ui.open_tabs` for startup restore.
  - `src/lib/tabWindows.ts` manages the runtime terminal window layout: which tabs live in each split window leaf, active tab per leaf, and window split ratios for the multi-tab/multi-split UI.
- `src/App.tsx` is the shell that composes activity bars, left/right panels, the terminal workspace, quick command / serial send bottom panels, OTP dialogs, transfer UI, recording, and lock screen.
- `src/components/terminal/XTerminal.tsx` is the xterm.js integration point. It wires Fit/Search/WebLinks addons, shell integration, command suggestions, reconnect hooks, and listens to per-session backend events.

### Backend runtime model
- `src-tauri/src/lib.rs` constructs and stores the shared backend managers in Tauri state:
  - `SessionManager`
  - `TunnelManager`
  - `RecordingManager`
  - `PendingAuthManager`
  - `HostKeyVerifyManager`
  - `QuickCommandsStore`
  - `CloudSyncManager`
  - `AgentApprovalManager` (gates AI agent command execution)
- Tauri commands are registered centrally in `src-tauri/src/lib.rs`; newer backend capability areas now include app, backup, clipboard, cloud sync, logging, and AI in addition to sessions/SFTP/settings/importers.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nyakang/nyaterm](https://github.com/nyakang/nyaterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
