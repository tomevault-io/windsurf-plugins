---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Silkweave is a TypeScript toolkit for building MCP (Model Context Protocol) servers and CLI tools from a single set of "Actions". Define an action once, then expose it via multiple adapters (MCP stdio, MCP HTTP, Fastify REST API, tRPC, or CLI).

## Commands

```bash
pnpm build          # Build all packages with tsdown (ESM output to build/)
pnpm check          # Lint + typecheck all packages
pnpm clean          # Clean all build outputs and turbo cache

# Run example servers (not automated tests - these start live servers)
pnpm -F @silkweave/example-core dev        # Run an Action directly without an adapter
pnpm -F @silkweave/example-cli dev         # CLI adapter (commander + clack)
pnpm -F @silkweave/example-mcp stdio       # MCP stdio server
pnpm -F @silkweave/example-mcp http        # MCP streamable HTTP server on :8080
pnpm -F @silkweave/example-mcp http-auth   # MCP HTTP with bearer token auth on :8080
pnpm -F @silkweave/example-mcp http-oauth  # MCP HTTP with Google OAuth 2.1 on :8080
pnpm -F @silkweave/example-mcp cli-proxy   # MCP CLI proxy client (connects to http example)
pnpm -F @silkweave/example-fastify dev     # Fastify REST API with Swagger on :8080
pnpm -F @silkweave/example-trpc dev        # tRPC standalone HTTP server on :8080/trpc/
pnpm -F @silkweave/example-typegen dev     # Generate .d.ts from action Zod schemas
pnpm -F @silkweave/example-nestjs dev      # NestJS controllers exposed as MCP tools via @Mcp, on :8080
pnpm -F @silkweave/example-nextjs dev      # Next.js App Router: one action set as MCP (/api/mcp) + tRPC (/api/trpc), on :8080

# AI chat example (Vite + React + useChat + tRPC subscriptions)
ANTHROPIC_API_KEY=sk-... pnpm -F @silkweave/example-ai dev

# MCP Inspector (connects to MCP stdio example via .mcp.json)
pnpm mcp
```

## Architecture

The core pattern is **Action → Adapter → Silkweave**:

- **Action** (`packages/core/src/util/action.ts`): A named operation with a Zod `input` schema, an optional Zod `output` schema, and an async `run(input, context)` function. Actions are adapter-agnostic - they receive a `Logger` via context. The `output` schema is used by the typegen and tRPC adapters to generate typed response interfaces. An optional `kind: 'query' | 'mutation'` field (default `'mutation'`) controls how the action is exposed over tRPC - queries are GET-cacheable, mutations are POST. the `@silkweave/fastify` REST adapter additionally honors three optional routing fields: `method` (`'GET' | 'POST' | 'PUT' | 'DELETE'`, default `POST` or `GET` when `kind: 'query'`), `path` (a route template that may contain `:param` placeholders, e.g. `'spaces/:spaceId/users'`), and `queryParams` (input fields read from the URL query string instead of the body, e.g. `['offset', 'limit']`). Path placeholders and query params must be keys of the input schema; the input is merged from path + query + body and validated as one (see [REST routing](#rest-routing) below). An optional `toolResult(response, context)` hook lets actions control how results are formatted as MCP `CallToolResult` (e.g. returning embedded resources for large payloads); an optional `disposition: 'json' | 'smart'` field sets the *default* MCP result format (`jsonToolResult` vs `smartToolResult`) that a client's `_meta.disposition` can still override. `Action<I, O, N, K>` is generic over input/output types, the literal `name`, and `kind` - literal types are preserved through `createAction()` so the `Silkweave<Actions>` builder can thread action types to type-aware adapters like tRPC. Actions can also be **streaming**: declare a `chunk` Zod schema and an `async function*` `run` that yields chunks; adapters detect this via `isStreamingAction()` and switch to per-chunk wire delivery (see [Streaming](#streaming) below).
- **Adapter** (`packages/core/src/util/adapter.ts`): Translates actions into a specific transport. `AdapterFactory<T>` takes config options, returns an `AdapterGenerator` that takes `SilkweaveOptions` and produces an `Adapter` with `start(actions)` / `stop()`.
- **Silkweave** (`packages/core/src/lib/silkweave.ts`): Fluent builder - `silkweave(opts).adapter(generator).action(action).start()`. `Silkweave<Actions extends Record<string, Action>>` is generic over accumulated actions so `typeof server` carries action type info forward; type-aware adapters (e.g. `@silkweave/trpc`'s `InferTrpcRouter<typeof server>`) extract this for end-to-end type safety.

### Packages

| Package | Path | Description |
|---------|------|-------------|
| `@silkweave/core` | `packages/core` | Core library - actions, adapters, builder, context, logger, utilities |
| `@silkweave/auth` | `packages/auth` | Auth - OAuth 2.1 proxy (PKCE, refresh tokens, CIMD, dynamic client registration), bearer token validation, protected resource metadata (RFC 9728) |
| `@silkweave/mcp` | `packages/mcp` | MCP adapters - stdio, streamable HTTP, CLI proxy |
| `@silkweave/cli` | `packages/cli` | CLI adapter - commander + clack terminal UI |
| `@silkweave/fastify` | `packages/fastify` | Fastify REST adapter - auto-generated OpenAPI/Swagger docs |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [silkweave/silkweave](https://github.com/silkweave/silkweave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
