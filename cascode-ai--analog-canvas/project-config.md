---
trigger: always_on
description: Treat the repository as an engineering project: bounded targets, explicit
---

# Agent Working Rules

Treat the repository as an engineering project: bounded targets, explicit
ownership, risk-proportional validation, and decisions recorded where the work
is. What a change did and why is stated in its commit message, which travels
with the diff through `git log`, `git blame`, and the pull request.

Notes you keep while working — scratch plans, checklists, drafts — belong in
the untracked `plan/` directory. They are a local working area and are not
published.

## Development and Delivery

The cadence is local iteration followed by a delivered pull request. A request
to change the editor proceeds through both stages unless the user explicitly
asks for local-only work or a branch backup instead of publication.

1. **Local iteration.** Continue the current local batch branch, or create
   `codex/local-batch` from current `main` when starting a new batch. Use
   `pnpm dev` for feedback and focused checks for each bounded target. Keep
   separate, explanatory local commits for separate targets. A completed local
   target ends with its commit and validation, then proceeds to Delivery unless
   the user limited the request to local work. A branch pushed only for backup
   is not a delivery.
2. **Delivery.** Open one pull request for the completed change or batch, run
   the mainline delivery gate, queue it for merge, wait for the required
   checks in the merge queue, and verify Production. Every non-documentation merge to `main` follows this one
   route. Batching is optional. A change means one independently useful feature, fix, or
   improvement; supporting tests, repair commits, files, and formatting do not
   count as extra changes. Prepare any intended release version before merging.

Keep a short working list in `plan/local-batch.md` while a batch is in
progress: the branch, completed changes and count, validation still needed,
and current stage. Read it when continuing the batch and reconcile it with
local commit messages; commit count alone is not a change count. Start the next
list after the batch is delivered. Decisions and validation that must survive
delivery belong in the commits and pull request. A squash merge's description
must preserve the target summary, test impact, validation, and unresolved
limitations.

## Operating Discipline

Before starting a target:

1. Run `git status --short --branch` from the repository root.
2. Audit dirty state by ownership:
   - Proceed normally when the worktree is clean.
   - When it is dirty, identify whether each changed path belongs to the
     current target, the user, another worker, or an earlier target.
   - Unrelated dirty paths do not automatically block work.
   - Stop before editing when dirty paths overlap the target's owned files,
     ownership is unclear, or a dirty shared contract affects the target.
   - Say so in the commit message when proceeding with unrelated dirty files
     present.
3. Identify the target owner, goal, expected files, shared dependencies, and
   validation surface.
4. Read `README.md` and any closer domain instructions.
5. Review validation intent before editing:
   - Run `pnpm gate:plan -- --path <expected-path>` for the expected owned
     paths when practical.
   - Expand the selected commands and identify platform, release, generated-
     artifact, and golden-state assumptions.
   - State which gates you chose, and why, in the commit message.

## Before Editing

Know the boundary of the target before editing tracked files: the goal, which
paths it owns, what the dirty state means for it, and which shared contracts
it touches. Hold that in a local note if it helps; the repository does not
require a file for it.

Do not edit outside the target's boundary without deciding, deliberately, that
the boundary has moved.

## During Work

- Keep each target small and reviewable; exclude unrelated cleanup.
- Define a target by one ownership and validation boundary, not by every
  visible symptom. Closely related micro-fixes that share files, contracts,
  and validation belong in one target; independent changes do not.
- Protect shared contracts, generated artifacts, binary assets, and user-owned
  work unless the target explicitly claims them.
- Decide deliberately before expanding scope or taking on a new dependency, and
  say so in the commit.
- Regenerate the advisory gate plan from the real diff with
  `pnpm gate:plan -- --base <base-ref>` before expensive validation. If the
  actual selection differs materially from the gates you chose, revisit the
  choice before proceeding.
- For local iteration, plan and validate the current target's delta rather than
  repeatedly treating the accumulated batch as a new change. Start with focused
  unit/browser checks directly. Run
  `pnpm gate:preflight -- --base <target-base>` before executing selected
  affected, build, or release gates, and use
  `pnpm gate:affected -- --base <target-base>` for the bounded gates the target
  needs. The Test-Impact check reads commit messages, so check its declaration
  after committing. A planned `full-delivery` gate is owed at batch delivery,
  where the merge queue's required checks and the short local check in the
  Mainline Delivery Gate meet it; it is not an instruction to rerun full

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cascode-ai/analog-canvas](https://github.com/cascode-ai/analog-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
