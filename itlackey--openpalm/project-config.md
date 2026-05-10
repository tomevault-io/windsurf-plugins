---
trigger: always_on
description: > **CRITICAL:** All work must comply with [`docs/technical/core-principles.md`](docs/technical/core-principles.md).
---

# AGENTS.md — OpenPalm

> **CRITICAL:** All work must comply with [`docs/technical/core-principles.md`](docs/technical/core-principles.md).
> That document is the **authoritative source of architectural rules** for this project.
> No implementation may violate its Core Goals, Security Invariants, or Filesystem Contract.
> **IT IS VERY IMPORTANT THAT YOU AVOID AND/OR REMOVE ALL COMPLEXITY THAT YOU CANNOT PROPERLY JUSTIFY. ALWAYS CALLOUT ANY COMPLEXITY THAT YOU FIND AND CANNOT JUSTIFY**

---

## Project Overview

OpenPalm is a self-hosted personal AI platform built on Docker Compose and OpenCode. It manages a stack of containers orchestrated by the host CLI or an optional admin web UI.

Four core containers: **guardian** (HMAC ingress), **assistant** (OpenCode runtime), **memory** (vector-backed agentic memory), **scheduler** (cron/automations). Channels (chat, API, Discord, Slack, voice) and services (Ollama, etc.) are added as addon compose overlays.

Repo layout convention:
- `packages/*` — app/package source workspaces
- `core/*` — container/runtime assembly assets and image build contexts

```
CLI (host)            ->  Docker Compose (lifecycle)    <- primary orchestrator
Admin UI / Assistant  ->  Admin API  ->  Docker Compose  <- optional web orchestrator
External clients      ->  Channel    ->  Guardian (HMAC/validate)  ->  Assistant
```

See [`docs/technical/core-principles.md`](docs/technical/core-principles.md) for the filesystem/volume-mount contract.

---

## Architecture

- **Lib** (`packages/lib/`) — Shared control-plane library (`@openpalm/lib`). All portable lifecycle, staging, secrets, channels, connections, scheduler logic. Both CLI and admin import from this package.
- **CLI** (`packages/cli/`) — Host-side orchestrator. Manages Docker Compose directly. Serves setup wizard during install. Self-sufficient without admin.
- **Admin** (`packages/admin/`) — SvelteKit app: optional operator web UI + API. Uses Docker socket via docker-socket-proxy. Behind `profiles: ["admin"]` compose profile.
- **Guardian** (`core/guardian/`) — Bun HTTP server: HMAC verification, replay detection, rate limiting for all channel traffic.
- **Assistant** (`core/assistant/`) — OpenCode runtime with tools/skills. No Docker socket. When admin is present, calls Admin API for stack operations.
- **Memory** (`packages/memory/`, `core/memory/`) — Bun-based memory service with sqlite-vec. Provides vector-backed agentic memory.
- **Scheduler** (`packages/scheduler/`) — Lightweight Bun sidecar: sole automation engine. Runs cron jobs (http, shell, assistant, api actions). Reads enabled automation files from `config/automations/`.
- **Channel runtime** (`core/channel/`) — Unified `channel` image build and startup entrypoint.
- **Channel adapters** (`packages/channel-chat/`, `packages/channel-api/`, `packages/channel-discord/`, `packages/channel-slack/`, `packages/channel-voice/`) — Translate external protocols into signed guardian messages.
- **Channels SDK** (`packages/channels-sdk/`) — Shared SDK for channel adapters: signing, assistant client, base classes.
- **Assistant-tools** (`packages/assistant-tools/`) — Memory tools and session hooks for the assistant. No admin dependency.
- **Admin-tools** (`packages/admin-tools/`) — Admin API tools for the assistant. Only loaded when admin is present.
- **Stack** (`.openpalm/stack/`) — Repo-shipped Docker Compose foundation. Contains the core compose file only. Runtime enabled addons live under `~/.openpalm/stack/addons/`.

---

## Commands

### Development

```bash
# Admin (SvelteKit admin + API)
cd packages/admin && npm install && npm run dev     # Dev server on :8100
npm run build                                       # Production build
npm run check                                       # svelte-check + TypeScript

# Guardian (Bun)
cd core/guardian && bun install && bun run src/server.ts

# Channel Chat (Bun)
cd packages/channel-chat && bun install && bun run src/index.ts

# Root shortcuts
bun run admin:dev        # Runs admin dev from root
bun run admin:build      # Builds admin from root
bun run admin:check      # svelte-check + TypeScript for admin
bun run guardian:dev     # Runs guardian server
bun run channel:chat:dev    # Runs chat channel dev server
bun run channel:api:dev     # Runs api channel dev server
bun run channel:discord:dev # Runs discord channel dev server

# Dev environment setup
./scripts/dev-setup.sh --seed-env       # Creates .dev/ dirs, seeds configs

# Setup wizard (dev)
bun run wizard:dev                      # Runs install --no-start --force with OP_HOME=.dev
```

### Type Checking

```bash
cd packages/admin && npm run check
# or from root:
bun run check            # Runs admin:check + sdk:test
```

### Tests

The project has ~100 test files across all packages using Bun test, Vitest, and Playwright.

| Runner | Command | Scope |
|--------|---------|-------|
| `bun test` (root) | `bun run test` | channels-sdk, guardian, cli, all channel packages (excludes admin) |
| `bun test` (sdk) | `bun run sdk:test` | packages/channels-sdk unit tests |
| `bun test` (guardian) | `bun run guardian:test` | core/guardian security tests |
| `bun test` (cli) | `bun run cli:test` | packages/cli tests |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [itlackey/openpalm](https://github.com/itlackey/openpalm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
