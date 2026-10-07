---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DGX Manager is a full-stack system for managing a DGX Spark cluster. It handles node provisioning via SSH, real-time GPU metrics collection, model deployment (via [sparkrun](https://github.com/spark-arena/sparkrun) — the head-node agent runs `sparkrun run`), inference load balancing, and fine-tuning job orchestration.

## Architecture

npm workspaces monorepo with three packages:

- **`packages/server`** — Express 5 backend (REST API + WebSocket hubs), port 4000
- **`packages/dashboard`** — Next.js 15 frontend (React 19 + Tailwind CSS 4), port 3000
- **`packages/agent`** — Node.js agent deployed on DGX nodes, collects GPU metrics via nvidia-smi

Communication flow:
```
Dashboard <--WS /ws/dashboard--> Server <--WS /ws/agent--> Agent (on DGX node)
                                   |
                              Prisma (SQLite)
```

The server has REST routes at `/api/*`. An inference gateway fronting every running deployment is in progress — see [ADR 0001](docs/adr/0001-inference-gateway.md); the load balancer it replaces has been removed.

## Running the App

**Always use Docker Compose to run the app:**

```bash
# One-time host setup: register QEMU binfmt handlers so we can cross-build
# the amd64 agent bundle from an arm64 manager (and vice versa).
docker run --privileged --rm tonistiigi/binfmt --install all

# Build per-arch agent bundles (amd64 + arm64) before every compose build.
./scripts/build-agent-bundles.sh

# Start (set your machine's IP and SSH user)
MANAGER_ADVERTISE_HOST=192.168.44.36 SSH_USER=daniel docker compose up -d

# Rebuild after code changes (non-disruptive — won't kill active deployments)
./scripts/build-agent-bundles.sh && \
  MANAGER_ADVERTISE_HOST=192.168.44.36 SSH_USER=daniel docker compose up -d --build

# Stop (WARNING: removes docker network, may disrupt active Ray clusters)
docker compose down

# Logs
docker compose logs server -f
docker compose logs dashboard -f
```

- Server runs on port 4000, dashboard on port 3000
- SQLite DB persists in a Docker volume (`dgx-data`)
- SSH keys mounted from host `~/.ssh` for node management
- `NEXT_PUBLIC_*` vars are baked into the dashboard at build time via build args

## Development Commands

```bash
npm run dev              # Run server + dashboard locally (dev mode, no Docker)
npm run dev:server       # Server only (tsx watch, port 4000)
npm run dev:dashboard    # Dashboard only (next dev, port 3000)
npm run build            # Build all packages (tsc + next build)

npm run db:push          # Apply Prisma schema changes to SQLite
npm run db:generate      # Regenerate Prisma client
npm run db:studio        # Open Prisma Studio GUI
```

## Key Source Locations

### Server (`packages/server/src/`)
- `index.ts` — Express setup, WebSocket hub init, route mounting
- `routes/` — REST handlers: nodes, models, deployments (recipeFile | recipePath | inline recipeYaml), finetune, recipes
- `deployments/recipe-path.ts`, `deployments/recipe-inline.ts` — validate the path / inline-YAML deploy sources (security boundary)
- `ws/agent-hub.ts` — Manages agent WebSocket connections, processes metrics
- `ws/dashboard-hub.ts` — Broadcasts updates to connected dashboards
- `ssh/provisioner.ts` — Audits prerequisites, auto-installs packages on nodes (incl. sparkrun install + non-interactive setup)
- `routes/agent-bundle.ts` — Serves per-arch agent tarballs + generates the token install script
- `gateway/` — the inference gateway at `/v1` (OpenAI-compatible). `models.ts` builds the model list from published names; `router.ts` is the allowlist — anything not served is refused without reaching a node. Mounted BEFORE `express.json()` (see ADR 0001)
- `openapi.ts` — builds the OpenAPI 3 spec (served at `/api/openapi.json`, Swagger UI at `/api/docs`)

### Dashboard (`packages/dashboard/`)
- `app/` — Next.js App Router pages (overview, nodes, models, deployments, finetune)
- `components/` — node-card, metric-gauge, deployment-table, finetune-log
- `lib/api.ts` — Fetch wrapper; `lib/ws.ts` — useWebSocket hook

### Agent (`packages/agent/src/`)
- `index.ts` — WebSocket client, reconnection logic, 5-second metrics loop; `cmd:deploy`/`cmd:undeploy` handlers + reconnect reconciliation
- `metrics.ts` — nvidia-smi wrapper for GPU metrics
- `recipes.ts` — recipe catalog from `sparkrun list`, mapped to the wire `Recipe` shape
- `runtime/sparkrun.ts` — deploy lifecycle: `sparkrun run`/`stop`, `cluster check-job` liveness, and the `sparkrun logs` follower that streams container logs
- `runtime/sparkrun-args.ts` · `sparkrun-parse.ts` · `sparkrun-metrics.ts` — pure argv builder, CLI output parsers, vLLM `/metrics` scrape
- `runtime/ollama.ts`, `runtime/finetune.ts` — Ollama deploys; fine-tune training/merge (fine-tune deploy also goes through sparkrun)

## Database

SQLite via Prisma ORM. Schema at `prisma/schema.prisma`. Core models: Node, Model, Deployment, MetricSnapshot, FineTuneJob.

## Environment Variables

Copy `.env.example` to set up. Key vars:
- `DATABASE_URL` — Prisma DB path (default: `file:./dev.db`)
- `PORT` / `MANAGER_HOST` — Server bind config

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kreuzhofer/dgx-manager](https://github.com/kreuzhofer/dgx-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
