---
trigger: always_on
description: This file is the entry point for AI coding agents (and new humans) working on
---

# AGENTS.md — Buzrr guide for coding agents

This file is the entry point for AI coding agents (and new humans) working on
Buzrr. It tells you what the system is, what to read for a given task, the
rules you must not break, and how to keep this documentation alive.

> **Code is the source of truth. Documentation must never override the
> implementation.** If a doc contradicts the code, trust the code, fix the
> doc, and note the discrepancy in your change summary.

## What Buzrr is

Open-source "QuizUp + Kahoot in one app":

- **Classic mode (Kahoot-style)** — a signed-in host creates a quiz and opens a
  room; anonymous players join with a 6-char code and answer live over
  WebSockets.
- **Duel mode (QuizUp-style)** — signed-in users fight ranked 1v1 battles via
  ELO matchmaking (with bot fallback) or unrated friend-invite links.
- Extras: AI quiz generation (Gemini), image questions (Cloudinary), community
  question moderation, roles (user/admin/superadmin).

## Repository shape

Turborepo + Yarn 4 workspaces:

| Path                                                   | What it is                                                                                                              |
| ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------- |
| `apps/web`                                             | Next.js 15 frontend (React 19, App Router). Also hosts **Better Auth** (`/api/auth/*`) — the only web-owned API routes. |
| `apps/server`                                          | NestJS 11 — REST API (`/api/*`) + Socket.IO gateway + the server-authoritative game engine.                             |
| `apps/ai`                                              | **Buzrr-AI** — Python 3.12 + FastAPI + arq worker. Knowledge Spaces, document ingestion, RAG quiz generation. Optional. |
| `packages/prisma`                                      | `@buzrr/prisma`: Prisma schema, migrations, generated client, shared by both apps.                                      |
| `packages/eslint-config`, `packages/typescript-config` | Shared lint/tsconfig presets.                                                                                           |
| `scripts/setup.mjs`                                    | One-command local bootstrap (Docker Postgres+Redis, .env files, schema push).                                           |
| `docs/`                                                | Architecture docs, ADRs, current-state context (see below).                                                             |

Local dev: `yarn setup` then `yarn dev` (web :3000, api :3001). The AI service
is opt-in — `yarn workspace ai setup` then `yarn workspace ai dev` (:3002); with
`NEXT_PUBLIC_AI_API_URL` unset it is invisible to the rest of the app. Details:
[docs/architecture/infrastructure.md](docs/architecture/infrastructure.md).

## Read this before touching anything

1. This file (you're here).
2. [docs/CONTEXT.md](docs/CONTEXT.md) — current state, active work, known debt.
3. [docs/architecture/invariants.md](docs/architecture/invariants.md) — rules
   you must not break.
4. Then the area doc(s) for your task, from the map below.

## Task → reading map

| If your task touches…                                                           | Read                                                                                                   |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| Anything (orientation)                                                          | [ARCHITECTURE.md](ARCHITECTURE.md) then [docs/architecture/overview.md](docs/architecture/overview.md) |
| Live gameplay, phases, timers, scoring, reconnect, kick/ban                     | [docs/architecture/realtime.md](docs/architecture/realtime.md)                                         |
| Matchmaking, duel invites, bots, ELO                                            | [docs/architecture/duels.md](docs/architecture/duels.md) + realtime.md                                 |
| Database schema, Redis keys, what's stored where                                | [docs/architecture/data.md](docs/architecture/data.md)                                                 |
| Login, JWTs, socket auth, roles, guards                                         | [docs/architecture/auth.md](docs/architecture/auth.md)                                                 |
| REST endpoints, Nest modules, validation, rate limiting, moderation             | [docs/architecture/backend.md](docs/architecture/backend.md)                                           |
| React pages, components, Redux/React-Query state, socket hooks                  | [docs/architecture/frontend.md](docs/architecture/frontend.md)                                         |
| Knowledge Spaces, document ingestion, embeddings, RAG, the Python service       | [docs/architecture/ai.md](docs/architecture/ai.md)                                                     |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [buzrr/buzrr](https://github.com/buzrr/buzrr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
