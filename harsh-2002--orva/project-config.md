---
trigger: always_on
description: Self-hosted Function-as-a-Service (FaaS) for homelab and on-premises use. Users write JavaScript (Node.js 24), Python (3.14), or TypeScript functions — two generic runtimes, `node` and `python`, latest-stable only; Orva deploys them into nsjail sandboxes and exposes them over HTTP with a built-in dashboard, CLI, MCP server, and an in-product AI chat assistant (the **Chat** sidebar item, route `/web/ai`) that operates the instance end-to-end via in-process tool calling (BYO provider keys, embedde
---

# Orva

Self-hosted Function-as-a-Service (FaaS) for homelab and on-premises use. Users write JavaScript (Node.js 24), Python (3.14), or TypeScript functions — two generic runtimes, `node` and `python`, latest-stable only; Orva deploys them into nsjail sandboxes and exposes them over HTTP with a built-in dashboard, CLI, MCP server, and an in-product AI chat assistant (the **Chat** sidebar item, route `/web/ai`) that operates the instance end-to-end via in-process tool calling (BYO provider keys, embedded Bifrost gateway).

> **Operational contract:** read [`CONTRACT.md`](CONTRACT.md) **before proposing changes** — the canonical commands, build/CI invariants, ports, and must-not-break rules. Each directory's `CLAUDE.md` (mirrored as `AGENTS.md` so Codex/opencode read it too) covers how that subsystem works.
>
> **Docs ship with the code that changed them.** If your change alters anything an
> operator or a function author can observe, update the affected docs **in the same
> commit** — [`CONTRACT.md` §6a](CONTRACT.md) has the table of what to touch for what.
> This is not tidiness: a 2026-08-25 audit of every document against the source found
> 181 defects, 81 factually wrong, including a handler contract whose two headline
> examples produced an HTTP 500 and a silently wrong answer. It accumulated one
> un-updated commit at a time. Note that `frontend/src/utils/aiPrompts.js` counts as
> documentation — it is the prompt the in-product assistant generates code from, so a
> stale claim there ships as broken code rather than a stale sentence.

@CONTRACT.md

## Quick Start

```bash
# Docker (recommended)
docker compose up -d
# → dashboard at http://localhost:3000  (compose maps host 3000 → container 8443)

# Dev mode (frontend hot-reload + backend foreground process)
make dev
```

## Build Commands

```bash
make build          # backend binary → build/orva  (calls adapters-embed + docs-embed)
make build-all      # embed UI then build           (full release artifact)
make test           # go test -count=1 ./...  (from repo root)
make lint           # go vet ./...  (from repo root)
make ui             # cd frontend && npm install && npm run build
make embed          # build UI, copy dist/ → backend/internal/server/ui_dist/
make cli            # static CLI binary → build/orva (current OS)
make cli-all        # cross-compile CLI: linux/{amd64,arm64}, darwin/{amd64,arm64}, windows/{amd64,arm64}
make adapters-embed # sync runtimes/ → backend/cmd/orva/adapters/ (auto-called by build)
make docs-embed     # sync docs/reference.md → mcp + frontend (auto-called by build/ui)
make clean          # remove build/ and embedded artefacts
```

## Repo Layout

```
go.mod, go.sum    Single Go module rooted at the repo (covers backend/ + cli/ + internal/)
backend/          Go server (see backend/CLAUDE.md)
  cmd/orva/       Server entry: registers commands.NewRoot() + serve/setup
  internal/       Server packages (config, database, pool, proxy, mcp, …)
  runtimes/       Runtime adapter source: node, python
cli/              Slim standalone CLI codebase (see cli/CLAUDE.md)
  cmd/orva/       Slim CLI entry point (no server packages — ~20 MB binary)
  commands/       Cobra subcommand library — single source of truth for
                  both binaries (server imports it for its CLI surface)
internal/         Shared utilities accessible to both backend/ and cli/
  client/         HTTP client + ~/.orva/config.yaml loader
  ids/            UUIDv7 generator
frontend/         Vue 3 dashboard (see frontend/CLAUDE.md)
docs/             Operator and developer documentation (see docs/CLAUDE.md)
scripts/          Installers (install.sh = server, install-cli.{sh,ps1} = CLI),
                  Docker entrypoint (entrypoint.sh); the systemd + OpenRC units
                  are emitted inline by install.sh, not separate files
test/             Shell-based integration test suite (see test/CLAUDE.md)
  cli/            CLI-specific tests (build matrix, install-cli, upgrade, command-tree)
  install/        Server-install e2e harness (privileged systemd-in-docker)
Makefile          All build/test/release targets
docker-compose.yml  Single-node Docker deployment
Dockerfile        Multi-stage image (dev and production — single file)
```

## Data & Configuration

- **Data dir**: `/var/lib/orva` (Docker volume `orva-data`) — contains `orva.db` (SQLite WAL) and `functions/<id>/versions/`
- **Server config**: environment variables only; full reference in `docs/CONFIG.md`
- **CLI config**: `~/.orva/config.yaml` with `endpoint` and `api_key`

## Release Policy

**Two-stage policy: verify on push/PR, ship on tag.** The release workflow does
**no testing** — it gates on the tagged commit's checks already being green, then builds and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Harsh-2002/Orva](https://github.com/Harsh-2002/Orva) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
