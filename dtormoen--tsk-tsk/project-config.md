---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Important files:
- @justfile - key development commands. These should be used when possible over raw cargo commands.
- @README.md - user facing documentation for the project. This could cover key user facing details without going into too much detail. Make sure it stays up to date with your changes.

## Architecture Overview

`tsk` implements a command pattern with dependency injection for testability. The core workflow: queue tasks → execute in containers (Docker or Podman) → create git branches for review. `tsk` can run in server mode for continuous task processing across multiple repositories.

### Key Components

- **CLI Commands** (`src/commands/`): Command handlers for all `tsk` subcommands. See README.md for the full command reference.
- **Task Management** (`src/task.rs`, `src/context/task_storage.rs`, `src/task_manager.rs`, `src/task_runner.rs`): `TaskBuilder` for task creation, SQLite-backed `TaskStorage`, task lifecycle (Queued → Running → Complete/Failed/Cancelled). Two execution paths: server-scheduled (via `add`) and inline (via `run`/`shell`).
- **Docker Integration** (`src/docker/`): `DockerImageManager` (layered image builds: base → stack → project → agent), `ProxyManager` (proxy lifecycle with per-configuration instances), `DockerManager` (container execution). Security-first with dropped capabilities and per-container network isolation.
- **Server Mode** (`src/server/`): `TskServer` daemon with `TaskScheduler` and `WorkerPool` for parallel task execution. Parent-aware scheduling, cascading failure handling, and auto-cleanup of old tasks.
- **TUI** (`src/tui/`): Interactive terminal dashboard using ratatui/crossterm. Two-panel layout (task list + log viewer) with `ServerEvent` channel from scheduler.
- **Git Operations** (`src/git.rs`, `src/git_sync.rs`, `src/git_operations.rs`, `src/repo_utils.rs`): Repository cloning with configurable copy mode (`CopyMode`: working directory overlay or committed-only), branch creation, result fetching. Uses git CLI (not libgit2) for commits and post-overlay renormalization to support clean/smudge filters (e.g., Git LFS). `GitSyncManager` for concurrent access safety. Supports submodules, git worktrees, and Git LFS.
- **Storage/Config** (`src/context/`): `AppContext` (DI container), `TskEnv` (directory paths), `TskConfig` (user/project config with layered resolution). See README.md for config file reference.
- **Agents** (`src/agent/`): `Agent` trait for AI agent integration (claude, codex, integ, no-op). Handles command building, validation, warmup, and version tracking.
- **Auto-Detection** (`src/repository.rs`): Stack detection from project files and project name from directory name.
- **Skills Marketplace** (`skills/`, `.claude-plugin/marketplace.json`): Claude Code skills following the Agent Skills open standard.

### Key Design Principles

- **Docker client is lazy**: Not part of `AppContext`; constructed only at command entry points that need containers. Commands like `add`, `list`, `clean`, `delete` work without a Docker daemon.
- **Config snapshotting**: At task creation, the resolved config is serialized and stored with the task. Execution uses the snapshot, not live config files. Chained tasks inherit the parent's snapshot.
- **Dual-lock concurrency**: `ProxySyncManager` and `GitSyncManager` both use in-process `tokio::Mutex` + cross-process `flock(2)` for safe concurrent access across threads and processes.
- **Two output paths**: Task-scoped messages use `TaskLogger` (writes to `agent.log`, optionally prints to stdout). Global messages use `emit_or_print` (sends `ServerEvent` in TUI mode, prints otherwise).
- **Graceful degradation**: Submodule setup failures fall back to regular files. Git-town errors log warnings and continue. Missing config uses defaults.

## Development Workflows

- `just test` — run the test suite
- `just format` — auto-format Rust source
- `just lint` — clippy with warnings as errors
- `just precommit` — format, lint, test, and integration tests (if in a TSK container). Run before committing.
- `just integration-test` — stack layer integration tests (requires Docker/Podman)

## Coding Conventions

- Avoid `#[allow(dead_code)]` directives
- Avoid `unsafe` blocks
- Keep documentation up to date following rustdoc best practices
- Keep CLAUDE.md simple but up to date

### Commit Conventions

Commits use conventional commit prefixes mapped to changelog groups via `release-plz.toml`:

- `feat`: New user-facing functionality (appears in release notes under "added")
- `fix`: Bug fixes to existing behavior (appears in release notes under "fixed")
- `docs`: Documentation-only changes (appears in release notes under "documentation")
- `test`: Adding or updating tests (excluded from release notes)
- `refactor`: Code restructuring with no behavior change (excluded from release notes)
- `chore`: Maintenance tasks like dependency updates, CI config, releases (excluded from release notes)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dtormoen/tsk-tsk](https://github.com/dtormoen/tsk-tsk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
