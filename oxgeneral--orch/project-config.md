---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Positioning

ORCH is an **AI agent runtime** — not just a CLI tool. The core engine (domain + application + infrastructure) has zero dependencies on CLI/TUI layers. The npm package `@oxgeneral/orch` exports the full engine API via `src/index.ts`, enabling programmatic use from any Node.js application.

**User-facing messaging**: "Turn your AI tools into a coordinated engineering team"
**Technical category**: "AI agent runtime"
**Internal architecture term**: "ORCH Engine"

## Commands

```bash
npm run build          # Build ESM + DTS bundles (tsup)
npm run dev            # Run via tsx (no build needed)
npm test               # Run all tests (vitest)
npm test -- test/unit/application/orchestrator-resilience.test.ts  # Single file
npm test -- --grep "state machine"                                  # By pattern
npm run typecheck      # tsc --noEmit (strict mode)
npm run coverage       # Tests with coverage report
```

Release: `./scripts/release.sh patch|minor|major && git push && git push --tags`

## Architecture

Layered DDD with dependency injection — no frameworks, no decorators.

```
Domain (models, state machine, errors)
  ↓
Application (services, orchestrator, event bus)
  ↓
Infrastructure (adapters, file storage, process manager, templates, workspaces)
  ↓
CLI (Commander.js) / TUI (Ink/React)
```

### Container (DI)

`src/container.ts` defines two tiers:
- **LightContainer**: stores + services only — used by read-only commands (task, agent, logs, config, context, msg, goal, team, status)
- **Container** (extends Light): + orchestrator + adapters + ProcessManager + LiquidJS — used by run, tui, doctor

The CLI entry point (`src/bin/cli.ts`) determines which tier to build based on the subcommand, then lazy-imports only the needed command module.

### State Machine

`src/domain/transitions.ts` — tasks flow through: `todo → in_progress → review → done`, with `retrying` and `failed` branches. All transitions validated by `canTransition()`. Terminal statuses: done, failed, cancelled.

### Orchestrator Tick Loop

`src/application/orchestrator.ts` runs three phases per tick:
1. **Reconcile** — check PIDs, detect stalls, handle zombies
2. **Dispatch** — claim idle agents, launch adapter processes
3. **Collect** — process completed runs, update stats, transition tasks

Uses a promise-chain mutex (`stateMutex`) for serializing state mutations. Lock ownership checked via `isOwner` getter.

### Adapter System

`src/infrastructure/adapters/interface.ts` — adapters implement `IAgentAdapter`:
- `execute(params)` spawns an external process, returns `{ pid, events: AsyncGenerator<AgentEvent> }`
- Available: claude (Claude CLI), opencode (OpenCode, multi-provider), codex (OpenAI), cursor (Cursor IDE), shell (any command)
- Registered in `AdapterRegistry`, resolved by agent's `adapter` field

### Storage

All state in `.orchestry/` — no database:
- Tasks/Agents/Goals/Teams: individual YAML files
- Runs: JSON metadata + append-only JSONL events
- State: single `state.json`
- Context/Messages: JSON files

Key utilities in `src/infrastructure/storage/fs-utils.ts`:
- `atomicWrite()` — temp file + rename for corruption safety
- `readJsonlTail()` — reads last N lines without loading entire file (OOM protection)
- All stores use `Promise.all()` for parallel file reads

### TUI

`src/tui/App.tsx` — Ink/React dashboard with batched message queue (80ms flush) to prevent OOM from high-frequency events. Detail strings capped at 2KB. RunId maps have LRU cap of 500.

### Skill Library

`src/infrastructure/skills/skill-loader.ts` — loads plain Markdown skills from `skills/library/` (26 skills). Two types:
- **Library skills**: alphanumeric+hyphen names (e.g. `qa`, `review`, `ship`) — Markdown content injected into agent prompts at dispatch time
- **MCP skills**: colon-separated names (e.g. `testing-suite:generate-tests`) — handled natively by Claude CLI, skipped by SkillLoader

`SkillLoader` has process-lifetime in-memory cache, parallel reads via `Promise.all()`, and path traversal prevention (`VALID_SKILL_NAME` regex). Skills are auto-loaded by the orchestrator during dispatch based on each agent's `skills` field.

### Serve Mode

`orch serve` — headless daemon for CI/CD and server deployments. Files: `src/cli/commands/serve.ts`, `src/cli/serve/once-runner.ts`, `src/cli/serve/structured-logger.ts`.

- `--once` — process all todo tasks, exit when all reach terminal status (exit code 1 if any failed)
- `--log-format json|text` — structured JSON (default) or human-readable text logs to stdout
- `--log-file <path>` — append logs to file alongside stdout
- `--tick-interval <ms>` — override polling interval
- `--verbose` — include high-frequency `agent:output` events

`StructuredLogger` transforms `OrchestratorEvent` into flat log records. Idle tick throttling (every 6th idle tick logged) prevents log spam. Graceful shutdown with stream flush. Designed for pm2, systemd, or background execution.

## Conventions

- **ESM**: `"type": "module"` — all local imports require `.js` extension (`import { Foo } from './foo.js'`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oxgeneral/ORCH](https://github.com/oxgeneral/ORCH) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
