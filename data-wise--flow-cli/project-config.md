---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

**flow-cli** - Pure ZSH plugin for ADHD-optimized workflow management. Zero dependencies. Standalone (works without Oh-My-Zsh or any plugin manager).

- **Architecture:** Pure ZSH plugin (no Node.js runtime required)
- **Current Version:** v7.6.0
- **Install:** Homebrew (recommended), or any plugin manager
- **Source:** `source /opt/homebrew/opt/flow-cli/flow.plugin.zsh` (via Homebrew)
- **Optional:** Atlas integration for enhanced state management
- **Health Check:** `flow doctor` for dependency verification
- **User ZSH Config:** `~/.config/zsh/` (not `~/.zshrc`)

### What It Does

- Instant workflow commands: `work`, `dash`, `finish`, `hop`
- 15 smart dispatchers: `g`, `mcp`, `obs`, `qu`, `r`, `cc`, `tm`, `wt`, `dots`, `sec`, `tok`, `teach`, `prompt`, `v`, `em`
- ADHD-friendly design (sub-10ms response, smart defaults)
- Session tracking, project switching, quick capture
- Teaching workflow with Scholar integration
- macOS Keychain secret management

---

## Git Workflow & Standards

**CRITICAL:** Follow these mandatory workflow rules when developing for flow-cli.

### Branch Architecture

- **main**: Production. PROTECTED. No direct commits. Only merges from `dev`.
- **dev**: Planning & Integration Hub. All features start here.
- **feature/**: Isolated implementation branches (via worktrees).

### Mandatory Workflow Steps

#### 1. Plan on `dev` Branch

**Before writing any code:**

````bash
git checkout dev && git pull origin dev
```diff

- Analyze requirements on `dev` branch
- Create comprehensive implementation plan
- Document in `docs/specs/SPEC-*.md`
- **Wait for user approval**
- Commit approved plan to `dev`

**Constraint:** Never write feature code on `dev` branch

#### 2. Create Worktree + Orchestration Plan

```bash
git worktree add ~/.git-worktrees/flow-cli/<feature> -b feature/<feature> dev
git worktree list
```zsh

After creating the worktree, write an `ORCHESTRATE-<feature>.md` file **to the worktree** with the full implementation plan (task list, file changes, verification steps). Commit it to the feature branch.

#### 3. STOP - NEW Session Required

**CRITICAL:** Do NOT start implementing in the worktree from the dev/planning session. The dev session's job ends after creating the worktree and committing the orchestration plan. Tell user to `cd` into worktree and start a new `claude` session.

#### 4. Atomic Development (In Worktree)

**Conventional Commits:** `feat:`, `fix:`, `refactor:`, `docs:`, `test:`, `chore:`

**Before each commit:** Run `./tests/run-all.sh`, verify `source flow.plugin.zsh`

#### 5. Integration (feature -> dev)

```bash
git fetch origin dev && git rebase origin/dev
./tests/run-all.sh
gh pr create --base dev
# After merge: git worktree remove ~/.git-worktrees/flow-cli/<feature>
```bash

#### 6. Release (dev -> main)

```bash
gh pr create --base main --head dev --title "Release vX.Y.Z"
git tag -a vX.Y.Z -m "vX.Y.Z" && git push --tags
```diff

### ABORT Conditions

1. About to commit to main -> Redirect to PR workflow
2. About to commit to dev -> Confirm if spec/planning commit
3. Push to main/dev without PR -> Block, require PR
4. Working in worktree from planning session -> Stop, tell user new session
5. About to implement code after creating worktree on dev -> STOP, write orchestration plan only

**See:** `docs/contributing/BRANCH-WORKFLOW.md`

---

## Layered Architecture

flow-cli is Layer 1 of a 3-layer stack: **flow-cli** (pure ZSH, <10ms) < **aiterm** (Python CLI, rich viz) < **craft** (Claude Code plugin). flow-cli owns instant operations, session management, ADHD motivation, quick navigation, and simple dispatchers.

---

## Quick Reference

### Core Commands

```bash
work <project>    # Start session (cd + context, no editor)
work <proj> -e    # Start session + open $EDITOR
finish [note]     # End session (optional commit)
hop <project>     # Quick switch (tmux)
dash [category]   # Project dashboard
catch <text>      # Quick capture
js                # Just start (auto-picks project)
flow doctor       # Health check
flow doctor --fix # Interactive install missing tools
```text

### Dopamine Features

```bash
win <text>        # Log accomplishment (auto-categorized)
yay               # Show recent wins
yay --week        # Weekly summary + graph
flow goal set 3   # Set daily win target
```text

### Active Dispatchers (15)

```bash
g <cmd>       # Git workflows
mcp <cmd>     # MCP server management
obs <cmd>     # Obsidian notes
qu <cmd>      # Quarto publishing
r <cmd>       # R package dev
cc [cmd]      # Claude Code launcher
tm <cmd>      # Terminal manager
wt <cmd>      # Worktree management
dots <cmd>    # Dotfile management (chezmoi)
sec <cmd>     # Secret management (Keychain/Bitwarden)
tok <cmd>     # Token management (create/rotate/expire)
teach <cmd>   # Teaching workflow
prompt <cmd>  # Prompt engine switcher
v <cmd>       # Vibe coding mode
em <cmd>      # Email management (himalaya)
at <cmd>      # Atlas bridge (project intelligence, optional)
```

**Get help:** `<dispatcher> help` (e.g., `r help`, `teach help`, `at help`)

### Teaching Subcommands


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Data-Wise/flow-cli](https://github.com/Data-Wise/flow-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
