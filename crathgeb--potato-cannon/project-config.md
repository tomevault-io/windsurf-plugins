---
trigger: always_on
description: Multi-agent software engineering daemon for autonomous development pipelines. Orchestrates Claude Code sessions through configurable workflow phases with adversarial review loops.
---

# Potato Cannon

Multi-agent software engineering daemon for autonomous development pipelines. Orchestrates Claude Code sessions through configurable workflow phases with adversarial review loops.

## Commands

```bash
# Development
pnpm install          # Install all dependencies
pnpm dev              # Start daemon + frontend (concurrent)
pnpm dev:daemon       # Start daemon only (watches + rebuilds)
pnpm dev:frontend     # Start frontend only (Vite dev server)
pnpm dev:desktop      # Start Electron desktop app

# Build
pnpm build            # Build all packages
pnpm build:shared     # Build shared types first (if dependency issues)
pnpm typecheck        # TypeScript check all packages

# Testing
pnpm test             # Run all tests
cd apps/frontend && pnpm test  # Frontend tests (Vitest)
cd apps/daemon && pnpm test    # Daemon tests (Node test runner)

# CLI (after build)
./apps/daemon/bin/potato-cannon.js start          # Start daemon
./apps/daemon/bin/potato-cannon.js status         # Check daemon status
./apps/daemon/bin/potato-cannon.js stop           # Stop daemon
```

## Code Style

- TypeScript strict mode, ES modules (`"type": "module"`)
- Prefer `async/await` over raw promises
- Use factory functions for dependency injection (e.g., `createTicketStore(db)`)
- SQLite operations are synchronous (better-sqlite3), file I/O is async
- ISO 8601 timestamps for all dates stored in database
- Path alias `@/` for frontend imports from `src/`
- Tailwind CSS for styling, container queries over media queries
- Radix UI for accessible primitives, shadcn/ui patterns for components

## Architecture

```
potato-cannon-monorepo/
├── apps/
│   ├── daemon/        # @potato-cannon/daemon - Express server, MCP, SQLite
│   ├── frontend/      # @potato-cannon/frontend - React 19, Vite, Tailwind
│   └── desktop/       # @potato-cannon/desktop - Electron wrapper
└── packages/
    └── shared/        # @potato-cannon/shared - Types, constants
```

### Data Flow

```
User (Web UI / Telegram)
         │
         ▼
┌─────────────────────────────────────────────────────┐
│  Daemon (Express on :8443)                          │
│  ├── /api/*          REST endpoints                 │
│  ├── /events         SSE for real-time updates      │
│  └── /mcp/*          MCP proxy bridge               │
└─────────────────────────────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────────┐
│  MCP Proxy (per Claude session)                     │
│  stdio ↔ HTTP bridge to daemon                      │
└─────────────────────────────────────────────────────┘
         │
         ▼
Claude Code (PTY process)
```

### Worker Execution Model

Workflow phases define a `workers` array. Workers form a tree:

| Type | Purpose |
|------|---------|
| `agent` | Single Claude session execution |
| `ralphLoop` | Adversarial review until approved or maxAttempts |
| `taskLoop` | Iterates nested workers over task queue |

State is persisted per-ticket at `~/.potato-cannon/projects/{projectId}/tickets/{ticketId}/worker-state.json` for crash recovery.

Key files:
- `apps/daemon/src/services/session/worker-executor.ts` - Tree interpreter
- `apps/daemon/src/services/session/loops/ralph-loop.ts` - Review loop
- `apps/daemon/src/services/session/loops/task-loop.ts` - Task iteration

## Key Files

### Daemon

| Path | Purpose |
|------|---------|
| `apps/daemon/src/server/server.ts` | Express server, routes, SSE |
| `apps/daemon/src/mcp/proxy.ts` | stdio↔HTTP bridge for Claude |
| `apps/daemon/src/mcp/tools/*.ts` | MCP tool definitions |
| `apps/daemon/src/services/session/session.service.ts` | Claude session lifecycle |
| `apps/daemon/src/stores/*.ts` | SQLite data access layer |
| `apps/daemon/src/config/paths.ts` | File system path constants |
| `apps/daemon/templates/workflows/` | Workflow templates + agents |

### Frontend

| Path | Purpose |
|------|---------|
| `apps/frontend/src/routes/` | TanStack Router file-based routes |
| `apps/frontend/src/components/ui/` | shadcn-style UI primitives |
| `apps/frontend/src/hooks/queries.ts` | TanStack Query API wrappers |
| `apps/frontend/src/stores/` | Zustand client state |
| `apps/frontend/src/api/client.ts` | API fetch functions |

### Shared

| Path | Purpose |
|------|---------|
| `packages/shared/src/types/` | TypeScript interfaces |
| `packages/shared/src/constants/` | Shared constants |

## Database

**Location:** `~/.potato-cannon/potato.db` (SQLite with WAL mode)

| Table | Purpose |
|-------|---------|
| `projects` | Registered projects with template info |
| `tickets` | Ticket metadata, phase, worker_state JSON |
| `brainstorms` | Brainstorm sessions |
| `conversations` | Chat containers (linked from tickets/brainstorms) |
| `conversation_messages` | Message history |
| `sessions` | Claude session tracking |
| `tasks` | Tasks within tickets |
| `artifacts` | Artifact metadata (files on disk) |

**Migrations:** `apps/daemon/src/stores/migrations.ts` - uses `user_version` pragma

**Store APIs:** See `apps/daemon/src/stores/CLAUDE.md` for detailed store documentation

## File System Layout

```
~/.potato-cannon/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [crathgeb/potato-cannon](https://github.com/crathgeb/potato-cannon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
