---
trigger: always_on
description: > This file is Claude Code's briefing document for the Sprint Zero repo.
---

# Sprint Zero — CLAUDE.md

> This file is Claude Code's briefing document for the Sprint Zero repo.
> Read this before touching any file. It is the single source of truth for
> what this project is, how it's wired, and what the rules are.

---

## What is Sprint Zero?

Sprint Zero is a cloneable Claude Code kit that turns a reference URL plus three scoping answers into a complete spec set and a working product. The user points Sprint Zero at something similar to what they want to build, answers one multi-part scoping question, and a team of sub-agents produces:

- A full spec set in `docs/` (scope, reference brief, PRD, decisions, user stories, API contract)
- A working build in `server/` (Express + Supabase) and `client/` (React + Vite)
- Playwright-driven QA covering the auth flow and the core product loops

The audience is PMs, founders, and non-developers who can follow terminal output but don't write code. Every decision in this repo prioritises:

- Clarity over cleverness
- Demo-readiness over production hardening
- Visible outputs over elegant internals
- The user's judgement over inferred defaults

---

## Scope levels

The user picks one of three levels up front. The scope level drives every downstream agent's behaviour.

| Level       | What it produces                                                                                                  |
| ----------- | ----------------------------------------------------------------------------------------------------------------- |
| `clickable` | Mock backend, fake data, no auth. Useful for pitching and flow reviews.                                           |
| `MVP`       | Real Supabase, real auth, one core loop works end-to-end. The main v1 target.                                     |
| `Prod`      | MVP plus error boundaries, loading states, input validation, and Playwright happy path + one error path per loop. |

`docs/scope.md` is the lever that carries this choice through the pipeline.

---

## The fixed stack

Sprint Zero v1 produces one stack. Not configurable, not inferred.

- **Frontend:** React + Vite
- **Backend:** Express (Node)
- **Database + Auth:** Supabase (Postgres + Supabase Auth)
- **Testing:** Playwright via Playwright MCP

The user brings their own Supabase project. Credentials go in `.env` (see `.env.example`).

---

## Architecture, one line

URL → scoping conversation → spec set (PRD, decisions, stories, contract) → parallel build (backend + frontend) → QA with browser-driven tests → working product.

---

## Repo structure

```
sprint-zero/
├── CLAUDE.md                 ← you are here
├── README.md                 ← public-facing description
├── plan.md                   ← phased build plan for this repo itself
├── LICENSE                   ← MIT, Yousuf Alvi
├── .env.example              ← SUPABASE_URL + SUPABASE_PUBLISHABLE_KEY + SUPABASE_SECRET_KEY
template
├── .gitignore
├── .claude/
│   ├── commands/             ← slash commands (Phase 2)
│   └── agents/               ← sub-agents (Phase 3)
├── docs/                     ← generated spec files live here per run
└── examples/
    └── mini-twenty/          ← worked example (Phase 5, committed build)
```

Files generated at runtime (per user run):

- `docs/scope.md` — scoping answers, structured
- `docs/reference-brief.md` — extracted brief of the reference URL
- `docs/prd.md` — product requirements
- `docs/decisions.md` — scope cuts and tradeoffs, tied to the chosen level
- `docs/user-stories.md` — user stories
- `docs/api-contract.md` — the contract all agents build to
- `server/` — Express + Supabase build
- `client/` — React + Vite build
- `.claude/settings.local.json` — demo-time permissive settings, gitignored

---

## Agent topology

Sprint Zero's build layer has four sub-agents. One orchestrator, two engineers, one QA. The user talks to the orchestrator. The orchestrator handles everything else.

```
User
  │
  └─ tech-lead ──┬─ backend-engineer   (Express + Supabase)
                 ├─ frontend-engineer  (React + Vite + Supabase Auth)
                 │
                 └─ qa-engineer        (Playwright MCP)
```

### tech-lead

- Reads `docs/prd.md`, `docs/api-contract.md`, `docs/decisions.md`, and `docs/scope.md`
- Briefs the user on what it understood
- Spawns `backend-engineer` and `frontend-engineer` in parallel in a single turn
- Waits for both, then spawns `qa-engineer`
- Returns a delivery summary
- Does not write application code

### backend-engineer

- Builds the Express API in `server/`
- Uses `@supabase/supabase-js` for data access
- JWT verification middleware on protected routes
- Seed script uses the Supabase admin client
- Builds strictly to `docs/api-contract.md`

### frontend-engineer

- Builds the React app in `client/` using Vite
- Supabase Auth: login page, signup page, session context, protected route wrapper
- Product screens come from `docs/user-stories.md` and the contract
- Form patterns standardised so QA can drive them

### qa-engineer

- Runs the full auth dance (signup, session, logout, login, protected route, 401 on expired token)
- Runs the core product loop end-to-end
- At `Prod` scope, adds one error-path test per loop
- Reports pass/fail back to `tech-lead`

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yousuf-labs/sprint-zero](https://github.com/yousuf-labs/sprint-zero) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
