---
trigger: always_on
description: Architecture rules live in `CLAUDE.md`. This file holds the rules for
---

# grove — agent notes

Architecture rules live in `CLAUDE.md`. This file holds the rules for
how agents open, verify, and merge PRs.

## Before shipping a change

Run **`npm run check`** — the fast local gate. It runs:

1. `npx tsc --noEmit` — typecheck.
2. `npm test` — the full vitest suite.

The same checks run in CI (`check / test` + `check / audit` + `check / secrets`)
plus a `plan-drift` job that validates `PLAN.md` tasks against the code.
CI must be green before merge.

## Pre-push hook (opt-in)

`.githooks/pre-push` runs `npm run check` before every push so local
regressions don't burn a CI minute. Opt in once per clone:

```bash
git config core.hooksPath .githooks
```

Opt out with `git config --unset core.hooksPath`. Bypass a single push
with `git push --no-verify` — don't make a habit.

## Merging PRs — standing authorization

You are authorized to merge PRs into `main` without asking first, *if*
all of the following are true:

- CI is green: `test`, `plan-drift`, `audit`, `secrets` all SUCCESS.
- `mergeable == "MERGEABLE"` and `mergeStateStatus` is `CLEAN`.
- Not a draft.
- No `changes requested` review.
- No label named `needs-human`, `wip`, or `do-not-merge`.
- For Dependabot PRs: any version bump passing CI is fair game,
  including majors of dev tooling. `@modelcontextprotocol/sdk` and
  `better-sqlite3` **majors** are always off-limits — Dependabot is
  configured to skip them but check anyway. `@anthropic-ai/sdk` updates
  (including majors) are fine; the SDK tracks Claude releases we want.

**Default for your own PRs:** `gh pr merge <n> --auto --squash --delete-branch`.
This uses GitHub's native auto-merge queue — the PR sits until required
checks pass, then merges without a second touch.

**For existing Dependabot PRs that are already green:**
`gh pr merge <n> --squash --delete-branch`.

Stale PRs from Dependabot: comment `@dependabot rebase` and re-check
status before merging.

### Ask before merging when

- The PR changes `CLAUDE.md`, `AGENTS.md`, or `PLAN.md` substantively.
- The PR removes tests, lowers the CI bar, or disables status checks.
- The PR touches `src/db-migration*.ts` or `src/db.ts` schema — the
  deploy job has a schema-change guard that requires explicit
  confirmation.
- CI is red for a reason that isn't "rebase onto current main."
- The PR author is a first-time external contributor.

### GitHub auth note

The `gh` CLI used for merges must have the `workflow` scope to merge
PRs that touch `.github/workflows/*`. If `gh pr merge` fails with
"refusing to allow an OAuth App to create or update workflow", run
`gh auth refresh -s workflow` once to grant the scope.

## Autonomous batch shipping (`scripts/ship.ts`)

`npm run ship` spawns parallel Claude Code agents in worktrees, merges
their commits into a `ship/<batch-id>` branch, opens a PR, and lets
GitHub's auto-merge land it. Replaces the bash orchestrators that used
to live in `scripts/{run-batch,ship-*}.sh`. See `docs/ship.md` for
full operator usage.

### Authorization

Shipping via `scripts/ship.ts` is authorized whenever **manual PR
merges are authorized** (see above). The orchestrator uses the same
`gh pr merge --auto --squash --delete-branch` command you'd type
yourself — GitHub's auto-merge queue respects the same required
status checks (`test`, `plan-drift`, `audit`, `secrets`). If CI turns
red, the PR stays open; `ship.ts` times out after 30 minutes of
no-merge and halts so a human can decide.

### Schema-change gate (`noAutoMerge`)

The "Ask before merging when PR touches `src/db.ts` schema" rule
above is a human convention that GitHub's auto-merge doesn't know
about. To prevent `ship.ts` from auto-merging schema changes,
`scripts/ship/batches.ts` marks those batches `noAutoMerge: true`.
The orchestrator then opens the PR, logs the URL, and halts the run.

Flow for a `noAutoMerge` batch:
1. ship.ts opens `ship/<batch-id>` PR and halts with a visible
   "HALTED — batch requires human review" message.
2. Human reviews the diff, merges via the normal `gh pr merge`.
3. Human triggers `workflow_dispatch` on `deploy` with
   `confirm_schema_change=true` input (the Tier 2 guard refuses
   otherwise because SQLite migrations don't roll back safely).
4. Resume with `npm run ship -- --from <next-batch-id>`.

Phase 8's `p8a-1` and `p8b-1` are both flagged; `npm run ship -- --list`
shows `[manual merge]` next to them.

### Cross-repo asymmetry

- **grove** → `ship.ts` opens a PR against `origin/main`. Branch
  protection enforces the 4 required checks. No direct push.
- **grove-www** → `ship.ts` pushes directly to `origin/main`.
  grove-www has no branch protection today. If that ever changes,
  update `groveWwwSyncAfter()` in `scripts/ship.ts` to use the same
  PR flow. Do not silently add protection to grove-www without
  updating ship.ts first — agents will silently fail to push.

### Batch registry

New work gets added to `scripts/ship/batches.ts` as a `Batch` entry
with 1-N `BatchEntry` agents. Commit discipline (message format,
grove-www rules, exit behavior) comes from the project system prompt
in `.claude/settings.json` — keep per-batch prompts spec-only.

## Claude Code hooks

`.claude/settings.json` wires two hooks active for every agent session in this repo:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmilinovich/grove](https://github.com/jmilinovich/grove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
