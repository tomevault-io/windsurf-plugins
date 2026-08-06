---
trigger: always_on
description: For any tracked repository change beyond a trivial typo fix, create or use a
---

# Agent Instructions

## Commit And PR Hygiene

### Worktree And PR Gate

For any tracked repository change beyond a trivial typo fix, create or use a
dedicated clean `git worktree` on a `codex/` branch from latest `origin/main`
before editing files. Do not implement changes directly in a dirty primary
worktree, even when the task starts by inspecting that dirty tree.

When a dirty worktree contains potentially valuable changes, first classify it
read-only, then copy or reapply the valuable subset into the dedicated clean
worktree and open a PR from that branch. Reset or clean the original dirty
worktree only after the valuable subset has been merged or explicitly judged
obsolete. Leave unrelated untracked local artifacts alone.

Every tracked repository change must be pushed on a branch and reviewed through
a pull request before it reaches `main`. Do not push broad mixed commits or
direct commits to `main`.

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

For small, low-risk PRs, maintainers may self-merge after validation when all
of the following are true:

Here, "自合并" means: 自己 review/refine, then admin-bypass merge after the
required validation and authorization.

- the PR only touches public docs, contributor metadata, or narrow cleanup;
- the change is single-purpose and easy to review from the diff;
- required checks or focused smokes have passed;
- private state, raw benchmark evidence, credentials, local paths, and
  generated logs are excluded;
- there is no runtime behavior, benchmark adapter, permission, destructive git,
  or public evidence-policy change that needs separate review.

Small benchmark seam/refactor PRs may also be self-merged when they are like
PR #145: they add or clarify a reusable adapter/control-plane contract, include
focused public smokes, do not launch benchmark jobs, do not change scoring or
runner behavior for an existing benchmark, and do not include temporary probes,
raw evidence, private state, credentials, local paths, or generated logs.

Benchmark helper/runtime PRs may also be self-merged after owner authorization
when they are limited to public benchmark helper code, status/runtime
observation, reducer/closeout plumbing, or benchmark developer workflow support;
focused smokes or compile checks pass; public/private boundary scans are clean;
and the PR does not change benchmark scoring, task semantics, leaderboard or
submission behavior, permission boundaries, or launch new benchmark jobs.

After self-merging, sync local `main`, leave unrelated untracked local artifacts
alone, and continue with the next safe project batch.

Before self-merging non-trivial LoopX changes, run
`loopx canary premerge --from-git-diff` or an equivalent risk-based validation
set. The PR comment must name the changed surfaces, checks run, failures/skips,
manual holds, and why the coverage is enough. One hand-picked smoke is not
enough for runtime, quota/status, scheduler, todo, install, dashboard,
benchmark-boundary, or public/private evidence changes.

### Release Contributor Attribution

Keep the shipped product changes as the primary release narrative. When the tag
range contains merged work from community contributors other than project
founder `@huangruiteng`, add a prominent `## Community Contributors` section
after the English product groups and a matching `### 社区贡献者` section after
the Chinese product groups. Place both before compatibility, validation, or
update material so the credit remains visible without replacing the release
summary.

Link each eligible contributor's GitHub handle and relevant pull requests, and
describe the concrete contribution instead of publishing an unannotated name
list. Explicitly highlight external and first-time contributors when
applicable. Do not list or thank `@huangruiteng` in contributor sections;
founder stewardship is implicit in LoopX releases. Omit both contributor
sections when the tag range has no eligible community contribution.

Derive attribution from the previous-tag-to-current-tag Git range plus merged
PR metadata. Do not guess from commit display names, omit contributors because
their work is summarized elsewhere, let contributor credit displace product

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huangruiteng/loopx](https://github.com/huangruiteng/loopx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
