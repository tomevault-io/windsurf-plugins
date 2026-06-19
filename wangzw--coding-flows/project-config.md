---
trigger: always_on
description: |
---


# coding-flows — Coder / Reviewer cycle

Paired-agent GitHub workflow for a single repo. **One invocation = one cycle**
for one role. Long-running monitoring is handled by the wrapping scheduler
(`slock reminder` or `/loop`), not by this skill staying alive.

## Quick start

```
role=coder       # turn issues into PRs and merge approved PRs
role=reviewer    # review open PRs until they meet merge bar
```

The caller MUST specify the role. If not specified, **stop and ask**.

## Per-repo configuration

The skill reads `.coding-flows.json` from the repo root. It controls: acting user,
merge method, scope rules (which standards apply to which paths), high-risk
label rules, AC-mapping / LGTM-coverage requirements, and flaky-test policy.
If the file is absent, sensible defaults apply (LGTM coverage and AC mapping
stay required — those are core invariants).

- Schema: **[references/config-schema.md](references/config-schema.md)**
- Example: **[examples/coding-flows.example.json](examples/coding-flows.example.json)**

## Pre-flight (every cycle)

1. `gh auth status` — must match `acting_user` from `.coding-flows.json` (or current
   authenticated user if no config). On mismatch, abort with `auth-broken`.
2. cwd is inside a GitHub repo: `git rev-parse --show-toplevel` + `gh repo view`.
   Worktree cwds are valid — `find_repo_config` falls back to the main repo via
   `git rev-parse --git-common-dir`.
3. **Main-checkout `git status` is no longer pre-flight-blocking.** Per-issue
   work happens in worktrees; the main checkout's state is independent.
   Per-worktree state is checked per-item in Coder Phase B / E.
4. Coder only: `scripts/coding-flows-worktree-prune` — retire worktrees whose PRs
   have merged or closed since the last cycle (best-effort).
5. Read project conventions: `CLAUDE.md` / `AGENTS.md`, `commitlint.config.*`,
   `.github/PULL_REQUEST_TEMPLATE.md`, `.github/CODEOWNERS`, build commands
   from `Makefile` / `package.json` / `pyproject.toml`.
6. **Mandatory scope detection.** Run `scripts/coding-flows-detect-scope` against
   the diff (Coder: working branch; Reviewer: target PR). It enumerates which
   coding-standards skills + invariant docs apply per `.coding-flows.json`
   `scope_rules`. **Read each one into context before Phase B/C.** This closes
   the "invariant never loaded" failure mode.

## Cycles

- **Coder** — see **[references/coder-cycle.md](references/coder-cycle.md)**.
  Per-issue **worktree** isolation — worktrees default to siblings of the
  main checkout at `<parent-of-main-repo>/<branch-slug>/`; see
  [references/worktrees.md](references/worktrees.md). Within one cycle the
  Coder is serial (one build/test at a time) but multi-task across items,
  with **strict two-stage ordering**: process every actionable PR
  (address review comments → fix CI failures → merge) before touching any
  issue. When an item hands off to CI or Reviewer, the cycle rotates to
  the next actionable item rather than blocking. **The scheduler's
  interval is polling frequency, not a per-cycle time budget**: take
  however long an item needs (20 min, an hour) before rotating —
  partial work and draft-PR shortcuts are not part of the workflow.
  Iteration frequency comes from the wrapping scheduler (`slock
  reminder` / `/loop`), not from this skill.
- **Reviewer** — see **[references/reviewer-cycle.md](references/reviewer-cycle.md)**.
  Default-deny adversarial stance. Builds a **local** review plan (under
  `~/.cache/coding-flows/<owner>/<repo>/pr-<N>/`); the plan never touches the PR.
  Publishes only `/lgtm` with a coverage marker (compact, machine-verifiable)
  or `--request-changes` with natural-language findings.

## Merge gates (hard, machine-checked)

A PR is merge-ready only when **all** pass. Enforced by `scripts/coding-flows-merge` —
agents **must not** call `gh pr merge` directly.

1. **CI green** on the latest commit (flaky-test policy applied first — see
   [references/flaky-test-policy.md](references/flaky-test-policy.md)).
2. **Linked issue** present in PR body (`Closes #N` / `Fixes #N` / `Resolves #N`).
3. **AC mapping table** present and complete — see
   [references/ac-mapping.md](references/ac-mapping.md).
4. **LGTM coverage** — the union of `acs=`, `invariants=`, and
   `risks-reviewed=` claims across current-head `/lgtm` markers covers every
   AC ID, every triggered invariant, and every Coder-declared risk category.
   See [references/review-plan.md](references/review-plan.md).
5. **Current LGTM** — most recent `/lgtm` comment is bound to the current
   head SHA; any new push invalidates the prior LGTM.
6. **No unresolved review threads.**
7. **Squash or rebase only** (per `merge.method` in config; allowance against
   repo settings is verified live or via `CODING_FLOWS_REPO_ALLOWS`).
8. **High-risk PRs require dual LGTM** from two distinct reviewer markers —
   see [references/high-risk-pr.md](references/high-risk-pr.md).
9. **Risks declared** — `## Risks` table present with structured
   Category/Description/Mitigation rows + matching HTML marker.
10. **Scope envelope** — every changed file is claimed by an AC mapping row,
    a `## Support files` entry, or `scope_envelope.exclude_paths`.
11. **Test plan complete** — every bullet in the PR body's `## Test plan`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wangzw/coding-flows](https://github.com/wangzw/coding-flows) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
