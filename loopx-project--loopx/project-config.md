---
trigger: always_on
description: Before selecting non-trivial work, resolve the current requested outcome from
---

# Agent Instructions

## Goal-Oriented Development

Before selecting non-trivial work, resolve the current requested outcome from
user direction, the linked issue/task, accepted contract or demonstrated bug.
For cross-cutting LoopX work, consult the [overall roadmap](docs/architecture/rfcs/loopx-overall-roadmap-v0.md)
and relevant domain acceptance; do not make every fix wait for every RFC or
invent a roadmap id. Check latest `main`, related PRs and canonical Todos so an
older task description cannot override corrected direction or duplicate work.

Carry one compact delivery brief from task to PR: goal/source, current gap,
observable result, owning boundary and decisive acceptance evidence. Reuse the
existing task/PR fields; keep private Goal state out of public artifacts.
Choose a complete, independently reviewable and reversible outcome slice.
Small diffs, fields, receipts, test counts and merged PR counts do not establish
progress. Characterization, prerequisites, research, docs and maintenance are
valid when they remove an evidenced gap or enable a named real next step.

Continue through the selected slice's implementation, integration, negative
cases and readback while authorized work remains feasible. Do not stop after
setup, a serializer, a mock or an isolated smoke when the useful outcome is
still missing. Do not expand scope merely to make a PR larger. When a staged
boundary is necessary, name the delivered delta, remaining gap, next owner/
dependency and why the boundary improves verification or rollback. Reuse or
update an existing successor; do not create ceremonial follow-up tasks for a
completed request. Real authorization, cost and operational stop gates remain.

For multi-Agent changes, qualify the relationship the user needs: dependency
artifacts, receiver adoption, claim/lease handling, independent acceptance and
result return as applicable. Sending a message or registering workers does not
prove collaboration. Missing frontend/Lark/CLI companion work makes a product
journey partial even when a backend slice is ready to merge.

Before delivery, reconcile the result with the original/current goal and
update its task and RFC checkpoint when the boundary changes. Preserve passed,
failed and untested distinctions. If user feedback exposes the same missing
outcome, repair the owning rule, active task or projection through self-repair;
do not merely append stronger instructions. PR review must execute the current
capability-owned `problem_context` delivery judgment; author declarations and
this prose do not certify it or settle a Goal.

## Commit And PR Hygiene

### Worktree And PR Gate

For any tracked repository change beyond a trivial typo fix, create or use a
dedicated clean `git worktree` on a `codex/` branch. Use latest `origin/main`
unless the user explicitly names an integration or release branch; in that
case, fetch that branch and use its latest remote head as both the worktree
baseline and pull-request base. Before pushing, verify the merge base and PR
base so unrelated `main` history cannot leak into a stacked integration PR.
Do not implement changes directly in a dirty primary worktree, even when the
task starts by inspecting that dirty tree.

When a dirty worktree contains potentially valuable changes, first classify it
read-only, then copy or reapply the valuable subset into the dedicated clean
worktree and open a PR from that branch. Reset or clean the original dirty
worktree only after the valuable subset has been merged or explicitly judged
obsolete. Leave unrelated untracked local artifacts alone.

Every tracked repository change must be pushed on a branch and reviewed through
a pull request before it reaches `main`. Do not push broad mixed commits or
direct commits to `main`.

### DCO Sign-Off

Every commit in a pull-request branch must include a
`Signed-off-by: Your Name <your.email@example.com>` trailer, or the `DCO`
check will reject the PR. Always commit with `git commit -s`. If a commit is
already missing the trailer, amend it with `git commit --amend -s` (or an
interactive rebase for multiple commits) before pushing. See CONTRIBUTING.md
for the full DCO policy.

Only skip this worktree/PR gate when the user explicitly says the change is
local-only and must not be proposed for the repository.

For non-trivial repository changes, especially anything that touches benchmark
adapters, smoke tests, public docs, or commit/push workflows, use the
`git-split-commit-pr` workflow before staging:

1. Establish ground truth with `git status --short --branch`,
   `git diff --stat`, `git diff --name-only`, and
   `git ls-files --others --exclude-standard`.
2. Classify every changed path before staging:
   - core product code;
   - core documentation;
   - durable validation smoke;
   - local/private state;
   - low-value or obsolete artifact.
3. Scan candidate paths for credentials, private state, local absolute paths,
   raw benchmark logs, trajectories, verifier output, and internal links.
4. Stage by explicit pathspecs only. Do not use `git add .`.
5. Split commits by reviewer logic:
   - runtime/API behavior;
   - public docs and protocol notes;
   - focused validation or cleanup.
6. Push a branch and open a PR for reviewable batches.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [loopx-project/loopx](https://github.com/loopx-project/loopx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
