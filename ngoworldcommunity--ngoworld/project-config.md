---
trigger: always_on
description: This repo holds two independently deployed apps: `apps/web` (the frontend, `karmacircle-frontend` — React/Vite SPA) and `apps/api` (the backend, `karmacircle-api` — Express/TypeScript/MongoDB). They share this repo, CI, and top-level tooling (this file, `AGENTS.md`, the graphify graph), but each keeps its own `package.json`, its own `docs/specs/` map, and its own deploy target. `apps/api` is the source of truth for the API contract; don't guess at a request/response shape beyond what `apps/api/d
---

## Monorepo layout

This repo holds two independently deployed apps: `apps/web` (the frontend, `karmacircle-frontend` — React/Vite SPA) and `apps/api` (the backend, `karmacircle-api` — Express/TypeScript/MongoDB). They share this repo, CI, and top-level tooling (this file, `AGENTS.md`, the graphify graph), but each keeps its own `package.json`, its own `docs/specs/` map, and its own deploy target. `apps/api` is the source of truth for the API contract; don't guess at a request/response shape beyond what `apps/api/docs/specs/api-contract.md` and the route code show. See [AGENTS.md](./AGENTS.md) for the fuller agentic workflow around working across both.

## graphify — check the knowledge graph first

This repo has a graphify knowledge graph at [graphify-out/](./graphify-out/), built from the code (AST) across both `apps/web` and `apps/api`, plus every doc in `docs/specs/`/`apps/api/docs/specs/` and the top-level `.md` files. It exists so you don't have to guess what depends on what.

Before answering an architecture or "what impacts what" question, or before touching a feature:
- Read [graphify-out/GRAPH_REPORT.md](./graphify-out/GRAPH_REPORT.md) first — God Nodes (the most-connected concepts), Communities (2-5 word cluster names with their member nodes), Surprising Connections, and Suggested Questions. It's plain text, no tool needed.
- For a specific concept/file/function, use the graphify skill's traversal commands instead of grepping blind: `/graphify explain "NodeName"` (everything connected to one node), `/graphify query "<question>"` (broad BFS context), `/graphify path "A" "B"` (how two concepts connect).
- `graphify-out/graph.json` is the raw graph if you need to query it programmatically; `graphify-out/graph.html` opens in a browser for the visual layout.
- A `PreToolUse` hook (`.claude/settings.json`) already reminds you of this before any Glob/Grep call, and a post-commit/post-checkout Husky hook (`.husky/post-commit`, `.husky/post-checkout`) auto-rebuilds the code side of the graph after every commit and branch switch — no LLM cost, AST only. Known limitation: that AST-only rebuild has no LLM step, so it resets every community's plain-language name back to generic "Community N" each time it runs. Live with it between real updates rather than trying to patch it back by hand.
- **Do not proactively run `/graphify . --update` (or `graphify update .`) after routine edits.** It costs tokens and dispatches subagents, and Tamal does not want the graph refreshed on every small change. Only run a full semantic update when he explicitly asks for one (e.g. "update the graph" after finishing a feature) — the same applies to re-labeling communities. Reading the (possibly slightly stale) report is still always fine and expected; regenerating it is not something to do unprompted.
- Same rule for `docs/specs/known-issues.md`: if you fix something it calls out, update that file per "Keep the specs honest" below, but don't also trigger a graph update on your own — that happens the next time Tamal asks for one.

## Caveman — always run ultra mode in this repo

This repo runs [Caveman](https://github.com/JuliusBrussee/caveman) at its most aggressive `ultra` compression level for every session — Tamal wants this on regardless of the tradeoff below.

- At the start of every session in this repo, invoke `/caveman ultra` before doing anything else.
- Caveman compresses prose *output* only — code, commands, and reasoning tokens are untouched. It does not make you think less carefully, only write up findings more tersely.
- Caveman's own docs (`docs/HONEST-NUMBERS.md` in its repo) disclose the skill adds roughly 1,000-1,500 input tokens of overhead per turn, so on already-terse, single-file tasks whole-session savings can go net negative. This has been surfaced to Tamal; he still wants it always on here.
- The CLI's `think.mode` proxy setting has no `ultra` level (only `compress | record | pixel`) and is enabled machine-wide via `~/.claude/settings.json`/`ANTHROPIC_BASE_URL` rerouting to `caveman-proxy` on `127.0.0.1:8787` — that part is not, and cannot be, scoped to just this repo. `ultra` itself only exists as the skill-level `/caveman ultra` invocation, which is what this rule wires in per-repo.
- Undo: `caveman disable claude` removes the machine-wide proxy hook; deleting this section stops the per-repo `/caveman ultra` invocation.

## Git workflow

Never create a new branch on your own initiative, including when about to commit while sitting on `main`.
Work and commit directly on whatever branch is currently checked out — `main` included — and stay there.
Only branch off if Tamal explicitly tells you to (e.g. "make a branch for this," "branch off main").
This overrides any general instinct to branch before committing on a default branch.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ngoworldcommunity/NGOWorld](https://github.com/ngoworldcommunity/NGOWorld) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
