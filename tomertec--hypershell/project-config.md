---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

HyperShell (package name: hypershell) — a Windows-first desktop SSH and serial terminal with integrated SFTP file browser, built with Electron + React + xterm.js, packaged as a pnpm monorepo.

Full documentation: [`docs/INDEX.md`](docs/INDEX.md)

## Build & Dev Commands

```bash
pnpm build                  # Build all workspaces
pnpm test                   # Run all Vitest unit tests
pnpm lint                   # Lint all workspaces

# Per-workspace
pnpm --filter @hypershell/ui test
pnpm --filter @hypershell/desktop test

# E2E (Playwright, headless Chromium)
pnpm --filter @hypershell/ui test:e2e
pnpm --filter @hypershell/ui test:e2e:headed

# CI pipelines
pnpm ci:build
pnpm ci:test
pnpm ci:test:e2e

# Windows release
pnpm release:windows:unsigned
pnpm release:windows:signed
```

**Important:** After changing main process or preload code, you must `pnpm --filter @hypershell/desktop build` and restart Electron. UI changes are picked up by Vite HMR automatically — unless Electron is loading the bundled renderer (delete `apps/desktop/dist/renderer/` to force Vite dev server in development).

## Monorepo Structure

Five pnpm workspaces with clear dependency flow:

```
apps/desktop    → Electron main + preload (IPC boundary, window mgmt, tray, secure storage)
apps/ui         → React workbench (xterm.js terminals, host browser, tabs/panes, Zustand state)
packages/shared → IPC channel names, Zod request/response schemas, auth/transport enums
packages/session-core → Transport abstraction (SSH via PTY, serial, SFTP via ssh2), session lifecycle, connection pool, network monitor, tmux probe
packages/db     → SQLite via better-sqlite3, migrations (001-014), repositories
```

Dependency direction: `desktop` → `shared`, `session-core`, `db`; `ui` → `shared`; `session-core` → `shared`.

## Architecture

**Three-layer Electron model:**
1. **Main process** (`apps/desktop/src/main/`) — bootstraps app lifecycle, registers 40+ IPC handlers, manages sessions, tray, windows. Entry: `main.ts`.
2. **Preload bridge** (`apps/desktop/src/preload/`) — exposes `window.hypershell` API to renderer with Zod-validated typed IPC methods. Both request and response are validated.
3. **Renderer** (`apps/ui/`) — React SPA loaded by Electron. Vite dev server on port 5173.

**IPC contract pattern:** All IPC channels and payloads are defined in `packages/shared/src/ipc/` using Zod schemas. Both preload and main validate against the same schemas. Types are inferred via `z.infer`. See [`docs/ipc-reference.md`](docs/ipc-reference.md) for the full channel list.

**Session transports:** `session-core` provides a `SessionManager` that creates transport instances:
- **SSH** — spawns system `ssh` binary in node-pty (full agent/config/proxy compatibility)
- **Serial** — opens via `serialport` npm with configurable baud/parity/flow
- **SFTP** — programmatic ssh2 library (separate from SSH terminal, handles transfers/streams)

The SFTP transport tries all candidate key files sequentially (like system ssh) and strips Windows domain prefixes from usernames. When an `Ssh2ConnectionPool` is provided, SFTP reuses pooled connections instead of creating new ones.

**Connection pooling:** `session-core` provides an `Ssh2ConnectionPool` that manages shared ssh2 connections keyed by `host:port:user`. SFTP sessions and programmatic port forwards acquire from the pool; connections are ref-counted and idle-close after 30s with no consumers.

**Network-aware auto-reconnect:** `SessionManager` integrates with a `NetworkMonitor` that probes DNS every 10s. On disconnect, if the network is down, sessions enter `waiting_for_network` state (no reconnect attempts burned). When connectivity returns, attempts reset and reconnection starts immediately. Per-host config: `autoReconnect`, `reconnectMaxAttempts`, `reconnectBaseInterval`.

**Tmux session detection:** Per-host opt-in (`tmuxDetect` on host record). Before connecting, spawns a one-shot `ssh host 'tmux ls -F ...'` via `child_process.execFile` (reuses `buildSshArgs()` for identical auth). Parses output into session list, shows a `TmuxSessionPicker` modal. On attach, sends `tmux attach -t '<name>'` as terminal input after SSH connects. Requires key-based auth — password-only hosts are skipped. All probe failures silently fall back to normal connection. Key files: `session-core/tmux/tmuxProbe.ts`, `desktop/ipc/tmuxIpc.ts`, `ui/features/tmux/TmuxSessionPicker.tsx`.

**State management:** UI uses Zustand stores — `layoutStore` (tabs/panes, drag-and-drop reorder), `settingsStore`, `sessionRecoveryStore`, `broadcastStore`, `sftpStore` (per SFTP session), `transferStore`, `tunnelStore` (port forward manager), `snippetStore` (snippets panel).

**Session logging:** `loggingIpc.ts` provides a `createSessionLogger()` that intercepts terminal data events in `registerIpc.ts`, strips ANSI escape sequences, and writes to user-chosen files. Controlled per-session via recording button in TerminalPane (visibility controlled by `general.showRecordingButton` setting).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomertec/HyperShell](https://github.com/tomertec/HyperShell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
