---
trigger: always_on
description: > Repository-wide custom instructions for GitHub Copilot CLI integration.
---

# 🔄 Copilot Ralph Mode Instructions

> Repository-wide custom instructions for GitHub Copilot CLI integration.

## Overview

This repository implements **Ralph Mode** - an iterative, self-referential AI development loop for GitHub Copilot CLI. These instructions help Copilot understand how to work within the Ralph loop pattern.

---

## Detecting Ralph Mode

Check if `.ralph-mode/state.json` exists in the workspace root. If it exists, Ralph Mode is active.

```bash
# Check if active
test -f .ralph-mode/state.json && echo "Ralph Mode is ACTIVE"
```

---

## When Ralph Mode is Active

### Step 1: Read the State

```bash
cat .ralph-mode/state.json
```

Important fields:
- `iteration`: Current iteration number
- `max_iterations`: Maximum allowed (0 = unlimited)
- `completion_promise`: Text to output when genuinely done
- `started_at`: When the loop started
- `mode`: `single` or `batch`
- `current_task_index`: Only in batch mode
- `model`: The AI model being used

### Step 2: Read the Task

```bash
cat .ralph-mode/prompt.md
```

This contains the task you need to work on.

### Step 3: Check Previous Work

```bash
# See recent changes
git diff HEAD~1

# Or check the history
cat .ralph-mode/history.jsonl
```

### Step 4: Work on the Task

- Make incremental improvements each iteration
- Run tests if applicable
- Fix errors you encounter
- Build on previous work (visible in files and git history)

### Step 5: Check Completion

Are ALL requirements met?

- **YES**: Output `<promise>COMPLETION_PROMISE_VALUE</promise>`
- **NO**: Continue working, iterate again

---

## Batch Mode

When `mode` is `batch`:

- Task list: `.ralph-mode/tasks.json`
- Task files: `.ralph-mode/tasks/*.md`
- Current task: `current_task_index` in `state.json`

After completing one task, the loop automatically moves to the next.

---

## Critical Rules

### ⚠️ Completion Promise

If a completion promise is set:

1. **ONLY** output `<promise>VALUE</promise>` when the task is **GENUINELY COMPLETE**
2. The statement must be **COMPLETELY AND UNEQUIVOCALLY TRUE**
3. **NEVER** lie to exit the loop
4. If stuck, document blockers instead of false promises

### 🔄 Iteration Pattern

Each iteration:
1. The SAME prompt is fed to you
2. Your previous work is visible in files
3. Git history shows your changes
4. You improve incrementally until done

---

## Memory System (mem0-inspired)

Ralph Mode includes a multi-level memory system inspired by [mem0](https://github.com/mem0ai/mem0) that persists knowledge across iterations.

### Memory Levels

| Level | Purpose | Persistence |
|-------|---------|-------------|
| **Working** | Current iteration scratch notes | Cleared each iteration |
| **Episodic** | Per-iteration summaries (what happened, what changed) | Survives across iterations |
| **Semantic** | Extracted facts, patterns, relationships | Survives across tasks |
| **Procedural** | Learned workflows ("to fix X, run Y then Z") | Long-term |

### Memory Categories

`file_changes`, `errors`, `decisions`, `blockers`, `progress`, `patterns`, `dependencies`, `test_results`, `environment`, `task_context`

### How Memory Works in the Loop

Each iteration automatically:
1. **Clears** working memory (fresh start)
2. Runs the AI with full context including Memory Bank
3. **Extracts** episodic memories from output (files changed, errors, test results)
4. **Extracts** semantic facts (dependencies, decisions, fix patterns)
5. **Decays** old memory scores (recency bias)
6. **Promotes** frequently-accessed episodic memories to semantic

### Using Memory from CLI

```bash
python3 ralph_mode.py memory stats       # View memory bank statistics
python3 ralph_mode.py memory show        # Display formatted memory for context
python3 ralph_mode.py memory search "auth login"  # Search by relevance
python3 ralph_mode.py memory add "project uses React" --category dependencies
python3 ralph_mode.py memory extract     # Extract from last output
python3 ralph_mode.py memory decay       # Apply temporal decay
python3 ralph_mode.py memory promote     # Promote episodic → semantic
python3 ralph_mode.py memory reset       # Clear all memories
```

---

## File Editing Best Practices

### BEFORE Editing

1. **Read first** — always read the file (or relevant section) before editing.
2. **Check git** — run `git diff` to see if a previous iteration already made changes.
3. **Check Memory Bank** — search for prior edits to the same file.

### WHEN Editing

1. Use precise, targeted edits — replace only the lines that need to change.
2. Include enough context (3+ surrounding lines) to anchor the edit uniquely.
3. Preserve whitespace, indentation, and coding style.
4. Never guess at file contents — always read before editing.

### AFTER Editing

1. **Verify** — read the modified section back to confirm correctness.
2. **Test** — run relevant tests or linters if available.
3. **Record** — note what you changed in progress.

### Common Mistakes

- Editing without reading first (stale content).
- Remaking changes that already exist from a previous iteration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sepehrbayat/copilot-ralph-mode](https://github.com/sepehrbayat/copilot-ralph-mode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
