---
trigger: always_on
description: - Never run `git reset --hard`, `git checkout -- <path>`, or a tree-wide `git stash` in a checkout
---

# AGENTS.md

## Danger: commands agents must not run

- Never run `git reset --hard`, `git checkout -- <path>`, or a tree-wide `git stash` in a checkout
  whose state matters; they overwrite tracked work or hide shared worktree-family state.
- Never run `rm -rf`, `find ... -delete`, or equivalent recursive cleanup through an empty,
  unresolved, relative, root, home, workspace, or otherwise unproven target path.
- Never remove or move a linked worktree with `rm -rf` or `mv`, and never hand-delete
  `.git/worktrees/*`; use `git worktree remove` / `move` / `prune` / `repair`.
- Never run sandboxed `git switch --track` or `git branch -D` directly; wrap the complete command
  with `utils/git-sandbox-guard.sh --repo <root> -- <git command>`.
- Never delete or move a full-clone folder until its working tree, stashes, local refs, and registered
  worktrees prove that it contains no unique or depended-on state.
- Never run `validate.sh` or `test/*.sh` from a linked worktree or a full clone whose state matters;
  run mutation-heavy gates only in a separate disposable full clone.

Read [`WORKTREE-SAFETY.md`](WORKTREE-SAFETY.md) for the rationale, recovery paths, and safe patterns.

> **Safety and warranty:** XYZ Forge is provided **“AS IS,” without warranty**, under the applicable
> license. Coding models may choose commands through their own runtimes and safety controls, outside
> the intended harness workflow. XYZ Forge cannot guarantee model behavior or data integrity; maintain
> tested, independent backups and follow industry-standard backup and recovery practices.

Read `ROUTER.md` first for startup order and canonical files.

Read `GUIDING-PRINCIPLES.md` for the product north stars.

Read `PROJECT/PDDA.md` when the task touches project docs, `ROADMAP.md`, or `CHANGELOG.md`.

Read `HARNESS-MODELS-REGISTRY.md` for evaluated agent harnesses, model compatibility grades, and CLI flags.

Read `TESTS-RESULTS/README.md` for committed test artifacts, telemetry receipts, and benchmark logs.

## Runtime default

Entry-point shims run their **Python** implementation by default (`XYZ_PYTHON` unset → Python). To
force the legacy Bash path for a single run, prefix it with `XYZ_PYTHON=0`; for a whole session,
`export XYZ_PYTHON=0`. The Bash body stays inline in every shim, so the opt-out is always available.

## What this file owns

This file is the behavioral playbook for work in this repo: decision quality, reversibility, blast
radius, planning shape, and proof.

Do not restate routing, roadmap, changelog, or active-doc contracts here. Those live in
`ROUTER.md` and `PROJECT/PDDA.md`.

After merging any PR into `development`, run `python3 utils/py/wave_reconcile.py --pr <N>` before
ending the task — the reconciler is single-command but nothing triggers it for you; `pdda.sh
issue-doc-sync` is the deterministic drift detector when in doubt.

Maintainer-only workflow defaults (branch discipline, express-to-development, fresh-clone-per-task)
live in `SOP.md` → "Opinionated SOPs" — optional for downstream users, binding for us. That section
is a standing carve-out from the "do not create new git branches automatically" rail: it
pre-authorizes one `feat/`/`fix/` branch per fresh task clone, nothing more.

## Operating principles

### 1. Lead with the line that survives skimming

Your first sentence gives the verdict, current state, or call. No setup first.

### 2. Make the bet explicit before acting

State the assumption, tradeoff, and failure mode that matter before you commit to a path. If a future
reader could not say "that assumption was wrong," you have not made the real bet legible yet.

### 3. Use one reversibility scale

Consequential changes get a read on the shared scale: **Easy / Costly / One-way door**, with one line
of why. If undoing it would take more than a day of focused work, it is at least Costly. Costly
changes need a rollback path. One-way doors need explicit confirmation before proceeding.

### 4. Size the blast radius before changing shared surfaces

Before a refactor, schema change, dependency bump, coordination-kernel change, or relay-containment
change, say what ripples, what might break, and who notices. A change you cannot size is not ready.

### 5. One plan, one ordered list

When you give executable steps, put them in one numbered list in execution order. Keep verification
inline (`-> expect ...`). Do not scatter action items across prose.

### 6. Verified beats plausible

Do not claim success without the relevant test, script, or observable proof. If verification was
skipped or failed, say that plainly and include the result.

An uncommitted `provenance.jsonl` is not proof (GH-430). Any run cited as evidence in an issue, PR,
ROADMAP entry, or decision record must have its `provenance.jsonl` committed in the same PR — a path
you merely ran and can no longer show counts as no claim at all.

### 7. Record only consequential bets

If a change is Costly, One-way door, or assumption-heavy, record the bet in `CHANGELOG.md` per
`PROJECT/PDDA.md`. Below that threshold, skip the ritual.

### 8. Stay quiet on trivial work

Most edits are small and reversible. Do not manufacture ceremony for a rename, typo fix, or other
local change.

## Repo-specific rails


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HiQS-Labs/XYZ-forge](https://github.com/HiQS-Labs/XYZ-forge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-27 -->
