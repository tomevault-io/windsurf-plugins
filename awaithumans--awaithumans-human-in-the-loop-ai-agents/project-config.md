---
trigger: always_on
description: This file is the source of truth for how code is written in this repo. It is
---

# awaithumans — Codebase Guide

This file is the source of truth for how code is written in this repo. It is
read by human developers, AI coding agents (Claude Code, Copilot, Cursor),
and CI. Follow it exactly.

---

## What This Project Is

An open source infrastructure package that lets AI agent workflows delegate
tasks to human beings — with task routing, notifications (Slack, email),
a review dashboard, AI verification of completed work, and a callback system
to resume the agent when the human is done.

**One primitive:** `await_human()` / `awaitHuman()` — the agent awaits a human
like it awaits a promise.

---

## Architecture Overview

The project has three packages:

1. **Python package** (`packages/python/`) — the API server, CLI, channels,
   verification, AND the Python SDK. Published as `awaithumans` on PyPI.
   This is the brain of the system.

2. **Dashboard** (`packages/dashboard/`) — Next.js 16 web UI. Pre-built to
   static files and bundled into the Python package for production. Separate
   dev server for development.

3. **TypeScript SDK** (`packages/typescript-sdk/`) — the npm `awaithumans`
   package. A thin HTTP client to the Python API server. Includes adapter
   subpath exports for Temporal and LangGraph.

```
Developer's agent code                  The awaithumans system
─────────────────────                   ──────────────────────

  Python agent                           ┌─────────────────────────┐
  ┌──────────────┐      HTTP             │  API Server (Python)    │
  │ from         │ ──────────────────►   │  FastAPI + SQLModel     │
  │ awaithumans  │                       │                         │
  │ import       │                       │  Owns:                  │
  │ await_human  │                       │  - Task store (DB)      │
  └──────────────┘                       │  - Slack channel        │
                                         │  - Email channel        │
  TypeScript agent                       │  - AI verification      │
  ┌──────────────┐      HTTP             │  - Webhook dispatch     │
  │ import {     │ ──────────────────►   │  - Long-poll endpoint   │
  │  awaitHuman  │                       │  - Audit trail          │
  │ } from       │                       └────────────┬────────────┘
  │ "awaithumans"│                                    │
  └──────────────┘                                    │ serves static files
                                         ┌────────────▼────────────┐
                                         │  Dashboard (Next.js)    │
                                         │  Pre-built static files │
                                         │  Task queue, audit log  │
                                         └─────────────────────────┘
```

---

## Monorepo Structure

```
awaithumans/
├── packages/
│   ├── python/                       # PyPI: awaithumans (SDK + server + CLI)
│   │   ├── awaithumans/
│   │   │   ├── __init__.py           # SDK public API: await_human, types, errors
│   │   │   ├── client.py             # await_human() async + await_human_sync()
│   │   │   ├── types.py              # Pydantic models: AwaitHumanOptions, TaskRecord, etc.
│   │   │   ├── errors.py             # Error classes (what → why → fix → docs pattern)
│   │   │   ├── temporal.py           # Temporal adapter (pip install "awaithumans[temporal]")
│   │   │   ├── langgraph.py          # LangGraph adapter (pip install "awaithumans[langgraph]")
│   │   │   ├── verifier_claude.py    # Claude verifier config helper
│   │   │   │
│   │   │   ├── server/               # FastAPI server (pip install "awaithumans[server]")
│   │   │   │   ├── __init__.py
│   │   │   │   ├── app.py            # FastAPI app factory + dashboard static mount
│   │   │   │   ├── routes/           # One file per route group (tasks, webhooks, auth, health)
│   │   │   │   ├── db/               # SQLModel schema, Alembic migrations, connection
│   │   │   │   ├── services/         # Business logic (task lifecycle, notifications, verification)
│   │   │   │   ├── channels/         # Slack (slack-sdk) + Email (resend) — server-side
│   │   │   │   └── verification/     # AI verifier execution (anthropic SDK, etc.)
│   │   │   │
│   │   │   └── cli/                  # CLI commands
│   │   │       └── main.py           # `awaithumans dev`, `awaithumans add-user`, `awaithumans version`
│   │   │
│   │   ├── pyproject.toml            # One package, multiple extras: [server], [temporal], [langgraph]
│   │   └── tests/
│   │
│   ├── dashboard/                    # Next.js 16 web UI (standalone React app)
│   │   ├── app/                      # App Router pages
│   │   ├── components/               # shadcn/ui components (brand palette: #0A0A0A / #F5F5F5 / #00E676)
│   │   ├── lib/                      # API client, hooks, utilities
│   │   ├── generated/                # TypeScript types auto-generated from OpenAPI spec
│   │   └── package.json
│   │
│   └── typescript-sdk/               # npm: awaithumans (TS SDK — HTTP client only)
│       └── src/
│           ├── index.ts              # Public API: awaitHuman, types, errors (re-exports only)
│           ├── types.ts              # All TypeScript interfaces

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awaithumans/awaithumans-human-in-the-loop-ai-agents](https://github.com/awaithumans/awaithumans-human-in-the-loop-ai-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
