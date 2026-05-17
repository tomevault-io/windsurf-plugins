---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Isotopes is a self-hostable AI agent framework for multi-agent collaboration across chat platforms (Discord today, Feishu future). Agents have self-evolving prompts (SOUL.md, MEMORY.md), per-account routing, cron automation, sandbox execution, and daemon mode.

## Commands

```bash
pnpm install           # Install dependencies (pnpm is canonical, not npm)
pnpm build             # Compile TypeScript (plain tsc -> dist/)
pnpm dev               # Run without building (tsx src/legacy/cli.ts)
pnpm lint              # ESLint
pnpm lint:fix          # ESLint with auto-fix
pnpm typecheck         # tsc --noEmit
pnpm test              # Vitest (unit tests only, excludes integration/)
pnpm test:watch        # Vitest in watch mode
pnpm ci                # lint + typecheck + test (full local validation)

# Single test file
npx vitest run src/agent/tools/index.test.ts

# Single test by name
npx vitest run -t "registers a tool"

# Integration tests (requires DISCORD_TOKEN + DISCORD_TEST_CHANNEL env vars)
pnpm test:integration
```

## Architecture

**Module resolution**: ESM-only (`"type": "module"`). All imports use `.js` extensions (NodeNext resolution). Target: ES2022. Node >= 20.

### Top-level src/ layout

- `agent/` — Agent runtime, runners, tools, workspace loading, and per-agent host/sandbox middleware (executor, fs bridge, docker container manager, sandbox config). The new home for everything that defines what an agent *is* and how it runs.
- `gateway/` — Typed Gateway abstraction (steer-only): the canonical entrypoint for channel adapters to dispatch inbound messages, stream callbacks, and abort sessions.
- `channels/` — Channel adapters. Today: `channels/discord/` (full Discord adapter — inbound pipeline, outbound streaming, dedupe, channel history, image attachments, /stop interception, A2A sink for spawn_agent threads, react, allowlists).
- `sessions/` — Session type definitions only; the in-memory + JSONL impl lives in `agent/pi/session-store.ts`.
- `automation/` — `CronScheduler` (cron-based task scheduling) and `HeartbeatManager` (periodic agent wake-ups). `types.ts` holds the config-shape `CronActionConfig`.
- `daemon/` — macOS-only LaunchAgent install/uninstall/restart/status (`launchd.ts`). Other platforms: run `isotopes` in the foreground or supervise it yourself.
- `init/` — `isotopes init` setup wizard built with Ink.
- `logging/` — `createLogger("tag")` factory.
- `extensions/` — Discovery for user-managed customization at `~/.isotopes/extensions/`. Three typed slots: `pi/loader.ts` (pi-coding-agent extensions from `~/.isotopes/extensions/pi/*.ts`), `ui/loader.ts` (static SPA dirs from `~/.isotopes/extensions/ui/<id>/`, mounted at `/ui/<id>`), and `channels/loader.ts` (loads built-in channel adapters from `channels/`).
- `legacy/` — Transitional area being decomposed PR-by-PR. New code should not land here.
- Standalone files: `app.ts` (daemon wiring), `config.ts` (YAML config + schema), `paths.ts` (`ISOTOPES_HOME` resolution), `test-helpers.ts` (shared test mocks).

### `src/agent/`

- `runtime.ts` — `AgentRuntime`: in-memory agent registry + per-run dispatcher. Validates `RunRequest`, resolves session ID, delegates to a runner.
- `runtime-adapter.ts` — Chat-style decorator over `runtime.run` for callers that want a single `responseText` instead of an event stream.
- `types.ts` — `RegisteredAgent`, `RunRequest`, `RunInfo`, `AgentConfig`, `ProviderConfig`, `RunValidationError`.
- `pi/` — Pi backbone (default agent runtime, not a swappable adapter). Wraps `@mariozechner/pi-agent-core` + `@mariozechner/pi-coding-agent`: `runner.ts`, `session-factory.ts`, `session-store.ts`, `messages.ts`, `system-prompt-override.ts`, `tool-result-truncation.ts`. Other isotopes modules (transports, HTTP, gateway) are allowed to depend on these directly — pi is the host, not a guest.
- `adapters/claude/` — Third-party adapter for the Claude Agent SDK. Implements the same `Runner` interface but lives under `adapters/` to signal "alternative entry point, not the backbone".
- `tools/` — Built-in agent tools (`web` for `web_fetch`, `react` for channel reactions) and the registry (`index.ts`) that assembles per-agent tool sets.
- `workspace/` — Loads `SOUL.md` / `TOOLS.md` / `MEMORY.md` / `BOOTSTRAP.md` into system prompts; manages workspace state and template files.

### `src/legacy/` (transitional)

- `cli.ts` — CLI entry point. Parses args, dispatches subcommands, runs foreground. Includes `isotopes service install/uninstall/restart/status` for macOS LaunchAgent management. Dynamically imports `init/wizard.tsx` and `tui/index.tsx`.
- `tui/` — Terminal UI for interactive chat mode.
- `http/` — REST API server using raw Node `http` (no Express); routes for chat, sessions, cron, logs, status. Instantiated directly from `app.ts`.
- `version.ts` — Build version constant.

### Key patterns

- **Pluggable runner**: `AgentRuntime` dispatches to a runner per agent (`pi` default, `claude` alternative). Swap runners without touching gateway, sandbox, or transport code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GhostComplex/isotopes](https://github.com/GhostComplex/isotopes) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
