---
trigger: always_on
description: - This file is the top-level authority for work inside `SillyTavern-AstraProjecta`.
---

# AstraProjecta Agent Guide

## Purpose

- This file is the top-level authority for work inside `SillyTavern-AstraProjecta`.
- AstraProjecta is a third-party SillyTavern UI extension, not a SillyTavern core fork and not a server plugin.
- This repository's current working-tree shape is the active AstraProjecta baseline. Keep documentation aligned to what exists here now instead of preserving speculative alternate layouts.
- Keep architectural rules grounded in this repository's current source, tests, and documentation instead of speculative alternate layouts.

## Owned Paths / Responsibilities

- Own everything under this extension directory only.
- Never modify SillyTavern core files outside this extension repository for AstraProjecta work.
- Use this repository to define AstraProjecta-owned hosts, runtime contracts, adapters, features, styles, and documentation.
- Keep `AGENTS.md` files current whenever a folder gains stable responsibilities, SillyTavern touchpoints, or non-obvious constraints.

## Repository Workflow

- Treat the repository root as the canonical active working tree for AstraProjecta implementation, testing, build, commit, and handoff.
- Run agent tooling, `git status`, `npm run format`, `npm run test:run`, `npm run build`, commits, and pushes from the repository root only.
- For ordinary feature, bugfix, documentation, and refactor work, create or switch to a short-lived branch from the repository root before editing, typically with `git switch -c codex/<topic>` or `git switch <existing-branch>`.
- Use the canonical repository root as the active working tree for that task branch; do not keep the root pinned to `main` merely to preserve isolation.
- Do not maintain a second writable checkout for the same repository under another path. If another filesystem entry is needed for convenience, it must be a symlink or other pointer to this canonical working tree instead of a separate clone.
- Use `git worktree` only for explicit parallel or high-risk isolation needs; treat it as a supplement to the root short-lived-branch workflow rather than the default way to isolate work.
- If an auxiliary worktree is used, do not leave its branch locked away from the repository root when the user is expected to continue from the root. Before final handoff, either move the changes back to a root-usable branch and detach or remove the auxiliary worktree, or explicitly state that the branch is checked out only at the worktree path and cannot be switched to from the root until the worktree releases it.
- Maintainers may keep machine-specific instructions in ignored `AGENTS.local.md`, using `AGENTS.local.example.md` as the public-safe template.

## Public vs Local Agent Instructions

- `AGENTS.md` files are public, tracked repository guidance for general architecture, workflow, ownership, and contributor rules.
- `AGENTS.local.md` files are private, ignored local notes for personal paths, private reference repositories, machine-specific workflows, personal authorization phrases, or other user-specific instructions.
- Keep private or personally customized content out of tracked `AGENTS.md` files. If a local rule becomes useful to contributors generally, move a public-safe version into the relevant tracked `AGENTS.md`.

## Git Workflow

- These git rules apply only inside this AstraProjecta repository and do not authorize git actions in a parent SillyTavern checkout.
- Before any commit, inspect `git status`, check whether the working tree includes unrelated changes, and stop for scope clarification instead of guessing when the commit boundary is ambiguous.
- Before handoff, report the current branch, whether it is ahead of `main`, and whether it appears ready to merge back into `main`.
- When the task appears complete, explicitly state whether the branch is a candidate to merge into `main` and whether it can be deleted after that merge. Reporting merge readiness is allowed; performing `git merge`, `git push`, opening pull requests, or deleting branches still requires explicit authorization.
- Before handoff, run `git worktree list` when any auxiliary worktree was used, and confirm no worktree-owned branch remains checked out in a way that blocks later merge, switch, or cleanup from the canonical root.
- Before any commit, run `npm run format`, `npm run test:run`, and `npm run build` from this repository, and treat any warning or error as a blocker.
- Only proceed to `git add` and `git commit` after formatting, tests, and build finish with zero warnings and zero errors.
- Write the commit message on the user's behalf using a single-topic, readable conventional-style summary or concise imperative summary unless the user requests specific wording.
- Do not push, merge, open pull requests, or delete branches on another contributor's behalf unless that action is explicitly authorized in the current collaboration context.

## Structure Tree

```text
SillyTavern-AstraProjecta/
├─ AGENTS.md                         # repository rules
├─ locales/                          # English catalog source
├─ manifest.json                     # extension manifest
├─ package.json                      # toolchain contract
├─ scripts/                          # repository automation
├─ src/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RivelleDays/SillyTavern-AstraProjecta](https://github.com/RivelleDays/SillyTavern-AstraProjecta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
