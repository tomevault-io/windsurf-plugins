---
trigger: always_on
description: This repo hosts roborev, a local daemon + CLI for AI-assisted code review. Use this guide to
---

# AGENTS.md

## Purpose

This repo hosts roborev, a local daemon + CLI for AI-assisted code review. Use this guide to
navigate the codebase quickly, understand subsystem boundaries, and avoid missing adjacent
changes when working in a growing Go project.

## Start Here

- Product behavior and user-facing workflows: `README.md`
- CLI entry point and command registration: `cmd/roborev/main.go`
- Command discovery: `rg '^func .*Cmd\(' cmd/roborev`
- Daemon HTTP API and route wiring: `internal/daemon/server.go`
- Worker pool and job execution: `internal/daemon/worker.go`
- SQLite schema and migrations: `internal/storage/db.go`
- Config loading and resolution: `internal/config/config.go`
- Prompt construction: `internal/prompt/prompt.go`
- TUI entry point: `cmd/roborev/tui/tui.go`

## Architecture At A Glance

```text
CLI (roborev) -> HTTP API -> Daemon -> Worker Pool -> Agent adapters
                      |            |
                      |            -> Hooks / Activity log / CI poller
                      |
                      -> SQLite DB <-> Sync worker <-> PostgreSQL (optional)
```

- The daemon is the long-lived control plane. Many CLI commands are thin HTTP clients.
- Background daemon work must not modify the user's checked-out working tree.
- Foreground agentic flows such as `roborev fix` and `roborev refine` may modify code.
- Isolated background fix work uses temporary git worktrees and stores patches in the DB.

## Package Map

| Path | Purpose | Start with |
|------|---------|------------|
| `cmd/roborev/` | Cobra CLI commands and daemon-facing client logic | `main.go`, `review.go`, `fix.go`, `refine.go`, `analyze.go`, `daemon_cmd.go` |
| `cmd/roborev/tui/` | Bubble Tea terminal UI | `tui.go`, `api.go`, `fetch.go`, `handlers.go`, `render_*.go` |
| `internal/daemon/` | HTTP server, handlers, worker pool, SSE, hooks, runtime, CI poller | `server.go`, `worker.go`, `client.go`, `runtime.go`, `ci_poller.go` |
| `internal/storage/` | SQLite models/queries/migrations, sync logic, PostgreSQL mirror | `db.go`, `models.go`, `jobs.go`, `reviews.go`, `sync.go`, `postgres.go` |
| `internal/agent/` | Agent interface, registry, command-backed implementations, ACP support | `agent.go`, `codex.go`, `claude.go`, `acp.go`, `test_agent.go` |
| `internal/config/` | Global config, repo config, validation, key metadata, resolve helpers | `config.go`, `keyval.go` |
| `internal/prompt/` | Review prompt builder and template loading | `prompt.go`, `templates.go`, `prompt/analyze/` |
| `internal/review/` | Daemon-free batch review, synthesis, comment sizing/formatting | `batch.go`, `synthesis.go`, `result.go` |
| `internal/git/` | Shared git helpers for refs, diffs, branch logic, repo discovery | `git.go` |
| `internal/worktree/` | Temporary worktree creation, patch capture/apply/check | `worktree.go` |
| `internal/skills/` | Embedded Codex/Claude skill files and installer logic | `skills.go`, `internal/skills/claude/`, `internal/skills/codex/` |
| `internal/streamfmt/` | Formatting streamed agent output for CLI and TUI | `streamfmt.go`, `render.go` |
| `internal/githook/` | Hook install/upgrade logic | `githook.go` |
| `internal/github/` | GitHub REST helpers used by CI/comment flows | `comment.go` |
| `internal/ghaction/` | GitHub Actions integration | `ghaction.go` |
| `internal/update/` | Self-update logic and release fetches | `update.go` |
| `internal/testenv/` | Test environment setup helpers | `testenv.go` |
| `internal/testutil/` | Temp git repos and shared test helpers | `git.go`, `testutil.go` |
| `docs/plans/` | Design notes for larger features | open the matching date/topic file |
| `skills/` | Human-readable skill docs shipped to agents | `README.md` and `roborev-*.md` |

## Command Map

- Review queueing and results: `review.go`, `wait.go`, `status.go`, `list.go`, `show.go`, `comment.go`, `compact.go`
- Agentic fix flows: `fix.go`, `refine.go`, `run.go`, `analyze.go`
- Daemon lifecycle and transport: `daemon_cmd.go`, `daemon_client.go`, `stream.go`, `log_cmd.go`
- Repo/bootstrap/hooks: `init_cmd.go`, `hook.go`, `repo.go`, `remap.go`
- Configuration and maintenance: `config_cmd.go`, `skills.go`, `update.go`, `version.go`, `check_agents.go`
- Automation/integration: `ci.go`, `ghaction.go`, `sync.go`
- TUI entry command: `tui_cmd.go`

## Follow The Flow

### Review path

`roborev review` typically flows through:

1. CLI validation and repo/ref resolution in `cmd/roborev/review.go`
2. Daemon startup/runtime lookup in `cmd/roborev/daemon_cmd.go` and `cmd/roborev/daemon_client.go`
3. `POST /api/enqueue` handled by `internal/daemon/server.go`
4. Job persistence in `internal/storage/`
5. Prompt building in `internal/prompt/prompt.go`
6. Agent execution from `internal/agent/`
7. Review/result persistence plus broadcasts/hooks in `internal/daemon/`

### Fix path

- `roborev fix` runs agents synchronously in the foreground from `cmd/roborev/fix.go`.
- It often discovers candidate jobs through daemon APIs, but the actual code modification happens locally.
- When behavior changes, inspect both the CLI-side flow and the job state transitions in storage.

### Refine path

- `roborev refine` is the automated review-fix-review loop in `cmd/roborev/refine.go`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roborev-dev/roborev](https://github.com/roborev-dev/roborev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
