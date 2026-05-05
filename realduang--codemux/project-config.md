---
trigger: always_on
description: **CodeMux** is a multi-engine AI coding assistant client. It runs as an Electron desktop app (or standalone web server) that connects to multiple AI coding engines — OpenCode, GitHub Copilot, and Claude Code — through a unified WebSocket gateway. Users can access it locally or remotely via Cloudflare Tunnel.
---

# CodeMux - AI Agent Development Guide

## Project Overview

**CodeMux** is a multi-engine AI coding assistant client. It runs as an Electron desktop app (or standalone web server) that connects to multiple AI coding engines — OpenCode, GitHub Copilot, and Claude Code — through a unified WebSocket gateway. Users can access it locally or remotely via Cloudflare Tunnel.

### Tech Stack

| Layer | Technology |
|-------|-----------|
| Desktop Shell | Electron 40 (`electron: ^40.6.1`) |
| Build System | electron-vite (Vite 5) |
| Frontend | SolidJS 1.8 + TypeScript 5 |
| Styling | Tailwind CSS v4 + CSS Modules |
| Routing | @solidjs/router (HashRouter in Electron, BrowserRouter in web) |
| i18n | @solid-primitives/i18n (en, zh, ru) |
| Markdown | marked 11 + shiki 1.22 (syntax highlighting) |
| Backend Comm | WebSocket (ws) with custom JSON-RPC protocol |
| Packaging | electron-builder (DMG for macOS, NSIS for Windows) |
| Testing | Vitest 4 (unit) + Playwright (e2e) |
| Scripts | Bun |

### Project Structure

```
codemux/
├── electron/
│   ├── main/
│   │   ├── index.ts                  # Main process entry (service orchestration)
│   │   ├── ipc-handlers.ts           # IPC handler registration
│   │   ├── window-manager.ts         # BrowserWindow creation
│   │   ├── engines/                   # Engine adapters
│   │   │   ├── engine-adapter.ts         # Abstract base class
│   │   │   ├── opencode/                 # OpenCode CLI (HTTP REST + SSE)
│   │   │   │   ├── index.ts
│   │   │   │   ├── converters.ts
│   │   │   │   └── server.ts
│   │   │   ├── copilot/                  # GitHub Copilot (@github/copilot-sdk)
│   │   │   │   ├── index.ts
│   │   │   │   ├── converters.ts
│   │   │   │   └── config.ts
│   │   │   ├── claude/                   # Claude Code (@anthropic-ai/claude-agent-sdk)
│   │   │   │   ├── index.ts
│   │   │   │   ├── converters.ts
│   │   │   │   └── cc-session-files.ts
│   │   │   └── mock-adapter.ts           # Mock engine for testing
│   │   ├── gateway/                   # WebSocket Gateway
│   │   │   ├── ws-server.ts              # WebSocket server
│   │   │   └── engine-manager.ts         # Engine routing & lifecycle
│   │   ├── channels/                  # External messaging channels
│   │   │   ├── channel-adapter.ts        # Abstract channel base class
│   │   │   ├── channel-manager.ts        # Channel lifecycle & config persistence
│   │   │   ├── gateway-ws-client.ts      # Internal WS client (channel → gateway)
│   │   │   └── feishu/                   # Feishu (Lark) bot integration
│   │   │       ├── feishu-adapter.ts
│   │   │       ├── feishu-card-builder.ts
│   │   │       ├── feishu-command-parser.ts
│   │   │       ├── feishu-message-formatter.ts
│   │   │       ├── feishu-session-mapper.ts
│   │   │       └── feishu-types.ts
│   │   └── services/                  # Backend services
│   │       ├── auth-api-server.ts        # Auth API (HTTP on :4097, internal)
│   │       ├── device-store.ts           # Authorized devices persistence
│   │       ├── conversation-store.ts     # Session persistence (filesystem)
│   │       ├── update-manager.ts         # Auto-update management
│   │       ├── logger.ts                 # Logger + settings.json read/write
│   │       ├── production-server.ts      # Production HTTP server
│   │       └── tunnel-manager.ts         # Cloudflare Tunnel management
│   └── preload/
│       └── index.ts                  # contextBridge (electronAPI)
├── shared/                            # Code shared between main & renderer
│   ├── auth-route-handlers.ts            # Auth route handling logic
│   ├── device-store-base.ts              # Device store base class
│   ├── device-store-types.ts             # Device store type definitions
│   ├── http-utils.ts                     # HTTP utility functions
│   └── jwt.ts                            # JWT token handling
├── src/                               # SolidJS renderer
│   ├── main.tsx                       # App mount point
│   ├── App.tsx                        # Router setup & i18n provider
│   ├── pages/
│   │   ├── EntryPage.tsx              # Landing page (local auto-auth / remote login)
│   │   ├── Chat.tsx                   # Main chat interface
│   │   ├── Settings.tsx               # Settings page (engines, models, channels)
│   │   └── Devices.tsx                # Device management
│   ├── components/
│   │   ├── SessionSidebar.tsx         # Sidebar: project groups + session list
│   │   ├── SessionTurn.tsx            # Single assistant turn (steps, tool calls)
│   │   ├── MessageList.tsx            # Message rendering
│   │   ├── PromptInput.tsx            # Input area (agent/plan/autopilot modes)
│   │   ├── AddProjectModal.tsx        # Add project dialog
│   │   ├── HideProjectModal.tsx       # Hide project dialog
│   │   ├── FeishuConfigModal.tsx      # Feishu channel config dialog
│   │   ├── ContextGroup.tsx           # Context group display
│   │   ├── InputAreaPermission.tsx    # Permission request in input area

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [realDuang/codemux](https://github.com/realDuang/codemux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
