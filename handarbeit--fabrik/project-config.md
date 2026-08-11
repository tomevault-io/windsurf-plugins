---
trigger: always_on
description: Fabrik is a Go CLI that orchestrates Claude Code through an SDLC pipeline defined on a GitHub Project board. Issues are the unit of work. The pipeline stages (Specify → Research → Plan → Implement → Review → Validate) are configured via YAML files.
---

# Fabrik — Development Guide for Claude

## Project Overview

Fabrik is a Go CLI that orchestrates Claude Code through an SDLC pipeline defined on a GitHub Project board. Issues are the unit of work. The pipeline stages (Specify → Research → Plan → Implement → Review → Validate) are configured via YAML files.

## Build & Test

```bash
go build -o fabrik .     # Build
go test ./...            # Run all tests
go test -race ./...      # Run with race detector
go vet ./...             # Lint
```

## Documentation bundle (docs/llms-full.txt)

When you modify any of the canonical doc pages — `docs/USER_GUIDE.md`, `docs/state-machine.md`, `docs/stage-lifecycle.md`, or `docs/positioning.md` — you MUST regenerate `docs/llms-full.txt` in the same commit:

```bash
bash scripts/generate-llms-full.sh
git add docs/llms-full.txt
```

CI's `docs-drift` workflow (`.github/workflows/docs-drift.yml`) runs the regen and fails the PR if the committed bundle differs from the regen output. Forgetting the regen costs an extra round-trip; doing it consistently keeps PRs green on first push. This requirement applies to **every** stage (Specify, Research, Plan, Implement, Review, Validate) — wherever a doc edit happens, the bundle must follow.

## Architecture

- `cmd/root.go` — CLI entry point, flag parsing, .env loading
- `engine/engine.go` — Engine struct, Config, construction, Run() entry point
- `engine/poll.go` — Main poll loop, idle-upgrade, concurrent worker dispatch
- `engine/item.go` — Per-issue processing: stage runs, comment processing, blocking/pausing
- `engine/pr.go` — Output posting: issue comments, PR comments, summary extraction
- `engine/comments.go` — Comment detection and filtering logic
- `engine/context.go` — Context files (.fabrik-context/) and stage comment lookup
- `engine/repo.go` — Per-repo identity helpers (parseOwnerRepo, repoName, issueKey)
- `engine/claude.go` — Claude Code invocation, prompt building, marker extraction
- `engine/worktree.go` — Git worktree lifecycle (create, update, push, cleanup)
- `engine/merge_train.go` — Merge-train worker: trial branch assembly, inline conflict resolution, integration PR creation and CI polling (ADR-059 D3)
- `engine/ci_settle.go` — `settleAwaitingCIScan`: the per-poll settle scan that owns `fabrik:awaiting-ci` items, sourced directly from `board.Items` so it is independent of `itemMayNeedWork`/`selectDeepFetchCandidates` admission (the fifth instance of the "dedicated settle scan" pattern — ADR-1270). Runs the shared `catchUpPhase1Handlers` chain, primes the store with a live check-run read (`RefreshCheckRunsLive`) so a stale cached PENDING classification cannot wedge the item, and applies an **unconditional `CIWaitTimeout` backstop** ahead of the handler chain so an item escalates even when some gate claims it before `checkCIGate` is ever reached. The main poll loop deliberately no longer processes these items (`poll.go` admits `hasComplete`-only); see ADR-1270 and #1303/#1325.
- `engine/queued_review_settle.go` — `settleQueuedReviewFindings`: the per-poll settle scan that detects unresolved review-thread feedback arriving on a Queued merge-train member's linked PR — a window the `HoldingStage` exclusion otherwise blacks out entirely (the sixth instance of the "dedicated settle scan" pattern — ADR-1270). Ejects a flagged member off `Queued` via `ejectMember`/`ejectQueuedMemberForReviewFindings` (`engine/merge_train.go`) so the ordinary review-reinvoke path can address the finding, either directly (no merge-train worker in flight for the repo) or via a mutex-guarded pending-eject signal the worker itself consumes at its own checkpoints (`applyPendingReviewEjects`, mirroring the runaway guard's "poll writes a signal, worker checks it at a checkpoint" shape) when one is. See ADR-1208, `docs/state-machine.md` §6.16, and #1208.
- `engine/interfaces.go` — GitHubClient and ClaudeInvoker interfaces (for testing)
- `github/project.go` — GraphQL board fetching (single query for all items + comments + linked PRs)
- `github/client.go` — HTTP client construction and shared request helpers
- `github/labels.go` — Label mutations (add, remove, ensure)
- `github/comments.go` — Comment mutations (add, update, reactions)
- `github/prs.go` — PR mutations (create draft, mark ready)
- `github/status.go` — Project board status updates
- `github/rest.go` — Low-level HTTP helpers
- `github/types.go` — Shared data types (ProjectItem, Comment, ReactionGroup)
- `stages/stages.go` — YAML stage config loading
- `stages/examples/` — Default stage YAML sources, embedded in binary via `//go:embed`
- `stages/embed.go` — Exposes embedded default stages as `stages.DefaultStages`
- `plugin/embed.go` — `FabrikPlugin` embed.FS; source of truth for all built-in skills
- `plugin/refresh.go` — `RefreshPlugin()` overwrites `.fabrik/plugin/` from the embedded source
- `cmd/init.go` — `fabrik init` subcommand; extracts embedded YAMLs to `.fabrik/stages/`
- `.fabrik/stages/` — Live stage configs for this project (tracked in git)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [handarbeit/fabrik](https://github.com/handarbeit/fabrik) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
