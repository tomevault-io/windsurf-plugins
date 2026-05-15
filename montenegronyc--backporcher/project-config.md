---
trigger: always_on
description: Backporcher is a parallel Claude Code agent dispatcher. GitHub Issues are the task queue: label an issue with `backporcher`, and the daemon picks it up, runs a sandboxed `claude -p` agent in a git worktree, creates a PR, reviews it via a coordinator agent, monitors CI, auto-merges on success, and closes the issue.
---

# CLAUDE.md — Backporcher

Backporcher is a parallel Claude Code agent dispatcher. GitHub Issues are the task queue: label an issue with `backporcher`, and the daemon picks it up, runs a sandboxed `claude -p` agent in a git worktree, creates a PR, reviews it via a coordinator agent, monitors CI, auto-merges on success, and closes the issue.

## Architecture

```
GitHub Issue (label: backporcher)
  → Issue Poller (30s)
    → Batch Orchestrator (haiku, for 2+ issues per repo)
      → SQLite queue (with priority + dependency chain)
        → Conflict check (haiku, ~$0.001) — serializes overlapping tasks
          → Task Executor (semaphore: 2 concurrent, respects dependencies)
            → Credential sync (auto if stale)
            → Navigation Context (sonnet queries code graph → relevant files/symbols)
              → claude -p in sandboxed worktree (with stack info + learnings + navigation map)
                → Build verification (optional, per-repo)
                  → git push + gh pr create
                    → Code Graph (Tree-sitter → blast radius BFS)
                      → Coordinator Review (claude -p reviews diff + impacted code)
                      → CI Monitor (retries up to 3x on failure)
                        → Merge gate (hold for approval or auto-merge)
                          → Close issue (label: backporcher-done)
```

## Six Concurrent Loops

The worker daemon (`src/worker.py`) runs up to 6 async loops via `asyncio.gather()`:

1. **Issue Poller** (every 30s) — scans GitHub for issues labeled `backporcher`, deduplicates (including failed tasks, excludes completed no-op tasks with no PR), batch-orchestrates 2+ issues per repo (assigns priorities, dependencies, models via haiku), creates tasks, claims issues with `backporcher-in-progress` label
2. **Task Executor** (every 5s) — claims queued tasks (bounded by semaphore), syncs credentials, generates navigation context (sonnet + code graph → relevant files/symbols for the agent), runs `claude -p` in isolated worktrees with structured prompt (stack info + learnings + navigation map + task), optionally runs build verification, creates PRs. Auto-retries transient failures (auth, permissions, stale branches)
3. **Coordinator Reviewer** (every 15s) — reviews each PR diff via `claude -p`, checks for conflicts with other open PRs, approves or rejects. Backfills missing `pr_number` from `pr_url`
4. **CI Monitor** (every 60s) — checks CI status on approved PRs, auto-merges passing PRs (squash), auto-retries failures with CI log context, closes GitHub issues on success
5. **Artifact Cleanup** (every 5 min) — removes worktrees and remote branches for terminal tasks older than 10 minutes
6. **Dashboard** (optional) — aiohttp web server with HTTP Basic Auth, real-time SSE updates every 5s. Warm light theme: cream/beige base (#F5F5DC), translucent glass panels with backdrop blur, SF Pro + system font typography, pulsing badges for active states, warm red-orange success ring gauges. Theme CSS served from `backporcher-theme.css` (editable without restart). Only starts when `BACKPORCHER_DASHBOARD_PASSWORD` is set. Features: inline Approve/Hold/Reject/Escalate/Re-queue buttons, task detail panel with timeline, edit modal for prompt/model/priority rewriting, pipeline summary with metrics (merged count, success rate, avg time, retry rate), global Pause/Resume toggle. API: `POST /api/tasks/{id}/approve|hold|reject|edit|requeue|escalate`, `POST /api/pause|resume`, `GET /api/stats`

## Task Status Flow

```
queued → working → pr_created → reviewing → reviewed → ci_passed → completed (merged)
                                                                 → hold=merge_approval (review-merge mode)
                                                                   → backporcher approve → completed
                                                     → retrying → pr_created (retry loop, up to 3x)
                                                     → failed (max retries exhausted)
                              → reviewing → failed (coordinator rejected PR)
       → hold=dispatch_approval (review-all mode) → backporcher approve → working
       → failed (agent error / exit != 0)
       → failed (safety scan blocked PR — secrets or dangerous patterns detected)
       → completed (agent ran but no changes to push)
       → queued (auto-retry on transient failure, up to 2x)
       → hold=circuit_breaker (repo failure rate too high, auto-releases after cooldown)
any    → cancelled (manual via CLI)
```

## Orchestrator Mode

Controls how much human oversight the pipeline requires. Set via `BACKPORCHER_APPROVAL_MODE`:

| Mode | Dispatch | Merge | Default |
|------|----------|-------|---------|
| `full-auto` | automatic | automatic | |
| `review-merge` | automatic | approval required | yes |
| `review-all` | approval required | approval required | |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [montenegronyc/backporcher](https://github.com/montenegronyc/backporcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
