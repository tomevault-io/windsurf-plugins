---
trigger: always_on
description: VibeShell is a modern SSH/SFTP desktop terminal built with **Tauri 2** (Rust backend) and **React 18** (TypeScript frontend). It supports multi-session SSH, SFTP file management, SSH tunneling, local shell, session recording, jump hosts, and AI tool integration.
---

# VibeShell — Agent Guide

## Overview
VibeShell is a modern SSH/SFTP desktop terminal built with **Tauri 2** (Rust backend) and **React 18** (TypeScript frontend). It supports multi-session SSH, SFTP file management, SSH tunneling, local shell, session recording, jump hosts, and AI tool integration.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│  Frontend (React 18 + Vite 6 + TypeScript)              │
│  ├── Zustand stores (10 stores)                         │
│  ├── xterm.js terminal emulator                         │
│  └── Tailwind CSS (Tokyo Night theme)                   │
├─────────────────────────────────────────────────────────┤
│  IPC: Tauri invoke (frontend↔backend)                   │
│  IPC: Named pipe / Unix socket (CLI↔backend)            │
├─────────────────────────────────────────────────────────┤
│  Backend (Rust / Tauri 2)                               │
│  ├── SSH: russh 0.44 (async, non-Clone Handle/Channel)  │
│  ├── SFTP: russh-sftp                                   │
│  ├── DB: rusqlite (SQLite, bundled)                     │
│  ├── Async: tokio runtime                               │
│  └── Local Shell: portable-pty                          │
└─────────────────────────────────────────────────────────┘
```

## Critical Knowledge

### russh 0.44 Constraints
- `client::Handle<H>` and `Channel<S>` are **NOT Clone**
- Handle sharing: wrap in `Arc<tokio::sync::Mutex<Option<Handle>>>`
- Channel I/O: use `channel.into_stream()` → `tokio::io::split()` for bidirectional data

### Frontend → Backend Communication
- All Tauri calls go through `src/lib/tauri.ts` → `safeInvoke<T>(command, args)`
- Returns `InvokeResult<T>` = `{ success: true; data: T }` | `{ success: false; error: TauriError }`
- For latency-sensitive input: use `sendInputBatched()` with RAF batching
- For fire-and-forget: use `fireAndForgetInvoke()`

### Session Lifecycle
```
SessionManager.create_with_credentials()
  → SshClient.connect_password/connect_key()
  → SshClient.open_shell()
  → Session { input: mpsc::Sender, output: broadcast::Sender }
  → Frontend attaches via Tauri event listener
```

### Theme System
- CSS variables set on `document.documentElement` from `settingsStore`
- Tailwind classes: `bg-tokyo-bg`, `text-tokyo-fg`, `border-tokyo-bg-hl`, etc.
- **NEVER** use hardcoded hex colors — always use `tokyo-*` Tailwind utilities

## Project Structure

```
src/                          # Frontend
  ├── components/             # React components (each in own folder)
  ├── stores/                 # Zustand stores (10 stores)
  ├── lib/                    # Utilities (tauri.ts, utils.ts)
  ├── types/                  # TypeScript type definitions
  ├── App.tsx                 # Main layout
  └── styles.css              # Global styles + Tailwind

src-tauri/                    # Rust backend
  ├── src/
  │   ├── commands/           # Tauri command handlers
  │   ├── ssh/                # SSH client (russh)
  │   ├── sftp/               # SFTP operations
  │   ├── session/            # Session + SessionManager
  │   ├── tunnel/             # SSH tunneling (local/remote/dynamic)
  │   ├── logging/            # Session recording
  │   ├── storage/            # Database + models
  │   ├── local_shell/        # Local terminal (portable-pty)
  │   ├── ipc/                # CLI↔GUI IPC socket
  │   ├── mcp/                # MCP server for AI tools
  │   ├── install/            # AI tool skill installer
  │   └── lib.rs              # App entry, command registration
  └── Cargo.toml

cli/                          # CLI client (workspace member)
```

## Database Schema (SQLite)

| Table | Purpose |
|-------|---------|
| `servers` | SSH server configs (host, port, auth, jump_host, post_login_cmd) |
| `groups` | Server organization groups |
| `credentials` | Encrypted credential storage |
| `server_credentials` | Per-server saved credentials |
| `tunnel_configs` | Persistent SSH tunnel configurations |
| `command_snippets` | Saved command templates |
| `recordings` | Session recording metadata |
| `settings` | App settings (key-value) |

## Stores (Frontend State)

| Store | Purpose |
|-------|---------|
| `serverStore` | Server/group CRUD |
| `sessionStore` | SSH session lifecycle |
| `localShellStore` | Local terminal sessions |
| `settingsStore` | App settings + themes |
| `fingerprintStore` | SSH host key verification |
| `notificationStore` | Toast notifications |
| `navigationStore` | View routing (main/settings) |
| `tunnelStore` | SSH tunnel configs + active tunnels |
| `snippetStore` | Command snippet management |
| `recordingStore` | Session recording state |

## Tauri Commands (70+ commands)

Organized by module: `session`, `server`, `credential`, `sftp`, `fingerprint`, `local_shell`, `snippet`, `tunnel`, `logging`, `install`, `dialog`.

All registered in `src-tauri/src/lib.rs` → `invoke_handler`.

## Build & Run

```bash
# Development
npm run dev              # Start Vite dev server
npx tauri dev            # Start Tauri dev (frontend + backend)

# Production build

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veithly/vibeshell](https://github.com/veithly/vibeshell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
