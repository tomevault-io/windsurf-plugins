---
trigger: always_on
description: Reply in English unless the user explicitly asks otherwise. Always talk in ASD-STE100 Simplified Technical English.
---

# Repository-level Agent Guide

Reply in English unless the user explicitly asks otherwise. Always talk in ASD-STE100 Simplified Technical English.

TypeScript monorepo for **pythinker-code**, a provider-agnostic AI coding agent. This file covers product identity, project map, hard constraints, and workflow rules.

## Product Identity

**pythinker-code** plans, writes, tests, and iterates on code autonomously. The same runtime talks to any LLM through the `packages/kosong` abstraction layer.

### Wire Types

| Wire type          | SDK / transport         | Providers                                       |
| ------------------ | ----------------------- | ------------------------------------------------ |
| `anthropic`        | `@anthropic-ai/sdk`     | Anthropic (Claude family)                        |
| `openai`           | OpenAI Chat Completions | OpenAI (GPT-4o/4.1/4.5/5.x, GPT-3.5-turbo)     |
| `openai_responses` | OpenAI Responses API    | OpenAI (GPT-4.1/5.x, o-series)                  |
| `google-genai`     | `@google/genai`         | Google (Gemini 2.0–3.x)                          |
| `vertexai`         | Google Vertex AI        | Google Cloud–hosted Gemini                       |
| `pythinker`        | Pythinker managed API   | Any model proxied through Pythinker              |

Any OpenAI-compatible endpoint (DeepSeek, Qwen, GLM, Grok, Together AI, Fireworks, etc.) works via the `openai`/`openai_responses` wire with a custom `baseURL`.

### Model Selection

Flows through the **catalog** ([catalog.ts](packages/kosong/src/catalog.ts)):

1. JSON catalog maps `providerId → models[]` with context window, capabilities, cost, and modality metadata.
2. `inferWireType()` resolves provider → wire type (explicit `type` field, then heuristic on `npm`/`id`).
3. `createProvider()` instantiates the correct `ChatProvider`.
4. `getModelCapability()` returns per-model flags (vision, tool-use, thinking, fast-mode).

Adding an OpenAI-compatible provider requires **zero code changes** — just add a catalog entry.

## Working Principles

- Start from requirements and code facts; discuss unclear goals first.
- Code is the source of truth — don't read Markdown to understand implementation.
- Validate version claims against authoritative docs (Context7 MCP, Tavily).
- Read relevant source and follow the nearest `AGENTS.md` before changing code.
- Keep changes focused — no drive-by refactors.
- Implement current requirements directly; no backward-compatibility shims.
- Simplest implementation first: stdlib → established libraries → custom code. Use the `ponytail` skill when a change looks over-engineered.
- No co-author attribution or agent identity in commits/PRs.
- Git identity: `elkaix <melkholy@techmatrix.com>` — apply per command; never modify git config.

## Project Map

| Package | Description | Notes |
| ------- | ----------- | ----- |
| `apps/pythinker-code` | CLI / TUI app | Consumes `@pythoughts/pythinker-code-sdk`; no `agent-core` dep. Use `write-tui` skill. |
| `apps/pythinker-web` | Browser UI (Vue 3 + Vite + vue-i18n) | REST + WS `/api/v1`; no `agent-core` dep. See its `AGENTS.md`. |
| `apps/dashboard` | Session dashboard & replay | `server/` + `web/` subdirs. |
| `packages/agent-core` | Agent engine | Agent, Session, profile, skills, tools, plan, permission, DI. |
| `packages/node-sdk` | Public TS SDK & harness | |
| `packages/kosong` | LLM provider abstraction | Wire types, catalog, capability registry. |
| `packages/kaos` | Execution environment | File/process abstractions. |
| `packages/oauth` | Auth utilities | |
| `packages/telemetry` | Client-side telemetry | |
| `packages/server` | Server | Hosts `agent-core` over REST + WS `/api/v1`. See its `AGENTS.md`. |
| `packages/server-e2e` | E2E tests | `PYTHINKER_SERVER_URL` (default `http://127.0.0.1:58627`). See its `AGENTS.md`. |

## Environment

- **Node.js** ≥ 26.4.0 (`.nvmrc`). **pnpm** 10.33.0 (root `packageManager`). `engine-strict=true`.

## Monorepo Maintenance

- `pnpm-workspace.yaml` is source of truth, but `flake.nix` **hardcodes** `workspacePaths`/`workspaceNames`.
- **Update both** when adding/removing any workspace package. Missing a path silently drops files from Nix; missing a name breaks `pnpmConfigHook`.
- CI (`scripts/check-nix-workspace.mjs`) only validates the `@pythoughts/pythinker-code` closure — keep `flake.nix` updated by hand.

## Coding Rules

- English-only codebase. Use ASCII/Latin fixtures (e.g. `café`) for unicode tests.
- `packages/acp-adapter`: pin `@agentclientprotocol/sdk` `^0.23.0` (0.24+ broke session-model API).
- `tsgo` (`@typescript/native-preview`) available via `npx tsgo -p <tsconfig> --noEmit`; committed scripts use `tsc` — run both for type fixes.
- Pass `undefined` directly for optional props — no conditional spread.
- `user?: User`, not `user?: User | undefined`.
- Single-param internal methods stay single-param — no options-object wrapping.
- Non-root `index.ts`: prefer `export * from './module'`.
- `Agent` class must be standalone — no mandatory `Session`/`agentId`. Optional `sessionId` as provider hint only.
- Prefer adding tests to existing files. Fix failing tests first (unless there's a real impl bug).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PyModel/pythinker-code](https://github.com/PyModel/pythinker-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
