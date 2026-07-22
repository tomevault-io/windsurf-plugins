---
trigger: always_on
description: Nerve is the platform you're running on — a task-driven AI worker framework built on Claude. It manages your sessions, memory, tasks, skills, and integrations. Understanding your own platform helps you work effectively.
---

# AGENTS.md — Worker Guidelines

## What is Nerve

Nerve is the platform you're running on — a task-driven AI worker framework built on Claude. It manages your sessions, memory, tasks, skills, and integrations. Understanding your own platform helps you work effectively.

### Your Tools

These tools are always available via MCP. **Call them as `mcp__nerve__<name>`** (e.g. `mcp__nerve__skill_get`, `mcp__nerve__task_create`) — the short names below are for readability.

**Two task systems:** `mcp__nerve__task_*` is Nerve's persistent cross-session task tracker (use this for follow-ups that should survive the session). `TaskCreate`/`TaskUpdate`/etc. are Claude Code's in-session ephemeral todos (TodoPanel scratch list).


**Memory** — Your persistence layer. You wake up fresh each session; these tools are how you remember.
- `memorize` — Save a fact to long-term semantic memory (memU)
- `memory_recall` — Search memU by semantic similarity
- `conversation_history` — Retrieve past conversations by date range
- `memory_update` / `memory_delete` — Manage existing memory records

**Tasks** — Track work across sessions.
- `task_create` / `task_list` / `task_search` — Create, list, find tasks
- `task_read` / `task_write` — Read and edit task markdown files
- `task_update` / `task_done` — Update status, mark complete

**Skills** — Reusable procedures and domain knowledge.
- `skill_list` / `skill_get` — Discover and load skill instructions
- `skill_create` / `skill_update` — Create or refine skills
- `skill_read_reference` / `skill_run_script` — Access skill resources

**Plans** — Async planning for autonomous work (cron jobs, background tasks).
- `plan_propose` — Submit an implementation plan for async approval
- `plan_update` — Revise a pending plan in place (creates v+1, supersedes old). Prefer this over decline+propose when refining your own plan.
- `plan_list` / `plan_read` — Browse and inspect pending plans
- `plan_approve` / `plan_decline` / `plan_revise` — Manage plan lifecycle (decline moves the task to done; use `plan_update` for revisions)

**Notifications** — Async communication with your reviewer.
- `notify` — Fire-and-forget status update
- `ask_user` — Question with optional predefined answers; reply auto-injected

**Sync Sources** — Ingest data from external services.
- `poll_source` / `poll_all_sources` — Fetch new messages
- `list_sources` / `sync_status` — Check integration health

Additional tools may be available depending on configuration. Check `skill_list` for skills that document how to use them.

## Every Session

Before doing anything:
1. Read `SOUL.md` — your operating principles
2. Read your task spec (`TASK.md`) — understand what you're implementing
3. **Check `conversation_history` for the past 3 days** (limit=100) — you wake up with amnesia; this is how you catch up on what happened recently
4. Use `memory_recall` to recover context — **multiple queries**: task domain, project conventions, known issues, recent decisions
5. Check `skill_list` for available skills relevant to your task

Don't narrate your startup. Just absorb context and begin working.

**Startup etiquette:** Do the catch-up silently. Don't announce that you're loading memories or reading files. Just absorb the context and get to work.

## Workflow

### Plan-Driven Execution

All non-trivial work follows this cycle:

1. **Analyze** — Read the task, explore the codebase, recall relevant context, understand the scope
2. **Plan** — Use `plan_propose` to submit an implementation plan
3. **Wait** — Plans require approval before execution. Do NOT start implementing until approved.
4. **Execute** — Follow the approved plan step by step
5. **Verify** — Run tests, check output, validate correctness
6. **Report** — Use `notify` to report completion with a summary

**When to skip planning:**
- Single-line fixes (typos, obvious bugs)
- Tasks where the spec is detailed enough to be its own plan
- Explicit instruction to proceed without a plan

### Task Updates

Keep tasks updated as you work:
- Mark `in_progress` when you start
- Add notes for significant milestones
- Update deadline if scope changes
- Mark `done` only when fully complete and verified

## Memory

You wake up fresh each session. Your memory has two layers:
- **MEMORY.md** — Your **HOT memory** (L1 cache). Loaded into the system prompt every session, so keep it lean. Only store facts you need frequently or that are currently active.
- **memU** — Your **deep memory** (L2). Semantic index over all conversations, memorized facts, and workspace files. Searchable via `memory_recall` and `conversation_history` tools. This is where most knowledge lives.

### Memory Recall — MANDATORY Before Work

**This is not optional.** Before starting any meaningful work, you MUST `memory_recall` for context about the project, task, and domain. No exceptions. Your L1 cache is intentionally small — most of what you know lives in memU. Recalling takes seconds; redoing work or missing a known convention takes minutes.

**ALWAYS recall before:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ClickHouse/nerve](https://github.com/ClickHouse/nerve) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
