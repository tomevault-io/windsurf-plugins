---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Obsidian AI Workspace - A Docker-based microservices solution integrating Obsidian with AI Agent executors:
- **Agent Service** - NestJS executor manager with WebUI, Skills system (port 8000)
- **Obsidian** - Desktop app via noVNC (optional, port 3000)
- **Claude Code Executor** - Claude Code CLI HTTP API (port 3002)
- **Playwright Executor** - Web scraping with Playwright (port 53333)

## Build and Run Commands

```bash
# Build and start all containers
docker compose up -d --build

# Start specific services (without Obsidian desktop)
docker compose up -d agent-service claude-executor playwright-executor

# Rebuild without cache
docker compose build --no-cache && docker compose up -d

# View logs
docker compose logs -f [service-name]

# Stop
docker compose down
```

### Production Deployment

Two deployment options available in `deploy/`:
- `deploy/allinone/` - Full stack with Obsidian desktop
- `deploy/headless/` - Agent service and executors only

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                       Agent Service (:8000)                      │
│              NestJS - Executor Manager + WebUI + Skills          │
└────────────────────────────┬────────────────────────────────────┘
                             │ manages/monitors/invokes
        ┌────────────────────┼────────────────────┐
        ▼                    ▼                    ▼
┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐
│    Obsidian     │  │ Claude Code     │  │  Playwright     │
│  (linuxserver)  │  │   Executor      │  │   Executor      │
│     :3000       │  │     :3002       │  │    :53333       │
└────────┬────────┘  └────────┬────────┘  └────────┬────────┘
         │                    │                    │
         └────────────────────┴────────────────────┘
                              │
                        ./vaults (shared)
```

## Service Details

### Agent Service (`agent-service/`)

NestJS application managing executors and providing a skills workflow engine.

**Tech Stack:** NestJS + TypeScript + js-yaml + playwright-core

**Key Modules:**
- `src/executors/` - Executor CRUD and health monitoring
- `src/executor-types/` - Handler implementations (claudecode, playwright, puppeteer, agent)
- `src/skills/` - Workflow engine with YAML-defined skills
- `src/skills/skill-scheduler.service.ts` - Cron/interval scheduling for skills
- `src/skills/task-queue.service.ts` - Task queue for sequential execution

**API Endpoints:**
- `GET /api/executors` - List executors
- `POST /api/executors/:name/invoke` - Invoke executor action
- `POST /api/executors/check-all` - Health check all
- `GET /api/executor-types` - List supported executor types
- `GET /api/skills` - List available skills
- `POST /api/skills/:id/execute` - Execute a skill with user inputs
- `GET /api/skills/:id/execute-stream` - Execute skill with SSE event streaming
- `GET /api/skills/definitions` - Get all skill definitions
- `POST /api/skills/smart-fetch` - Built-in web clip workflow
- `GET /api/skills/schedules` - Get all skill schedule statuses
- `GET /api/skills/schedules/history` - Get execution history
- `GET /api/skills/task-queue/status` - Get task queue status
- `PUT /api/skills/:id/schedule` - Update skill schedule
- `POST /api/skills/:id/schedule/trigger` - Manually trigger scheduled skill

**Configuration:**
- `config/executors.yaml` - Executor instances configuration
- `config/skills.yaml` - Skills definitions and smartFetch config

**Development:**
```bash
cd agent-service
npm install
npm run start:dev   # Watch mode
npm run build       # Build
npm run lint        # ESLint
npm run format      # Prettier
```

### Claude Code Executor (`claude-executor/`)

Express server wrapping Claude Code CLI. Spawns `claude` CLI process per request.

**Endpoints:**
- `GET /health` - Health check
- `POST /v1/chat/completions` - OpenAI-compatible API (streaming SSE)
- `POST /api/chat` - Simplified chat endpoint

**Key implementation:** `src/routes/chat.ts` - spawns Claude CLI with `--output-format stream-json --dangerously-skip-permissions`

**Development:**
```bash
cd claude-executor
npm install
npm run dev     # ts-node development mode
npm run build   # TypeScript build
npm start       # Production mode
```

### Playwright Executor

Uses pre-built image `jacoblincool/playwright:chromium-light-server` exposing WebSocket endpoint at `/playwright`.

Agent Service connects via `playwright-core` library to perform:
- `fetch` - Page content extraction with @mozilla/readability
- `screenshot` - Full page or viewport screenshots

Handler implementation: `agent-service/src/executor-types/handlers/playwright.handler.ts`

## Skills System

The skills system allows defining multi-step workflows combining different executors.

**Skill Definition Structure (in `config/skills.yaml`):**
```yaml
skills:
  - id: unique-id
    name: Display Name
    description: What this skill does
    enabled: true
    builtinVariables:
      currentDate: true
      randomId: true
    userInputs:
      - name: url
        label: Input Label

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sunshow/obsidian-ai-workspace](https://github.com/Sunshow/obsidian-ai-workspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
