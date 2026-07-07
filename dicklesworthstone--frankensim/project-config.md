---
trigger: always_on
description: > Guidelines for AI coding agents working in this Rust simulation, geometry,
---

# AGENTS.md - FrankenSim

> Guidelines for AI coding agents working in this Rust simulation, geometry,
> optimization, and rendering codebase.

---

## RULE 0 - THE FUNDAMENTAL OVERRIDE PREROGATIVE

If I tell you to do something, even if it goes against what follows below, YOU
MUST LISTEN TO ME. I AM IN CHARGE, NOT YOU.

---

## RULE NUMBER 1: NO FILE DELETION

**YOU ARE NEVER ALLOWED TO DELETE A FILE WITHOUT EXPRESS PERMISSION.** Even a
new file that you yourself created, such as a test file. You must always ask and
receive clear, written permission before deleting a file or folder of any kind.

---

## Irreversible Git & Filesystem Actions - DO NOT EVER BREAK GLASS

1. **Absolutely forbidden commands:** `git reset --hard`, `git clean -fd`,
   `rm -rf`, or any command that can delete or overwrite code/data must never be
   run unless the user explicitly provides the exact command and states, in the
   same message, that they understand and want the irreversible consequences.
2. **No guessing:** If there is any uncertainty about what a command might
   delete or overwrite, stop immediately and ask the user for specific approval.
3. **Safer alternatives first:** When cleanup or rollbacks are needed, use
   non-destructive inspection first: `git status`, `git diff`, backups, or
   explicit hand-written patches.
4. **Mandatory explicit plan:** Even after explicit user authorization, restate
   the command verbatim, list exactly what will be affected, and wait for
   confirmation that your understanding is correct.
5. **Document the confirmation:** When running any approved destructive command,
   record the user text that authorized it, the command actually run, and the
   execution time in your final response.

---

## Git Branch: ONLY Use `main`, NEVER `master`

When this directory is a git repository, the default branch is `main`.

- All work happens on `main`.
- Never create, switch to, or push feature branches unless the user explicitly
  overrides this file.
- Never reference `master` in code or docs. If you see it, treat it as a bug.
- If the remote also needs a legacy `master` ref, synchronize it from `main`
  only when the user or project automation asks for that exact operation.

---

## RULE 2: NO GIT BRANCHES. NO GIT WORKTREES. EVER.

`main` is the one and only branch. There is no "temporary" branch, no per-agent
branch, no per-task branch, and no scratch worktree.

### FORBIDDEN

- `git branch <anything-other-than-main>`
- `git checkout -b <foo>` or `git switch -c <foo>`
- `git worktree add ...`
- Pushing non-main refs to `origin`
- Creating pull requests or draft PRs from feature branches
- Working in scratch clones at paths like `/tmp/frankensim-*`,
  `/data/projects/frankensim-*`, or `~/projects/frankensim-*` to isolate work
- Using any tool or harness that creates branches or worktrees as a side effect

### WHAT YOU DO INSTEAD

- Commit directly to `main` when the user asks for commits and the work is ready.
- Keep unfinished work in the working tree.
- Coordinate through Agent Mail reservations when multiple agents are active.
- Use Beads issue IDs and file reservations as the isolation mechanism, not git
  branches.
- If another agent changed files, do not revert or stash their work. Work with
  the current tree.

---

## Project Truth Sources

This repository is currently plan-first. The authoritative design document is:

- `COMPREHENSIVE_PLAN_FOR_FRANKENSIM.md`

Read it before broad design work. It defines the Decalogue, architecture,
roadmap, crate atlas, Gauntlet verification program, performance targets, and
flagship pipelines. This `AGENTS.md` is the operating contract for agents; the
plan is the technical constitution.

If a `README.md`, crate `CONTRACT.md`, `TESTING.md`, or `.beads/` directory is
added later, read the relevant files before making nontrivial edits.

---

## FrankenSim - This Project

FrankenSim is intended to be a single memory-safe Rust continuum for:

- computational geometry
- certified representation conversion
- physics simulation
- adjoint and derivative-free optimization
- uncertainty quantification
- rendering and scientific visualization
- replayable design ledgers and agent-native orchestration

The mission is not to wrap legacy CAD/FEM/CFD/optimization tools. The mission is
to build one typed algebra where geometry, fields, operators, derivatives, error
bounds, budgets, provenance, and cancellation travel together through every
layer.

### Load-Bearing Principles

- **Pure Rust first:** runtime dependencies are limited to `std`, asupersync,
  FrankenSQLite, FrankenNumpy, FrankenTorch, FrankenScipy, FrankenPandas, and
  FrankenNetworkx.
- **Memory safety:** default code must be safe Rust. `unsafe` is allowed only in
  narrow audited leaf modules where there is no safe alternative.
- **Determinism:** deterministic mode must be bit-stable across runs and thread
  counts on the same ISA wherever the plan claims it.
- **Cancellation-correct compute:** every unit of compute runs under explicit
  asupersync scopes and checks cancellation at bounded tile boundaries.
- **Budgets first:** accuracy, time, memory, and capability budgets are explicit
  values, not comments.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Dicklesworthstone/frankensim](https://github.com/Dicklesworthstone/frankensim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
