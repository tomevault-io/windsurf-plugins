---
trigger: always_on
description: Full-stack web app that deploys Claude Code (via `@anthropic-ai/claude-agent-sdk`) as a persistent headless server. Users submit prompts ("jobs") through a web UI; the server executes them with full tool access in named project directories and streams real-time output via WebSocket. Features include an approval system for interactive tools, slash command integration, subagent nesting, and a command palette for quick navigation.
---

# CLAUDE.md — claude-code-server

## Project Overview

Full-stack web app that deploys Claude Code (via `@anthropic-ai/claude-agent-sdk`) as a persistent headless server. Users submit prompts ("jobs") through a web UI; the server executes them with full tool access in named project directories and streams real-time output via WebSocket. Features include an approval system for interactive tools, slash command integration, subagent nesting, and a command palette for quick navigation.

## Repository Layout

```
claude-code-server/
├── package.json               # Root — concurrently (dev), pm2 scripts (prod)
├── ecosystem.config.cjs       # PM2 process config (auto-restart, logging)
├── setup.sh                   # One-click install + launch (dev/prod/install-only)
├── projects/                  # Runtime data dir
│   ├── .state.json            # Persisted projects + jobs (last 200 logs/job)
│   └── <ProjectName>/         # One dir per project; Claude's cwd
├── server/
│   ├── package.json
│   ├── tsconfig.json
│   └── src/
│       ├── index.ts           # ENTIRE server — single file (~900+ lines)
│       └── types.ts           # Shared TypeScript types
└── client/
    ├── package.json
    ├── vite.config.ts
    ├── tsconfig.json
    └── src/
        ├── main.tsx           # Entry (React.StrictMode enabled)
        ├── App.tsx            # Root component + resizable panels + command palette
        ├── types/index.ts     # Client types (manual copy of server types)
        ├── hooks/
        │   ├── useStore.ts    # WebSocket client + global state
        │   ├── api.ts         # REST API client (thin fetch wrapper)
        │   ├── useResizable.ts    # Mouse-drag panel resizer with localStorage persistence
        │   └── useSuggestions.ts   # @file and /command autocomplete engine
        ├── components/
        │   ├── Sidebar.tsx            # Project list + WS connection indicator
        │   ├── JobList.tsx            # Job cards for selected project
        │   ├── JobDetail.tsx          # Chat view, tool renderers, thinking blocks, subagent nesting
        │   ├── NewJobModal.tsx
        │   ├── NewProjectModal.tsx
        │   ├── ApprovalList.tsx       # Pending AskUserQuestion / plan approvals
        │   ├── CommandPalette.tsx     # Cmd+K fuzzy search across all jobs
        │   └── SuggestionDropdown.tsx # Autocomplete popup for @files and /commands
        └── styles/global.css  # All CSS — custom properties, dark theme
```

## Commands

```bash
# One-click setup (interactive, checks Node ≥18, prompts for API key)
./setup.sh              # dev mode: Vite HMR + Express
./setup.sh --prod       # production: builds client + starts PM2 daemon
./setup.sh --prod --fg  # production: builds client + foreground server (no PM2)
./setup.sh --install    # install deps only, no start

# Manual install
cd server && npm install && cd ../client && npm install && cd ..
npm install

# Dev (both server + client with HMR)
npm run dev
# Server: http://localhost:3001
# Client: http://localhost:5173 (Vite, open this)

# Production via PM2 (same as ./setup.sh --prod)
npm start            # builds client + pm2 start (daemon)
npm stop             # pm2 stop
npm restart          # rebuild client + pm2 restart
npm run logs         # pm2 logs
npm run status       # pm2 describe

# Build client only
npm run build:client   # outputs to client/dist/
```

No build step for server — `tsx` executes `.ts` directly. In production, Express serves `client/dist/` as static files. Server restart required on server code changes (no HMR). Client has Vite HMR in dev.

**Production modes**: `./setup.sh --prod` and `npm start` both use PM2 to daemonize the server (auto-restart, log rotation, survives terminal close). Use `./setup.sh --prod --fg` for a foreground process if you need direct stdout/stderr (e.g., debugging, Docker containers).

## Tech Stack

- **Server**: Express 4, ws 8, `@anthropic-ai/claude-agent-sdk`, uuid, cors, tsx (no build)
- **Client**: React 18, Vite 5, lucide-react (icons), no CSS framework
- **Root**: concurrently (dev), pm2 (production)
- **TypeScript**: strict mode in both server and client

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `PORT` | `3001` | Express server port |
| `PROJECTS_ROOT` | `../projects` (relative to server/) | Root dir for project workspaces |
| `ANTHROPIC_API_KEY` | *(required)* | Anthropic API key for Claude Agent SDK |
| `ANTHROPIC_BASE_URL` | *(optional)* | Custom API base URL |

## Architecture

### Communication: REST + WebSocket

Vite proxies `/api` → `http://localhost:3001` and `/ws` → `ws://localhost:3001` in dev. In production, Express serves client/dist and all paths from a single port. Client uses only relative paths.

**REST endpoints** (`/api`):
- `GET/POST /projects` — CRUD projects
- `GET/POST /jobs` — list/create jobs; `GET /jobs/:id` returns full logs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/whi497) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
