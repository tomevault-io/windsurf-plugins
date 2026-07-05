---
trigger: always_on
description: Repo-level guidance for AI coding agents (Cursor, Claude Code, Codex, etc.) working in this repository. Cursor users see the same directive via `.cursor/rules/puppetmaster-workflow.mdc`.
---

# AGENTS.md

Repo-level guidance for AI coding agents (Cursor, Claude Code, Codex, etc.) working in this repository. Cursor users see the same directive via `.cursor/rules/puppetmaster-workflow.mdc`.

## Default subagent runtime: Puppetmaster

When this repo is open, **Puppetmaster's MCP tools are the primary path for fanning out work**. Do not roll your own grep/read exploration loops or invoke ad-hoc subagents when a Puppetmaster swarm is available.

## When to start a Puppetmaster swarm (do this by default)

For any request that involves more than a single trivial change, start a Puppetmaster swarm **before** spending tokens on your own exploration:

- Broad repo investigation, audit, or risk analysis ("review this", "find issues", "what could break").
- Multi-file refactors, migrations, or cross-cutting cleanups.
- Debugging that requires tracing call graphs, data flow, or test coverage across multiple files.
- Planning the next implementation slice when scope or risks are unclear.
- Comparing approaches, surfacing trade-offs, or producing decision artifacts.

Default routing:

1. `puppetmaster_start_cursor_swarm` — multi-role read-only analysis (the daily-driver entry point).
2. `puppetmaster_start_implement` — durable, patch-producing implementation that runs on whatever platform the lock enables (cursor preferred, then claude-code). Use this as the default implement verb so it works regardless of platform lock. `puppetmaster_start_cursor_implement` / `puppetmaster_start_claude_implement` force a specific platform.
3. `puppetmaster_start_cursor_review` / `puppetmaster_start_cursor_plan` — lightweight single-pass review or plan.

Every implement verb runs full-edit in a clean worktree (clean-tree guard; set `allow_dirty` to override) and captures the resulting diff as a `PATCH` artifact.

Start tools return a `job_id` immediately. Do **not** wait inside one long MCP call.

## Match the verb to the task shape (single feature ≠ swarm)

A swarm is for **read-only, decomposable analysis** — explore, review, audit, plan, redteam — where independent roles can run in parallel without touching the same code. It is **not** the right shape for one coupled feature. Fanning out a single tightly-coupled change makes parallel workers re-ingest the same context and land commits that are unaware of each other, which produces conflicts, rework, and broken delivery — and erases any token savings.

So:

- **Implementing one feature / fixing one ticket → a single `puppetmaster_start_implement` worker** in a clean worktree. One worker keeps the change coherent and yields one reviewable `PATCH`. Reserve swarms for the explore/review/audit passes *around* the feature, not the edit itself.
- **A focused single edit, or any change that builds on uncommitted work → `puppetmaster_edit`** (in-place, `allow_dirty`, cheapest sufficient model + CodeGraph, reviewable `PATCH`). This is the verb for last-mile work — "finish the module I just wrote", "add tests for the code I just added". `puppetmaster_start_implement` branches off HEAD in an isolated worktree, so it **cannot see uncommitted modules** and would clobber or rebuild them; never reach for it when the work depends on dirty-tree state. Keep truly trivial edits (typo/rename/comment) inline.
- **Genuinely independent slices** (e.g. "add the same header to 30 unrelated endpoints") can fan out — but split by non-overlapping files so workers never collide.
- **Broad investigation / audit / "find all X" → a read-only swarm.** That is what it is built for.

Puppetmaster's edge is mixed workloads, heavy trivial sub-tasks, long-context horizons, and zero-token artifact recall across sessions — not winning a single hard implementation against one strong steered agent. Output-compression and context-hygiene tools (e.g. RTK) are **additive** to Puppetmaster, not competitors: let them shrink tool-output tokens while Puppetmaster owns orchestration and durable state.

## When NOT to route through Puppetmaster

Use native tooling directly for:

- Trivial single-file edits with obvious intent (rename, add comment, fix typo).
- Questions answerable from the current visible file or recent context.
- Conversational follow-ups that don't change repo state.
- Anything explicitly framed as "just answer me" / "no swarm".

## Browser swarms (live-site QA)

For QA that needs a **real browser against a live site** — capturing actual
network payloads instead of mocked ones — use the first-class browser verb, not
a read-only swarm. A read-only swarm can't reach it: the MCP swarm specs hardcode
a `file,web,vision` toolset list with no `browser`, and the cursor swarm adapter
has no browser at all. Only the **Hermes** adapter can drive a browser
(`hermes chat -t browser`).

- **Verb:** `puppetmaster_start_browser_swarm` (MCP) / `python -m puppetmaster browser "<task>" ["<task2>" ...]` (CLI). Each task becomes one parallel Hermes worker. Requires the Hermes platform enabled.
- **Single source of truth:** `puppetmaster/browser.py`. It bakes in three hard-won guardrails so callers don't re-derive them:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [professorpalmer/Puppetmaster](https://github.com/professorpalmer/Puppetmaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
