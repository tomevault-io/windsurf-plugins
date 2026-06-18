---
trigger: always_on
description: Use when editing files in a git project and you need safe, traceable, reversible changes. Creates a separate .gitmem repository to track every agent edit independently from the main git history. Triggers for: undo/rollback agent edits, compare file versions, restore previous states, create checkpoints before risky changes, or inspect recent agent modifications. Distinguishes from normal git operations - use this for agent edit safety, not for standard git workflows like branching, merging, or pu
---


# Gitmem Safe Editing

Treat GitMem as an **agent editing safety layer**, not as user-facing version control and not as general memory.

Core outcome:

1. record every agent file edit immediately
2. make each edit easy to diff and roll back
3. give the user a safe way to undo or time-travel when the agent starts degrading the codebase

## Operating model

Use two repositories side by side:

- the user's main repository in `.git/`
- an independent agent edit log in `.gitmem/.git`

GitMem must remain isolated from the main repository.
Do not reuse `.git/` for GitMem history.
Do not introduce databases, mcp servers, or external storage.

Expected layout:

```text
project/
  src/
  docs/
  .git/
  .gitmem/
    .git/
```

## Auto-initialization on skill activation

When this skill is triggered, **you MUST immediately execute these commands** using the Bash tool:

### Step 1: Check initialization status

Run this command to check if GitMem is initialized:

```bash
test -d .gitmem/.git && echo "initialized" || echo "not initialized"
```

**You MUST run this command and check the output.** If it outputs "not initialized", proceed to Step 2.

### Step 2: Auto-initialize if needed

If GitMem is not initialized, **MUST run these commands**:

```bash
mkdir -p .gitmem
git init .gitmem
git --git-dir=.gitmem/.git --work-tree=. config user.email "agent@gitmem.local"
git --git-dir=.gitmem/.git --work-tree=. config user.name "GitMem Agent"
```

Then inform the user: "GitMem initialized at `.gitmem/`"

### Step 3: Start auto-watch mode

**MUST run these commands** to start the file watcher:

```bash
# Check if watch is already running, if not start it in background
if ! pgrep -f "gitmem-watch" > /dev/null 2>&1; then
  # Find the watch script
  WATCH_SCRIPT="$(find ~/.claude/skills -name 'gitmem-watch' 2>/dev/null | head -1)"
  if [ -x "$WATCH_SCRIPT" ]; then
    nohup "$WATCH_SCRIPT" > /tmp/gitmem-watch.log 2>&1 &
    disown
  fi
fi
```

Then inform the user: "GitMem auto-watch started - all file changes will be automatically tracked"

**Note:** If `inotify-tools` (Linux) or `fswatch` (macOS) is not installed, inform the user: "Auto-watch requires `inotify-tools` (Linux) or `fswatch` (macOS). Please install for automatic tracking, or manually commit changes using `gitmem commit <file>`."

## Required edit workflow

For every edited file, follow this sequence:

```text
read file
edit file
commit that file to gitmem immediately
```

Never batch multiple edits and commit only at the end.
Never edit repeatedly without checkpointing history in `.gitmem`.

After changing one file, create a GitMem commit for that file before editing the next file.

## Command setup

Use GitMem commands through the separate git dir and the project worktree:

```bash
git --git-dir=.gitmem/.git --work-tree=. <command>
```

Run commands from the project root unless the user explicitly specifies another path.

## Core operations

### 1. commit an edited file

Use this after each file change.

```bash
git --git-dir=.gitmem/.git --work-tree=. add -- <file>
git --git-dir=.gitmem/.git --work-tree=. commit -m "agent(edit): <file>\n\nreason: <why the change was made>"
```

Rules:

- add only the current file
- never use `git add .`
- explain the reason in the commit message
- keep each commit scoped to the current file change

Commit message format:

```text
agent(edit): <file>

reason: <why the change was made>
```

Example:

```text
agent(edit): src/api.ts

reason: add retry logic
```

### 2. inspect file history

Use when the user asks what changed before, which version was better, or when the agent is unsure how the file evolved.

```bash
git --git-dir=.gitmem/.git --work-tree=. log -- <file>
```

Summarize at least:

- commit id
- commit message
- time

### 3. diff two states

Use when deciding whether to keep, revert, or compare changes.

```bash
git --git-dir=.gitmem/.git --work-tree=. diff <commit_a> <commit_b> -- <file>
```

### 4. roll back one file

Use when the user says the earlier file version was better or asks to restore one file.

```bash
git --git-dir=.gitmem/.git --work-tree=. checkout <commit> -- <file>
```

After rollback, commit the restored file to GitMem with a clear reason so the recovery itself is traceable.

### 5. create a safe checkpoint

Use when the user is satisfied, a feature reaches a stable state, or before risky edits.

```bash
git --git-dir=.gitmem/.git --work-tree=. tag gitmem-checkpoint-<name>
```

Checkpoint names should be short and descriptive, such as:

```text
gitmem-checkpoint-feature-working
gitmem-checkpoint-tests-passing
```

### 6. undo the last agent change

Use when the user says the most recent agent edit broke something and wants the last GitMem step removed quickly.

```bash
git --git-dir=.gitmem/.git --work-tree=. reset --hard HEAD~1
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacazjx/GitMem-Skill](https://github.com/jacazjx/GitMem-Skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
