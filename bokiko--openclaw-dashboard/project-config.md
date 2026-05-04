---
trigger: always_on
description: You are setting up the OpenClaw Dashboard for the user. Follow these instructions exactly. Do not skip steps. Do not improvise alternatives. Every step has been tested.
---

# OpenClaw Dashboard — Agent Setup Guide (v2.0.0)

You are setting up the OpenClaw Dashboard for the user. Follow these instructions exactly. Do not skip steps. Do not improvise alternatives. Every step has been tested.

This guide works with any AI agent — Claude, GPT, Gemini, Kimi, LLaMA, Copilot, or any other model.

---

## What This Is

A real-time web dashboard (v2.0.0) for the [OpenClaw](https://openclaw.io) agent swarm. It provides full operational visibility: agents, tasks, routines, chat, notifications, and metrics.

The dashboard supports three deployment modes:

| Mode | What you need | What you get |
|------|--------------|--------------|
| **File-based** (minimal) | Node.js only | Reads task JSON files from disk, Kanban board, metrics |
| **DB-backed** (full platform) | Node.js + PostgreSQL | Everything above + task CRUD, checklists, comments, notifications, chat, routines, WebSocket real-time updates |
| **Gateway** (live cluster) | Node.js + OpenClaw gateway | Everything above + live session data from the OpenClaw agent cluster via WebSocket RPC |

All three can be combined. The dashboard auto-detects which backends are available.

---

## Prerequisites

Before starting, confirm these exist on the machine:
- **Node.js 18+** (`node --version`)
- **npm** (`npm --version`)
- **git** (`git --version`)

Optional:
- **PostgreSQL** — enables full v2.0 features (task CRUD, notifications, chat, routines). Docker works: `docker ps | grep postgres`
- **OpenClaw gateway** — enables live cluster integration (sessions, agents, cron jobs)

**Minimum viable setup:** Node.js + npm only. The dashboard runs with file-based tasks and no auth.

If any required prerequisite is missing, install it before continuing.

---

## Installation

### Step 1: Clone and install

```bash
git clone https://github.com/bokiko/openClaw-dashboard.git
cd openClaw-dashboard
npm install
```

### Step 2: Create environment file

```bash
cp .env.example .env.local
```

### Step 3: Configure .env.local

Ask the user which mode they want, then set the appropriate variables:

```bash
# ── File-based mode (always available) ────────────────────────────
# Path to the directory containing task JSON files.
# The dashboard reads *.json from this directory.
OPENCLAW_TASKS_DIR=./tasks

# ── Auth (recommended for production) ─────────────────────────────
# Operator password for login protection.
# Session stored as JWT (HS256) in httpOnly cookie.
# If unset, dashboard has open access (fine for localhost).
DASHBOARD_SECRET=

# ── Database mode (optional, enables full v2.0) ──────────────────
# PostgreSQL connection string.
# Enables: task CRUD, checklists, comments, notifications, chat, routines.
# If unset, falls back to file-based mode.
# DATABASE_URL=postgresql://user:password@localhost:5432/openclaw_dashboard

# ── Gateway mode (optional, live cluster integration) ─────────────
# OpenClaw gateway WebSocket RPC connection.
# Enables: live sessions, worker status, cron job management, agent chat via gateway.
# GATEWAY_WS_URL=ws://127.0.0.1:18789
# GATEWAY_TOKEN=your-gateway-token
# GATEWAY_HEALTH_URL=http://127.0.0.1:18792
# Dashboard origin sent as WS header (must match gateway's controlUi.allowedOrigins)
# GATEWAY_ORIGIN=http://192.168.1.100:3000
# Client ID for gateway handshake (default: openclaw-control-ui)
# GATEWAY_CLIENT_ID=openclaw-control-ui
# Scopes to request (default: operator.admin,operator.read,operator.write,operator.talk)
# GATEWAY_SCOPES=operator.admin,operator.read,operator.write,operator.talk

# ── Agent chat (optional) ────────────────────────────────────────
# Anthropic API key for direct agent chat (when not using gateway).
# ANTHROPIC_API_KEY=
```

**Important:** Only uncomment and fill in the sections the user needs. File-based mode requires zero configuration beyond `OPENCLAW_TASKS_DIR`.

**Gateway note:** If using gateway mode, the OpenClaw gateway needs two config changes:
1. Add the dashboard's origin (e.g. `http://192.168.x.x:3000`) to `controlUi.allowedOrigins`
2. Set `controlUi.allowInsecureAuth: true` if the dashboard is on HTTP (local network without HTTPS)

Without these, the gateway strips RPC scopes after auth and all data calls fail with "missing scope: operator.read". Run `openclaw doctor --non-interactive` after updating the gateway config to apply.

### Step 4: Create the tasks directory

```bash
mkdir -p tasks
```

### Step 5: Database setup (if using DB mode)

Skip this step if not using PostgreSQL.

```bash
# Run all migrations
npm run setup-db
```

Or run manually:
```bash
psql $DATABASE_URL -f scripts/migrations/002_dashboard_tasks.sql
psql $DATABASE_URL -f scripts/migrations/003_agents.sql
psql $DATABASE_URL -f scripts/migrations/004_activity_log.sql
psql $DATABASE_URL -f scripts/migrations/005_notifications.sql
psql $DATABASE_URL -f scripts/migrations/006_chat_messages.sql
psql $DATABASE_URL -f scripts/migrations/007_routines.sql
psql $DATABASE_URL -f scripts/migrations/008_task_extras.sql
```

To migrate existing file-based tasks into the database:
```bash
npm run migrate-files
```

### Step 6: Personalize the dashboard


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bokiko/openClaw-dashboard](https://github.com/bokiko/openClaw-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
