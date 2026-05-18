---
trigger: always_on
description: **Package:** `@cyanheads/mcp-ts-core`
---

# Developer Protocol

**Package:** `@cyanheads/mcp-ts-core`
**Version:** 0.9.1
**Engines:** Bun ≥1.3.0, Node ≥24.0.0
**MCP SDK:** `@modelcontextprotocol/sdk` ^1.29.0
**Zod:** ^4.4.3
**GitHub:** [cyanheads/mcp-ts-core](https://github.com/cyanheads/mcp-ts-core)
**npm:** [@cyanheads/mcp-ts-core](https://www.npmjs.com/package/@cyanheads/mcp-ts-core)
**Docker:** [ghcr.io/cyanheads/mcp-ts-core](https://ghcr.io/cyanheads/mcp-ts-core)

> **Developer note:** Never assume. Read related files and docs before making changes. Read full file content for context. Never try to edit a file before reading it.

---

## Consumers

This package serves two consumer paths. When making changes, know which audience your change affects:

| Path | On-ramp | Affected by changes to |
|:--|:--|:--|
| **Direct package import** — existing project pulls in the package | `bun add @cyanheads/mcp-ts-core` → `import { createApp, tool, z } from '@cyanheads/mcp-ts-core'` | Public API surface (`src/`) — existing consumers feel changes immediately on upgrade |
| **Init-scaffolded server** — fresh project bootstrapped from this repo's templates | `bunx @cyanheads/mcp-ts-core init [name]` copies `templates/` into the new directory | `templates/` — only affects newly scaffolded servers, not existing ones |

Both paths share the same public API. Init copies starter `package.json`, configs (`tsconfig`, `biome.json`, `vitest.config.ts`), `.env.example`, `Dockerfile`, `CLAUDE.md`/`AGENTS.md`, and example definitions. `_`-prefixed files (e.g. `_.gitignore`) drop the prefix on copy. After init, consult the `setup` skill.

---

## Core Rules

- **Logic throws, framework catches.** Pure, stateless `handler` functions, no `try/catch`. Plain `Error` works — framework catches, classifies, formats. Use `McpError(code, message, data, options?)` only when you need a specific JSON-RPC code or structured data; 4th arg `{ cause }` chains.
- **Full-stack observability.** The framework automatically instruments every tool/resource call — OTel span, duration/payload/memory metrics, structured completion log. Use `ctx.log` for additional domain-specific logging within handlers (external API calls, multi-step operations, business events). `requestId`, `traceId`, `tenantId` auto-correlated. No `console` calls.
- **Unified Context.** Handlers receive `ctx` with logging (`ctx.log`), tenant-scoped storage (`ctx.state`), optional protocol capabilities (`ctx.elicit`, `ctx.sample`), and cancellation (`ctx.signal`).
- **Decoupled storage.** `ctx.state` for tenant-scoped KV. Never access persistence backends directly.
- **Canvas tokens are capabilities, not tenant-scoped state.** A `canvasId` is a 10-char URL-safe token; possession grants full read/write/drop on that canvas. Tokens are designed to be shareable — between agents in one session, and across users in single-tenant deployments (see tenant resolution table under `ctx.state`). Tools should accept the token in `input` (or omit to create fresh) and return it in `output`; collaboration is opt-in via explicit token exchange.
- **Runtime parity.** All features work with `stdio`/`http` and Worker bundle. Guard non-portable deps via `runtimeCaps` from `@cyanheads/mcp-ts-core/utils` — a frozen capability object (`isNode`, `isBun`, `isWorkerLike`, `hasBuffer`, `hasProcess`, etc.) computed once at module load. Prefer runtime-agnostic abstractions (Hono + `@hono/mcp`, Fetch APIs).
- **Definition linting is build-time only.** Run `bun run lint:mcp` (standalone) or `bun run devcheck` (gate). Not invoked at server startup — new lint rules are additive and never break deployed servers. Every diagnostic links to the rule reference in `api-linter` skill; see that skill for the full rule catalog.
- **Elicitation for missing input.** Use `ctx.elicit` when the client supports it.

---

## Exports Reference

| Subpath | Key Exports | Purpose |
|:--------|:------------|:--------|
| `@cyanheads/mcp-ts-core` | `createApp`, `tool`, `resource`, `prompt`, `appTool`, `appResource`, `APP_RESOURCE_MIME_TYPE`, `Context`, `createFail`, `createRecoveryFor`, `TypedFail`, `TypedRecoveryFor`, `ReasonOf`, `HandlerContext`, `z` | Main entry point |
| `/worker` | `createWorkerHandler`, `CloudflareBindings` | Cloudflare Workers entry |
| `/tools` | `ToolDefinition`, `AnyToolDefinition`, `ToolAnnotations` | Tool definition types |
| `/resources` | `ResourceDefinition`, `AnyResourceDefinition` | Resource definition types |
| `/prompts` | `PromptDefinition` | Prompt definition type |
| `/tasks` | `TaskToolDefinition`, `isTaskToolDefinition` | Task tool escape hatch |
| `/errors` | `McpError`, `JsonRpcErrorCode`, `notFound`, `validationError`, `unauthorized`, ... | Error types, codes, and factory functions |
| `/config` | `AppConfig`, `config`, `parseConfig`, `parseEnvConfig`, `resetConfig`, `ConfigSchema`, `FRAMEWORK_NAME`, `FRAMEWORK_VERSION` | Zod-validated config, framework identity, env-var helper |
| `/auth` | `checkScopes` | Dynamic scope checking |
| `/storage` | `StorageService` | Storage abstraction |
| `/storage/types` | `IStorageProvider` | Provider interface |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cyanheads/mcp-ts-core](https://github.com/cyanheads/mcp-ts-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
