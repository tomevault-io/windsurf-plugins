---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What is Talon?

Talon (`talond`) is a self-hosted autonomous AI agent daemon (~22K lines TypeScript). It receives messages from humans across multiple channels (Telegram, Slack, Discord, WhatsApp, Email, Terminal), processes them through a durable queue, runs agents through a provider layer (Codex default, with Gemini CLI and Codex CLI supported), executes tools through capability-gated host-tools, and sends responses back. All data stays on the operator's hardware.

## Build & Development Commands

```bash
npm run build          # tsc + copy SQL migrations to dist/
npm run dev            # tsx watch src/index.ts
npm test               # vitest run (full suite — SLOW, ask before running)
npm run test:watch     # vitest watch mode
npx vitest run tests/unit/queue/queue-manager.test.ts  # single test file
npm run lint           # eslint src/**/*.ts
npm run format         # prettier src + tests
```

Entry points: `node dist/index.js` (daemon), `node dist/cli/index.js` (CLI/talonctl).

## Architecture Overview

### Message Flow

```
Channel Connector → MessagePipeline (normalize, dedup, route, persist)
  → Durable Queue (SQLite-backed FIFO per thread)
  → QueueProcessor (concurrency-limited dequeue)
  → AgentRunner (provider runtime with provider-specific session handling)
  → Host-Tools Bridge (Unix socket MCP server, capability-filtered)
  → Channel Connector (format + send response)
```

### Source Layout

| Module    | Path                       | Purpose                                                             |
| --------- | -------------------------- | ------------------------------------------------------------------- |
| Daemon    | `src/daemon/`              | Lifecycle state machine, agent runner, bootstrap, watchdog          |
| Channels  | `src/channels/connectors/` | 7 adapters: telegram, slack, discord, whatsapp-business, whatsapp-baileys, email, terminal |
| Pipeline  | `src/pipeline/`            | Inbound normalization, dedup, routing, persistence                  |
| Queue     | `src/queue/`               | Durable SQLite queue, retry with exponential backoff, dead-letter   |
| Scheduler | `src/scheduler/`           | Cron/interval/one-shot task execution                               |
| Memory    | `src/memory/`              | Per-thread fact/summary/note storage + context assembly             |
| Tools     | `src/tools/`               | 11 host-tools + capability-based filtering via `tool-filter.ts`     |
| MCP       | `src/mcp/`                 | MCP server registry and lifecycle                                   |
| Personas  | `src/personas/`            | Persona config loading + capability merging                         |
| Skills    | `src/skills/`              | Declarative skill bundles with lazy loading (metadata-only in system prompt, full content on demand via `skill_load` tool) |
| SubAgents | `src/subagents/`           | Loader, model resolver, runner with per-subagent model overrides and failover |
| Config    | `src/core/config/`         | Zod-validated YAML config loader (`config-schema.ts` is the schema) |
| Database  | `src/core/database/`       | better-sqlite3 wrapper, 14 repositories, SQL migrations             |
| IPC       | `src/ipc/`                 | Unix socket daemon↔CLI communication                                |
| CLI       | `src/cli/`                 | 36 talonctl commands (Commander.js)                                 |

### Key Architectural Decisions

- **neverthrow `Result<T, E>`** everywhere — expected errors are typed, no raw throws across module boundaries. All repository methods return `Result`.
- **SQLite (better-sqlite3)** with WAL mode — single-file, no external DB dependency. Repository pattern allows future migration.
- **Provider runtime runs on host** (not in container) — AgentRunner executes the configured provider strategy. Codex uses the SDK path with `sessionId` persistence; Gemini and Codex use CLI strategies.
- **Capability-based security** — default-deny. Persona `capabilities.allow` lists what tools/channels are accessible. `requireApproval` triggers human confirmation.
- **Skills are declarative** — two formats: `skill.yaml` + `prompts/*.md` (legacy) or single `SKILL.md` with YAML frontmatter (preferred). No executable code in skills.
- **Lazy skill loading** — only skill name + description injected into system prompts. Full content loaded on demand via `skill_load` tool (in-process MCP server for Codex SDK, external MCP server for Gemini CLI, Codex CLI, and openai-compatible). Per-skill `eager: true` opt-in in frontmatter forces the body into the system prompt at startup (use for reflexive skills on small models). Background agents use eager loading.
- **Internal MCP server prefix** — `__talond_` prefix is reserved for internal MCP servers (`__talond_host_tools`, `__talond_skill_loader`). User-defined servers with this prefix are rejected.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ivo-toby/talon](https://github.com/ivo-toby/talon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
