---
trigger: always_on
description: Parallel Claude Code execution system. Overwhelm features with coordinated zergling instances.
---

# ZERG

Parallel Claude Code execution system. Overwhelm features with coordinated zergling instances.

## Quick Start

These are Claude Code slash commands. Use them inside a Claude Code session:

```claude
/zerg:init               # Set up project infrastructure
/zerg:brainstorm         # Discover what to build (optional)
/zerg:plan user-auth     # Plan a feature
/zerg:design             # Design architecture (after approval)
/zerg:rush --workers=5   # Launch the swarm (after approval)
/zerg:status             # Monitor progress
```

## How It Works

1. **Plan**: You describe what to build. ZERG captures requirements.
2. **Design**: ZERG creates architecture and breaks work into atomic tasks with exclusive file ownership.
3. **Rush**: Multiple Claude Code instances execute tasks in parallel, organized by dependency levels.
4. **Merge**: Orchestrator merges branches after each level, runs quality gates.

## Key Concepts

**Levels**: Tasks grouped by dependencies. All zerglings finish Level 1 before any start Level 2.

**File Ownership**: Each task owns specific files. No conflicts possible.

**Spec as Memory**: Zerglings read spec files, not conversation history. Stateless and restartable.

**Verification**: Every task has an automated verification command. Pass or fail, no subjectivity.

## Claude Code Task Ecosystem (MANDATORY — READ THIS FIRST)

> **This section has been rearchitected before due to drift. Pay close attention.**

**The Claude Code Task system is the authoritative backbone for all ZERG task state.** Every ZERG command MUST use Claude Code Tasks (TaskCreate, TaskUpdate, TaskList, TaskGet) for tracking work. This is non-negotiable.

### Core Rules

1. **Tasks are the source of truth.** State JSON files (`.zerg/state/`) are supplementary. If Task system and state JSON disagree, the Task system wins.
2. **Every command tracks itself.** All `/zerg:*` commands must TaskCreate on start, TaskUpdate to `in_progress`, and TaskUpdate to `completed` on finish.
3. **Workers claim via Tasks.** Before executing a ZERG task, workers MUST call TaskUpdate with `status: "in_progress"`. After completion or failure, update accordingly.
4. **Dependencies use Task fields.** Use `blocks`/`blockedBy` via TaskUpdate — not just task-graph.json.
5. **Sessions share via CLAUDE_CODE_TASK_LIST_ID.** Workers inherit the orchestrator's `CLAUDE_CODE_TASK_LIST_ID` by default. If the orchestrator has no value set, workers use the same default task list. Teams can explicitly `export CLAUDE_CODE_TASK_LIST_ID=feature` before starting Claude Code for feature-scoped lists.
6. **Tasks persist in `~/.claude/tasks/`.** They survive session restarts. They are the coordination mechanism between parallel Claude Code instances.
7. **Resume checks existing Tasks.** `/zerg:rush --resume` calls TaskList first and only creates tasks that don't already exist.

### Task Subject Convention

All ZERG tasks use bracketed prefixes for discoverability:
- `[Plan] Capture requirements: {feature}`
- `[Design] Architecture for {feature}`
- `[L{level}] {task title}` — execution tasks from task-graph.json
- `[Brainstorm] Discovery: {domain}`
- `[Init]`, `[Cleanup]`, `[Review]`, `[Build]`, `[Test]`, `[Security]`, etc. — utility commands

### Why This Matters

Without Task ecosystem integration, parallel workers can't coordinate, sessions lose state on restart, and `/zerg:status` has no authoritative data source. The Task system IS the project management layer for ZERG.

### Anti-Drift Rules

**This system has drifted before. These rules exist to prevent it from happening again.**

1. **Never remove Task tool calls from command files.** If a command file references TaskCreate, TaskUpdate, TaskList, or TaskGet, those references are load-bearing. Do not remove, comment out, or "simplify" them.

2. **Every command file MUST contain Task tool calls.** All command files in `zerg/data/commands/` are required to have Task ecosystem integration. There are zero exceptions. The minimum pattern for any command is:
   ```
   On invocation:  TaskCreate (subject with bracketed prefix)
   Immediately:    TaskUpdate status "in_progress"
   On completion:  TaskUpdate status "completed"
   ```

3. **Backbone commands have additional requirements.** These 5 files have deeper integration beyond the minimum pattern. Do not reduce them to the minimum:
   - `worker.md` — TaskUpdate to claim tasks, TaskUpdate for failures/checkpoints, TaskList at completion
   - `status.md` — TaskList as primary data source, cross-reference with state JSON, flag mismatches
   - `merge.md` — TaskUpdate after quality gates per level, TaskList verification at finalize
   - `stop.md` — TaskUpdate with PAUSED/FORCE STOPPED annotations
   - `retry.md` — TaskGet to read state, TaskUpdate to reset to pending, TaskUpdate on reassignment

4. **State JSON is the fallback, not the primary.** If you find yourself writing code that reads `.zerg/state/` without also consulting TaskList, you are drifting. State JSON supplements Tasks, not the other way around.

5. **New commands get Task integration on creation.** If you create a new `/zerg:*` command file, it MUST include the minimum Task tracking pattern before it is considered complete.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rocklambros/zerg](https://github.com/rocklambros/zerg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
