---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Ralph UI is an HTTP/WebSocket server application for orchestrating autonomous AI coding agents using the Ralph Wiggum Loop technique. The app enables multi-agent parallel development sessions where progress persists in files and git history rather than LLM context.

The backend is built with Rust (Axum), and the frontend is a React SPA that connects via HTTP/WebSocket.

## Development Commands

```bash
# Development
bun run dev                    # Frontend only (Vite dev server on port 1420)
bun run server:dev             # Backend server in dev mode

# Production
bun run server                 # Run HTTP/WebSocket server (release build)
bun run server:build           # Build server binary only

# Testing
bun run test                   # Unit tests (Vitest) - NOTE: use "bun run test", not "bun test"
bun run test:run               # Run tests once
bun run test:coverage          # With coverage report
bun run cargo:test             # Rust backend tests (650+ tests)
/e2e                           # E2E tests via Claude Code skill

# Code Quality
bun run lint                   # ESLint (strict, 0 warnings allowed)
bun run lint:fix               # Auto-fix lint issues
bun run format                 # Prettier format (frontend)
cargo fmt                      # Rust format (run from server/ dir)
cargo clippy -- -D warnings    # Rust lints (run from server/ dir)

# Building
bun run cargo:build            # Production backend build
bun run build                  # Production frontend build
```

## Quick Start

```bash
# Terminal 1: Start the backend server
bun run server:dev

# Terminal 2: Start the frontend dev server
bun run dev

# Open http://localhost:1420 in browser
# Enter the auth token displayed by the server

# Install git hooks (recommended, one-time setup)
./scripts/install-hooks.sh
```

## Running the Server

```bash
# Start server with default settings (port 3420, bind 0.0.0.0)
bun run server

# Development mode (faster builds, debug symbols)
bun run server:dev

# Development with fixed token (no need to re-enter after restart)
bun run server:dev:token

# Custom port/bind/token
cd server && cargo run -- --port 8080 --bind 127.0.0.1 --token my-secret-token

# Or use environment variable
RALPH_SERVER_TOKEN=my-secret-token bun run server:dev
```

The server displays a startup banner with:
- Server URL (e.g., `http://0.0.0.0:3420`)
- Auth token (32-char hex string, or your custom token)

## Browser Connection

When accessing the frontend from a browser, a connection dialog appears automatically. Enter:
1. **Server URL**: `http://localhost:3420` (or your server address)
2. **Auth Token**: Copy from server startup output

The connection is stored in localStorage and persists across page reloads.

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/invoke` | POST | Command proxy - routes to backend commands |
| `/ws/events` | GET | WebSocket for real-time events |
| `/health` | GET | Health check |
| `/` | GET | Connection instructions page |

## Authentication

All requests require Bearer token authentication:
```bash
curl -X POST http://localhost:3420/api/invoke \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"cmd": "get_sessions", "args": {"projectPath": "/path/to/project"}}'
```

WebSocket connections pass token as query parameter: `/ws/events?token=YOUR_TOKEN`

## Architecture

### Frontend (src/)
- **React 19 + TypeScript** with Vite bundler
- **Zustand stores** (`src/stores/`): Feature-isolated stores (prdChat, prdWorkflow, project, ui, toast, ralphLoop, connection, onboarding, terminal, toolCall)
- **API wrappers** (`src/lib/`): Centralized HTTP/WebSocket calls to backend
- **shadcn/ui components** (`src/components/ui/`): Radix UI + Tailwind CSS
- **Feature components** (`src/components/`): mission-control, tasks, agents, git, prd, dashboard, parallel, etc.

### Backend (server/)
- **Axum + Rust** with tokio async runtime
- **HTTP/WebSocket server** (`src/server/`): API proxy, auth, events
- **Command handlers** (`src/commands/`): 14 modules for business logic
- **File storage** (`src/file_storage/`): JSON files in `.ralph-ui/` for sessions, PRDs, chat
- **Git operations** (`src/git/`): git2-rs integration for branches, worktrees, commits
- **Agent management** (`src/agents/`): Process spawning, rate limiting, log parsing, PTY support
- **Parallel orchestration** (`src/parallel/`): Pool, scheduler, worktree coordination

### Server Architecture

```
server/src/server/
├── mod.rs      # Server setup, router, CORS
├── auth.rs     # Bearer token middleware (tower::Layer)
├── proxy.rs    # Command routing (150+ commands via routing macros)
├── events.rs   # WebSocket broadcaster
└── state.rs    # Shared application state
```

The server uses the **Command Proxy Pattern** - a single `/api/invoke` endpoint routes HTTP requests to command handler functions.

**Proxy routing patterns** (`proxy.rs`):
- `route_async!` / `route_sync!` - Commands returning values
- `route_unit!` / `route_unit_async!` - Commands returning nothing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dario-valles/Ralph-UI](https://github.com/dario-valles/Ralph-UI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
