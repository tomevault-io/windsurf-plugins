---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Auto-Claude is a Bash-based CLI tool that runs Claude Code in a continuous loop with persistent context. It automates the full PR lifecycle: Claude makes changes → creates branch → opens PR → waits for CI checks → merges → repeats. The system maintains context across iterations via a shared markdown file (`SHARED_TASK_NOTES.md` by default).

**Key concept**: Instead of one-shot AI assistance, this enables long-running autonomous development where Claude can work on multi-step projects over time while you sleep or work on other things.

## Core Architecture

### Main Script: `auto_claude.sh`

The script orchestrates a loop that:
1. Creates a new git branch for each iteration
2. Runs Claude Code with an enhanced prompt (includes workflow context + user prompt + notes from previous iteration)
3. Claude makes changes and updates the shared notes file
4. Commits changes automatically via a nested Claude call
5. Pushes branch and creates PR via GitHub CLI
6. Waits for all CI checks and code reviews to pass (30min timeout)
7. Merges PR using specified strategy (squash/merge/rebase)
8. Pulls latest main and repeats

### Context Persistence

**`SHARED_TASK_NOTES.md`** (or custom file via `--notes-file`):
- External memory shared across iterations
- Claude reads it at start of each iteration and updates it at the end
- Enables "relay race" handoffs between runs
- Should be concise, actionable notes (not verbose logs or reports)

**Enhanced Prompt Structure**:
```
## CONTINUOUS WORKFLOW CONTEXT
[Instructions about iterative work + completion signal]

## PRIMARY GOAL
[User's original prompt]

## CONTEXT FROM PREVIOUS ITERATION
[Contents of SHARED_TASK_NOTES.md if it exists]

## ITERATION NOTES
[Instructions to update the notes file]
```

### Execution Limits

The loop can be bounded by three independent constraints:
- `--max-runs N`: Stop after N successful iterations (0 = infinite)
- `--max-cost X.XX`: Stop when total cost reaches $X.XX USD
- `--max-duration Xh/Xm/Xs`: Stop after time duration (e.g., `2h`, `30m`, `1h30m`)

Whichever limit is reached first will stop the loop.

### Error Handling

- **3 consecutive errors = fatal exit**: Prevents infinite error loops
- **Failed CI checks**: PR is closed, branch deleted, iteration fails (but counted toward error threshold)
- **Successful iteration**: Resets error counter, decrements extra iterations
- **Extra iterations**: Added for each error to compensate for failed attempts

### Completion Signals

If multiple agents independently determine the entire project is complete (not just their current task), they output the completion signal (default: `AUTO_CLAUDE_PROJECT_COMPLETE`). When `--completion-threshold N` consecutive iterations emit the signal, the loop stops early.

### Git Worktrees (Parallel Execution)

`--worktree <name>` creates/uses a git worktree in `../auto-claude-worktrees/<name>/` allowing multiple instances to run simultaneously on different tasks without conflicts. Each worktree:
- Pulls latest from main on startup
- Works in isolation
- Can be cleaned up with `--cleanup-worktree`

## Development Commands

### Running Tests

```bash
# Run BATS test suite
./tests/libs/bats/bin/bats tests/test_auto_claude.bats

# Setup test dependencies (installs BATS and libraries)
./tests/setup.sh
```

Tests use BATS (Bash Automated Testing System) with bats-support and bats-assert libraries.

### Installation

```bash
# Automated install (downloads to ~/.local/bin/auto-claude)
curl -fsSL https://raw.githubusercontent.com/iKislay/auto-claude/main/install.sh | bash

# Manual install
chmod +x auto_claude.sh
sudo mv auto_claude.sh /usr/local/bin/auto-claude
```

### Common Usage Patterns

```bash
# Basic: 5 iterations with auto-detected GitHub repo
auto-claude -p "add unit tests" -m 5

# Budget-limited: stop at $10 spent
auto-claude -p "improve code quality" --max-cost 10.00

# Time-boxed: run for 2 hours
auto-claude -p "refactor module" --max-duration 2h

# Infinite loop until manually stopped
auto-claude -p "increase test coverage" -m 0

# Parallel execution in different worktrees
auto-claude -p "Add unit tests" -m 5 --worktree tests
auto-claude -p "Add docs" -m 5 --worktree docs  # Run in another terminal

# Testing without creating PRs
auto-claude -p "test changes" -m 2 --disable-commits --dry-run

# Custom completion threshold
auto-claude -p "fix all bugs" -m 20 --completion-threshold 3

# Forward flags to Claude Code
auto-claude -p "task" -m 3 --allowedTools "Write,Read" --model claude-haiku-4-5
```

## Key Functions

### Main Loop (`main_loop` at line 1616)
- Controls iteration flow based on limits (runs/cost/duration)
- Calls `execute_single_iteration` for each run
- Checks for completion signals

### Single Iteration (`execute_single_iteration` at line 1530)
1. Creates branch via `create_iteration_branch`
2. Builds enhanced prompt with context from `SHARED_TASK_NOTES.md`
3. Runs Claude via `run_claude_iteration`
4. Parses JSON result via `parse_claude_result`
5. Handles success/error via `handle_iteration_success`/`handle_iteration_error`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iKislay/auto-claude](https://github.com/iKislay/auto-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
