---
trigger: always_on
description: This file is the canonical repo-level instruction set for coding agents working
---

# AGENTS.md — HybridClaw Engineering Protocol

This file is the canonical repo-level instruction set for coding agents working
in HybridClaw. Read it before any code change.

## Scope

- Follow this file first.
- If a deeper directory contains its own `AGENTS.md`, that file overrides this
  one for its subtree.
- Keep `CLAUDE.md` aligned with this file. `CLAUDE.md` should only carry
  tool-specific deltas.
- `templates/*.md` are product runtime workspace bootstrap files, not repo
  contributor onboarding docs.

---

## 1) Project Snapshot

HybridClaw is a personal AI assistant bot for Discord, powered by HybridAI.
Enterprise-grade Node.js 22 application with gateway service, TUI client, and
Docker-sandboxed container runtime.

**Version:** 0.12.6 &ensp;|&ensp; **Package:** `@hybridaione/hybridclaw`
&ensp;|&ensp; **License:** see `LICENSE`

Architecture: gateway (core runtime, SQLite persistence, REST API, Discord
integration) → container (Docker-sandboxed tool execution via file-based IPC) →
TUI (thin HTTP client). Agent workspaces are bootstrapped from `templates/` and
seeded with identity, memory, and context files managed by `src/workspace.ts`.

---

## 2) Project Map

```
src/
  cli.ts                CLI entry point and command dispatch
  types.ts              Core type definitions (ChatMessage, ContainerInput, ToolExecution, etc.)
  workspace.ts          Workspace bootstrap (SOUL.md, IDENTITY.md, USER.md, etc.)
  logger.ts             Structured logging (pino)
  tui.ts                Terminal UI
  onboarding.ts         Interactive onboarding
  model-selection.ts    Model selection logic
  agent/                Agent execution: conversation loop, tool executor, prompt hooks, delegation
  audit/                Append-only audit trail, approval tracking, hash-chain integrity
  auth/                 HybridAI and OpenAI Codex authentication flows
  channels/             Channel transports (discord, slack, telegram, email, whatsapp, msteams, voice, imessage)
  config/               CLI flag parsing, runtime config management
  doctor/               Doctor checks and resource hygiene maintenance
  gateway/              Core gateway service: HTTP APIs, health, session mgmt, approvals
  infra/                Container setup, IPC (file-based), worker signatures, runners
  memory/               SQLite database, semantic memory, compaction, consolidation, chunking
  providers/            Model providers (HybridAI, Anthropic, OpenAI, Ollama, LM Studio, vLLM)
  scheduler/            Scheduled task execution and cron management
  security/             Mount allowlists, approval policies, secret redaction, instruction audit
  session/              Session transcripts, token tracking, compaction, export
  skills/               Skill resolution, installation, trust-aware guard
  utils/                Shared utilities
  media/                Media handling and context management

container/              Sandboxed runtime (separate npm package)
  src/                  Container agent runtime, tool execution, provider adapters, MCP client
  Dockerfile            Container build definition
  package.json          Container-specific deps (Playwright, agent-browser, PDF, MCP SDK)

skills/                 Bundled SKILL.md skills (pdf, docx, xlsx, pptx, office, personality, etc.)
templates/              Runtime workspace bootstrap files seeded into agent workspaces
tests/                  Vitest suites: unit, integration, e2e, live
docs/                   Static site assets, development reference docs
console/                Web console workspace package
```

### Key Data Flows

```
User message → Gateway (HTTP/Discord) → ContainerInput (JSON)
  → Container spawns (Docker sandbox, file-based IPC)
    → Agent loop (tool calls, approvals, MCP)
  → ContainerOutput (JSON) → Gateway → User
  → Session persisted (SQLite), audit logged (wire.jsonl, hash-chained)
```

### Extension Points

| Extension     | Interface / Registration                                     | Playbook |
|---------------|--------------------------------------------------------------|----------|
| Skill         | `skills/<name>/SKILL.md` frontmatter                         | §7.1     |
| Provider      | `src/providers/<name>.ts` + factory                          | §7.2     |
| MCP Server    | `~/.hybridclaw/config.json` (`mcpServers.*`) → tool namespace | §7.3     |
| Approval rule | `.hybridclaw/policy.yaml`                                    | §7.4     |
| Template      | `templates/<name>.md` + `src/workspace.ts`                   | §7.5     |

### OpenTelemetry (Distributed Tracing)

The gateway supports optional OpenTelemetry instrumentation for distributed
tracing in cloud deployments. OTel is OFF by default (zero overhead).

| Env Var                          | Purpose                                                      |
|----------------------------------|--------------------------------------------------------------|
| `OTEL_ENABLED=true`             | Enable OTel SDK initialization                               |
| `OTEL_EXPORTER_OTLP_ENDPOINT`   | OTLP collector endpoint (also enables OTel if set)           |
| `OTEL_EXPORTER_OTLP_PROTOCOL`   | `grpc` (default) or `http/protobuf`                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HybridAIOne/hybridclaw](https://github.com/HybridAIOne/hybridclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
