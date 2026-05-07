---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ACPone is an ACP (Anthropic Client Protocol) Gateway Chat interface with a Go backend and Vue 3 + TypeScript frontend. It provides a web-based chat interface for communicating with AI agents (like Claude Code, Codex) through JSON-RPC, with support for multiple workspaces, sessions, and agent routing.

The project also includes a desktop tray application for macOS/Linux/Windows.

## Build Commands

### Frontend (web/)
```bash
cd web
npm install              # Install dependencies
npm run dev              # Dev server on :5173 (proxies API to :3000)
npm run build            # Build to web/dist
```

### Backend (backend/)
```bash
cd backend
go build -o acpone ./cmd/acpone              # Build web server binary
go run ./cmd/acpone                          # Run with embedded web
go run ./cmd/acpone -web ../web/dist         # Run with external web dir
go run ./cmd/acpone -port 8080               # Custom port (default: 3000)
```

### Desktop App (backend/)
```bash
cd backend
go build -o acpone-desktop ./cmd/desktop     # Build desktop tray app
```

### Full Build (embedded single binary)
```bash
# Build web assets and embed into Go binary
cd web && npm run build && cd ../backend && go build -o acpone ./cmd/acpone
```

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Vue 3 Frontend                            │
│  ┌─────────┐ ┌──────────────┐ ┌─────────────┐ ┌──────────────┐  │
│  │ Sidebar │ │ChatContainer │ │ ChatInput   │ │SettingsModal │  │
│  └────┬────┘ └──────┬───────┘ └──────┬──────┘ └──────────────┘  │
│       │             │                │                           │
│       └─────────────┼────────────────┘                           │
│                     ▼                                            │
│              ┌─────────────┐                                     │
│              │session.ts   │  (Reactive store)                   │
│              └──────┬──────┘                                     │
│                     ▼                                            │
│              ┌─────────────┐                                     │
│              │  api/       │  HTTP + SSE                         │
│              └──────┬──────┘                                     │
└─────────────────────┼───────────────────────────────────────────┘
                      │ HTTP/SSE
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                        Go Backend                                │
│  ┌─────────────┐                                                 │
│  │ api/server  │ ──► Routes: /api/chat, /api/sessions, etc.     │
│  └──────┬──────┘                                                 │
│         │                                                        │
│    ┌────┴────┐                                                   │
│    ▼         ▼                                                   │
│ ┌──────┐ ┌────────┐                                              │
│ │Router│ │Session │ (storage/)                                   │
│ └──┬───┘ │Storage │                                              │
│    │     └────────┘                                              │
│    ▼                                                             │
│ ┌────────────────┐                                               │
│ │ Agent Manager  │                                               │
│ └───────┬────────┘                                               │
│         │ JSON-RPC                                               │
│         ▼                                                        │
│ ┌────────────────┐                                               │
│ │ Agent Process  │ (subprocess: claude-code, codex, etc.)       │
│ └────────────────┘                                               │
└─────────────────────────────────────────────────────────────────┘
```

## Key Data Flow

### Chat Message Flow
1. User sends message via `ChatInput` → `POST /api/chat` (SSE)
2. Backend `Router` selects agent based on @mentions or keywords
3. `AgentManager` spawns/connects to agent process via JSON-RPC
4. Agent streams responses → Backend relays via SSE → Frontend updates UI
5. Stream items (text/tool calls) are held in `streamItems` ref until completion
6. On next user message, `commitStreamItems()` moves stream content to messages array

### Permission Flow
Agent requests permission → Backend sends SSE event → `PermissionRequest.vue` displays → User confirms → `POST /api/permission/confirm` → Agent proceeds

### File Upload Flow
1. User uploads file via ChatInput → `POST /api/upload` with multipart form
2. Backend stores file in `.acpone-uploads/` directory in workspace
3. File path is added to chat request and formatted as `@filename` reference in prompt
4. Agent can access uploaded files via file path
5. On session end or manual cleanup → `POST /api/upload/cleanup` removes upload directory

## Key Files

| Path | Purpose |
|------|---------|
| `backend/cmd/acpone/main.go` | Web server entry point, embeds web assets |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daodao97/acpone](https://github.com/daodao97/acpone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
