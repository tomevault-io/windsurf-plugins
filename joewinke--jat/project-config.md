---
trigger: always_on
description: <!-- This is the canonical source for agent workflow instructions.
---

# JAT Agent Instructions

<!-- This is the canonical source for agent workflow instructions.
     It is imported by CLAUDE.md (for Claude Code agents) and read directly
     by other agent programs (Pi, Codex, Gemini, OpenCode, Aider).
     Keep this file self-contained - no @import syntax. -->

You are running as part of a **multi-agent development system** called JAT (Jomarchy Agent Tools). This system enables parallel, coordinated work across codebases using multiple AI coding agents.

## Quick Start

```bash
/jat:start                 # Start a session (pick a task)
/jat:start jat-abc123      # Start a specific task
/jat:complete              # Complete your task
/jat:verify                # Verify in browser
```

> **Command prefix:** Claude Code uses `/jat:` commands. Other agents (Pi, Codex, Gemini, etc.)
> use `/skill:jat-` skills. The behavior is identical.

## Core Principles

1. **One agent = one session = one task** - Each session handles exactly one task
2. **File declarations prevent conflicts** - Declare files when starting a task via `--files`
3. **Memory coordinates work** - Past session context surfaces via `.jat/memory/`
4. **JAT Tasks is the task queue** - Pick from ready work, update status, close when done
5. **Signals track your state** - The IDE monitors agents through `jat-signal`

## JAT Tasks (Task Management)

This project uses `jt` (JAT Tasks) for issue tracking.

```bash
jt ready                    # Find available work (highest priority, no blockers)
jt show <id>                # View task details
jt show <id> --json         # JSON format
jt update <id> --status in_progress --assignee AgentName
jt close <id> --reason "Completed"   # If requester is set → status=submitted, assignee=requester
jt list --status open       # List all open tasks
jt search "keyword"         # Search tasks
jt create "Title" --requester "username"  # Set who must review/accept completed work
```

**Status values** (use underscores, not hyphens):

Agent-workable (agents pick these up via `jt ready`):
- `open` - Available to start
- `in_progress` - Being worked on

Paused / mid-flight (agents do NOT pick these up):
- `waiting` - Ball in counterparty's court (awaiting their input)
- `blocked` - Blocked by external dependency
- `submitted` - In requester's queue for acceptance (see Requester Workflow below)
- `accepted` - Requester approved, pending deploy
- `deployed` - Shipped, pending archive/closeout

Terminal / special:
- `closed` - Completed
- `dev` - Internal/dev-only (hidden from clients)

**Task types:** `bug`, `feature`, `task`, `epic`, `chore` (recurring scheduled task), `chat`

### Requester Workflow

`requester` is the person who originally asked for the work. Set it at task creation so `jt close` knows where to route the completed task.

```bash
jt create "Fix login bug" --requester "mike"   # mike must accept when done
jt create "Refactor cache" --requester "jw"    # jw must accept (or auto-accepts if jw delegated it)
```

**What happens on `jt close`:**

| Condition | Outcome |
|-----------|---------|
| No `requester` | `closed` immediately |
| `requester` set, third party delegated | `submitted` → requester's queue for accept/reject |
| `requester` set, requester delegated to agent themselves | `accepted` automatically (requester already endorsed the work by spawning the agent) |

**The self-accept rule:** when `previous_assignee == requester`, the requester was the one who handed the task to the agent — they've implicitly accepted the outcome. No review queue needed.

**Example:**
- jw creates task (`requester=jw`), spawns agent → agent completes → **`accepted`** (jw delegated it)
- mike creates task (`requester=mike`), jw spawns agent → agent completes → **`submitted`** to mike for review

### Dependencies

```bash
jt dep add parent-id child-id   # parent depends on child
jt dep tree task-id             # Show dependency tree
jt dep remove parent-id child-id
```

### Epics

Epics are **blocked by their children** (children are READY, epic waits):

```bash
# Create epic
jt create "Epic title" --type epic --priority 1

# Create children
jt create "Child task" --type task --priority 2

# Set dependencies: epic depends on children (NOT children on epic)
jt dep add epic-id child-id
```

## Agent Registry (Identity)

Agent identities for multi-agent coordination. All tools are in `~/.local/bin/`.

```bash
# Identity
am-register --name AgentName --program pi --model sonnet
am-whoami
am-agents                                         # List all agents

# See what other agents are doing (real-time state from tmux + signals)
jt agents                                         # Active agents with state, task, files
jt agents --json                                  # JSON output for programmatic use
jt agents --project jat                           # Filter by project
jt agents --all                                   # Include inactive agents

# File Declarations (prevent conflicts) - via jt on the task itself
jt update task-id --status in_progress --assignee AgentName --files "src/**/*.ts"
```

Cross-session context is handled by agent memory (`.jat/memory/`).

## Signals (IDE State Tracking)

The IDE tracks your state through signals. Emit them in order:

```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joewinke/jat](https://github.com/joewinke/jat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
