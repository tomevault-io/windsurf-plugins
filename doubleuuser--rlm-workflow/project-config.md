---
trigger: always_on
description: Orchestrates the RLM repo workflow end-to-end with phase gates, locked artifacts, addenda, traceability, and automatic bootstrap/upsert of AGENTS/PLANS scaffolding. Trigger phrases: "Implement requirement <run-id>", "Run RLM Phase <N>", "resume requirement", "lock Phase <N>", "verify locks".
---


# RLM Workflow

## Overview

Implement repository work using the canonical RLM process in `.agent/PLANS.md`, with invocation conventions from `.codex/AGENTS.md`. Treat repository artifacts as the source of truth and keep prompts as path-based commands.

## Trigger examples

- `Implement requirement '2026-02-24-add-oauth'`
- `Run RLM Phase 2 for .codex/rlm/2026-02-24-add-oauth/`
- `Resume requirement '2026-02-24-add-oauth' after manual QA`
- `Verify locks for .codex/rlm/2026-02-24-add-oauth/`
- `Lock Phase 3 for run '2026-02-24-add-oauth'`

## Bootstrap preflight (always run first)

Before doing anything else, ensure the repo scaffolding exists and is up to date:

- `.codex/AGENTS.md` contains the RLM Workflow block (managed upsert block)
- `.agent/PLANS.md` contains the canonical workflow block from `references/plans-canonical.md` (managed upsert block)
- `.codex/rlm/` exists
- Any other files created by the installer scripts (`scripts/install-rlm-workflow.ps1` or `scripts/install-rlm-workflow.py`) exist

If any are missing/outdated, run:

```powershell
# Windows PowerShell:
powershell -ExecutionPolicy Bypass -File ./scripts/install-rlm-workflow.ps1 -RepoRoot .

# PowerShell 7+ (pwsh):
pwsh -NoProfile -File ./scripts/install-rlm-workflow.ps1 -RepoRoot .

# Python (Windows/macOS/Linux):
python ./scripts/install-rlm-workflow.py --repo-root .
# or:
python3 ./scripts/install-rlm-workflow.py --repo-root .

# Bash wrapper (macOS/Linux):
bash ./scripts/install-rlm-workflow.sh --repo-root .
```

If script execution isn't possible, perform an equivalent manual bootstrap:

- Create missing directories/files listed above
- Upsert canonical plans from `references/plans-canonical.md` into `.agent/PLANS.md` using managed markers
- Upsert the "RLM Workflow Skill" block into `.codex/AGENTS.md` using managed markers

Then continue with the workflow phases.

## Read Order

1. Read `.codex/AGENTS.md` intro sections for local invocation conventions.
2. Read `.agent/PLANS.md` for canonical phase rules and requirements.
3. If AGENTS wording and PLANS wording differ, follow PLANS (AGENTS declares PLANS canonical) and note the mismatch in the current phase artifact when relevant.

## Trigger Examples

- `Implement requirement '<run-id>'`
- `Run RLM Phase 2 for .codex/rlm/<run-id>/`
- `Create .codex/rlm/<run-id>/02-to-be-plan.md with Coverage and Approval gates`
- `Update tests and lock Phase 4 artifact for this run`

## Invocation Mode

- Single-command mode:
  - On `Implement requirement '<run-id>'`, resolve run folder and execute phases sequentially.
  - Pause only for manual QA sign-off in Phase 5.
- Single-phase mode:
  - On `Run RLM Phase N`, execute only that phase and write only that phase outputs, but only when all required earlier phases are lock-valid.

## Single-Command Contract (Mandatory)

- Resolve run folder at `.codex/rlm/<run-id>/`.
- If run folder or `00-requirements.md` is missing, stop and ask for it. Do not invent requirements.
- **Auto-resume from current state:**
  - **Phase 0 (Requirements):** Confirm `00-requirements.md` exists (user-created starting point). Stop if missing.
  - **Phase 0 (Worktree):** Create/enter an isolated worktree, then execute the run from that worktree.
  - If a phase artifact exists as `DRAFT` or with failing gates, resume that phase.
  - If a phase artifact is missing, create it for the next phase in sequence.
  - Never back-edit locked prior-phase artifacts.
- Execute in order: Phase 0 through Phase 7.
- **For Phase 0 (Worktree Isolation - REQUIRED):**
  - Treat `00-requirements.md` as the starting input for the run (it must already exist).
  - Check if worktree exists at `.worktrees/<run-id>` or configured location.
  - If on main/master branch: require explicit consent or auto-create worktree.
  - Verify worktree directory is git-ignored (if project-local).
  - Run project setup (npm install, cargo build, etc.).
  - Verify clean test baseline.
  - Create/lock `00-worktree.md` before proceeding.
  - **Skill:** `skills/rlm-worktree/SKILL.md`
- **For Phase 1.5 (Debug Mode - optional):**
  - Determine if requirement needs debugging (bug fixes, test failures, unexpected behavior).
  - If yes: create/lock `01.5-root-cause.md` before Phase 2.
  - Phase 1.5 uses systematic debugging (see `skills/rlm-debugging/SKILL.md`).
- For Phase 5:
  - Write `05-manual-qa.md` with scenarios in `DRAFT`.
  - Pause and request user results/sign-off.
  - On next invocation, record results, lock Phase 5, then continue to Phase 6 and 7.

## Phase Transition Guardrail (Mandatory, Hard Stop)

- Before starting Phase `N`, validate the lock chain for all prior phases (`0..N-1`) using `.agent/PLANS.md`.
- **If Phase 0 exists:** It must be lock-valid before Phase 1/2 can begin (worktree isolation verified).
- **If Phase 1.5 exists:** It must be lock-valid before Phase 2 can begin.
- A prior phase is considered lock-valid only when its base artifact and phase-local addenda are `LOCKED`, include `LockedAt` and `LockHash`, and end with `Coverage: PASS` and `Approval: PASS`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doubleuuser/rlm-workflow](https://github.com/doubleuuser/rlm-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
