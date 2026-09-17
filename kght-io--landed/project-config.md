---
trigger: always_on
description: <!-- BEGIN:nextjs-agent-rules -->
---

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# Keep the CoWork brief in sync

[`instructions/README.md`](instructions/README.md) — the tracked repo folder that `INSTRUCTIONS_ROOT`
resolves to ([backend/src/config.ts](backend/src/config.ts)) — is the **single source of truth** that briefs the
CoWork agent on how this system works. It is consumed by a live agent, not just humans — a stale
brief makes CoWork act on the wrong model. The playbooks are generic repo source (candidate-specific
detail lives in the profile config, not the prose). Edit them **in the repo** — the in-app Guides /
Instructions views are read-only (playbooks carry the agents' MCP wiring, so the UI never writes them).

When you change code that the brief describes, update `instructions/README.md` in the same change:
- **MCP tools** added/removed/renamed in [mcp/jobhunt-server.mjs](mcp/jobhunt-server.mjs) → update the tool lists.
- **Job types / playbooks** added/removed → update the "Job types" index and add/remove the `<type>.md` playbook.
- **Asset layout** changes (what lives on disk vs. in the DB, folder names, the slug convention) → update the layout section.
- **The run flow** (how the queue is processed, the discovery funnel, the Apply boundary) changes → update the matching section.

Do not create a second doc that re-describes the system — fold it into `instructions/README.md` instead.

# Architecture at a glance

Local-first job-search command center. **One SQLite DB is the source of truth**
(`data/jobhunt.db`), edited by two actors — **You** (human, via the UI) and **CoWork** (the
agent, via MCP) — and every change is attributed to one of them.

The repo is an **npm-workspaces monorepo** of five flat workspaces: the frontend and the backend are
separate packages that happen to run in one process (`next dev`). Which workspace a file belongs in
is not cosmetic — it is enforced (see "The workspace boundary" below).

```
frontend/   Next.js — pages, components, hooks, and the /api route handlers
backend/    server-only — DB, job queue, agent orchestration, on-disk paths
shared/     client-safe — domain types, pure logic, formatting (browser-safe)
mcp/        the stdio MCP server (zero-dep; a thin HTTP client over frontend)
desktop/    the Electron app — runs the agent on the user's machine (see below)

data/  instructions/  scripts/  tests/    (repo-root, shared by every workspace)
```

- **UI** — Next.js 16 (App Router), React 19, Tailwind 4. Routes in [frontend/app/](frontend/app/),
  components in [frontend/components/](frontend/components/).
- **Data** — Drizzle ORM over `better-sqlite3`. Schema + queries in
  [backend/src/db/](backend/src/db/).
- **Job queue** — [backend/src/jobs/](backend/src/jobs/) is the work spine: jobs are
  created, atomically *claimed* (lease-based), and their results *ingested* back into the DB.
  `store.ts` is the core; `registry.ts` maps agent result records onto postings/companies.
- **Agent surface** — [mcp/jobhunt-server.mjs](mcp/jobhunt-server.mjs) exposes MCP tools; CoWork is
  briefed by `INSTRUCTIONS_ROOT/README.md` (defaults to `./instructions`; see the sync rule above).
- **Desktop app** — [desktop/](desktop/) is where the agent actually runs. A supervisor long-polls
  `/api/jobs/wait`, spawns `claude` on the user's own subscription scoped to the folder they picked,
  and streams the transcript into a window. It exists for the one thing nothing else can do: drain
  the queue while nobody is watching. It ships a SECOND MCP server (`landed-local`) for the
  machine's own files — `mcp/jobhunt-server.mjs` stays a pure HTTP client and knows nothing about
  disks. `npm run desktop` runs it; `npm run desktop:package` builds a DMG.
- **On-disk paths** — anchor them on `REPO_ROOT`
  ([backend/src/paths.ts](backend/src/paths.ts)), never `process.cwd()`. npm runs a
  workspace script with cwd set to the *package*, so `next dev` reports cwd = `frontend`; a
  cwd-relative `data/jobhunt.db` silently opens a second, empty database.
- **Untyped boundaries** — agent/JSON results arrive as `unknown`. Coerce them through
  [shared/src/util/coerce.ts](shared/src/util/coerce.ts) (`num` never returns NaN; `str`
  maps empty→undefined). Don't hand-roll `Number(x)` on agent input — it re-introduces the
  NaN-defeats-`?? fallback` bug.

# The workspace boundary

Five invariants, enforced by `npm run boundary` (part of `npm run check`), configured in
[.dependency-cruiser-boundary.cjs](.dependency-cruiser-boundary.cjs):

| Rule | Meaning |
| --- | --- |
| `backend -/-> frontend` | the backend never reaches into the frontend |
| `shared -/-> backend` | at **runtime**; type-only imports are exempt (TypeScript erases them) |
| `shared -/-> node builtins` | `shared` ships to the browser |
| `frontend -/-> desktop` | the web app must build without Electron |
| `desktop -/-> backend` | at **runtime**; the desktop app talks HTTP, never opens the DB |

**`desktop -> frontend` is deliberately allowed, and it is the only direction across that pair.**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kght-io/Landed](https://github.com/kght-io/Landed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
