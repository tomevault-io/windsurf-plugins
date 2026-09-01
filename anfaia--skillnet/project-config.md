---
trigger: always_on
description: > **Asked to start the project? → [`RUNNING.md`](RUNNING.md).** Five steps, one decision
---

# AGENTS.md

> **Asked to start the project? → [`RUNNING.md`](RUNNING.md).** Five steps, one decision
> (API key, local model, or neither), and it says what to verify afterwards. Do not
> reconstruct the commands from this file — the seed step is easy to miss and without it the
> dashboard is empty.

## Project

SkillNet — open-source adaptive learning system. It creates courses from an idea or source material
and can adapt their explanations, activities and interfaces to each person. Self-hosted, with
organization and individual workspace modes.

## Stack

| Layer | Technology |
|-------|------------|
| Frontend | React 19 + Vite + Tailwind v4 + React Router + TanStack Query |
| Backend | Python + FastAPI + fastapi-users |
| Database | PostgreSQL + pgvector |
| Auth | Session cookies (httpOnly, 7-day expiry) via fastapi-users CookieTransport |
| Real-time | SSE (Server-Sent Events) for streaming LLM responses |
| AI orchestration | LangGraph |
| LLM | Any OpenAI-compatible API (user configures endpoint + key + model) |
| Deployment | Docker Compose |

## Repo structure

```
skillnet/
├── AGENTS.md                         # This file (root instructions)
├── RUNNING.md                        # How to start the project — read this first
├── apps/
│   ├── skillnet-api/                 # FastAPI + LangGraph + pgvector (backend)
│   ├── skillnet-web/                 # React SPA (frontend)
│   │   └── AGENTS.md                 # Frontend-specific instructions
│   ├── skillnet-a2a/                 # A2A agent server (optional, `--profile a2a`)
│   └── skillnet-site/                # Public landing site
├── packages/
│   ├── a2tl-video/                   # A2TL-Video — compact spec for agent-generated video (TypeScript)
│   ├── a2tl-web/                     # A2TL-Web — compact spec for agent-generated web pages (TypeScript)
│   ├── mcp-md-reader/                # Markdown reader MCP server (TypeScript)
│   └── skillnet-mcp/                 # SkillNet MCP server — thin client of /ext/v1 (TypeScript)
├── docs/
│   ├── design/
│   │   ├── v1-scope.md               # What v1 is and isn't
│   │   ├── v2-dynamic-courses.md     # The v2 design (Spanish) — implemented, chosen per course
│   │   ├── ai-course-design.md       # Stateless AI endpoints and multi-model routing for course design (Spanish)
│   │   ├── openui-adoption.md        # Why OpenUI, and what its reactive layer would cost (Spanish)
│   │   ├── tuning.md                 # The dials for generation quality, and what each does
│   │   ├── course-packages.md        # A course as an installable directory (no LLM, no key)
│   │   ├── architecture.md           # Architecture decisions (decided + deferred)
│   │   ├── data-model.md             # PostgreSQL schema (v1 body + v2 appendix)
│   │   ├── screens.md                # Screen specs
│   │   └── design-system.md          # Visual design tokens and component patterns
│   └── research/                     # Investigation by topic
└── assets/
```

## Current phase: v1 and v2 always available

Both v1 (static courses) and v2 (dynamic courses) are always available. The choice is
**per-course** via `delivery_mode`: a course is dynamic (v2) when it has
`delivery_mode='dynamic'` **and** `schema_status='validated'`. Every other course stays on v1.

Consequences for anything you change:

- `src/services/course_delivery.resolve_delivery` is the single decision point for v1 vs v2;
  do not add a second one.
- `tests/integration/test_v1_regression.py` exists to catch a break in v1 behaviour.
- `docs/design/v1-scope.md` still defines the v1 product and still wins on v1 questions. It no
  longer wins on "is v2 implemented" -- it is not a forward-looking document any more.
- `docs/design/v2-dynamic-courses.md` is the design of record for everything v2.
- Tuning generation quality: `docs/design/tuning.md` plus
  `apps/skillnet-api/scripts/quality_bench.py`.

## Architecture (key decisions)

Full details in `docs/design/architecture.md`. Summary:

- **Database:** PostgreSQL + pgvector. Single DB for relational and vector data. Schema in `docs/design/data-model.md`
- **API:** Pragmatic REST. CRUD for resources + action endpoints for operations (`POST /courses/:id/generate`, `POST /courses/:id/publish`, `POST /exercises/:id/attempt`)
- **Auth:** Session cookies via fastapi-users. No JWT tokens in frontend. Browser sends cookie automatically
- **Frontend:** Single SPA with React Router. Fixed routes, dynamic content. TanStack Query for server state, `useState` for UI state
- **Real-time:** SSE for streaming agent responses. `StreamingResponse` in FastAPI
- **Self-hosted:** One instance per company. `organizations` table scopes data but has one row per deployment
- **LLM:** Provider-agnostic via litellm. User sets `LLM_MODEL` (e.g. `anthropic/claude-sonnet-4-20250514`, `deepseek/deepseek-chat`, `ollama/llama3`) in env vars. Any provider litellm supports works

## Commands

```bash
# Frontend (from apps/skillnet-web/)
pnpm install
pnpm dev              # dev server on localhost:5173
pnpm build            # production build
pnpm lint             # oxlint

# Backend (from apps/skillnet-api/)
uv sync
uv run uvicorn src.main:app --reload

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ANFAIA/SkillNet](https://github.com/ANFAIA/SkillNet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
