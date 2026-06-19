---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Covel is an AI RPG plugin-based framework (modular monolith). Core philosophy: **plugins carry gameplay logic, the kernel provides primitives and orchestration**. Each plugin is a self-contained Agent Runtime that declares its own trigger rules, context injection, tool whitelist, and write proxies; the kernel routes turns, assembles context, drives LLM tool-calls, and commits proposals.

Deployable as Web or Electron (desktop). Production desktop builds should use `pnpm build:electron`.

## Documentation Index

Before changing anything non-trivial, consult the matching reference doc — they are the source of truth, CLAUDE.md only points at them.

| Topic                                               | Authoritative doc                                                                          |
| --------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| Project intro, quick start, roadmap                 | [README.md](./README.md) · [docs/README.md](./docs/README.md)                              |
| End-to-end turn pipeline, full architecture         | [docs/architecture/flow.md](./docs/architecture/flow.md)                                   |
| Plugin registry (all plugins, priorities, triggers) | [docs/reference/plugins.md](./docs/reference/plugins.md)                                   |
| World Data (`worldData`, source import, overrides)  | [docs/reference/world-data.md](./docs/reference/world-data.md)                             |
| Tool registry (builtin + local, approval policy)    | [docs/reference/tools.md](./docs/reference/tools.md)                                       |
| HTTP API (all endpoints, request/response, curl)    | [docs/reference/api.md](./docs/reference/api.md)                                           |
| Protocol (SSE events, envelope, Transport layer)    | [docs/reference/protocol.md](./docs/reference/protocol.md)                                 |
| Right-panel tabs, json-render declarative UI        | [docs/reference/ui-panels.md](./docs/reference/ui-panels.md)                               |
| Prompt assembly (segments, cache_control)           | [docs/reference/prompt-structure.md](./docs/reference/prompt-structure.md)                 |
| DataStore transactions (begin/commit/rollback)      | [docs/reference/transactions.md](./docs/reference/transactions.md)                         |
| Writing a plugin (tutorial + frontmatter fields)    | [docs/guide/plugin-authoring.md](./docs/guide/plugin-authoring.md)                         |
| Plugin UI + runtime guidelines                      | [docs/guide/plugin-ui-runtime-guidelines.md](./docs/guide/plugin-ui-runtime-guidelines.md) |
| Plugin testing (harness + examples)                 | [docs/guide/plugin-testing.md](./docs/guide/plugin-testing.md)                             |
| UI component catalogue (json-render primitives)     | [docs/reference/ui-components.md](./docs/reference/ui-components.md)                       |
| Terminology glossary (session / runtime / slot / …) | [docs/glossary.md](./docs/glossary.md)                                                     |
| E2E plugin verify harness                           | [docs/guide/e2e-plugin-verify.md](./docs/guide/e2e-plugin-verify.md)                       |
| Environment variable registry                       | [docs/guide/env-registry.md](./docs/guide/env-registry.md)                                 |
| Desktop config (paths, sidecar, safeStorage)        | [docs/guide/desktop-config.md](./docs/guide/desktop-config.md)                             |
| Desktop packaging (Electron), signing, notarisation | [apps/desktop/PACKAGING.md](./apps/desktop/PACKAGING.md)                                   |
| Contributing & release workflow                     | [docs/CONTRIBUTING.md](./docs/CONTRIBUTING.md)                                             |

## Commands

```bash
# Install & dev
pnpm install
pnpm dev              # web (5173) + server (3001), SqliteStore (default, ./data/covel.db)
pnpm dev:web          # web only
pnpm dev:server       # server only (SqliteStore default; STORE_BACKEND=memory for ephemeral)
pnpm dev:pg           # server only, STORE_BACKEND=pg (needs pnpm db:up first)

# Build & check
pnpm build            # build all
pnpm lint             # tsc --noEmit across workspace
pnpm test             # vitest via turbo (cached)
pnpm test:coverage    # + @vitest/coverage-v8
pnpm clean

# Single package tests — add --watch for watch, --run for single run
pnpm --filter @covel/runtime test
pnpm --filter @covel/<pkg> test

# Database (Docker)
pnpm db:up / db:down / db:generate / db:migrate / db:studio

# E2E
pnpm e2e              # Playwright headless
pnpm e2e:ui

# Real-LLM E2E scripts (need .env.llm)
npx tsx --env-file=.env --env-file=.env.llm scripts/e2e-plugin-verify.ts --slot e2e_local --turns 3

# Docker (full stack)
pnpm docker:build / docker:up / docker:down / docker:logs

# Desktop
pnpm dev:electron                        # Electron dev shell (real sidecar)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ackness/covel](https://github.com/ackness/covel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-19 -->
