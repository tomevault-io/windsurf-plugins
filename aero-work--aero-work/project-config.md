---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Aero Work** is a cross-platform AI code agent application built with Tauri 2.0 (Rust backend) + React 19 (TypeScript frontend). It provides a visual interface for interacting with AI coding agents like Claude Code via the Agent Client Protocol (ACP).

**GitHub**: https://github.com/aero-work/aero-work

## Development Commands

```bash
# Install dependencies
bun install

# Desktop app development (Tauri + Vite hot reload)
bun run tauri dev

# Web-only development (frontend only)
bun run dev

# Headless mode (web frontend + WebSocket server concurrently)
bun run headless

# Build production
bun run tauri build        # Desktop app (Windows, macOS, Linux)
bun run build              # Web app only
bun run headless:build     # Web + server

# Run standalone WebSocket server (for web/PWA mode)
cargo run --bin aero-server --manifest-path src-tauri/Cargo.toml

# Rust checks
cargo check --manifest-path src-tauri/Cargo.toml
cargo clippy --manifest-path src-tauri/Cargo.toml
cargo test --manifest-path src-tauri/Cargo.toml

# TypeScript checks
bunx tsc --noEmit
```

### Android Build (WebView-only client)

```bash
# First time setup
bun run tauri android init
./scripts/android-post-init.sh    # Configure cleartext traffic & signing

# Build
bun run tauri android build --target aarch64 --debug   # Debug APK
bun run tauri android build --target aarch64           # Release APK

# Release signing (optional - uses debug keystore by default)
ANDROID_KEYSTORE_PATH=~/keys/app.keystore \
ANDROID_KEYSTORE_PASSWORD=secret \
ANDROID_KEY_ALIAS=aerowork \
ANDROID_KEY_PASSWORD=secret \
bun run tauri android build --target aarch64
```

### macOS Installation

```bash
# One-line install from releases
curl -fsSL https://raw.githubusercontent.com/aero-work/aero-work/main/scripts/install-mac.sh | bash

# Or install from local DMG
./scripts/install-local-mac.sh /path/to/AeroWork.dmg
```

### Headless Server Mode (Linux without display)

The compiled binary automatically detects headless environments and runs in server-only mode. Web frontend assets are embedded in the binary - no external files needed.

```bash
# Auto-detect: runs headless if no DISPLAY/WAYLAND_DISPLAY
./aero-work

# Force headless mode
./aero-work --headless

# Custom ports (defaults: ws=9527, web=9521)
./aero-work --headless --ws-port 9527 --web-port 9521

# Or via environment variables
AERO_WS_PORT=9527 AERO_WEB_PORT=9521 ./aero-work --headless
```

In headless mode, two servers start:
- **Web Client Server** (default: 9521) - serves the embedded frontend UI
- **WebSocket Server** (default: 9527) - handles agent communication

Ports auto-increment if occupied (tries +1 to +100, then OS-assigned).

**Note**: Build requires `bun run build` before `cargo build` to embed the frontend assets.

## Architecture

### Key Design Decisions

- **Unified Rust Backend**: Single Tauri backend serves both desktop (IPC) and web (WebSocket) modes
- **WebSocket-First**: Both desktop and web use WebSocket for agent communication; desktop IPC is only used for window management
- **ACP Protocol**: JSON-RPC 2.0 over stdio to spawn Claude Code agent (`npx @zed-industries/claude-code-acp`)
- **Backend is Source of Truth**: Session state lives in Rust; frontend subscribes via hooks

### Data Flow

```
Frontend (React)
    ↓
Zustand stores → api.ts → WebSocketTransport
    ↓
WebSocket Server (axum, port 9527)
    ↓
Core Services (AgentManager, SessionRegistry)
    ↓
ACP Client → Agent Process (stdio)
```

### Key Directories

| Directory | Purpose |
|-----------|---------|
| `src-tauri/src/acp/` | ACP protocol: `client.rs` (JSON-RPC over stdio), `types.rs` |
| `src-tauri/src/core/` | Business logic: agent, session, config, terminal managers |
| `src-tauri/src/server/` | WebSocket server: `websocket.rs` (JSON-RPC dispatch) |
| `src/stores/` | Zustand stores: session, agent, file, settings, terminal |
| `src/services/` | Backend API: `api.ts`, `transport/websocket.ts` |
| `src/hooks/` | React hooks: `useSessionData.ts` (key subscription hook) |
| `src/components/chat/` | Chat UI: MessageList, ToolCallCard, ChatInput |
| `scripts/` | Build and install scripts |

### Critical Files

- `src-tauri/src/server/websocket.rs` - All JSON-RPC method handlers
- `src-tauri/src/acp/client.rs` - Agent process spawn, message parsing
- `src-tauri/src/core/session_state_manager.rs` - Session state sync
- `src/hooks/useSessionData.ts` - Frontend session subscription
- `src/services/transport/websocket.ts` - WebSocket transport layer
- `src/services/api.ts` - High-level API with connection management
- `src/main.tsx` - Must call `enableMapSet()` from immer for Set/Map support
- `src/components/layout/MobileLayout.tsx` - Mobile layout with Android keyboard handling
- `src-tauri/gen/android/app/src/main/java/com/aerowork/dev/MainActivity.kt` - Android native keyboard detection (manually edited, preserved across rebuilds)

## Configuration

User config files are stored in `~/.config/aerowork/`:
- `config.json` - General settings
- `models.json` - Model provider configuration (Anthropic, Bedrock, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aero-work/aero-work](https://github.com/aero-work/aero-work) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-30 -->
