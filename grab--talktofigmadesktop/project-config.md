---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

**TalkToFigma Desktop** is an Electron-based desktop application that bridges Figma with AI tools (Cursor, Codex, VS Code) using the Model Context Protocol (MCP). It enables AI assistants to read and manipulate Figma designs through a structured protocol.

**Current Version:** 2.0.0
**Bundle ID:** com.grabtaxi.klever
**Architecture:** Electron + React + TypeScript

### Migration History
This project was migrated from Kotlin Compose Desktop to Electron in January 2026 (v2.0.0). The Electron version maintains feature parity while adding cross-platform support (macOS, Windows, Linux) and better developer experience with modern JavaScript tooling.

## Development Commands

### Start Development Server
```bash
npm start
# Launches Electron app with hot reload
# Main process: src/main.ts
# Renderer process: src/renderer.tsx
```

### Build & Package
```bash
npm run package  # Package for current platform
npm run make     # Create distributable installers
```

### Linting
```bash
npm run lint     # Run ESLint on TypeScript files
```

### Building Individual Components
The project uses Vite with multiple entry points configured in `forge.config.ts`:
- Main process: `src/main.ts` → `vite.main.config.ts`
- Preload script: `src/preload.ts` → `vite.preload.config.ts`
- Renderer: React app → `vite.renderer.config.ts`
- MCP stdio server: `src/main/server/mcp-stdio-server.ts` → `vite.stdio.config.ts`

## Architecture

### High-Level Overview

```
MCP Clients (Cursor, Codex, etc.)
    │ spawn independent process per client
    ▼
stdio MCP Servers (one per client)
    │ WebSocket connection (ws://127.0.0.1:3055)
    ▼
WebSocket Server (Electron App - Port 3055)
    │ Channel-based routing
    ▼
Figma Plugin
```

### Core Architectural Concepts

#### 1. **Multi-Process Architecture**
- Each MCP client (Cursor, Codex) spawns its own independent stdio server process
- stdio servers are NOT managed by the Electron app - they're spawned by the MCP clients themselves
- All stdio servers connect to the same WebSocket server (port 3055) in the Electron app
- This allows multiple AI tools to work with Figma simultaneously

#### 2. **Stdio Server Installation**
The MCP stdio server is automatically installed to platform-specific locations:
- **macOS**: `~/Library/Application Support/TalkToFigma/mcp-server.cjs`
- **Windows**: `%APPDATA%\TalkToFigma\mcp-server.cjs`

Installation happens on first app launch via `src/main/utils/stdio-installer.ts`. These paths are sandbox-safe for App Store distribution.

#### 3. **Channel-Based Isolation**
- The WebSocket server uses a channel system to isolate different Figma files
- Each MCP client must call `join_channel` tool with a Figma file ID before sending commands
- Messages are routed only to clients and plugins in the same channel
- This prevents command collisions when multiple Figma files are open

#### 4. **Communication Flow**
1. User sends command in MCP client (e.g., Cursor)
2. MCP client → stdin → MCP stdio server process
3. stdio server → WebSocket (port 3055) → Electron app
4. Electron app → WebSocket → Figma plugin
5. Figma executes command and returns result
6. Result flows back through the same chain via message ID matching

### Electron Process Architecture

#### Main Process (`src/main.ts`)
- Creates BrowserWindow (900x650, sidebar layout)
- Initializes singleton services:
  - `TalkToFigmaService` - Unified service orchestrator
  - `TalkToFigmaServerManager` - WebSocket server lifecycle manager
  - `TalkToFigmaTray` - System tray with quick actions
- Handles OAuth authentication flow
- Installs stdio server on startup
- Manages IPC communication with renderer

#### Renderer Process
- React 19 application (`src/renderer.tsx` → `src/App.tsx`)
- Three-page sidebar interface:
  - **Terminal**: Real-time log viewer with log streaming
  - **Settings**: MCP client configuration manager (generates config snippets)
  - **Help**: Documentation and troubleshooting
- Real-time status updates via IPC events
- Dark/light theme support

#### Preload Script (`src/preload.ts`)
- Security bridge with context isolation
- Exposes typed API surface via `window.electron`:
  - `server.*` - Server control (start/stop/status)
  - `figma.*` - Figma operations
  - `auth.*` - OAuth authentication
  - `mcp.*` - MCP configuration utilities
  - `log.*` - Log streaming

### Key Services Architecture

#### Service Layer Pattern
All services in `src/main/server/services/` extend `BaseFigmaService`:
- `document-service.ts` - Document operations (get nodes, search)
- `creation-service.ts` - Node creation (frames, rectangles, text, etc.)
- `channel-service.ts` - Channel management (join/leave/list)
- `rest-api-service.ts` - Figma REST API operations (comments, reactions)

Services use adapter pattern to decouple from UI layer - see `main.ts` for adapter implementations.

#### WebSocket Server (`src/main/server/TalkToFigmaWebSocketServer.ts`)
- Listens on port 3055
- Tracks two types of connections:
  - **MCP clients** (stdio servers spawned by Cursor/Codex)
  - **Figma plugins** (running in Figma)
- Channel-based message routing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grab/TalkToFigmaDesktop](https://github.com/grab/TalkToFigmaDesktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
