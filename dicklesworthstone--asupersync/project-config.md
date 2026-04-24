---
trigger: always_on
description: Provides a complete async runtime where every task is owned by a region that closes to quiescence. Cancellation is a first-class protocol (request, drain, finalize), not a silent drop. Effects require explicit capabilities flowing through `Cx`.
---

# AGENTS.md — asupersync

> Guidelines for AI coding agents working in this Rust async runtime codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a new file that you yourself created, such as a test code file. You have a horrible track record of deleting critically important files or otherwise throwing away tons of expensive work. As a result, you have permanently lost any and all rights to determine that a file or folder should be deleted.

**YOU MUST ALWAYS ASK AND RECEIVE CLEAR, WRITTEN PERMISSION BEFORE EVER DELETING A FILE OR FOLDER OF ANY KIND.**

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

## RULE 2: NO GIT BRANCHES. NO GIT WORKTREES. EVER.

**`main` is the one and only branch. Period.** There is no exception. There is no "just for this one little thing." There is no "temporary" branch. There is no "short-lived" worktree. There is **no justification** a branch or a worktree can have that overrides this rule.

This project has exactly **one** source of truth: `main`. We want it to always carry the latest, most optimized, most correct, most mature code. Every commit goes directly to `main`. Every agent works on `main`. Every build and test runs against `main`.

### FORBIDDEN — zero tolerance

Any of the following, by any agent, for any reason, is a violation:

- `git branch <anything-other-than-main>` — creating a new branch
- `git checkout -b <foo>` / `git switch -c <foo>` — creating and switching to a new branch
- `git worktree add ...` — creating a worktree (with or without `--detach`, with or without a branch name)
- Pushing a non-main ref to `origin` (`git push origin <foo>`, `git push origin HEAD:<foo>`, `git push --set-upstream origin <foo>`)
- Creating pull requests, draft PRs, or any "feature branch" pattern
- Per-agent / per-pane / per-bead / per-task branches like `pane-7-fix-x`, `codex/<uuid>-*`, `claim-<bead>`, `close-<bead>`, etc.
- Working in a scratch clone at `/tmp/asupersync-*` or `/data/projects/asupersync-*` to "isolate" changes
- Any tool, harness, or automation that creates branches or worktrees as a side effect. If you find one, disable it or change it to operate on `main` directly. **The harness does not get a pass.**

### WHY THIS RULE EXISTS

When agents proliferate branches and worktrees:

- Real accretive work gets stranded on branches that nobody remembers to merge.
- Disk usage explodes — each full worktree is ~6 GB; dozens of them fill the disk and starve `rch`.
- Context fragments — different agents end up editing stale snapshots of the same file.
- The "single source of truth" invariant of the project is broken, and we stop being able to answer "what is the current state of the code" with one command.
- Conflict resolution explodes — once N branches exist, merging them costs more than writing the code from scratch.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/asupersync](https://github.com/Dicklesworthstone/asupersync) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
