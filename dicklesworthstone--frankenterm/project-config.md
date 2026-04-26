---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — frankenterm

> Guidelines for AI coding agents working in this Rust codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 0.5: NEVER REMOVE crates/frankenterm-core

**THE crates/frankenterm-core CRATE IS A PERMANENT PART OF THIS WORKSPACE. REMOVING IT IS FORBIDDEN.**

This rule exists because agents have REPEATEDLY deleted this entire crate (860+ files, 624K+ lines of code) under the guise of "refactoring" or "consolidation." It has happened at least 3 times and caused enormous damage each time.

**ABSOLUTE PROHIBITIONS:**
- Do NOT delete, remove, or "consolidate" `crates/frankenterm-core/`
- Do NOT stage deletions of files in `crates/frankenterm-core/`
- Do NOT create commits that remove files from `crates/frankenterm-core/`
- Do NOT propose, plan, or suggest removing this crate for any reason
- The words "remove frankenterm-core", "consolidate into main crate", or "crate removal" should NEVER appear in any commit message

**If you see frankenterm-core files missing from disk but present in git:**
- This is the EXACT bug that keeps happening — an agent on another machine deleted them
- Do NOT commit the deletions. Run `git checkout -- crates/frankenterm-core/` to restore them
- Report the issue to the user

**This crate is not optional. It is not redundant. It is not "extracted." It IS the core.**

---

## AGENT MAIL (am) PROCESS PROTECTION — DO NOT TOUCH

**NEVER run any of these commands:**
- `am service restart` / `am service stop`
- `am doctor fix` / `am doctor repair` / `am doctor reconstruct`
- `kill` targeting any `am`, `am serve-http`, or `mcp-agent-mail` process

The `am serve-http` process is a **shared singleton** that all agents depend on. Restarting or killing it disrupts every other agent. Multiple agents running `am service restart` create a **restart cascade** that makes the service permanently unavailable.

**If `am` commands fail or the API is unreachable:** retry once after a few seconds, then proceed with your work WITHOUT agent-mail. Do NOT attempt to diagnose, repair, or restart the service.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

---

## RULE NUMBER 2: ABSOLUTELY NO GIT WORKTREES

**GIT WORKTREES ARE STRICTLY FORBIDDEN IN THIS REPO. DO NOT USE THEM.**

1. **Never run:** `git worktree add`, `git worktree remove`, `git worktree prune`, or any related worktree command.
2. **No exceptions by convenience:** Do not create temporary directories, detached worktrees, or parallel checkout trees for agent work.
3. **Use branches in the main repo only:** All agent work must happen on normal branches in the primary checkout.
4. **If you discover existing worktrees:** stop and report them, then rescue useful commits back into normal branches.

---

## FIRST-TIME SETUP ON ANY MACHINE

Run this after cloning or on any machine where agents work on frankenterm:
```bash
bash scripts/install-hooks.sh
```
This installs a pre-commit guard that blocks mass deletions and any deletion of `crates/frankenterm-core/`.

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

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankenterm](https://github.com/Dicklesworthstone/frankenterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
