---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — meta_skill

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## RULE 2 - BEADS/BR DATABASE SAFETY (ABSOLUTE)

**SQLite + WAL = DATA LOSS RISK.** The beads system uses SQLite with Write-Ahead Logging. Improper handling WILL destroy uncommitted data.

**Note:** `br` (beads_rust) is non-invasive—it has NO daemon and NEVER executes git commands automatically. You must manually run `git add .beads/ && git commit` after `br sync --flush-only`.

### BEFORE Running Parallel Agents That Use `br`

You MUST complete this checklist BEFORE launching any parallel agents/subagents that will run `br update`, `br create`, or any br write operations:

```bash
# 1. Check for stale br processes
lsof .beads/beads.db 2>/dev/null | wc -l
# Should be 0 or 1. If more, wait for other agents to finish.

# 2. Run doctor checks
br doctor 2>&1 | grep -E "(✖|FAIL|Error)"
# If any failures, STOP. Ask user.

# 3. Verify sync status
br sync --status 2>&1
# Check if DB and JSONL are in sync
```

**If ANY check fails: STOP and ask the user. Do NOT proceed.**

### DURING Parallel Agent Work

- **FLUSH AFTER EACH BATCH**: Run `br sync --flush-only` after each agent completes
- **COMMIT PERIODICALLY**: Run `git add .beads/ && git commit` to persist changes
- **Monitor for failures**: If any `br` command fails, STOP ALL AGENTS

### FORBIDDEN ACTIONS (Will Destroy Data)

1. **NEVER kill processes holding `.beads/beads.db`**
   - The WAL may contain uncommitted transactions

2. **NEVER delete or modify these files manually:**
   - `.beads/beads.db`
   - `.beads/beads.db-wal`
   - `.beads/beads.db-shm`

3. **NEVER run `rm .beads/beads.db*` to "fix" issues**

### When `br sync` Fails

**STOP IMMEDIATELY. Ask the user.** Do not attempt to:
- Kill processes
- Delete database files
- Delete WAL files

The correct response is: "br sync is failing with [error]. I need your guidance before proceeding."

### Recovery After Disaster

If data was lost, check these locations for recovery:
- Agent output files: `/tmp/claude/-data-projects-*/tasks/*.output`
- Git history: `.beads/issues.jsonl` is git-tracked and may have recoverable versions

---

## Irreversible Git & Filesystem Actions — DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`, `rm -rf`, or any command that can delete or overwrite code/data must never be run unless the user explicitly provides the exact command and states, in the same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might delete or overwrite, stop immediately and ask the user for specific approval. "I think it's safe" is never acceptable.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, request permission to use non-destructive options (`git status`, `git diff`, `git stash`, copying to backups) before ever considering a destructive command.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate the command verbatim, list exactly what will be affected, and wait for a confirmation that your understanding is correct. Only then may you execute it—if anything remains ambiguous, refuse and escalate.
5. **Document the confirmation:** When running any approved destructive command, record (in the session notes / final response) the exact user text that authorized it, the command actually run, and the execution time. If that record is absent, the operation did not happen.

---

## Git Branch: ONLY Use `main`, NEVER `master`

**The default branch is `main`. The `master` branch exists only for legacy URL compatibility.**

- **All work happens on `main`** — commits, PRs, feature branches all merge to `main`
- **Never reference `master` in code or docs** — if you see `master` anywhere, it's a bug that needs fixing
- **The `master` branch must stay synchronized with `main`** — after pushing to `main`, also push to `master`:
  ```bash
  git push origin main:master
  ```

**If you see `master` referenced anywhere:**
1. Update it to `main`
2. Ensure `master` is synchronized: `git push origin main:master`

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (nightly required)
- **Minimum Rust version:** 1.85
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Single-crate project (not a workspace)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/meta_skill](https://github.com/Dicklesworthstone/meta_skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
