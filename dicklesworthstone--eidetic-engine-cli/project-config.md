---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust codebase.
---

# AGENTS.md — ee (Eidetic Engine CLI)

> Guidelines for AI coding agents working in this Rust codebase.

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

**The default branch is `main`.** All work happens on `main` — commits, PRs, feature branches all merge to `main`. Never reference `master` in code or docs. If you see `master` anywhere, it's a bug that needs fixing.

---

## RULE NUMBER 2: NO WORKTREES. EVER. NO EXCEPTIONS.

**`git worktree add` is ABSOLUTELY FORBIDDEN. Period.**

You may **NEVER** create a git worktree for any reason. Not to "verify a build in isolation," not to "stage a push," not to "test a rebase," not to "compare two commits," not to "run a parallel cargo build." **NEVER.**

The user has been burned by agents leaving stray worktrees littering the filesystem with detached HEADs, abandoned rebases, and orphaned commits. This wastes hours of recovery work and risks losing real code.

**HARD CONSTRAINTS:**

- All work happens in the single canonical checkout at `/data/projects/eidetic_engine_cli` on the `main` branch.
- **Never run `git worktree add ...`.**
- **Never create or work on a feature branch.** All commits land directly on `main`.
- **Never run `git rebase` interactively or otherwise.** If you think you need to rebase, you do not.
- **Never run `git checkout <other-ref>`** to detach the HEAD or move off `main`. The only acceptable reset of HEAD is `git pull --rebase origin main` to sync with the remote, and even that goes onto `main`.
- **Never run `git stash`** to "park" changes — commit them or discard them properly. Stashing is how work gets lost.
- If your tooling (codex's `/review`, an agent script, anything) wants to spawn a worktree, **disable it immediately or work around it.** A worktree-using helper is broken; do not use it.

**ENFORCEMENT:**

If you see a stray worktree on this host (anything other than `/data/projects/eidetic_engine_cli` itself in `git worktree list`), the orchestrator will:

1. Cherry-pick any unique commits from the stray worktree onto `main`.
2. `git worktree remove --force <path>` every stray worktree.
3. Find which agent created it and dispatch a corrective prompt.

Repeat offenders are killed and replaced.

**NO WORKTREES. NO EXCEPTIONS. THIS IS NOT NEGOTIABLE.**

---

## Toolchain: Rust & Cargo

We only use **Cargo** in this project, NEVER any other package manager.

- **Edition:** Rust 2024 (nightly required — see `rust-toolchain.toml`)
- **Dependency versions:** Explicit versions for stability
- **Configuration:** Cargo.toml only (single binary crate with library surface in the same package; not a workspace in phase 0)
- **Unsafe code:** Forbidden (`#![forbid(unsafe_code)]`)

### Forbidden Dependencies (Hard Rule, Audited By CI)

These are **not allowed** anywhere in the dependency tree. If a transitive dependency pulls one in, the feature must be disabled or the dep quarantined behind an explicit adapter with a removal plan.

| Crate | Reason |
|-------|--------|
| `tokio`, `tokio-util` | Asupersync is the runtime; no Tokio. |
| `async-std`, `smol` | Same — only Asupersync. |
| `rusqlite` | FrankenSQLite via SQLModel is the storage layer. |
| `sqlx`, `diesel`, `sea-orm` | SQLModel is the ORM. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/eidetic_engine_cli](https://github.com/Dicklesworthstone/eidetic_engine_cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
