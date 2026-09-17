---
trigger: always_on
description: Reply in the same language as the user.
---

# Repository-level Agent Guide

Reply in the same language as the user.

This is a TypeScript monorepo built for agent-assisted development. Keep the root `AGENTS.md` limited to hot-path rules: the project map, hard constraints, and workflow requirements — things every task needs to know.

## Working Principles

- Think from first principles. Start from real requirements, code facts, and verification results; if the goal is unclear, discuss it with the user first.
- Treat code, not documentation, as the source of truth. Unless the user explicitly says otherwise, do not read ordinary Markdown just to understand the implementation.
- Before making code changes, read the relevant code and the most recent constraints, and follow the nearest `AGENTS.md` in the directory tree.
- Keep changes focused. Do not slip in unrelated refactors along the way.
- When committing, do not add any co-author attribution, and do not reveal the identity of the agent in commit messages, PR descriptions, or any explanatory text.

## Project Map

- `apps/kimi-code`: the CLI / TUI application. It consumes core capabilities through `@moonshot-ai/kimi-code-sdk` and must not depend directly on `@moonshot-ai/agent-core`. When writing or modifying its terminal UI, use the `write-tui` skill (`.agents/skills/write-tui/SKILL.md`).
- `apps/kimi-web`: the browser web UI, a peer to the TUI. Vue 3 + Vite + vue-i18n; talks to the server over REST + WebSocket under `/api/v1`. It must not depend on `@moonshot-ai/agent-core` (wire types are re-implemented locally). Debug against the two engines via the root `pnpm dev:v1` / `pnpm dev:v2` backend scripts — the dev Sidebar shows the active backend and switches it at runtime. See `apps/kimi-web/AGENTS.md`.
- `apps/vis`, `apps/vis/server`, `apps/vis/web`: visual debugging tools for sessions and replays.
- `apps/kimi-inspect`: web inspector for the v2 (kap-server) `/api/v2` surface — workspace/session browser, per-session chat, and live Service panels (data + trigger buttons) for the Session and Agent scopes. Built on its own old-klient-style channel layer (`src/channel/`: the VS Code `ProxyChannel` model — service-bound `IChannel`, HTTP `ProxyChannel` for calls routed to `/api/v2`, `WsChannel` over the shared `/api/v2/ws` socket for events), typed by `agent-core-v2` Service interfaces; `GET {rpcBasePath}/channels` loads every wire protocol 1:1 — probing `/api/v1/debug` first (dev, whitelist-free) and falling back to `/api/v2`. The Vite dev server proxies `/api` to a running kap-server (`KIMI_SERVER_URL`, default `http://127.0.0.1:58627`) and exposes `GET /__inspect/servers` (`vite/serverDiscovery.ts`), which scans the local kap-server instance registry (`~/.kimi-code/server/instances` + legacy `lock`) and the home token so the app can zero-config auto-connect and switch servers from the header dropdown at runtime.
- `packages/agent-core`: the unified agent engine, including Agent, Session, profile, skills, tools, plan, permission, background, records, the in-process DI service layer (`src/services/`), and other core capabilities.
- `packages/node-sdk`: the public TypeScript SDK and harness.
- `packages/kosong`: the LLM / provider abstraction layer.
- `packages/kaos`: the execution environment and file/process abstractions.
- `packages/oauth`: Kimi OAuth and managed auth utilities.
- `packages/telemetry`: shared client-side telemetry infrastructure.
- `packages/kap-server`: the Kimi Code server, backed by the DI × Scope agent engine (`@moonshot-ai/agent-core-v2`). Exposes sessions over REST + WebSocket (`/api/v1` and the native `/api/v2` RPC surface); bootstrapped from `src/start.ts` and consumed by `apps/kimi-code`. With `--debug-endpoints` on a loopback bind it additionally mounts `/api/v1/debug/*` — the same reflection dispatcher as `/api/v2` but without the channel whitelist (every scoped Service callable, `src/transport/registerDebugRoutes.ts`); internal only, repo dev scripts pass the flag.
- `packages/klient`: the client SDK — a contract-driven facade over agent-core-v2 with aggregated `global.*` / `session(id).*` / `agent(id).*` methods, zod validation on every call, and klient-level typed event forwarding. Transport is chosen once at creation via subpath entry (`@moonshot-ai/klient/http|ipc|memory`); all three return the same `Klient`. The package also hosts the e2e suites: dual-backend session/agent suites (`test/e2e/dual/`, in-memory + in-process server), `/api/v2` wire tests (`test/e2e/v2/`), the legacy `/api/v1` live suites (`test/e2e/legacy/`), and the docker e2e runner (`pnpm --filter @moonshot-ai/klient docker:e2e`). See `packages/klient/AGENTS.md`.

## Environment Requirements

- **Node.js**: `>=24.15.0` (from the root `package.json` `engines`; `.nvmrc` is `24.15.0`, used by nvm / fnm / mise to pick the minimum recommended version).
- **pnpm**: `10.33.0` (from the root `package.json` `packageManager`).
- `pnpm install` will fail when the Node version is not satisfied, because `.npmrc` sets `engine-strict=true`.

## Monorepo Workspace Maintenance

- `pnpm-workspace.yaml` is the source of truth for workspace membership, but `flake.nix` also contains **hardcoded** `workspacePaths` and `workspaceNames` lists.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NitrogenT7/kimi-code-security](https://github.com/NitrogenT7/kimi-code-security) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-17 -->
