---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**schmux** is a multi-agent AI orchestration system that runs multiple AI coding agents (Claude Code, Codex, etc.) simultaneously across tmux sessions, each in isolated workspace directories. A web dashboard provides real-time monitoring and management.

## ⚠️ Worktree Isolation — Do Not Cross Workspace Boundaries

Each schmux agent is sandboxed to its own worktree. Do not edit, write, or modify files in another worktree's directory (e.g., `/Users/stefanomaz/code/workspaces/schmux-NNN/` where NNN is a different agent's worktree) — these operations fail with `EPERM`. Do not `git checkout` a branch already checked out in another worktree — Git rejects this with "already used by worktree". If you need changes from another branch, create a new branch from it instead.

## ⚠️ React Dashboard Builds — Use Go Wrapper, NOT npm

**NEVER run `npm install`, `npm run build`, or `vite build` directly.**

The React dashboard MUST be built via `go run ./cmd/build-dashboard`. This Go wrapper:

- Installs npm deps correctly
- Runs vite build with proper environment
- Outputs to `assets/dashboard/dist/` which gets embedded in the Go binary

❌ **WRONG**: `cd assets/dashboard && npm install && npm run build`
✅ **RIGHT**: `go run ./cmd/build-dashboard`

## ⚠️ Frontend Tests — Use `./test.sh`, NOT `npx vitest` directly

**NEVER run frontend tests by `cd`-ing into `assets/dashboard/` and invoking `npx vitest run` or similar commands directly.**

Frontend tests are already included in `./test.sh --quick`. Running vitest from the subdirectory bypasses the project test wrapper and produces unreliable results.

❌ **WRONG**: `cd assets/dashboard && npx vitest run`
✅ **RIGHT**: `./test.sh --quick` (from repository root)

## ⚠️ E2E Tests — Use `./test.sh --e2e`, NOT `go test` directly

**NEVER run E2E tests with `go test -tags e2e` directly** — they immediately fail with "E2E tests must run in Docker". Always use `./test.sh --e2e` which handles Docker image building and container execution automatically.

❌ **WRONG**: `go test -tags e2e ./...`
✅ **RIGHT**: `./test.sh --e2e`

## Hot-Reload Development Mode

For active development with automatic rebuilding:

```bash
./dev.sh
```

This runs the Go backend and React frontend (via Vite) with workspace switching support:

- **Go changes**: Trigger rebuild from the dashboard Dev Mode panel
- **React changes**: Instant browser update via HMR (<100ms)
- **Workspace switching**: Switch which worktree's code is running from the dashboard UI
- **Access**: http://localhost:7337 (same URL as production)
- **Stop**: Ctrl+C

The dev mode panel in the sidebar lets you switch between workspaces:

- **FE**: Restart Vite pointed at a different worktree's `assets/dashboard/`
- **BE**: Rebuild Go binary from a different worktree, restart daemon
- **Both**: Both frontend and backend switch

Build failures are safe — the old binary keeps running if the new build fails.

First run installs npm dependencies if missing.

## Build, Test, and Run Commands

```bash
# Hot-reload development (recommended for active development)
./dev.sh

# Build the binary (outputs ./schmux)
go build ./cmd/schmux

# Generate TypeScript types from Go contracts:
go run ./cmd/gen-types

# Build the React dashboard (see warning above)
go run ./cmd/build-dashboard

# Run all tests (unit + E2E + scenarios) - RECOMMENDED
./test.sh --all

# Run tests with various options
./test.sh              # All tests (same as --all)
./test.sh --quick      # Fast tests only (backend + frontend, no Docker)
./test.sh --race       # All tests with race detector
./test.sh --coverage   # All tests with coverage report
./test.sh --e2e        # E2E tests only (requires Docker)
./test.sh --scenarios  # Scenario tests only (Playwright, requires Docker)
./test.sh --help       # See all options

# Or run tests directly with go
go test ./...          # Unit tests only
go test -race ./...    # Unit tests with race detector
go test -cover ./...   # Unit tests with coverage

# Build and run E2E tests manually
docker build -f Dockerfile.e2e -t schmux-e2e .
docker run --rm schmux-e2e

# Daemon management (requires config at ~/.schmux/config.json)
./schmux start      # Start daemon in background
./schmux stop       # Stop daemon
./schmux status     # Show status + dashboard URL
./schmux daemon-run # Run daemon in foreground (debug)
```

## Pre-Commit Requirements

**ALWAYS use `/commit` to create commits. NEVER run `git commit` directly.**

The `/commit` command enforces the definition of done before every commit:

- Runs `./test.sh` and aborts if tests fail
- Runs `./badcode.sh` (static analysis: deadcode, staticcheck, govulncheck, knip, npm audit, tsc) and aborts if it fails
- Checks that `docs/api.md` is updated when API-related packages change
- Requires a structured self-assessment (tests written, no architecture drift, docs current)

Before the `/commit` command runs, ensure:

1. **Format code**: `./format.sh` (or let the pre-commit hook handle it automatically)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sergeknystautas/schmux](https://github.com/sergeknystautas/schmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
