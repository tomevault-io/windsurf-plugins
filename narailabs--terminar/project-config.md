---
trigger: always_on
description: **NEVER use mock PTY, mock data, or any mock/stub options unless the user explicitly asks for it.** Always use `pnpm dev` (real PTY). Always use `cargo run -- --no-auth`. If a real dependency is unavailable, fail explicitly rather than silently using fakes.
---

# CLAUDE.md - terminar

## No Mocking Policy (MANDATORY)

**NEVER use mock PTY, mock data, or any mock/stub options unless the user explicitly asks for it.** Always use `pnpm dev` (real PTY). Always use `cargo run -- --no-auth`. If a real dependency is unavailable, fail explicitly rather than silently using fakes.

## Overview

Persistent terminal sessions that survive crashes and restarts. A Rust server manages PTY sessions on a Unix socket, with an Electron tray app as the distributed frontend. Think of it as a built-in tmux.

**Full architecture reference:** [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md)

## Directory Structure

```
terminar/
├── server/                  # Rust backend (terminar-server binary)
│   └── src/
│       ├── main.rs          # Server entry point
│       ├── lib.rs           # Server core: routing, message dispatch
│       ├── config.rs        # CLI args (clap)
│       ├── handlers/        # Message handlers (session, I/O, workspace)
│       ├── messages.rs      # Wire protocol types (Rust side)
│       ├── connection.rs    # Client connection tracking
│       ├── settings.rs      # Per-session settings
│       ├── workspace.rs     # Workspace persistence
│       ├── audit.rs         # Event logging
│       ├── constants.rs     # Defaults (ports, timeouts, limits)
│       ├── error.rs         # Error types
│       └── logging.rs       # Tracing configuration
├── web/                     # Web frontend (Svelte 5 + xterm.js + Vite) — dev-only, not shipped
│   └── src/
│       ├── App.svelte       # Root component
│       ├── components/      # ~20 components: Terminal, Pane, WorkspaceView, Sidebar, etc.
│       └── lib/             # ~50 modules: stores, managers, keybindings, themes
├── tray/                    # System tray app (Electron + Svelte 5) — shipped desktop app
│   └── src/
│       ├── main/            # Electron main process (tray, health, server, config, IPC)
│       ├── preload/         # contextBridge API (trayAPI)
│       └── renderer/        # Svelte: Settings window, embedded terminal
├── extension/               # VS Code extension (TypeScript)
│   └── src/                 # ServerController, SessionManager, NetSocketAdapter
├── packages/
│   └── shell-protocol/      # Shared TypeScript protocol (@narai/terminar-protocol)
│       └── src/             # Zod schemas, ShellClient, BaseWebSocketManager
├── npm/                     # npm distribution packages
│   ├── terminar/            # Main CLI package (bin/, lib/, app/)
│   ├── darwin-arm64/        # @narai/terminar-darwin-arm64 (server binary)
│   ├── darwin-x64/          # @narai/terminar-darwin-x64
│   ├── linux-x64/           # @narai/terminar-linux-x64
│   ├── linux-arm64/         # @narai/terminar-linux-arm64
│   └── win32-x64/           # @narai/terminar-win32-x64 (deferred)
├── docs/                    # API docs, architecture plans
│   ├── ARCHITECTURE.md      # Detailed architecture reference (READ THIS)
│   ├── API.md               # Protocol & REST API reference
│   └── plans/               # Design documents for major features
└── package.json             # Root workspace (pnpm monorepo)
```

## Architecture

### Local-Only Mode

```
Electron Tray (tray/) ──WebSocket──► terminar-server ◄──Unix Socket──► VS Code Extension
       │                                    │
  Embedded terminal                    PTY Sessions (portable-pty)
  Settings window
  Health polling
```

The server listens on a Unix socket and localhost HTTP. The Electron tray app is the shipped desktop frontend. The web frontend (`web/`) is used only during development.

### Components

- **terminar-server** (`server/`): Local PTY server. Unix socket + localhost HTTP/WebSocket. Token auth via `~/.terminar/token`.
- **Electron Tray** (`tray/`): Shipped desktop app. System tray icon, embedded terminal, settings, health polling. Bundles the server binary.
- **Web Frontend** (`web/`): Svelte 5 + xterm.js. Dev-only — not shipped in the distributed app.
- **VS Code Extension** (`extension/`): Connects via Unix socket with length-prefixed framing.
- **Protocol Package** (`packages/shell-protocol/`): Zod schemas, ShellClient, BaseWebSocketManager. Shared by web + extension.

## Build & Test Commands

### Root-level shortcuts (preferred)

```bash
pnpm dev                     # Start server (detached) + tray (USE THIS BY DEFAULT)
pnpm dev:web                 # Start server (detached) + web (for web frontend development)
pnpm dev:all                 # Start server (detached) + web + tray (everything)
pnpm server:status           # Check if the detached dev server is running
pnpm server:stop             # Stop the detached dev server
pnpm server:restart          # Rebuild and restart the detached dev server
pnpm server:logs             # Tail the detached dev server's log file
pnpm build                   # Build server (release) + tray
pnpm test                    # Test server
pnpm test:server             # Test server only
pnpm test:web                # Test web only
pnpm test:extension          # Test extension only

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [narailabs/terminar](https://github.com/narailabs/terminar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
