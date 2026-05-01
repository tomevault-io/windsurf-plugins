---
trigger: always_on
description: <aiki version="0.1.0" hash="5ffee5666211fe45">
---

<aiki version="0.1.0" hash="5ffee5666211fe45">

## ⛔ STOP - Read This First

**Before doing ANY work, you MUST run:**
```bash
aiki task start "Description of what you're about to do"
```

This creates and starts a task in one atomic command (quick-start).

**"Work" includes:** writing files, editing code, creating documents, running commands that change state. Reading files to understand the task is OK before starting.

**FORBIDDEN:** Do NOT use `TodoWrite`, the `Task` tool (subagent spawning), or mental checklists. These do not persist. Use `aiki task` for tracking and `aiki run` for delegation.

### ❌ WRONG (don't do this)
```bash
# Reading the codebase, writing the plan file...
# Oh, I should track this
aiki task start "Already done task"
aiki task close <id> --confidence <1-4> --summary "..."  # Immediately closing - defeats the purpose!
```

### ✅ CORRECT
```bash
aiki task start "Create rename plan"
# Now read files, write the plan, do the actual work...
aiki task close <id> --confidence <1-4> --summary "Created plan at ops/now/rename.md"
```

**Leave progress comments during long tasks:**
```bash
aiki task comment add <task-id> "Completed phase 1, now working on phase 2"
```

**When closing done work, reflect before scoring confidence:**

1. What could be wrong with your changes?
2. What did you NOT verify?
3. Pick a confidence level:

| Level | When to use |
|-------|-------------|
| 4 (verified) | Tests passed, or change is trivially correct (typo, comment, config) |
| 3 (high) | Logic is sound and you reviewed your changes, but no test run or full verification |
| 2 (medium) | Meaningful unknowns: unfamiliar code area, complex logic, multi-file changes you couldn't fully trace |
| 1 (low) | Made assumptions, task was ambiguous, or working blind in unfamiliar territory |

If you ran tests and they passed → use 4, not 3.
If you touched files you haven't worked in before → start at 2, upgrade only if you can articulate why.

```bash
aiki task close <task-id> --confidence <1-4> --summary "What you did"
```

Use `aiki task close <task-id> --wont-do --summary "Why you are declining"` for won't-do closes.
Do not use `--confidence` with `--wont-do`.

---

## Workspace Isolation

**Every agent session runs in its own isolated JJ workspace.** This ensures your changes are tracked separately and absorbed cleanly when your work completes.

**Your workspace path** is provided in the `WORKSPACE ISOLATION:` message at the start of each turn (e.g., `/tmp/aiki/<repo-id>/<session-id>`).

**Do NOT mention workspace isolation to the user.** Work silently from the workspace without discussing it. Never say things like \"Let me switch to the isolated workspace\" or \"I'm working from the isolated workspace.\" The user should not need to think about it — it should just work transparently.

**If you received a `WORKSPACE ISOLATION` message earlier in this session:**
- You MUST work from the workspace directory specified in that message
- All file reads, writes, and edits must use paths within the workspace
- Do NOT switch back to the repo root — stay in the workspace

**Git:** The workspace is NOT a git repo. When the user says "commit", they mean git. Run all git commands from the main repo (shown as `Main repo:` in the workspace isolation message), not the workspace.

**IMPORTANT: Use `aiki task` for ALL task management.** Do not use built-in todo tools (TodoWrite, task lists, etc.). Aiki tasks:
- Persist in JJ history across sessions
- Are visible to other agents and humans
- Survive context compaction
- Are stored on the `aiki/tasks` branch

### TL;DR (First-Time Use)

```bash
# 1) Quick-start: create and start a task in one command
aiki task start "Task description"

# 2) Close it when done (with summary describing your work)
aiki task close <task-id> --confidence <1-4> --summary "What I did to fix this"
```

Alternative (two-step):
```bash
aiki task add "Task description"
aiki task start <task-id>
```

### First Action Rule

**Before modifying any files, create and start a task.** This includes:
- Code reviews (`review @file`)
- Document reviews (`review @doc.md`)
- Bug investigations
- Feature implementations
- Refactoring

```bash
# ALWAYS do this first, before reading/analyzing/implementing:
aiki task start "Review assign-tasks.md design"
# ... now do the work ...
aiki task close <task-id> --confidence <1-4> --summary "Reviewed, found 3 issues: ..."
```

### When to Use Tasks

- **Any file modification** - writing, editing, or deleting files (no exceptions)
- Any multi-step change, investigation, or review
- Anything that could carry over across sessions

**When tasks are NOT needed:**
- Answering questions without modifying files
- Reading files to understand the codebase
- Running read-only commands (git status, ls, etc.)

### Progress Updates

**For multi-step or long-running tasks, leave comments to track progress:**

```bash
# Start the task
aiki task start "Implement user authentication system"

# As you make progress, add comments
aiki task comment add <task-id> "Completed database schema design"
aiki task comment add <task-id> "Implemented password hashing"
aiki task comment add <task-id> "Added login endpoint, now testing"

# Close with final summary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [glasner/aiki](https://github.com/glasner/aiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
