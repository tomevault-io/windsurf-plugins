---
trigger: always_on
description: **NodeSpace has ZERO users, NO production deployment, and NO releases.**
---

# NodeSpace Development Agent Guide

## CRITICAL: Pre-Release Development - NO BACKWARD COMPATIBILITY

**NodeSpace has ZERO users, NO production deployment, and NO releases.**

- ❌ **NO backward compatibility code** - Delete old patterns immediately when replaced
- ❌ **NO migration strategies** - We can reset the database anytime
- ❌ **NO gradual rollouts** - Implement new architecture directly, delete old code
- ❌ **NO transition periods** - No dual-mode support, no feature flags for compatibility
- ❌ **NO version support** - Don't maintain multiple versions of any API/method
- ❌ **NO "soak periods"** - No waiting weeks between changes
- ❌ **NO phased migrations** - Unless coordinating across multiple active worktrees
- ❌ **NO `#[deprecated]` attributes** - Delete old code, don't deprecate it
- ❌ **NO `#[allow(dead_code)]`** - Delete unused code, don't suppress warnings

Make breaking changes without hesitation. Fix breakage immediately in the same session. Implement final architecture directly — skip intermediate steps. If you find yourself writing "for backward compatibility" or "during the transition period" — **STOP. This is greenfield development.**

## Project Overview

NodeSpace is an AI-native knowledge management system: Rust backend, Svelte 5 frontend, Tauri 2.0 desktop. Stack: libsql/SQLite (local store — `packages/core/src/db/`), async/await trait-based Rust, $state/$derived/$effect runes. UI-first approach — build interfaces with mock data before storage integration.

**Before starting any task, read:**
- [`overview.md`](../nodespace-docs/development/overview.md) - Complete development process
- [`startup-sequence.md`](../nodespace-docs/development/startup-sequence.md) - Mandatory pre-implementation steps

## Mandatory Startup Sequence — NEW TASK

> **EXCEPTION: If continuing from a WIP commit, skip to the next section.**

1. **Check git status on the primary checkout**: `git status` — commit any pending changes first
2. **Pull latest `main`**: `git fetch origin && git pull origin main`
3. **Enter an isolated worktree**: `EnterWorktree({name: "issue-<number>-brief-desc"})`
   - Creates `~/.worktrees/issue-<number>-brief-desc/` on a new branch branched from `origin/main`
   - All subsequent commands run **inside the worktree**; primary `main` stays untouched
   - Naming: terse, no `feature/` prefix — branch name doubles as directory name, e.g. `issue-1122-agent-tools`
   - Continuing parent-issue work on a shared branch: `EnterWorktree({path: "~/.worktrees/<existing>"})`. If no worktree exists yet: `git worktree add ~/.worktrees/<name> <branch>` first
4. **Install dependencies**: `bun install`
5. **Run test baseline**: `bun run test` — frontend only (Rust tests require warm cache)
   - If you hit `Cannot find base config file "./.svelte-kit/tsconfig.json"`, run `bunx svelte-kit sync` from `packages/desktop-app/` once, then re-run
   - WAIT for complete output — look for "Test Files X passed" summary and "Duration" line
6. **Document baseline**: `bun run gh:comment <number> "Frontend: X passed"`

   > ⚠️ **All `bun run gh:*` commands MUST run from the worktree root, NOT from subdirectories. Do NOT pipe to gh:comment (it doesn't read stdin).**

7. **Assign issue**: `bun run gh:assign <number> "@me"`
8. **Update project status**: `bun run gh:status <number> "In Progress"`
9. **Select subagent**, read issue requirements, plan self-contained implementation

## Mandatory Startup Sequence — CONTINUING FROM WIP

1. **Enter the existing worktree**: `EnterWorktree({path: "~/.worktrees/issue-<N>-brief-desc"})`
   - If removed: `git worktree add ~/.worktrees/issue-<N>-brief-desc <branch>` first
2. **Check git status**: confirm you're on the right branch
3. **Pull latest**: `git fetch origin && git pull origin <branch-name>`
4. **Sync dependencies if needed**: `bun install` — only if WIP commit mentions new packages
5. **Review WIP commit message**: understand completed work and remaining tasks
6. **Resume** from the "Remaining Work" section

**DO NOT** re-run baseline, re-assign the issue, re-update status, or create a new worktree.

## Critical Process Violations

If you start implementation without completing the startup sequence: STOP, complete it, restart.

**Common mistakes:**
- Skipping `git pull` on main before EnterWorktree — worktree branches from stale local `main`
- Skipping test baseline — leads to undetected regressions
- Running `bun run gh:*` from a subdirectory — fails with "Script not found"
- Reading/editing files before EnterWorktree — edits land in wrong checkout
- Skipping EnterWorktree entirely and working on `main` — blocks parallel work
- Using TodoWrite without startup sequence as the first item

## Finding Tasks

```bash
bun run gh:list
bun run gh:view <issue-number>
bun run gh:edit <issue-number> --title "New Title"
bun run gh:edit <issue-number> --body "Updated description"
bun run gh:edit <issue-number> --labels "foundation,ui"
bun run gh:edit <issue-number> --state "closed"
```

When creating or modifying issues, follow the [Issue Workflow Guide](../nodespace-docs/development/issue-workflow.md).

Issue priority: `foundation` (highest) > `design-system` > `ui` > `backend`

## Architecture & Docs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NodeSpaceAI/nodespace-core](https://github.com/NodeSpaceAI/nodespace-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-10 -->
