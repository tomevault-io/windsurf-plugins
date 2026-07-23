---
trigger: always_on
description: roborev is an automatic code review daemon for git commits. It runs locally, triggered by post-commit hooks, and uses AI agents (Codex, Claude Code, Gemini, Copilot, etc.) to review commits in parallel. It also supports background fix jobs, CI integration via GitHub PRs, and PostgreSQL sync for multi-machine setups.
---

# CLAUDE.md

## Project Overview

roborev is an automatic code review daemon for git commits. It runs locally, triggered by post-commit hooks, and uses AI agents (Codex, Claude Code, Gemini, Copilot, etc.) to review commits in parallel. It also supports background fix jobs, CI integration via GitHub PRs, and PostgreSQL sync for multi-machine setups.

## General Workflow

When a task involves multiple steps (e.g., implement + commit + PR), complete ALL steps in sequence without stopping. If creating a branch, committing, and opening a PR, finish the entire chain.

### Handling review findings

When the user pastes review findings directly in the conversation (verdicts, severities, file paths, suggested fixes), treat them as direct instructions and fix the code normally. Do NOT invoke the `/roborev-fix` skill — that skill is only for discovering and fetching open reviews from the daemon, not for processing findings already present in the prompt.

## Go Development

After making any Go code changes, always run `go fmt ./...` and `go vet ./...` before committing. Stage ALL resulting changes, including formatting-only files.

## Git Workflow

Always commit after completing each piece of work — do not wait to be asked. When committing changes, always stage ALL modified files (including formatting, generated files, and ancillary changes). Run `git diff` and `git status` before committing to ensure nothing is left unstaged.

## Architecture

```
CLI (roborev) → HTTP API → Daemon (roborev daemon run) → Worker Pool → Agents
                              ↓                              ↓
                          SQLite DB ←──sync──→ PostgreSQL (optional)
                              ↓
                       CI Poller → GitHub PRs
```

- **Daemon**: HTTP server on port 7373 (auto-finds available port if busy)
- **Workers**: Pool of 4 (configurable) parallel review workers
- **Storage**: SQLite at `~/.roborev/reviews.db` with WAL mode
- **Config**: Global at `~/.roborev/config.toml`, per-repo at `.roborev.toml`
- **Data dir**: Set `ROBOREV_DATA_DIR` env var to override `~/.roborev`
- **Color mode**: `ROBOREV_COLOR_MODE=auto|dark|light|none` controls TUI color theme; `NO_COLOR=1` strips all colors
- **Runtime info**: Daemon writes PID/addr/version to `~/.roborev/runtime/daemon.<pid>.json` (kit runtime store); pre-v0.57 daemons wrote `~/.roborev/daemon.json`, still read for upgrade-time stop/cleanup

## Package Map

| Package | Purpose |
|---------|---------|
| `cmd/roborev/` | CLI entry point, 25+ Cobra subcommands |
| `cmd/roborev/tui/` | Bubbletea terminal UI (queue/review/task views) |
| `internal/agent/` | Agent interface, registry, 11 implementations |
| `internal/config/` | Config structs, loading, 20+ Resolve* functions |
| `internal/daemon/` | HTTP server, worker pool, CI poller, hooks, SSE |
| `internal/storage/` | SQLite + PostgreSQL, schema migrations, job CRUD |
| `internal/prompt/` | Review prompt construction, system prompts |
| `internal/git/` | Git operations (diff, commit info, branches) |
| `internal/worktree/` | Isolated git worktrees for fix jobs |
| `internal/review/` | Synthesis, batch processing, verdict parsing |
| `internal/github/` | GitHub REST API wrappers |
| `internal/githook/` | Git hook installation/management |
| `internal/ghaction/` | GitHub Actions integration |
| `internal/kata/` | Kata task-ledger client (CLI shell-out), ref parsing, context resolution |
| `internal/skills/` | Agent skill discovery and management |
| `internal/streamfmt/` | Streaming output formatting |
| `internal/testutil/` | Test helpers (TestRepo, HTTP fixtures) |
| `internal/testenv/` | Test environment setup |
| `internal/update/` | Self-update mechanics |
| `internal/version/` | Version string (injected at build time) |

## Key Files

| Path | Purpose |
|------|---------|
| `cmd/roborev/main.go` | CLI entry point, all Cobra commands |
| `internal/daemon/server.go` | HTTP API routes and handlers (~2000 lines) |
| `internal/daemon/worker.go` | Worker pool, job processing, retry/failover |
| `internal/daemon/ci_poller.go` | GitHub PR polling, synthesis, comment posting |
| `internal/storage/db.go` | SQLite schema definition, 18 migrations |
| `internal/storage/jobs.go` | Job CRUD, state transitions, verdict parsing |
| `internal/storage/models.go` | Core types: ReviewJob, Repo, Commit, Review |
| `internal/storage/postgres.go` | PostgreSQL schema (v1-v6) and operations |
| `internal/storage/sync.go` | Sync state management (cursors, machine IDs) |
| `internal/agent/agent.go` | Agent interface, registry, alias resolution |
| `internal/config/config.go` | Config/RepoConfig structs, Resolve* functions |
| `internal/prompt/prompt.go` | Prompt builder (single, range, dirty) |
| `internal/worktree/worktree.go` | Worktree create/patch-capture/apply |
| `internal/review/synthesis.go` | Multi-agent review synthesis for CI |
| `internal/kata/client.go` | Kata CLI client (Binding, List, Show, Create) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kenn-io/roborev](https://github.com/kenn-io/roborev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
