---
trigger: always_on
description: Operating guide for Claude Code and any other coding agent working on this
---

# CLAUDE.md

Operating guide for Claude Code and any other coding agent working on this
repository. This file is intentionally short, stable, and phase-independent.
For the system's shape (files, pipelines, invariants) read `ARCHITECTURE.md`;
for the project's history and status read `ROADMAP.md`; for long-term direction
read `BLUEPRINT.md`. The public `README.md` is a landing page, not a status file.

## 1. Project Mission

Agent OS is a lightweight **local-first project cockpit** — a small Agent
Operating System for managing multiple long-term projects through a single web
chat surface. It combines project-scoped conversations, structured markdown
memory, an orchestration layer, and a bounded execution layer that hands work
to a Coding Agent inside a sandboxed workspace, then delivers the result
through audited Git/GitHub and deployment contracts. The goal is not a
general-purpose assistant or a heavyweight agent platform; it is a clear,
controllable place for a single builder to plan, decide, and execute project
work.

## 2. Core Architecture Principles

- **Local-first.** Filesystem + SQLite + FastAPI + React. No cloud services,
  no queues, no external infra dependencies for the core.
- **Project isolation.** Each project has its own conversations, memory files,
  and execution workspace. Crossings are deliberate and bounded.
- **Structured markdown memory.** Important state lives in readable, editable
  `.md` files, not buried in chat history.
- **Main agent = brain.** Planner / memory steward / orchestrator. Does not
  edit code under `repo/`.
- **Coding Agent = hands.** Bounded executor inside the project's
  `execution_workspaces/{project_id}/repo/`. Does not edit project memory or
  other projects' workspaces.
- **All execution goes through `ProjectSandbox` + `ToolRuntime`.** No raw
  `os` / `pathlib` access to repo paths, no raw `subprocess`. Every tool call
  routes through `resolve_repo_path()` / `resolve_under()` or
  `validate_command()`.

## 3. Agent Roles

### Main agent (orchestrator)
**Does:** hold project conversations; load global + project memory and
assemble context; judge (via separate LLM calls) memory updates and delegation
intent; delegate execution to the Coding Agent; consume run summaries; read
specific repo files on demand through the bounded inspection channel; run
grant-gated web research.

**Does NOT:** edit code under any `repo/`; run shell commands; auto-inject
repo contents or diffs into its context; auto-dispatch a run from inferred
intent.

### Coding Agent (executor)
**Does:** run bounded JSON tool loops inside one project's workspace; edit
files under `repo/` via `ToolRuntime` (`list_files`, `read_file`,
`write_file`, `append_file`, `search_files`, `run_shell`); update the
per-project `TASK.md`; produce a concise `result.md` + `run.json` per run.

**Does NOT:** touch other projects' workspaces; edit project memory
(`projects/{id}/*.md`) or global memory (`memory/*.md`); bypass the sandbox;
run destructive shell commands (`rm -rf`, `git push --force`,
`git reset --hard`, …) — Git and Supabase CLIs are separate audited executors,
not agent tools.

## 4. Memory Policy

- **Global memory** lives in `memory/`: `USER.md`, `WORKSTYLE.md`, `SOUL.md`,
  `MEMORY.md`.
- **Project memory** lives in `projects/{project_id}/`: `PROJECT.md`,
  `STATUS.md` (carries the `## Task Queue` board — Completed / In Progress /
  Next), `DECISIONS.md`, `RESEARCH.md`, `LESSONS.md` (durable Main-Agent
  lessons; never written by the Coding Agent). `OPS.md` is the deployment
  ledger, written only by the deterministic `ops_ledger` — never by an LLM.
- `SOUL.md` is **read-only to every agent/LLM path**: loaded as the identity
  anchor every turn, never auto-written, never in any judge/reconciliation
  writeback allow-list. It is shown at the top of the Global Memory modal and
  editable **only by the user**, through the one explicit manual
  `/global-memory/update-file` endpoint — the sole SOUL.md write path.
- All other global/project memory files participate in **policy-filtered
  semantic writeback**: after each non-delegated turn a judge call proposes
  structured JSON updates (`{filename, section, content, action}`); the
  backend validates each against the writable-file set before touching disk.
  All writes go through the single atomic write path in `memory_engine.py`.
- Memory writes are **clean structured markdown**, not conversation dumps.
  Keep entries concise; this layer stays human-readable.

## 5. Execution Policy

- **Two trigger paths start a run, both explicit.** (a) `@code <task>`
  dispatches immediately. (b) A model-judged **pending execution plan**
  dispatches only when the user clicks "OK, run this" (the confirm endpoint).
  Nothing else dispatches runs; inferred intent never auto-runs code.
- **Implicit delegation is model-judged** (`judge_delegation`), with the
  rule-based detector as a fallback that never blocks chat and never triggers
  execution. Deterministic mode `@`-commands (`@plan`, `@design`, `@debug`,
  `@review`, `@inspect`, `@memory`) only shape the chat response.
- **Runs are background and observable.** Dispatch returns a `running` record
  immediately; finalization happens off-thread; crashes promote to `failed`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [earthwalker17/agent-os](https://github.com/earthwalker17/agent-os) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-19 -->
