---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bun-based TypeScript monorepo for `use-agently`, a CLI tool for the Agently platform — a decentralized marketplace for AI agents using ERC-8004 and the x402 payment protocol. The CLI manages local EVM wallets, discovers agents, and communicates with them via the A2A (Agent-to-Agent) protocol.

## Commands

```bash
# Install dependencies
bun install

# Build all packages (compiles to standalone binary via bun build --compile)
bun run build

# Run the CLI in development mode
bun run --filter use-agently dev

# Format code
bun run format

# Run tests
bun run test
```

All top-level commands use Turborepo (`turbo`) to orchestrate across packages.

## Local Dev Test Loop

`packages/localhost-aixyz` is the local testing infrastructure for the CLI. It is a private aixyz agent that serves a free A2A endpoint at `http://localhost:3000` so you can test `use-agently a2a` without hitting production agents or spending real funds.

```bash
# Terminal 1 — start the local agent (requires OPENAI_API_KEY in packages/localhost-aixyz/.env.local)
bun run --filter localhost-aixyz dev

# Terminal 2 — send a message via the CLI
use-agently a2a send --uri http://localhost:3000 -m "Hello!"
```

The agent exposes a free endpoint (`accepts: { scheme: "free" }`), so no wallet balance is needed. Set `OPENAI_API_KEY` in `packages/localhost-aixyz/.env.local` before starting the dev server.

Always use this agent when developing or testing changes to `use-agently` to maintain conformity and ensure every change is exercised end-to-end.

## Architecture

- **Monorepo**: Bun workspaces with `packages/*` layout, Turborepo for task orchestration
- **`packages/localhost-aixyz`**: Local development agent (private, not published)
  - `aixyz.config.ts` — Agent metadata and skills
  - `app/agent.ts` — Free A2A endpoint (`accepts: { scheme: "free" }`) with echo tool
  - `app/tools/echo.ts` — Simple echo tool for testing
- **`packages/use-agently-sdk`** (`@use-agently/sdk`): The SDK package (published to npm)
  - `client.ts` — `unstable_Client` type, `createClient()`, fetch wrappers (dry-run, payment)
  - `agently.ts` — Agently platform API (`search`, `getAgent` for ERC-8004 resolution)
  - `a2a.ts` — A2A protocol: `sendMessage`, `sendMessageStream`, `getAgentCard`, URL resolution
  - `mcp.ts` — MCP protocol: `listMcpTools`, `callMcpTool`, URL resolution
  - `wallets/` — Wallet interface and implementations (EVM private key)
  - `utils/` — Chain config, transaction mode types
- **`packages/use-agently`**: The CLI package
  - `src/bin.ts` — Executable entry point (`#!/usr/bin/env bun`)
  - `src/cli.ts` — Commander.js program setup, registers all commands
  - `src/config.ts` — Config persistence at `~/.use-agently/config.json` (load, save, backup)
  - `src/client.ts` — Creates `defaultClient` via `createClient()` with CLI user-agent; all CLI commands use this client
  - `src/commands/` — One file per CLI command (init, whoami, balance, search, a2a)
  - Build output: `build/use-agently` (standalone binary via `bun build --compile`)

### SDK Architecture (`@use-agently/sdk`)

The SDK is structured in three layers: **client**, **platform**, and **protocol**. All new SDK work must follow this layering.

#### Client Layer (`client.ts`)

`unstable_Client` is the low-level primitive. It holds a configured `fetch` (with User-Agent, etc.) and nothing else. It does **not** bundle protocol logic. Create one via `createClient({ userAgent })`.

```ts
const client = createClient({ userAgent: "my-app:1.0" });
```

The CLI creates a single `defaultClient` in `src/client.ts` and passes it through to all SDK calls.

#### Platform Layer (`agently.ts`)

The Agently API layer (`api.use-agently.com`). Provides `search(client, options)` and `getAgent(client, id)` for agent discovery and ERC-8004 metadata resolution. All functions take `unstable_Client` as the first argument.

#### Protocol Layer (`a2a.ts`, `mcp.ts`)

Each protocol file owns its own URL resolution and communication logic. Protocol functions take `unstable_Client` as the first argument.

- **A2A** (`a2a.ts`): `sendMessage(client, uri, message, options?)`, `sendMessageStream(client, uri, message, options?)`, `getAgentCard(client, uri)`, `createA2AClient(client, uri, fetchImpl)`
- **MCP** (`mcp.ts`): `listMcpTools(uri, options?)`, `callMcpTool(uri, tool, args?, options?)`

#### Key Design Rules

1. **`unstable_Client` is always the first parameter** — every SDK function that makes network requests takes `client: unstable_Client` as its first argument. This provides the configured `fetch`. Never use bare `fetch` or `clientFetch` in protocol-level code.

2. **Protocol-level URL resolution** — each protocol handles its own URI → URL mapping. A2A has `getAgentCardURL()` (appends `/.well-known/agent-card.json`). MCP has `resolveMcpUrl()`. The client layer does **not** resolve URLs.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AgentlyHQ/use-agently](https://github.com/AgentlyHQ/use-agently) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
