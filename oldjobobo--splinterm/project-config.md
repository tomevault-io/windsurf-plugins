---
trigger: always_on
description: The repository-root worktree on `main` is reserved for fetch, status inspection,
---

# Splinterm Agent Guardrails

## 0. Branch-First Repository Workflow

### 0.1 Main Worktree Is Coordination-Only

The repository-root worktree on `main` is reserved for fetch, status inspection,
review, approved integration, and release operations. Do not begin task-file
mutations or task commits there; approved integration and release-boundary
commits remain permitted.

Before the first edit for an authorized task:

* Fetch the current remote state.
* Create a short-lived branch from the reviewed `origin/main` base for 0.2 and
  general work, or from `origin/maint/0.1` for an explicitly authorized 0.1
  maintenance patch.
* Create or use a dedicated worktree for that branch.
* Confirm the branch and worktree path before editing.

Use milestone-oriented names such as `feat/binding-help-search`,
`fix/font-reload-race`, or `docs/configuration-font-sync`. Do not use one long-lived branch for all of a
release program.

If pre-existing changes are found on `main`, partition and preserve them before
creating the task worktree. Never sweep unrelated tracked or untracked files
into the new branch, stash, commit, move, or delete them without establishing
ownership. If ownership cannot be established, do not alter the changes; stop
and obtain an ownership decision.

### 0.2 One Writer Per Branch and Worktree

One writer owns each task branch and worktree. Read-only scouting, review, and
validation may inspect that worktree. Review fixes return to the same writer and
branch by default.

Concurrent writers still require the approval in Section 1.2 and separate
branches and worktrees. Branch isolation does not authorize uncontrolled
parallel edits. Serialize dependent milestones and work that overlaps known
convergence points such as `wayland.rs`, `keymap.rs`, `action_menu.rs`, or
shared release and package files.

### 0.3 Pull Request and Merge Boundary

Each coherent branch must pass its task's focused checks, the appropriate
non-graphical boundary, actual-diff inspection, `git diff --check`, and required
independent review before merge. Record exact validation and residual risks in
the pull request.

Prefer squash merge so the owning integration branch receives one coherent
milestone commit. Merge 0.2 and general work into `main`; merge 0.1 maintenance
patches into `maint/0.1` and forward-port applicable fixes to `main` through a
separate reviewed branch. Delete the merged task branch and remove its worktree
after verifying the merge. Release candidates, promotions, tags, and publication
may originate only from `main` or `maint/0.1`; the candidate and promotion must
use the same authority branch and retain their separate approval requirements.

## 1. Cost and Delegation Stop-Loss

### 1.1 Routine Subagent Authorization

A user request that authorizes implementation, review, investigation, or delegated work also authorizes the routine subagent launches required to complete that request.

For every subagent launch:

* Announce the launch.
* State the subagent’s role.
* State its bounded task.
* Do not wait for separate user confirmation unless Section 1.2 applies.

### 1.2 Subagent Launches Requiring Approval

Ask the user before launching a subagent if the launch would:

* Introduce a new concurrent writer.
* Materially expand the scope.
* Materially increase the expected cost.
* Perform graphical testing.
* Require a destructive or irreversible action.

### 1.3 Failed or Incomplete Subagents

Distinguish execution failure from review outcome:

* An execution failure means the subagent timed out, crashed, could not access its
  required inputs, exhausted its tools before producing the requested result, or
  returned incomplete work.
* A reviewer finding defects or rejecting a milestone is a successful review, not
  a failed subagent.

If a reviewer successfully returns actionable findings:

* Apply in-scope fixes directly in the single-writer parent without asking for
  another approval.
* Run the already-authorized non-graphical validation.
* Request user input only when a finding triggers Section 1.2 or Section 2.4.
* Do not describe the review itself as an agent failure.

### 1.4 Worktree Writers

Keep the active worktree single-writer by default.

The following work may run in parallel without creating additional writers:

* Read-only scouting.
* Read-only review.
* Read-only validation.

Intentionally concurrent writers must:

* Use separate worktrees.
* Receive user approval before launch.

### 1.5 Default Agent and Review Limits

For ordinary implementation, use no more than:

* One scout or planner.
* One writer.
* Two fresh, read-only reviewers.
* One fix writer, when justified.
* One review rounds.

Ask the user before exceeding any of these limits.

### 1.6 Launch Reporting

Before launching a subagent, state:

* The subagent’s role.
* Its bounded task.
* The expected files or area of responsibility.
* The validation command.

After launch, report:

* The subagent’s scope.
* The subagent’s outcome.

Do not introduce an additional confirmation gate after launch.

### 1.7 Subagent Launch-Readiness Gate

Do not use a subagent as a substitute for the parent’s first-pass debugging,
implementation, or validation. Before every reviewer or verifier launch, the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OldJobobo/splinterm](https://github.com/OldJobobo/splinterm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
