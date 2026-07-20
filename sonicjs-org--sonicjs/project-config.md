---
trigger: always_on
description: Use when scope is bounded AND output would bloat main context:
---

# SonicJS Agent Guidelines

These instructions mirror the workflow Claude Code already follows in Conductor so every coding agent behaves consistently.

## Token-Efficient Tooling (use FIRST, before grep/Read sprees)

This repo is indexed by **codegraph** (684 files, 5693 nodes). Use it as the default code-lookup tool. Hook also rewrites shell commands via **rtk** for 60-90% bash savings. Default response mode is **caveman** for terse output.

### Decision tree — code exploration

| Intent | Tool | Why |
|--------|------|-----|
| "How does X work / where does X live / trace flow X→Y" | `mcp__codegraph__codegraph_explore` | ONE call returns verbatim source grouped by file. Replaces many Read/Grep. |
| "Just the location of symbol named X" | `mcp__codegraph__codegraph_search` | Cheapest — name + path only. |
| "What calls this / what does this call / blast radius" | `codegraph_callers` / `codegraph_callees` / `codegraph_impact` | Edges already indexed; grep can't follow dynamic dispatch. |
| Pinpoint known file edit | `Read` + `Edit` | Codegraph not needed for known path. |
| Truly open-ended cross-cutting search | `Agent` w/ `Explore` or `cavecrew-investigator` | Compressed output, protects main context. |

**Rule:** Before any 3+ Grep/Read combo for "where is X", call `codegraph_explore` first. Treat codegraph as a pre-built index — don't re-run searches it already answers.

### rtk (Rust Token Killer)

- Bash commands auto-rewritten by hook — transparent, 0 overhead. No action needed.
- `rtk gain` to audit savings; `rtk discover` to find missed opportunities.
- Don't bypass rtk with raw shell unless debugging.

### caveman mode

- Active by default (full). Drops articles/filler. Code blocks, commits, security warnings stay normal.
- Toggle: `/caveman lite|full|ultra`, off via "stop caveman".

### Delegation — cavecrew subagents

Use when scope is bounded AND output would bloat main context:
- `cavecrew-investigator` — read-only locator ("where is X / what calls Y / map this dir"). ~60% smaller tool result vs vanilla Explore.
- `cavecrew-builder` — surgical 1-2 file edit (typo, single-fn rewrite, mechanical rename). Refuses 3+ files.
- `cavecrew-reviewer` — diff/PR review, one line per finding.

Skip cavecrew for: known-path edits, multi-file features, anything codegraph answers in one call.

### Anti-patterns (token waste)

- ❌ Running `grep -r` across the repo when `codegraph_search` would answer.
- ❌ Reading 5+ files to understand a flow — `codegraph_explore` returns the relevant slice.
- ❌ Spawning a subagent just to read one known file.
- ❌ Verbose narration. Caveman mode handles that — keep updates to one sentence.

## Project Structure & Stack
- Monorepo managed with npm workspaces. Core Hono + Workers code, routes, middleware, templates, plugins, utils, and DB lives in `packages/core/src`.
- Shared templates/components: `packages/templates/`. CLI scaffolder: `packages/create-app/`. Helper scripts: `packages/scripts/`.
- Marketing/docs site is `www/` (Next.js + MDX). Long-form docs, AI plans, and architecture notes live in `docs/`.
- E2E specs use Playwright in `tests/e2e/` (configs in `tests/playwright*.config.ts`). Postman and smoke docs sit under `tests/`.
- `my-sonicjs-app/` is the sand-boxed sample install; recreate freely and use it to exercise migrations (`npm run setup:db` produces a fresh branch-specific D1 DB).

## Preferred Workflow (Claude Code parity)
1. **Understand & Plan** – Read the issue and affected modules, skim relevant docs, and outline a plan/todo list before heavy editing. Keep changes minimal and targeted.
2. **Prep Environment** – From `my-sonicjs-app/`, run `npm run setup:db` for a clean Cloudflare D1 database tied to the worktree. Run `npm install` at repo root if dependencies changed.
3. **Implement** – Match existing TypeScript/Hono patterns (server templates, plugins, Drizzle schema, HTMX UI). Keep types explicit and favor async/await.
4. **Add Tests** – Unit tests go beside the feature in `packages/core/src/__tests__`. Every change also needs an accompanying Playwright spec under `tests/e2e/##-description.spec.ts`.
5. **Verify** – Run `npm run type-check`, `npm test`, and `npm run e2e` locally. Use `npm run e2e:smoke` or `npm run e2e:ui` only for debugging, but ensure the full suite passes before sign-off.
6. **Document** – Update README/docs or AI plans when APIs, migrations, or workflows change. Mention any fixtures or DB prep needed in the PR/test plan.

## Build & Test Commands
- Install deps: `npm install`
- Build core only: `npm run build:core`; full build + sample app: `npm run build`
- Dev servers: `npm run dev` (proxies to `my-sonicjs-app`), `npm run dev:www` (marketing)
- Type safety: `npm run type-check`
- Unit tests: `npm test` or `npm run test:watch`
- Playwright E2E: `npm run e2e`, smoke subset via `npm run e2e:smoke`, headed/UI via `npm run e2e:ui`
- Sample-app DB reset: `npm run db:reset` (or `npm run setup:db` inside `my-sonicjs-app`)

### Local secrets (`my-sonicjs-app/.dev.vars`)
- `wrangler dev` reads `my-sonicjs-app/.dev.vars` for local secrets. Auth refuses to initialize without `BETTER_AUTH_SECRET` (>=16 chars); requests to `/auth/*` then return 500 with "BETTER_AUTH_SECRET is missing or too short".

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SonicJs-Org/sonicjs](https://github.com/SonicJs-Org/sonicjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
