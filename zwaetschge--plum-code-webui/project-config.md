---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository

Web UI for Claude Code CLI (with multi-provider support for Codex and OpenCode). pnpm monorepo deployed as a single Docker container on Unraid.

## Commands

```bash
pnpm install                # install workspace deps
pnpm dev                    # run backend + frontend in parallel (tsx watch + vite)
pnpm build                  # build all packages (tsc for backend/shared, vite for frontend)
pnpm typecheck              # tsc --noEmit across workspace
pnpm lint                   # eslint
pnpm format                 # prettier --write
pnpm format:check           # prettier --check (CI)

./scripts/install.sh        # interactive installer (prereq check, .env gen, build, up, claude /login). Re-runnable; --reset wipes .env, --skip-login skips OAuth bootstrap, --non-interactive uses defaults.
./scripts/start-webui.sh    # dev helper: generates ephemeral SESSION_SECRET/JWT_SECRET, kills stale PIDs, logs to .logs/, writes PIDs to .pids/

# Backend-specific (run from packages/backend)
pnpm db:migrate             # apply SQLite migrations (better-sqlite3)
pnpm db:seed                # seed dev data
```

Dev ports: backend `3006`, frontend `5173`. Docker maps `4545:3001` (container listens on 3001).

Node `>=20`, pnpm `>=9` (packageManager pinned to `pnpm@9.15.0`).

## Architecture

### Packages

| Package | Purpose |
|---------|---------|
| `packages/backend` | Express + Socket.IO server, SQLite via better-sqlite3, spawns Claude/Codex/OpenCode CLIs as child processes |
| `packages/frontend` | React 18 + Vite SPA, Radix UI, Tailwind, Zustand, Socket.IO client |
| `packages/shared` | TypeScript types shared between backend and frontend |
| `packages/desktop` | Desktop shell wrapper |
| `packages/android` | Android client |

### Backend

Entry: `packages/backend/src/index.ts`. Routes live in `src/routes/` (~30 modules — sessions, auth, providers, files, git, github, mcp, etc.). Services in `src/services/` — the critical one is `src/services/claude/ClaudeProcessManager.ts`, which owns all CLI lifecycle.

**CLI process model** (`ClaudeProcessManager`):
- Spawns Claude CLI with `--print --verbose --output-format stream-json --input-format stream-json --include-partial-messages --dangerously-skip-permissions`.
- Also manages Codex (with per-turn respawn after `turn.completed`) and OpenCode processes.
- Parses stream-json events and forwards them over Socket.IO.
- Message queue accepts input while the CLI is working; interrupts via SIGINT.

**Key Socket.IO events** (server → client):
- `session:output` — streaming text deltas
- `session:message` — persisted messages
- `session:thinking` — thinking indicator (boolean)
- `session:tool_use` — tool lifecycle (started/completed/error)
- `session:agent` — subagent (Task tool) activity
- `session:status` — session state changes

**Auth**: Express session + JWT, Passport strategies for GitHub and Google OAuth, plus a Basic Auth guard stored in SQLite (`app_config` table).

**Security middleware** (`src/index.ts`): strict Helmet CSP (no `unsafe-inline` scripts), `trust proxy` configurable via `TRUST_PROXY`, per-bucket rate limiters in `src/middleware/rateLimiter.ts` (key = `userId` or `req.ip`, never raw `X-Forwarded-For`). CORS origins from `FRONTEND_URL` + `CORS_ALLOWED_ORIGINS`.

### Frontend

Entry: `packages/frontend/src/main.tsx`. Main chat view: `src/pages/SessionPage.tsx`. Tool rendering: `src/components/chat/ToolExecutionCard.tsx` — detects `Task`/`Agent` tools as subagents and renders them with a distinct border-left accent, tinted background, and "SUBAGENT" badge. Subagent icon mapping lives in the same file (`agentTypeMap`).

Store: `src/stores/useSessionStore.ts` (Zustand) holds per-session `toolExecutions`, `activity`, and `activeAgent` state.

WebSocket client: `src/services/socket.ts`.

## Deployment

Compose is split into two files:
- **`docker-compose.yml`** — portable, in git. Single `claude-code-webui` service on `${WEBUI_PORT:-4545}:3001`. Volumes use env-var-driven defaults (`${DATA_DIR:-./data}`, `${CONFIG_DIR:-./config}`, `${WORKSPACE_DIR:-./workspace}`). Safe to publish.
- **`docker-compose.override.yml`** — site-specific, **gitignored**. Holds Traefik labels for `code.zwaetschge-webui.ch`/`preview.code.zwaetschge-webui.ch`, the `group_add: 281` Unraid docker GID, absolute `/mnt/user/appdata/...` host paths, the `repair-bot` sidecar, the docker.sock mount, and the external `brian_traefik-public` network. Compose merges both automatically — `docker compose up -d --build` Just Works.

A template for other operators lives at `docker-compose.override.yml.example`.

```bash
./scripts/install.sh                # interactive: collects env, builds, starts, runs claude /login
docker compose up -d --build        # if you already have .env + an override
```

**docker.sock mount lives in the override file**, not the portable compose. Mounting it grants the in-container CLI full host Docker access — required for the repair-bot self-rebuild flow but not safe to ship as a default.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zwaetschge/plum-code-webui](https://github.com/zwaetschge/plum-code-webui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
