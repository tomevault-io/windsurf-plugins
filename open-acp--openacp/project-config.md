---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run

```bash
pnpm install            # Install dependencies
pnpm build              # TypeScript compile (tsc)
pnpm build:publish      # Bundle for npm publish (tsup → dist-publish/)
pnpm start              # Run: node dist/cli.js
pnpm dev                # Watch mode (tsc --watch)
pnpm test               # Run tests (vitest)
```

## Architecture

OpenACP bridges AI coding agents to messaging platforms via the Agent Client Protocol (ACP). The flow:

```
User (Telegram) → ChannelAdapter → OpenACPCore → Session → AgentInstance (ACP subprocess)
```

### Project Layout

```
src/
  cli.ts              — CLI entry (start, install, uninstall, plugins, --version, --help)
  main.ts             — Server startup, plugin boot
  index.ts            — Public API exports
  core/               — Core modules
    config/           — Zod-validated config, migrations, editor
    agents/           — Agent instance, catalog, installer, store
    sessions/         — Session, session-manager, session-bridge, permission-gate
    plugin/           — Plugin infrastructure (LifecycleManager, ServiceRegistry, MiddlewareChain, PluginContext)
    commands/         — System chat commands (session, agents, admin, help, menu)
    adapter-primitives/ — Shared adapter framework (MessagingAdapter, StreamAdapter, SendQueue, etc.)
    utils/            — Logger, typed-emitter, file utilities
    setup/            — First-run setup wizard
  plugins/            — All plugins (adapters + services)
    telegram/         — Telegram adapter (grammY)
    slack/            — Slack adapter (@slack/bolt)
    speech/           — TTS/STT (Edge TTS, Groq STT)
    tunnel/           — Port forwarding (Cloudflare, ngrok, Bore, Tailscale)
    security/         — Access control, rate limiting
    usage/            — Cost tracking, budget
    api-server/       — REST API + SSE
    file-service/     — File I/O for agents
    notifications/    — Cross-session alerts
    context/          — Conversation history
  cli/
    commands/         — CLI commands (start, plugins, dev, etc.)
    plugin-template/  — Scaffold templates for `openacp plugin create`
  packages/
    plugin-sdk/       — @openacp/plugin-sdk (types + testing utilities)
```

### Core Abstractions

**OpenACPCore** (`core.ts`) — Registers adapters, routes messages, creates sessions, wires agent events to adapters. Accesses services via ServiceRegistry.

**Session** (`session.ts`) — Wraps an AgentInstance with a prompt queue (serial processing), auto-naming, and lifecycle management.

**AgentInstance** (`agent-instance.ts`) — Spawns agent subprocess, implements full ACP Client interface. Converts ACP events to AgentEvent types.

**LifecycleManager** (`plugin/lifecycle-manager.ts`) — Boots plugins in dependency order (topo-sort), manages setup/teardown, handles version migration.

**ServiceRegistry** (`plugin/service-registry.ts`) — Central service discovery. Plugins register services, core accesses them via typed interfaces.

**CommandRegistry** (`command-registry.ts`) — Central command registry for chat commands. System and plugin commands registered here, adapters dispatch via generic handler.

**PluginContext** (`plugin/plugin-context.ts`) — Scoped API for plugins: events, services, middleware, commands, storage, logging.

### Plugin System

All features are plugins. Core only provides infrastructure (ServiceRegistry, MiddlewareChain, EventBus, LifecycleManager). Plugins register services, commands, and middleware in their `setup()` hook.

- 19 middleware hook points (message:incoming, agent:beforePrompt, permission:beforeRequest, etc.)
- 9 permission types (events:read, services:register, commands:register, etc.)
- Per-plugin settings via SettingsManager (~/.openacp/plugins/<name>/settings.json)

### Adapter Patterns

- **Topics** (Telegram): Each session gets its own topic
- **Callback routing**: Permission buttons use `p:` prefix, command buttons use `c/` prefix
- **Response renderers**: Adapters render CommandResponse types (text, menu, list, confirm, error, silent) per platform

## npm Publishing

Published as `@openacp/cli` on npm. Users install with `npm install -g @openacp/cli`.

- `pnpm build:publish` bundles CLI via tsup + builds SDK via tsc
- GitHub Action auto-publishes both `@openacp/cli` and `@openacp/plugin-sdk` on tag push (`v*`)
- Plugin system: `openacp plugin install <name>` installs from npm to `~/.openacp/plugins/`

## Versioning

Format: `YYYY.MMDD.<patch>` — e.g. `2026.327.1` is the first patch on March 27, 2026.

- `YYYY` — year
- `MMDD` — month (2-digit) + day (2-digit), leading zeros stripped by semver. Jan 5 = `105`, Mar 27 = `327`, Apr 1 = `401`, Dec 5 = `1205`
- `<patch>` — sequential patch number for that day, starting from 1

## Documentation Sync

When changing code, **you must update corresponding docs** to keep code and documentation in sync:

- **New features**: Must update README and `docs/` (GitBook) to describe the feature, usage, and config if applicable.
- **Feature updates**: Update related docs to reflect changes.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Open-ACP/OpenACP](https://github.com/Open-ACP/OpenACP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
