---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

ClawFleet — deploy and manage a fleet of isolated OpenClaw instances on a single machine, from a browser dashboard. Built on ClawSandbox, a purpose-built infrastructure layer for container orchestration and instance lifecycle management. Open-sourced on GitHub.

## Architecture Layers

ClawFleet is built on top of ClawSandbox, a purpose-built infrastructure layer
for container orchestration and instance lifecycle management.

### ClawSandbox (Infrastructure)
Packages: container/, state/, port/, config/, assets/, snapshot/, version/
Responsible for: Docker orchestration, instance state persistence, port allocation,
container networking, image management, snapshot archival.
Standard: production-grade reliability, defensive coding, thorough test coverage.

### ClawFleet (Product)
Packages: web/, cli/
Responsible for: REST API, WebSocket real-time updates, Dashboard UI, CLI commands,
asset management (models/channels/characters), skill management, i18n.
Standard: user experience, feature velocity, accessibility.

Dependency rule: ClawFleet → ClawSandbox (never reverse).

## Workflow

This is a multi-contributor project with rapid iteration. Before planning or starting any task in a session:

1. **Sync to latest main** — always pull the latest remote main branch first:
   ```bash
   git fetch origin
   git checkout main
   git pull origin main
   ```
2. **Build full context** — read the codebase, documentation (CLAUDE.md, README, SYSTEM_DESIGN, etc.), and project memory thoroughly. Understand recent changes by reviewing git log and any files touched by recent commits.
3. **Design from current state** — ensure all design decisions and implementation plans are based on a comprehensive understanding of the project's latest state, not assumptions from prior sessions.

Then create a feature branch from the up-to-date main. Never work directly on a stale branch.

**Never push directly to main.** All changes must go through a pull request, even for docs or config-only changes.

**One PR = one branch, all commits before merge.** Never push additional commits to a branch after its PR has been merged — those commits will not be on main. If you have more changes to make, create a new branch from the latest main and open a new PR. Always check `gh pr view <number> --json state` before pushing to a PR branch.

## Build/Test/Lint Commands

```bash
# Download dependencies
go mod tidy

# Build CLI binary → bin/clawfleet
make build

# Run tests
make test

# Build Docker image (first time, ~1.4GB, takes several minutes)
make docker-build
# or via the CLI itself (uses embedded Dockerfile):
./bin/clawfleet build

# Cross-compile for all platforms
make build-all
```

## Architecture

Go CLI tool (cobra) that manages Docker containers with an embedded Web Dashboard. Key packages:

- `cmd/clawfleet/` — binary entry point
- `internal/cli/` — cobra commands (build, create, list, start, stop, restart, destroy, desktop, logs, dashboard, config, version)
- `internal/container/` — Docker SDK wrappers (client, image build/check/pull, network, container lifecycle, stats)
- `internal/port/` — TCP port availability checker and allocator
- `internal/state/` — instance metadata store (`~/.clawfleet/state.json`), mutex-protected
- `internal/config/` — config file loader (`~/.clawfleet/config.yaml`)
- `internal/assets/` — embedded Docker build context (Dockerfile, supervisord.conf, entrypoint.sh)
- `internal/web/` — Web Dashboard: HTTP server, REST API handlers, WebSocket endpoints (stats/logs/events), embedded frontend
- `internal/version/` — build version info (injected via ldflags)

Each claw instance is a Docker container running: XFCE4 desktop + TigerVNC + noVNC (browser access on port 690N) + OpenClaw Gateway (port 1878N).

Container data is persisted at `~/.clawfleet/data/<name>/openclaw/` → `/home/node/.openclaw` inside the container.

## OpenClaw Integration

ClawFleet manages OpenClaw instances via `docker exec` CLI commands. Key integration points:

### Character / SOUL.md
- OpenClaw uses `SOUL.md` (Markdown) at `~/.openclaw/SOUL.md` for character/persona definition
- Gateway watches this file — hot-reloads on change, no restart needed
- ClawFleet renders `CharacterAsset` fields into SOUL.md and writes via `docker exec`

### Skills
- **Bundled Skills** (52): Ship with OpenClaw, status depends on binary/env requirements
- **Managed Skills**: Installed via `clawhub` CLI to `~/.openclaw/skills/`
- `openclaw skills list --json` returns structured skill data
- `npx clawhub --workdir ~/.openclaw --dir skills install/uninstall <slug>` manages community skills
- ClawHub has rate limits (~20 requests/minute) — handle errors gracefully

### Useful CLI Commands (run as `node` user inside container)
- `openclaw skills list --json` — list all skills with status
- `openclaw plugins list` — list all plugins (41 stock plugins)
- `openclaw config set <path> <value>` — set any config value
- `npx clawhub search "<query>"` — search community skills
- `npx clawhub --workdir /home/node/.openclaw --dir skills install <slug> --no-input` — install skill

## Engineering Principles


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [clawfleet/ClawFleet](https://github.com/clawfleet/ClawFleet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
