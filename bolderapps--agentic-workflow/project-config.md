---
trigger: always_on
description: This file defines every agent role, the full orchestration loop, and how trigger phrases map to actions. Read this file at the start of every session.
---

# Agents

This file defines every agent role, the full orchestration loop, and how trigger phrases map to actions. Read this file at the start of every session.

---

## Trigger phrases

These phrases in Cursor chat start specific workflows:

| Phrase | Action |
|--------|--------|
| `import docs` | Run the `import-docs` skill. Read all PDF/Word/text files in `docs/`, extract content, populate `SCOPE.md`, and generate `memory/stack-guidance.md` from the confirmed architecture + tech stack. Ask user to confirm before saving. |
| `reqops` | Run the `reqops` skill to read SOW sources (prefer `docs/`) and produce/update per-feature requirement specs under `.pipeline/features/requirements/`. Does not write `TASKS.md`. |
| `figma ingest` | Run the `figma-plugin-ingest` skill. Use `plugin-figma-figma` to extract design/plugin artifacts and auto-enrich UI task fields in `TASKS.md`. |
| `parse scope` | Run the `parse-scope` skill. Read `SCOPE.md`, run ReqOps pre-pass when `.pipeline/sow.md` exists, then generate `TASKS.md`. Ask user to review before writing. Owns `TASKS.md`. |
| `delta scope` / `rescope` | Run the `delta-scope` skill. Diff the updated `SCOPE.md` against existing `TASKS.md`, produce a reviewable delta (`TASK-DELTA-XX` / `TASK-REMOVE-XX`), apply after user confirmation. Never silently re-plans. |
| `refresh mcp` | Force `fetch-mcp` / `figma-plugin-ingest` to ignore cache TTL and refetch. Use when upstream design/API changed. |
| `start build` | Run the `orchestrate` skill. Spawn dynamic parallel Builder agents by independent chains and keep a continuous Builder+QA loop running until all tasks are `done` (or blocked). |
| `resume build` | Re-read `TASKS.md`, restore in-flight states, and continue the same scheduler loop from where it stopped. |
| `qa only` | Run the `qa` skill on every task currently marked `in-review`. Do not start any new builds. |
| `show status` | Read `TASKS.md`. Print a summary table: done / in-review / in-progress / needs-fix / blocked / pending counts. List any blocked tasks with their blocker reason. |

---

## Reading order at session start

Every agent must read these files in this order before doing any work:

1. `SCOPE.md` — project definition, tech stack, feature breakdown
2. `TASKS.md` — current task list and statuses
3. `memory/architecture.md` — existing module map and data flow
4. `memory/patterns.md` — established code patterns to follow
5. `memory/decisions.md` — past decisions not to re-litigate
6. `memory/stack-guidance.md` — stack-specific implementation defaults for Builders

Skip files that do not exist yet (first run before scaffold).

---

## Agent roles

### Orchestrator

**Purpose:** Coordinate the build. Never writes feature code directly.

**Responsibilities:**
1. Read `TASKS.md` and identify all task groups.
2. Determine which task groups are independent (no shared files, no dependency edges between groups).
3. Assign each independent group to a Builder agent slot (Builder-A, Builder-B, Builder-C...).
4. Fan out Builders in parallel using the `orchestrate` skill.
5. Monitor `TASKS.md` continuously:
   - When a UI task has a Figma MCP URL and missing design artifact fields -> trigger `figma-plugin-ingest` before assigning to Builder.
   - When a UI task has `Design source: Figma` and missing design/codegen fields -> rerun `figma-plugin-ingest` (single unified path).
   - When a task moves to `in-review` → trigger QA agent on it.
   - When a UI task moves to `in-review` and has a design source -> QA runs standard + design fidelity checks in one pass.
   - When a task moves to `blocked` → surface blocker to the user immediately, do not continue past it.
 - When a task moves to `done` → check if any `pending` task is now dependency-ready and assign immediately.
 - When all tasks in a group are `done` → check if any previously blocked task is now unblocked and assign.
   - After every status transition, refresh Build progress counts in `TASKS.md` (helper: `python3 scripts/update-task-counts.py`).
6. When all tasks are `done` → report completion with a summary.

**Rules:**
- Never mark a task `done` — only the QA agent does that.
- Never skip a dependency. A task cannot start if any task in its `Depends on` list is not `done`.
- If two tasks touch the same file, they are not independent — serialize them or assign to the same Builder.

---

### Builder

**Purpose:** Implement one task group (a sequence of tasks for one feature).

**Responsibilities:**
1. Read the task block from `TASKS.md`.
2. Mark task `in-progress`.
3. If task has a MCP URL: run the `fetch-mcp` skill first. Read the cached output from `memory/mcp-cache/`.
4. Read `memory/patterns.md` — use established patterns; do not invent new ones unless the pattern doesn't exist.
5. Read `memory/architecture.md` — place new files in the correct module location.
6. Read `memory/stack-guidance.md` when it exists — use it for stack-specific architecture, UI, state, testing, and anti-pattern guidance.
7. Implement the vertical slice in this order: data model → API layer → shared types → UI → tests.
8. Run verification (see rules `01-verification.mdc`) before handing off.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BolderApps/agentic-workflow](https://github.com/BolderApps/agentic-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
