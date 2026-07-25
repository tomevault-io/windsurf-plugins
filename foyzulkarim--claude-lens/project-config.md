---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo state: V2 active (Phase 4)

Phases 0–3 are complete: the V2 TypeScript app lives in `shared/`, `server/`, and `client/`, and Phase 4 page/feature work is next. The old Express dashboard is preserved under `legacy/`; don't extend it beyond keep-it-running fixes. Keep V2 work inside the active plan task and its settled specs.

## Commands (V2 — active app)

- `npm ci` — locked install; run separately in each worktree.
- `npm run dev` — backend on 4128 and Vite on 4129 by default; `CLAUDE_LENS_PORT_BASE=N` derives backend `N`, Vite `N+1`, E2E `N+2`, and Storybook `N+3` (so `npm run storybook` defaults to 4131, not Storybook's stock 6006).
- `npm run verify` — complete typecheck, lint, format, and unit/integration test gate.
- `npm run build` — production CLI and SPA bundle under `dist/`.
- `npm run test:e2e` — isolated fixture copy + built server + Cypress.
- `npm start` — run the built app. V1 instructions are in `legacy/README.md`.

## Before pushing (V2)

`npm run verify` runs the exact CI gate from `.github/workflows/ci.yml` — `typecheck` → `lint` → `format:check` → `test`, in that order. A Husky `pre-push` hook (`.husky/pre-push`, wired via the `prepare` script) runs it automatically on every `git push`, so this is enforced mechanically rather than by remembering — don't bypass it with `--no-verify` without a specific reason. `lint` and `format:check` are separate Biome checks (one is code-quality rules, the other is whitespace/wrapping); passing one says nothing about the other.

## Architecture

**V1** (`legacy/`, maintenance only): one ~640-line `server.js` — Express serving the static single-file `index.html`, with `/api/*` endpoints that re-scan and parse `~/.claude/projects/**/*.jsonl` transcripts on each request. No build step, no framework, pricing from env.

**V2** (active — the specs remain authoritative; this is just the map):

- **One npm package, one port**: Fastify serves the built SPA, `/api/*`, and a `/ws` upgrade. Three strict-TS roots: `shared/` (contracts), `server/`, `client/` (architecture §3; deps are pinned by §2 — deviating requires editing the doc first).
- **Ingest pipeline** (§5): discovery (fast-glob over roots) → poller (fast stat loop + slow re-glob) → tailer (byte-offset incremental reads, partial-line safe) → parser (JSONL line → `ApiCall`, `message.id` dedupe, malformed lines counted never thrown) → in-memory columnar store → derived turns/sessions → debounced per-session invalidation.
- **WS is an invalidation bus only** (§7): three message types, never data; the client refetches mounted queries by key prefix.
- **Metrics engine** (§8): a single `metrics(query) → Series[]` function (measure × dimension × grain, distributions, compare, smoothing). Every page is preset queries + layout over this engine — pages are deliberately cheap.
- **Tier system** (§4): transcript files alone give computed/estimated values (🟢 exact, 🟡 estimated); optional premium capture files (`<uuid>.cost.jsonl`, `<uuid>.turn-boundaries.jsonl`, `~/.claude/cost-log.jsonl`) upgrade to observed values per session; 🔴 = unavailable without them.
- **Client** (§11): React + wouter + TanStack Query; ECharts via a hand-rolled ~50-line wrapper (no `echarts-for-react`); global filter state lives in the URL query string (permalinks are a spec requirement).

Which doc for what: `docs/claude-lens-architecture.md` (how) · `specs/claude-lens-pages.md` (what — its per-page section tables are **binding over the HTML mockups**) · `specs/gates.md` (Report Card gates) · `specs/claude-lens-plan.md` (when — phases, tasks, decisions log) · `specs/claude-lens-phase4-parallelization.md` (Phase 4 start gates, live orchestration, worktrees, recovery, and opt-in maximum-throughput mode).

## The delivery pipeline

**Specs decide what, issues track what, start-time skills decide how, and the plan doc decides when.**

```
planned work:  specs/claude-lens-plan.md ──► /create-issue ─► /start-task <issue#> ─► /move-to-worktree ─► (/plan-architecture ─► /generate-tasks) ─► /implement ─► /review ─► /commit ─► PR merges, issue closes ─► /finish-worktree ─► /archive-issue
new ideas:     /plan-requirements ─► specs/requirements/REQ-<slug>.md ─► /create-issue ─► same as above
```

- **Every PR body must carry `Closes #N`** (or `Fixes`/`Resolves`) for its issue, so merging the PR actually closes it — nothing in `/commit` or any other skill does this automatically (`/commit`'s trailer is `Refs: {task-number}`, which does not auto-close). Skip it only when the PR has no associated issue, or when there's a specific reason to close manually (e.g. `NOT_PLANNED`/re-gated instead of shipped, as with #8) — but then close it explicitly before moving on, since `/archive-issue` refuses to touch an open issue and a forgotten close is exactly what leaves stale files behind (see next bullet).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [foyzulkarim/claude-lens](https://github.com/foyzulkarim/claude-lens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-25 -->
