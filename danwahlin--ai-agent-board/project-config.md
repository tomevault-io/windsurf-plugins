---
trigger: always_on
description: Agentic AI Kanban Board — a drag-and-drop Kanban board that delegates coding tasks to AI coding agents (GitHub Copilot, Claude Code, OpenAI Codex, OpenCode). Monorepo with npm workspaces.
---

# AGENTS.md

## Project Overview

Agentic AI Kanban Board — a drag-and-drop Kanban board that delegates coding tasks to AI coding agents (GitHub Copilot, Claude Code, OpenAI Codex, OpenCode). Monorepo with npm workspaces.

## Architecture

```
ai-agent-board/
├── packages/
│   ├── client/          # React 19 + Vite + Tailwind 4 + Framer Motion + xterm.js
│   │   └── src/
│   │       ├── components/  # Board, Column, TaskCard, TaskGroupCard, GroupPanel, AgentPanel, TerminalView, FilterChips, Header, dialogs
│   │       ├── hooks/       # useTasks, useTaskGroups, useTheme, useDebounce, useKeyboardShortcuts
│   │       ├── lib/         # api.ts (REST + WebSocket), agent-config.ts, priority-config.ts, columns.ts, utils
│   │       └── types/       # Client-side type re-exports
│   ├── server/          # Express + @codewithdan/agent-sdk-core + better-sqlite3/pg + ws
│   │   └── src/
│   │       ├── middleware/  # auth.ts (Bearer token auth)
│   │       ├── routes/      # tasks.ts, agent.ts, git.ts, templates.ts, groups.ts, helpers.ts
│   │       ├── services/    # agent-manager.ts (session orchestration, event caching)
│   │       ├── repositories/# sqlite.ts, postgres.ts, sqlite-templates.ts, postgres-templates.ts, sqlite-groups.ts, postgres-groups.ts, types.ts, template-types.ts, group-types.ts
│   │       ├── db.ts        # SQLite + PostgreSQL init + migrations
│   │       ├── websocket.ts # WebSocket broadcast
│   │       └── index.ts     # Express app setup + graceful shutdown
│   └── e2e/             # Playwright tests
├── shared/              # Shared types (Task, TaskGroup, TaskTemplate, AgentEvent, ColumnId, AgentType, etc.) + validation constants
├── scripts/             # test-sdk-e2e.sh
└── k8s/                 # Kubernetes manifests (namespace, deployments, services, ingress)
```

## Key Technical Decisions

- **Multi-agent support** — pluggable `AgentProvider`/`AgentSession` interfaces from `@codewithdan/agent-sdk-core`. Four providers: `copilot` (`@github/copilot-sdk`), `claude` (`@anthropic-ai/claude-agent-sdk`), `codex` (`@openai/codex-sdk`), `opencode` (`@opencode-ai/sdk`). Auto-detected at startup via `detectAgents()`.
- **Agent SDK abstraction** — all provider implementations live in the external `@codewithdan/agent-sdk-core` package. The server imports providers and the detection function from this package.
- **Dual database backends** — SQLite via `better-sqlite3` (default, zero config) or PostgreSQL via `pg` (set `DATABASE_URL`). Both implement the `TaskRepository` and `TemplateRepository` interfaces.
- **Route splitting** — REST API is split across `tasks.ts` (CRUD), `agent.ts` (start/stop/events/follow-up), `git.ts` (merge-local, create-pr, worktree cleanup, git-info), `templates.ts` (task template CRUD), and `groups.ts` (group CRUD + run/stop/archive).
- **API key auth** — optional Bearer token via `API_KEY` env var. When set, all API and WebSocket requests require `Authorization: Bearer <key>`. Middleware in `middleware/auth.ts`.
- **Task Groups** — parent entity with N child tasks, concurrency-controlled execution via `GroupQueue` in agent-manager. Groups move as a single card on the board; auto-advance to review when all children complete. Parallelism slider locked once running.
- **Event streaming** — SDK events mapped to `AgentEvent`s, persisted to database, broadcast via WebSocket. In-memory LRU cache (200 tasks max, 100 events per task).
- **Git worktrees** — optional per-task branch isolation. Agent works in worktree directory, path rewriting via `onPreToolUse` hook. Worktrees auto-cleaned after successful merge or PR creation.
- **Local merge** — `mergeLocal()` merges worktree branch into base branch locally with per-repo mutex to prevent concurrent checkout races. Auto-aborts on conflict.
- **Smart PR/merge buttons** — `GET /api/tasks/:id/git-info` checks for remote; UI shows "Create PR" only when remote exists, "Merge to main" always available.
- **Vite proxy** — client proxies `/api` and `/ws` to the server.
- **Shared validation** — `shared/constants.ts` exports validators (`isValidPriority`, `isValidColumnId`, etc.) and limits (`MAX_TITLE_LENGTH`, `MAX_DESCRIPTION_LENGTH`) used by both client and server.

## Running Locally

```bash
npm install
npm run dev:server   # Terminal 1 — port 3001
npm run dev:client   # Terminal 2 — port 4175
```

## Production (systemd)

The production server (`kanban.codewithdan.com`) runs via two systemd services:

| Service | Unit File | Port | Description |
|---------|-----------|------|-------------|
| `ai-agent-board-server` | `/etc/systemd/system/ai-agent-board-server.service` | 3001 | Express API + WebSocket + agent SDKs |
| `ai-agent-board-client` | `/etc/systemd/system/ai-agent-board-client.service` | 4175 | Vite dev server (proxied by nginx/reverse proxy) |

```bash
# Check status
systemctl status ai-agent-board-server ai-agent-board-client

# Restart after code changes
systemctl restart ai-agent-board-server ai-agent-board-client

# View logs
journalctl -u ai-agent-board-server -f
journalctl -u ai-agent-board-client -f

# Restart just one
systemctl restart ai-agent-board-server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanWahlin/ai-agent-board](https://github.com/DanWahlin/ai-agent-board) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
