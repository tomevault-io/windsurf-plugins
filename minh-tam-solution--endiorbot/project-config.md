---
trigger: always_on
description: <!-- From: /path/to/endiorbot/AGENTS.md -->
---

<!-- From: /path/to/endiorbot/AGENTS.md -->
# AGENTS.md - EndiorBot AI Agent Guidelines

## Overview

EndiorBot is a personal AI power tool for solo developers working on enterprise-scale projects.
It is a TypeScript/Node.js application that integrates with Claude Code as an Agent Orchestrator,
enabling `@agent` invocations with SDLC governance across CLI, Web, Telegram, and Zalo channels.

This document defines how AI agents should behave when working within the EndiorBot ecosystem.

## Technology Stack

| Layer | Technology | Notes |
|-------|------------|-------|
| Runtime | Node.js >= 20 | ES2022, NodeNext module resolution |
| Language | TypeScript 5.6 | Strict mode, explicit types, no `any` |
| Package Manager | pnpm 9 | Workspace monorepo (`packages: [".", "apps/*"]`) |
| Build | `tsc` | Outputs to `dist/`, declarations + source maps |
| Test Framework | Vitest 2.x | Unit + integration + e2e, v8 coverage, 70% threshold |
| Lint | ESLint 9 + `@typescript-eslint` | Explicit function return types, consistent type imports |
| AI SDKs | `@anthropic-ai/sdk`, OpenAI, Gemini, Ollama | Multi-model orchestration |
| Config Validation | Zod | Runtime schema validation with typed inference |
| Gateway | WebSocket + HTTP hybrid | JSON-RPC 2.0, port 18790 default |
| Desktop App | Electron 40 + React 19 + Vite | In `apps/desktop/` |
| Container | Docker multi-stage (node:20-alpine) | ~150MB production image |

## Project Structure

```
├── src/                    # Main TypeScript source (~40 modules)
│   ├── cli/                # Commander.js entry point
│   ├── commands/           # 35+ unified command handlers (CLI + OTT + Web)
│   ├── agents/             # Agent orchestration, SOUL templates, team registry
│   ├── bridge/             # Claude Code Bridge (tmux, sessions, launcher)
│   ├── bus/                # In-process MessageBus (EventEmitter, debounce, dedup)
│   ├── channels/           # Telegram + Zalo OTT adapters
│   ├── gateway/            # HTTP/WS server, Ingress, Web API, webhooks
│   ├── providers/          # AI model providers + multi-model orchestrator
│   ├── brain/              # 4-layer knowledge storage (Iceberg model)
│   ├── memory/             # ClawVault memory module
│   ├── sessions/           # Session lifecycle, checkpoints, resilience
│   ├── context/            # Context anchoring, sprint goals, spec snapshots
│   ├── sdlc/               # Gate engine, compliance, Vibecoding Index, scaffold
│   ├── security/           # Input sanitizer, output scrubber, SSRF guard, shell guard
│   ├── budget/             # Cost tracking, circuit breakers, escalation, approval queue
│   ├── evaluator/          # Evaluator-Optimizer feedback loop
│   ├── self-correction/    # Error classification + deterministic/AI fixes
│   ├── tools/              # Policy engine, Composio client, tool registry/executor
│   ├── mtclaw/             # MCP cross-system bridge to MTClaw agent platform
│   ├── search/             # ripgrep/ast-grep code search infrastructure
│   ├── errors/             # Unified error hierarchy + formatters
│   ├── logging/            # Structured logging with redaction
│   ├── config/             # Zod-based config, feature flags, timeout SSOT
│   └── utils/              # Type-safe utility library
├── tests/                  # Test suite mirroring src/ structure
│   ├── integration/        # 11 integration test files
│   ├── e2e/                # 2 end-to-end test files
│   ├── golden-scenarios/   # YAML-driven scenario runner
│   ├── security/           # Dedicated security test suite
│   └── performance/        # Performance tests
├── apps/desktop/           # Electron + React desktop application
├── scripts/                # Build helpers, linters, smoke tests
├── docs/                   # SDLC stage-shaped documentation (~328 .md files)
│   ├── 00-foundation/      # WHY — vision, business case
│   ├── 01-planning/        # WHAT — requirements, PRDs
│   ├── 02-design/          # HOW — ADRs, technical specs (50 ADRs)
│   ├── 04-build/           # Coding standards, sprint plans, CLI ref
│   ├── 05-test/            # Test plans, E2E reports
│   ├── 06-deploy/          # Deployment guides, env vars
│   ├── 07-operate/         # Usage guides, monitoring
│   └── 08-collaborate/     # Compliance, handover
├── endiorbot.mjs           # CLI bootstrapper (production vs dev mode detection)
├── package.json            # Root package @dttai/endiorbot
├── tsconfig.json           # TypeScript strict config with path aliases
├── vitest.config.ts        # Unit test config (70% coverage threshold)
├── vitest.e2e.config.ts    # E2E test config
├── eslint.config.js        # Lint rules
├── Dockerfile              # Multi-stage container build
└── .sdlc-config.json       # SDLC framework configuration (tier: STANDARD, current gate: G3)
```

## Path Aliases

TypeScript and Vitest resolve these aliases:

| Alias | Maps to |
|-------|---------|
| `@/*` | `./src/*` |
| `@config/*` | `./src/config/*` |
| `@utils/*` | `./src/utils/*` |
| `@agents/*` | `./src/agents/*` |
| `@security/*` | `./src/security/*` |
| `@sdlc/*` | `./src/sdlc/*` |
| `@providers/*` | `./src/providers/*` |

## Build and Test Commands

```bash
# Install dependencies
pnpm install

# Build TypeScript to dist/
pnpm build

# Watch mode development

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Minh-Tam-Solution/EndiorBot](https://github.com/Minh-Tam-Solution/EndiorBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
