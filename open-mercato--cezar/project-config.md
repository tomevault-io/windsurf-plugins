---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cezar is a team SaaS for running AI coding agents on GitHub issues — a **cockpit**
showing every agent run (queued / running / paused / failed / finished) with controls.
Incoming GitHub issues are auto-triaged; bug fixes run as a skill-driven, multi-step
**autofix workflow** (`verify-in-repo → root-cause → fix → review-loop → open PR`)
that ends in a draft PR. Each workflow step binds (in the web GUI) to a skill
(auto-discovered from `.ai/skills/` in the target repo), an agent backend (Anthropic
API · Claude Code CLI · Codex CLI), and a model. Agents run via a managed cloud path
(API key, the `/api/cron/dispatch` cron) or an optional self-hosted `@cezar/runner`
daemon.

**Status:** post-cutover. The agent-cockpit refactor is the live path — the
declarative workflow engine drives every autofix run, the `workflow_runs` /
`agent_runs` / `agent_run_events` tables back the cockpit, and the legacy
`flows`/`flow_events`/`ci_*`/`issue_autofix_candidates` tables + the 5 old
`/api/cron/{issue-match,issue-fix,ci-watch,ci-attribute,ci-fix}` routes + the
`/flows` UI are retired (migration `0011_retire_legacy_path.sql`). `issue-sync`
is kept as the GitHub → `issues`-table reconcile cron (auto-triage backlog +
missed-webhook safety net); `dispatch` and `triage-sweep` are the new path.
The design of record is `docs/REFACTOR-PLAN-agent-cockpit.md`; the activation
runbook is `MIGRATION.md`. The solo-use CLI (interactive hub + `init` / `sync`
/ `run` / `status` / `runs`) still works against a local file store.

## Commands

Yarn 4 monorepo (`packages/*`). Run from the repo root:

```bash
yarn build           # yarn workspaces foreach -A --topological-dev run build
yarn test            # all workspaces
yarn typecheck       # all workspaces
yarn lint            # all workspaces
yarn dev             # tsx watch the CLI

yarn workspace @cezar/core   run test       # core unit tests (vitest)
yarn workspace @cezar/core   run build      # build core
yarn workspace cezar         run build      # build the CLI
yarn workspace @cezar/runner run build      # build the runner daemon
yarn workspace @cezar/gui    run build      # Next.js build
```

Run a single core test file:
```bash
cd packages/core && npx vitest run tests/store/store.test.ts
```

> Known pre-existing failure: `packages/core` ends at 1 failing test —
> `tests/actions/stale/runner.test.ts > computes daysSinceUpdate correctly`
> (a date-arithmetic flake, unrelated to current work). Everything else green.

## Tech Stack

- **TypeScript 5.x** (strict, ES2022, NodeNext/ESM; `.js` on relative imports in core)
- **Node.js 20+** — native fetch, ESM; `node:util.parseArgs` (the runner CLI)
- **Commander.js** — CLI routing; **@inquirer/prompts** — interactive menus
- **@octokit/rest** — GitHub API; GitHub App auth via `@octokit/auth-app`
- **@anthropic-ai/sdk** — Claude API (streaming); **@anthropic-ai/claude-agent-sdk** — agent runs
- **Zod** — config + LLM-response validation
- **vitest** — test runner
- **cosmiconfig** — config discovery (`.issuemanagerrc.json`)
- **Next.js 15 + Supabase + Tailwind** — the `@cezar/gui` app (cockpit, settings, job queue, webhook receiver)
- **Local JSON store** at `.issue-store/store.json` (CLI) / Supabase (GUI) — no extra database for the CLI

## Architecture

### Data Flow (Three Phases)

1. **Fetch** — `init`/`sync` (CLI) or the `issue-sync` cron + the GitHub App webhook (GUI) pulls issues from the GitHub API into the store.
2. **Digest** — Claude generates compact per-issue summaries; comments are fetched and stored too.
3. **Analyze** — Actions run against digested issues; the workflow engine / autofix run on top.

### Key Design Patterns

**Action Plugin System** (`packages/core/src/actions/`): every analysis capability is a
self-contained action conforming to the `ActionDefinition` interface. Actions register via
side-effect imports in `packages/cli/src/index.ts`; the hub auto-discovers registered actions.
To add one: create `packages/core/src/actions/{name}/` with `prompt.ts`, `runner.ts`,
`interactive.ts`, `index.ts`; export the runner/prompt from `packages/core/src/index.ts`;
add the side-effect import to `packages/cli/src/index.ts`. The triage workflow reuses
`bug-detector`, `priority`, and `duplicates` (the real `dedupe-check` step against the
open-issue knowledge base); `auto-label`/`security`/etc. are used directly. (The four
display-only orphans — `issue-check`, `release-notes`, `milestone-planner`, `needs-response`
— were dropped in the legacy-path cleanup; if you need them, restore from git history.)

**Agent runner abstraction** (`packages/core/src/agents/`): `AgentRunner` interface with
three implementations — `AnthropicApiRunner`, `ClaudeCodeCliRunner`, `CodexCliRunner` —
and `createAgentRunner(backend, …)`. Normalized `AgentEvent` stream + `AgentRunResult`
(structured output + cost-weighted token usage). The Codex path (`codex exec --json`) is
implemented against the documented interface but not yet validated against a live binary
(`grep phase-4-verify`).


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [open-mercato/cezar](https://github.com/open-mercato/cezar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-31 -->
