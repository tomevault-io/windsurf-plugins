---
trigger: always_on
description: > **Write convention:** record durable project facts HERE, not in native harness memory.
---

# Project memory (canonical — read by ALL harnesses)

> **Write convention:** record durable project facts HERE, not in native harness memory.
> **Work loop:** `orch next --agent <you>` → work in the printed worktree → `orch submit`.
> **Review:** when `orch review-queue --agent <you>` is non-empty, review the other harness's PRs.

## Conventions
_(add project-specific facts, gotchas, and architectural decisions below)_

- **Batched issue lookups**: `eligibleIssues` (`src/board/board.ts`) and `reviewQueue` (`src/board/review.ts`) resolve dependency/PR issue state from a single `listIssues({state:"open"})` result indexed by `byNumber(...)` — never a per-dep/per-PR `getIssue`. An issue absent from that open-issue map (closed, missing, inaccessible) is treated as non-blocking / skipped, matching the old `openDeps` semantics. Keep new hot-path issue lookups on the batched map; reserve `getIssue` for single-target paths (`checkMergeGate`, `resolvePrIssue`).
- **GitHub list pagination** (`src/github/github.ts`): `listIssues`, `listPrs`/`listOpenPrs`, and `listLabels` page through `gh api repos/{owner}/{repo}/<issues|pulls|labels>` (`page`/`per_page=100`, looping until a page comes back short) instead of a single `gh issue|pr|label list --limit N` — those subcommands take only one `--limit` with no offset, so a fixed number silently truncates past that count. This means list reads use REST JSON shapes (lowercase `state`, `head.ref`, `merged_at`, label/assignee objects) mapped by dedicated `parseRest*` functions, distinct from the GraphQL shape (`--json ...` on `gh issue/pr view`) used by the single-target `getIssue`/`getPr` and parsed by `parseIssue`/`parsePr`. The `/issues` REST endpoint also returns PRs (flagged via a `pull_request` key) — `listIssues` filters those out to match `gh issue list` semantics.
- **Claim-lock lifecycle** (`refs/orch/lock/issue-<n>`): a lock is released on successful merge (`src/board/review.ts`), claim-setup rollback (`src/tasks/service.ts`), successful `orch abandon <n>` cleanup (`src/commands/abandon.ts`), and failed/no-commit runner recovery only after safe cleanup actually removes the worktree (`processNext` in `src/tasks/runner.ts`). Failed/no-commit runs, successful merges, plain abandon, and repair use safe worktree cleanup; only the human-explicit `orch abandon <n> --discard` path force-removes retained work. If safe cleanup retains dirty, untracked, detached, or committed work—or submit state is ambiguous after a successful harness run—recovery preserves both the worktree and its claim lock for repair. A retained worktree may be reused only when Git registers its path on the expected task branch.
- **Claim setup saga** (`claimSpecific` in `src/tasks/service.ts`): setup is observe/decide/execute/compensate across the Git lock, GitHub projection, and registered worktree. Each attempt stores a unique Git-object id in its lock ref, so ambiguous writes may roll forward only after re-observing that exact owner token (or the expected issue/worktree state). Compensation restores only the saga's issue-field delta, verifies that `status:claimed` is gone, and compare-deletes only its own lock; an unverified or conflicting worktree is preserved for reconciliation.
- **Run telemetry**: every completed `processNext` harness run appends exactly one best-effort record to `~/.orch/<project>/runs.jsonl`. Missing/malformed logs and telemetry IO errors must yield null usage or a warning without changing the task outcome.
- **Recoverable task lifecycle**: `deriveTaskState` (`src/tasks/lifecycle.ts`) is the pure authority over observed issue, lock, worktree, branch, PR, and latest unresolved telemetry facts. Its states are `ready | claimed | in-progress | in-review | needs-attention | done | inconsistent`; labels are output projections only and must never be read as lifecycle truth. Use `decideTaskTransition` before mutation, and repair every reported invariant violation before transitioning an inconsistent task. See ADR-0006.
- **Repository base branch**: `resolveBaseBranch` (`src/git/git.ts`) is the single authority for the configured `baseBranch` or GitHub-derived default. It keeps the GitHub branch name separate from its validated local/origin ref. Claim/worktree creation, runner commit detection, lifecycle reconciliation, and PR submission must use that resolution. `countCommitsAhead` requires both `merge-base` and `rev-list` to succeed; comparison failures are errors and must never be treated as zero commits.
- **Lifecycle reconciliation** (`src/tasks/reconcile.ts`): `orch repair [issue]` is read-only by default; `--apply` executes one idempotent action and then re-observes/re-plans, so interrupted or ambiguous writes resume from durable facts and repeated runs converge to no actions. Repair may restore projections/resources and safely remove only clean, attached, externally preserved worktrees; it never force-removes a worktree or deletes a branch. Destructive intent lives only at `orch abandon <issue> --discard`; plain abandon uses safe removal and retains the lock/projection when work cannot be proven safe to remove.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kehuydietlatoi/harness_orchestrator](https://github.com/kehuydietlatoi/harness_orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
