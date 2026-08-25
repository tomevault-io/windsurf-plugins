---
trigger: always_on
description: Guidance for AI agents (and humans) working **on** the projektor codebase.
---

# AGENTS.md

Guidance for AI agents (and humans) working **on** the projektor codebase.
Read this before making changes — it captures conventions that aren't obvious from the code alone.

> Portable source of truth across agent tools (Claude Code, Codex, Cursor, …). `CLAUDE.md` points here.

## What projektor is

A project management tool deployed on Cloudflare, combining AI-native design with tried-and-tested principles.

Design principles

1. Fast and lightweight.
2. Serverless, built on Cloudflare resources.

Implementation details:

- When implementing a feature or fixing a bug, always add a test that confirms the behaviour.
- **Runtime:** Hono on Cloudflare Workers
- **Data:** D1 (SQLite) for relational data, KV for caching (Access certs, user-by-email), R2 for file attachments
- **Schema:** Drizzle is the schema and primary query layer; raw `DB.prepare` remains in the auth/workspace middleware hot path, the dev bootstrap, and a handful of service queries (FTS, counters) where hand-written SQL is clearer.
- **Monorepo:** pnpm workspaces + turbo. `apps/api` (the Worker), `apps/web` (Astro + Preact static site, served in production via CF Workers Static Assets — see below), `apps/docs` (the Astro docs site linked throughout this file), `packages/*` (db, types, plugin-sdk), `plugins/*`
- **Deploy:** projektor publishes a self-contained **release artifact** on each `v*` tag; a config-only deploy repo (e.g. `projektor-deploy-example`) downloads it and ships it with `wrangler` — no submodule, no source checkout downstream. The Worker (`apps/api`) and the built frontend (`apps/web/dist`) ship together: `wrangler.toml` declares an `[assets]` binding with `run_worker_first = ["/api/*", "/mcp/*", "/wiki", "/.well-known/*"]`, so those paths always hit the Hono Worker while every other path serves the static Astro output (per-route HTML, asset-first). The release build compiles `apps/web` and bundles the Worker into a single `worker.js`.

## Coordination model (read this first)

Projektor expects multiple agents to work the same workspace concurrently. Before
editing anything:

- **Claim before editing.** File claims (`claim_files`) are path-level — they stop two
  agents editing the same files. Matching is **exact string equality**, not globs:
  claiming `src/` reserves nothing under `src/`, so name concrete paths and name them the
  same way the rest of the fleet does. Issue leases (`claim_issue`) are work-item-level —
  they stop two agents picking up the same ticket. The two are independent; you need both.
- **Your session goes stale if you stop heartbeating.** Liveness is heartbeat-based:
  `ACTIVE_TTL` in `apps/api/src/services/agents.ts` (mirrored as `SESSION_TTL_SECONDS` in
  `apps/api/src/services/issue-leases.ts`) is **120 seconds**. Register, then go quiet for
  two minutes without a `heartbeat_agent` call, and your session goes stale — you must
  heartbeat again before you can claim.
- **Both tiers self-heal.** An issue lease or file claim held by a stale session is
  reclaimed by the next claimer in the same call (`release_reason: "expired"`). Still call
  `release_files` and `end_agent` when you finish: reclaim only happens when someone else
  wants the path, so until then your claims sit there looking held, and a clean exit is
  what distinguishes you from a crash in the health data. A claim with no `agent_id` has no
  heartbeat to judge and is never auto-reclaimed — `force` is the only way past it.
- **There's a per-project cap on concurrently leased issues** —
  `DEFAULT_AGENT_WIP_LIMIT = 3` in `apps/api/src/services/issue-leases.ts`,
  overridable per project via `projects.agent_wip_limit`. It's admission control on
  the backlog, not a rate limit: it bounds how much work can be in flight at once,
  not how fast you can ask.
- **A refused claim tells you who to talk to.** Rejection is all-or-nothing: nothing is
  claimed, and the error names the issue and agent holding the path — message them with
  `post_message` if you need it. Nothing is pushed to the holder either way, including
  when you use `force` (that posts an audit message to *your* issue scope, not theirs), so
  if you override someone, tell them yourself. Every contended path is recorded regardless.

This is the mechanism; the mechanical call sequence for this repo is under "Fleet
coordination protocol" below, and the design rationale (why leases, claims, and the
WIP cap are shaped this way) is the [coordination model](https://tajd.github.io/projektor/philosophy/coordination-model/)
doc. The workflow rules themselves (definition of ready, state machine, human review
gates) live in exactly one place, the [workflow spec](https://tajd.github.io/projektor/agents/workflow-spec/)
— call `get_workflow` before claiming work; they aren't restated here.

## Planning and design docs live in the wiki, not the repo

Design records, implementation plans, and specs belong in the projektor wiki (`create_wiki_page`/`update_wiki_page`), not in a repo `docs/` folder. Keeping them in the wiki makes them discoverable and searchable (`search_wiki`) instead of buried in git history. Root-level user-facing docs (`README.md`, `AGENTS.md`, `CONTRIBUTING.md`, `SECURITY.md`) are the only docs that belong in the repo itself.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TAJD/projektor](https://github.com/TAJD/projektor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
