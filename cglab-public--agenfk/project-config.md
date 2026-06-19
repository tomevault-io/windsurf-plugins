---
trigger: always_on
description: Agile, measurable, and reliable workflow enforcement for AI-assisted engineering.
---


# AgenFK Engineering Framework (agenfk)

This skill enforces the core AgenFK Engineering workflow to ensure all software tasks are Agile, Measurable, Visual, Repeatable, Reliable, and Flexible.

## Strict Enforcement Mandate

> **MANDATORY**: You are strictly prohibited from modifying ANY file in the codebase without an active task in `IN_PROGRESS` status and a successful `workflow_gatekeeper` call. Bypassing this workflow is a critical operational failure.

### Hard Block Rules
1. **NO TASK = NO CODE**: If no task is `IN_PROGRESS`, stop immediately and create one.
2. **NO GATE = NO CODE**: Call `workflow_gatekeeper` before the first edit of every session.
3. **NO BYPASS**: Never use `git commit`, `npm test`, or direct file writes to circumvent `validate_progress`.
4. **MEASURE EVERYTHING**: Every task MUST have token usage logged via `log_token_usage` before completion.

## Operation Modes

AgenFK supports two distinct operation modes based on the slash command invoked:

### 1. Standard Mode (via `/agenfk`)
*   **Behavior**: Single-agent, proactive execution.
*   **Workflow**: The agent who starts the task is responsible for the entire lifecycle (Planning, Coding, Verification, and Closing) within a single session.
*   **Mandatory Log**: You MUST call `add_comment(itemId, content)` for EVERY significant tool execution or logical step (e.g. "Analyzed file X", "Implemented function Y", "Running tests").
*   **Proactivity**: For simple requests (TASK/BUG), the agent should proceed directly to implementation after basic analysis.
*   **Verification**: You MUST use `update_item({ status: "REVIEW" })` to enter REVIEW, then call `workflow_gatekeeper(itemId)` to get exit criteria, then `validate_progress` to advance through intermediate steps and reach DONE.
*   **Decomposition**: MANDATORY. Every piece of work must be minimally a **STORY with child TASKS** or an **EPIC with child STORIES and their TASKS**. Direct coding on a STORY or EPIC without child TASKS is prohibited.
*   **Handoff**: None. Do not spawn sub-agents.

### 2. Deep Mode (via `/agenfk-deep`)
*   **Behavior**: Multi-agent, automated orchestration.
*   **Trigger**: Use this mode for complex architectural changes, high-security code, or large features.
*   **Supervisor Pattern**: You act as a supervisor, responsible for decomposing the task and spawning specialized sub-agents via the `task` tool at every phase transition.
*   **Parallel Execution**: Deep Mode supports **parallel execution** of independent tasks.
    - If an EPIC or STORY has multiple independent sub-items, you SHOULD spawn multiple sub-agents simultaneously using the `task` tool.
    - When working in parallel, you MUST pass the `itemId` to the `workflow_gatekeeper(intent, role, itemId)` to authorize changes against the specific task.
*   **Plan & Pause**: Mandatory decomposition into sub-items. You **MUST PAUSE** and obtain human approval of the plan before moving any item to `IN_PROGRESS`.
*   **Automated Handover**:
    - **Coding to Review**: Automatically spawn a "Review Agent" after `update_item({ status: "REVIEW" })`.
    - **Review to Test**: Automatically spawn a "Test Agent" after successful review.
    - **Test to Done**: Automatically spawn a "Closing Agent" after successful testing.

---

## MCP Access Rules — MANDATORY

**ALWAYS** use MCP tool invocations (`list_items`, `create_item`, `update_item`, `get_item`, etc.) for all workflow state operations.

**NEVER** use any of the following shortcuts — PreToolUse hooks will block them mechanically:

| Forbidden | Use instead |
|-----------|-------------|
| Reading `.agenfk/db.sqlite` or `.agenfk/db.json` directly (via Bash or Read tool) | `list_items(projectId)` · `get_item(id)` |
| `curl` / `wget` to `http://localhost:3000` (direct REST API) | `list_items()` · `create_item()` · `update_item()` |
| `agenfk list`, `agenfk status`, `agenfk get`, `npx agenfk ...` (CLI state queries) | `list_items()` · `get_item()` · `list_projects()` |

If MCP tools are not available in your context, surface the connectivity problem clearly rather than falling back to a bypass route.

---

## What I do

1.  **Initialization**
    *   **Clean Start from Main (MANDATORY)**:
        1. Run `git status` — if the working tree has uncommitted or modified files, **STOP** and ask the user how to proceed (stash, commit, or discard). Never start new work on a dirty working tree.
        2. Run `git branch --show-current` — if NOT on `main` (or `master`) and the current branch does not belong to an item you're about to resume, run `git checkout main` (or `master`).
        3. Run `git pull` to ensure you have the latest upstream changes.
    *   **Action**:
        1. Check for `.agenfk/project.json` in the project root.
        2. If missing, DO NOT assume an existing project should be reused based on name alone.
        3. Call `list_projects()` via MCP to see existing projects.
        4. **MANDATORY**: Ask the user if they want to use an existing project (by name/ID) or create a new one (recommended).
        5. If creating a new one, use the current directory name as the default project name unless the user specifies otherwise.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cglab-public/agenfk](https://github.com/cglab-public/agenfk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
