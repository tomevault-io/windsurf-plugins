---
trigger: always_on
description: > Guide for AI assistants working on this repository.
---

# AGENTS.md - xopc Development Guide

> Guide for AI assistants working on this repository.

---

## Table of Contents

- [Project Overview](#project-overview)
- [Quick Start](#quick-start)
- [Tech Stack](#tech-stack)
- [Project Structure](#project-structure)
- [Model Registry](#model-registry-architecture)
- [Code Style](#code-style-guidelines)
- [Logging conventions](#logging-conventions)
- [Key Patterns](#key-patterns)
- [Common Tasks](#common-tasks)
- [Configuration](#configuration)
- [Environment Variables](#environment-variables)
- [Testing](#testing)
- [Web UI](#web-ui)
- [Debugging](#debugging)
- [Troubleshooting](#troubleshooting)
- [When Making Changes](#when-making-changes)

---

## Project Overview

**xopc** (`@xopcai/xopc`) is a personal AI assistant on Node.js + TypeScript: CLI, HTTP/SSE **gateway** (REST + Server-Sent Events), and a **React** gateway console (`web/`). Channels (e.g. Telegram) load as extensions; additional backends appear in config/registry as the project evolves.

| Metric | Value |
|--------|-------|
| Core | TypeScript on Node.js **>= 22** |
| LLM layer | **~23** built-in providers via `@earendil-works/pi-ai` (`KnownProvider`); more via `models.json` |
| Tests | **vitest** (`src/**/__tests__/*.test.ts`) |

---

## Quick Start

```bash
pnpm install
pnpm run dev -- <command>    # no build required for dev CLI
pnpm run build               # Node: tsdown (`tsdown.config.ts`) + web; types: `pnpm run typecheck`
pnpm test
```

Examples: `pnpm run dev -- agent -i` · `pnpm run dev -- agent -m "Hello"`

---

## Tech Stack

| Area | Stack |
|------|--------|
| Agent | `@earendil-works/pi-agent-core`, `@earendil-works/pi-ai` |
| CLI | `commander` |
| Config | `zod` |
| Tools (schemas) | `@sinclair/typebox` |
| Gateway console | **React** + Vite + Tailwind v4 (`web/` package) |
| Tests | `vitest` |

---

## Project Structure

**Runtime (`src/`)** — main areas agents touch:

| Path | Role |
|------|------|
| `agent/` | `AgentService`, tools, memory, orchestration (core entry files at root; helpers grouped under `context/`, `lifecycle/`, `prompt/`, `transcript/` — transcript hygiene, thinking-level types, etc.) |
| `channels/` | `ChannelPlugin`, manager, inbound/outbound, `attachments/`, `plugins/bundled.ts` |
| `gateway/` | HTTP + SSE server, API for UI; `heartbeat/` keep-alive service |
| `cli/` | Commands (self-registration via `registry`) |
| `config/` | Schema, loader, paths |
| `providers/` | `resolveModel`, API keys, pi-ai bridge |
| `session/` | Conversation session store |
| `infra/` | Infrastructure primitives (`retry`, rate-limit, `bus/` message bus) |
| `extensions/` | Extension runtime; `extensions/sdk/` re-exports `@xopcai/xopc/extension-sdk` |

Also present (follow local patterns): `auth/`, `chat-commands/` (in-chat slash commands), `cron/`, `daemon/`, `routing/`, `voice/stt/`, `voice/tts/`, `utils/` (`logger.ts` barrel → `logger/` implementation + `helpers.ts`), `markdown/`, `errors/`, etc.

**Gateway console (`web/`)** — React SPA (Vite + Tailwind v4): hash router, REST + SSE to the gateway, Zustand + SWR. Production build outputs to `dist/gateway/static/root` (same static root the gateway serves).

**Extensions (`extensions/`)** — optional add-ons; **Telegram / Weixin** channel *sources* live in `extensions/telegram` and `extensions/weixin` but are **`private`** workspace packages (not published). **`tsdown`** (Rolldown) **unbundle** mode emits `dist/src/**` and `dist/extensions/**` in one pass. Wiring: `src/generated/bundled-channel-plugins.ts` (`pnpm run generate:bundled-channels`) and `src/channels/plugins/bundled.ts`.

---

## Model Registry Architecture

`src/providers/index.ts` sits on **`@earendil-works/pi-ai`**: resolve models, map API keys from config/env, expose provider lists to CLI/UI. The built-in provider id set matches upstream **`KnownProvider`**; **`PROVIDER_META`** adds display names and categories (common / specialty / enterprise / oauth). Extra vendors are added only when present in `models.json`. Keys load at process start—restart after credential changes.

| Function | Purpose |
|----------|---------|
| `resolveModel(ref)` | Model id + optional `provider/` prefix |
| `getApiKey` / `isProviderConfigured` | Auth from config or env |
| `getAllProviders` / `getModelsByProvider` | Discovery for UI |

Details: [docs/models.md](./docs/models.md).

---

## Code Style Guidelines

- **Comments:** English only; minimal—non-obvious logic, edge cases, exported APIs (JSDoc).
- **Naming:** `camelCase` (code), `PascalCase` (types/classes), `UPPER_SNAKE_CASE` (constants), `_unused` for unused params, `_privateMethod` for private helpers.
- **Imports:** external deps → internal absolute → relative (blank lines between groups). Example in repo: `src/agent/tools/*.ts`.
- **Files:** `camelCase.ts` sources; `*.test.ts` tests; `*.types.ts` for dedicated type modules.

---

## Logging conventions

Use **`createLogger('Prefix')`** from `src/utils/logger.ts` (Pino under the hood). Prefer a **stable module prefix** (e.g. `AgentService`, `Hono:Auth`) so gateway **Log Manager** and file logs filter cleanly.

### Shape: object first, message second

```typescript
const log = createLogger('MyModule');

log.info({ sessionKey, durationMs }, 'Session saved');

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xopcai/xopc](https://github.com/xopcai/xopc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
