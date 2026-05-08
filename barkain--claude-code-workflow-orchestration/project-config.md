---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## Delegation Policy (Soft Enforcement)

The framework nudges via stderr when the main agent uses work-doing tools (`Bash`, `Edit`, `Write`, `Read`, `Glob`, `Grep`, `MultiEdit`, `NotebookEdit`) directly. **Nudges never block.** They escalate by per-turn violation count: silent → imperative STOP → imperative STOP (2nd call phrasing) → strong reminder explaining what's being lost. The counter resets each turn and zeros when `/workflow-orchestrator:delegate` runs.

The expected path for any multi-step or work-shaped request is:

```
/workflow-orchestrator:delegate <task description>
```

Subagents are immune (they're executing a delegation). New tools added by Claude Code never trigger nudges — only the 8 stable work primitives are tracked.

---

## Prerequisites

- **uv** - Python package manager (required for `uvx`, `uv run`)
- **bun** - JavaScript runtime (for ccusage cost tracking in statusline)
- **jq** - JSON processor (optional, for advanced features)

No `pyproject.toml` exists — all scripts use `uv run --no-project --script` mode.

---

## Build, Lint, and Test Commands

```bash
uvx ruff format .                # Format code (auto-fix)
uvx ruff check --no-fix .       # Lint code (check only)
uvx pyright .                    # Type checking
uvx deadcode hooks/ scripts/    # Dead code detection
```

The PostToolUse hook enforces a specific ruff rule subset on edited Python files:
```bash
uvx ruff check --select F,E711,E712,UP006,UP007,UP035,UP037,T201,S <file>
```

CI workflow exists (`.github/workflows/ci.yml`) but tests are currently disabled/placeholder.

---

## Available Commands

```bash
/workflow-orchestrator:delegate <task>   # Plan and execute task via native plan mode
/workflow-orchestrator:ask <question>    # Read-only question answering (forked context)
/workflow-orchestrator:add-statusline    # Enable workflow status display
```

**Installation:**
- Plugin: `claude plugin install workflow-orchestrator@barkain-plugins`
- Manual: `./install.sh [--scope=user|project]`

In plugin mode, all commands and agent names use the `workflow-orchestrator:` prefix.

---

## Architecture Overview

### Execution Flow

**Token overhead:** Conditional injection (stub ~200 tokens on startup, full ~7.5K tokens on first delegation, optional token-efficient guide ~1.9K) + per-agent delegation (~350 tokens)

```
User prompt
  → UserPromptSubmit hook (clear state, record turn timestamp, clear team state)
  → SessionStart hooks (inject stub orchestrator + output style + token efficiency)
    [Stub version provides just enough system direction, avoiding unnecessary tokens]
  → Task detection: if multi-step connectors found, enters native plan mode (EnterPlanMode)
  → plan mode: injects full workflow_orchestrator, explores codebase, decomposes, assigns agents, creates tasks via TaskCreate
  → plan mode: evaluates execution_mode (subagent vs team via TeamCreate tool availability)
  → plan mode: exits via ExitPlanMode (requires lead approval)
  → PostToolUse hook (remind_skill_continuation.py): creates workflow_continuation_needed.json on ExitPlanMode
  → After ExitPlanMode approval, main agent continues to Stage 1
  → Main agent: Stage 1 — parses execution plan JSON, renders dependency graph
  → SUBAGENT MODE (default):
    → For each wave: spawn agents via Agent tool (run_in_background: true)
    → Agents write to $CLAUDE_SCRATCHPAD_DIR, return DONE|{path}
    → SubagentStop hooks: remind task update, suggest verification
    → Main agent: TaskUpdate to mark completed, proceed to next wave
  → TEAM MODE (when TeamCreate tool is available):
    → Create .claude/state/team_mode_active + team_config.json
    → TeamCreate(team_name=...), then Agent(team_name=...) for each teammate with agent configs
    → Create shared tasks with dependencies, bridge to framework Tasks API
    → Teammates self-claim tasks, self-coordinate via messaging
    → Lead syncs team completions to TaskUpdate (bridge pattern)
    → Cleanup team state on completion
  → Stop hook: calculate turn duration, quality analysis
```

### Hook System (6 lifecycle events, 14 hooks)

| Event | Scripts | Purpose |
|-------|---------|---------|
| **PreToolUse** (`*`, `Bash`) | `validate_task_graph_compliance.py` (advisory), `require_delegation.py` (adaptive nudge), `token_rewrite_hook.py` (Bash only) | Hint on out-of-order Agent/Task spawns; emit per-turn escalating delegation nudge (silent → strong); rewrite Bash for token-efficient output |
| **PostToolUse** | `python_posttooluse_hook.py` (Edit/Write/MultiEdit, **blocking**), `remind_skill_continuation.py` (ExitPlanMode\|Skill\|SlashCommand), `validate_task_graph_depth.py` (advisory) + `remind_todo_after_task.py` (Agent/Task) | Python validation (Ruff, Pyright, security — only hard-blocking hook); workflow continuation + zero violations counter on `/workflow-orchestrator:delegate`; depth hint; task reminders |
| **UserPromptSubmit** | `clear-delegation-sessions.py` | Reset per-turn state (timestamp, violations counter), clear team state, rotate logs |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [barkain/claude-code-workflow-orchestration](https://github.com/barkain/claude-code-workflow-orchestration) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
