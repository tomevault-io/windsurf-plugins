---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

AI Triad Research — multi-perspective research platform for AI policy/safety literature. Berkman Klein Center, 2026. Two sibling repos: this one (code) and `../ai-triad-data` (structured JSON data, ~410 MB).

## Build & Test Commands

Build and test commands are role-specific — see the owning subtree's `AGENTS.md` (they load automatically when you work in that scope):

- **PowerShell module / Pester / manifest** → `scripts/AGENTS.md`
- **Taxonomy Editor / poviewer / summary-viewer (npm, vitest, tsc)** → `taxonomy-editor/AGENTS.md`
- **Debate engine (vitest)** → `lib/debate/AGENTS.md`
- **CI pipeline (`ci.yml` jobs)** → `operations/devops/AGENTS.md`

## Architecture

### Two-Repo Split

Code lives here; data lives in `../ai-triad-data`. The file `.aitriad.json` maps relative paths to data directories. Override with `$env:AI_TRIAD_DATA_ROOT`. Priority: env var > `.aitriad.json` > monorepo fallback.

### Orca Overlay Repo

Orca config files (`.orca.yaml`, `AGENTS.md`, `.orca/` directory) live in a **separate overlay repo** stored at `.orca-git/`. This keeps Orca infrastructure private while the main project repo stays public.

- **`git` commands** operate on the main project repo
- **`ogit` commands** (alias for `git --git-dir=.orca-git --work-tree=.`) operate on the overlay
- **Never `git add` or `git commit`** files tracked by the overlay: `.orca.yaml`, `.orca/`, `.orca-gitignore`, and every **nested** `AGENTS.md`
- Run `ogit` from the repo root — `.orca-git` is not visible from subdirectories
- A **new** nested `AGENTS.md` needs `ogit add -f` — the overlay whitelist alone does not stage it

**Creating a role/instance?** After `create_role`/`create_instance`, the generated nested `AGENTS.md` is tracked by **neither** repo until you overlay-track it — do this **before** your next commit:

1. `ogit add -f <new-role>/AGENTS.md`  (the overlay whitelist alone does not stage a *new* nested file — t/1971)
2. `sh .githooks/agent-file-owner.sh --audit`  → expect **clean**
3. Commit normally. **Never** `--no-verify` past the audit — that strands the file with no backing repo (F3 orphan; Pattern #146). If the audit instead flags a `.worktrees/<name>/AGENTS.md`, that is a worktree checkout of a main-tracked file — do **not** ogit-add it (t/2205).

**Which repo owns an `AGENTS.md`? Don't recall it — ask (t/2080):**

```
sh .githooks/agent-file-owner.sh --path <file>    # → main | overlay | NEITHER
```

The rule is a predicate, not a list: a file is **main-repo-tracked iff a public-repo consumer needs it without the overlay**. Today that is exactly two files — this root `AGENTS.md` (commit it with **`git`**, not `ogit`) and `operations/devops/azure/AGENTS.md`. Every other `AGENTS.md` is overlay-only.

The two sets are **disjoint by construction**: the code repo's allowlist lives in `.gitignore`, the matching re-exclusions in `.orca-gitignore`. Both `AGENTS.md` above were previously tracked in *both* repos and had silently diverged. The pre-commit hook runs `agent-file-owner.sh --audit` on every commit and **refuses** one that re-creates a double-track — or that leaves a nested `AGENTS.md` tracked by **neither** repo, the state that left two role files with a single unbacked copy on one machine.

### Shared-Checkout Commit Guard (git pre-commit hook)

The fleet shares one `main` checkout, so a commit made **directly on its `main` branch** strands work local-only and diverges `main` from `origin` (t/1926). A committed pre-commit hook (`.githooks/pre-commit`) **refuses** such commits. It also **refuses a commit on a detached HEAD inside a worktree** (t/2009, orphaned-commit guard) — so `/land-from-worktree` is now **branch-first** (`git worktree add -b <branch> ...`). Worktree commits on a **named branch**, non-`main` branches, and `--no-verify` are allowed, so landing is never blocked. Git does not auto-run committed hooks — **enable once per checkout**:

```
git config core.hooksPath .githooks
```

The hook is self-documenting (see its header comment). Owner / emergency override: `git commit --no-verify`.

**Feature work is worktree-only; the shared checkout stays on `main`.** A companion `.githooks/post-checkout` hook **warns** (advisory, non-blocking) the moment the shared tree's HEAD leaves `main` for a feature branch — pointing you to `git worktree add -b <branch>` instead (t/2209). It is silent inside linked worktrees (the correct home for feature work) and on `main`. Don't do feature work on the shared tree: it strands unpushed commits and turns the next `git add -A` into a cross-role sweep.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jpsnover/ai-triad-research](https://github.com/jpsnover/ai-triad-research) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-20 -->
