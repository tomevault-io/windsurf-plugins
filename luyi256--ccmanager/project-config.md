---
trigger: always_on
description: - Do not use `curl` against any remote server. Remote service checks and operations must use another explicitly permitted mechanism. `curl` is allowed only for local loopback services such as `127.0.0.1` or `localhost`.
---

# CC Manager

## Repository-specific agent rules

- Do not use `curl` against any remote server. Remote service checks and operations must use another explicitly permitted mechanism. `curl` is allowed only for local loopback services such as `127.0.0.1` or `localhost`.

Multi-device task management system for Claude Code — manage Claude Code task execution across multiple devices via Web UI.

## Tech Stack

| Package | Technology | Description |
|---|---|---|
| `@ccmanager/server` | Express + Socket.IO + better-sqlite3 | API server, WebSocket, SQLite |
| `@ccmanager/web` | React 18 + Vite + TailwindCSS + TanStack Query | SPA frontend |
| `@ccmanager/agent` | Socket.IO Client + child_process | Connects to server, spawns `claude` CLI to execute tasks |

- **Language**: TypeScript (strict mode)
- **Package Manager**: pnpm 9.0 (monorepo workspace)
- **Runtime**: Node.js >= 18
- **Process Manager**: PM2

## Architecture

```
Server Host
├── CCManager/              - Code repository
├── <DATA_PATH>/            - Data directory (SQLite DB, config, etc.)
├── ccm-server              - PM2 daemon (port 3001)
└── ccm-agent               - PM2 daemon (connects to local server)

Remote Machines (MacBook, Linux, etc.)
├── ccm-agent        - Connects to server to execute tasks
└── ccm-tunnel       - Cloudflare tunnel (optional)
```

## Access URLs

- **Web UI**: http://localhost:3001 (requires API Token login)
- **API**: http://localhost:3001/api (requires `Authorization: Bearer <API_TOKEN>` header)
- **Health Check**: http://localhost:3001/api/health (no auth required)
- **Web Dev**: http://localhost:5173 (proxies `/api` and `/socket.io` to 3001)

## Project Structure

```
packages/
├── server/src/
│   ├── index.ts              # Express entry, route registration, WebSocket
│   ├── cli/
│   │   ├── index.ts          # ccmng CLI entry
│   │   └── token.ts          # Device token create/list/revoke
│   ├── routes/
│   │   ├── agents.ts         # Agent routes
│   │   ├── auth.ts           # Device auth (GET /me, devices CRUD)
│   │   ├── projects.ts       # Project CRUD
│   │   ├── tasks.ts          # Task CRUD, cancel, retry, continue, plan review
│   │   ├── settings.ts       # Global settings, auth validation
│   │   └── transcribe.ts     # Voice-to-text
│   ├── services/
│   │   ├── database.ts       # SQLite connection and schema
│   │   ├── storage.ts        # Data access layer
│   │   ├── agentPool.ts      # Agent registration and task dispatch
│   │   ├── streamParser.ts   # Claude Code stream-json output parser
│   │   ├── waitingTasks.ts   # Background task polling (node-cron, checks every minute, max 20 retries)
│   │   └── claudemd.ts       # CLAUDE.md context management
│   ├── websocket/index.ts    # Socket.IO namespaces and events
│   └── types/index.ts
├── web/src/
│   ├── App.tsx, main.tsx, index.css
│   ├── pages/
│   │   ├── HomePage.tsx      # Project list
│   │   └── ProjectPage.tsx   # Task board
│   ├── components/
│   │   ├── Layout/AppLayout.tsx         # Top nav, connection status
│   │   ├── Project/                     # AddProjectModal, ProjectCard, ProjectList
│   │   ├── Task/                        # TaskBoard, TaskCard, TaskColumn, TaskDetail, TaskInput
│   │   └── common/                      # ErrorBoundary, Modal, SafeMarkdown, StatusBadge, VoiceInput
│   ├── contexts/WebSocketContext.tsx     # Socket.IO provider
│   ├── hooks/                           # useProjects, useTasks, useTaskStream, useVoiceInput
│   ├── services/api.ts                  # API client (3 retries, exponential backoff)
│   └── types/index.ts
├── agent/src/
│   ├── index.ts              # CLI entry, config loading and validation
│   ├── connection.ts         # WebSocket connection, heartbeat (30s), concurrent task Map
│   ├── executor.ts           # spawn claude CLI (stream-json, 4-hour timeout)
│   ├── docker.ts             # Docker container execution (mount /workspace, credential injection, HOME=/home/ccm)
│   ├── security.ts           # Path validation (incl. symlink check), env var whitelist
│   └── types.ts              # AgentConfig, TaskRequest, DockerConfig, TaskResult types
```

## Database Schema

```sql
config (key, value, updated_at)
agents (id, name, capabilities, executor, status, last_seen, created_at)
projects (id, name, agent_id, project_path, security_mode, auth_type, created_at, last_activity)
tasks (id, project_id, prompt, status, is_plan_mode, depends_on, worktree_branch,
       created_at, started_at, completed_at, error, waiting_until, wait_reason,
       check_command, continue_prompt, git_info, summary, security_warnings, pending_permission)
task_logs (id, task_id, timestamp, type, content)
```

## Development

```bash
pnpm install                  # Install dependencies

pnpm run dev                  # Start server (3001) + web (5173)
pnpm run dev:server           # Server only
pnpm run dev:web              # Web only

pnpm run build                # Build server + web
pnpm run build:server         # Build server only
pnpm run build:web            # Build web only

pnpm run start                # Start production server

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [luyi256/CCManager](https://github.com/luyi256/CCManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
