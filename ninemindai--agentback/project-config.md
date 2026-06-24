---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

ESM/Zod/MCP fork of LoopBack 4 — a slim modern subset of `@loopback/core` + REST for building HTTP and MCP services out of the same DI container. ESM-only, Node 22.13+, TypeScript 6.0, pnpm 11 workspaces. Alpha (v0.2.0 published to npm — all `@agentback/*` packages + the `create-agentback` scaffolder); API still settling. Scaffold a new app with `npm create agentback my-service [--template rest|mcp|hybrid]`.

For the framework's design thesis (boundary coherence between Zod, OpenAPI, MCP, and DI — and why that matters for AI-led development), see [docs/agent-ergonomics.md](docs/agent-ergonomics.md). Read it before adding a feature that might introduce a second source of truth alongside the Zod schemas.

## Commands

```bash
pnpm install                       # install workspace deps
pnpm build                         # tsc -b across the whole workspace (project references)
pnpm build:watch                   # incremental watch build
pnpm clean                         # tsc -b --clean + rm -rf each package's dist
pnpm test                          # vitest run — IMPORTANT: requires a prior `pnpm build`
pnpm test:watch                    # vitest watch
pnpm typecheck:client              # tsc --noEmit on the esbuild client bundles (NOT covered by build/test)
pnpm verify                        # full local CI mirror: build + typecheck:client + test + validate-templates
pnpm lint                          # eslint + prettier --check
pnpm lint:fix                      # eslint --fix + prettier --write

pnpm -F <pkg> build                # build a single workspace package (e.g. `pnpm -F @agentback/rest build`)
pnpm -F hello-rest start           # run an example (after build)
pnpm -F hello-hybrid start         # REST + MCP from one process
```

Running a single test file or pattern:

```bash
pnpm build
pnpm exec vitest run packages/core/dist/__tests__/unit/application.unit.js
pnpm exec vitest run -t "name of test"
```

## Critical: tests run against built `dist/`, not `src/`

`vitest.config.ts` globs `packages/*/dist/__tests__/**/*.{test,spec,unit,integration,acceptance}.js`. After editing any `.ts` you must `pnpm build` (or have `build:watch` running) before `pnpm test` will pick up the change. The same rule applies to running examples — they `import` from each package's `dist/`.

## Architecture

### Workspace layout

`pnpm-workspace.yaml` includes `packages/*` and `examples/*`. Each package is `@agentback/<name>` and emits to its own `dist/`. The root `tsconfig.json` is a project-references file listing build order; per-package `tsconfig.json` extends `tsconfig.base.json` and declares its own `references`. Adding a new package means: create `packages/<name>/{src,tsconfig.json,package.json}`, add it to the root `tsconfig.json` references in dependency order, and `pnpm install` to wire the workspace symlinks.

### Two layers

1. **Ported faithfully from upstream LoopBack 4** (ESM-ified, `.js` extensions on relative imports, `lodash` → `lodash-es`, `p-event` v6 named exports):
   - `metadata`, `context` — decorator metadata + DI container
   - `core` — `Application`, `Component`, `Server`, lifecycle
   - `http-server`, `express` — HTTP server with graceful stop, Express integration
   - `authentication`, `authentication-jwt`, `authentication-oauth2`, `authorization`, `security` — auth stack (`-oauth2` adds RFC 7662 introspection + JWKS bearer tokens; bring-your-own auth server)
   - `extension-health`, `extension-metrics` — observability extensions
   - `testlab` — test helpers
2. **Rewritten, not ported** (upstream carried too much baggage):
   - `openapi` — Zod-first decorators. Emits OpenAPI 3.1.1 directly from Zod via `z.toJSONSchema({target: 'draft-2020-12'})` instead of the upstream `@loopback/repository-json-schema` pipeline.
   - `rest` — minimal `RestServer` (routing + Zod request/body validation + error mapping + serves `/openapi.json`). Replaces upstream's ~10k LoC of sequences/actions/middleware composition. **Host is a class choice:** `RestApplication` (alias `ExpressRestApplication`) is the Node/Express host; `EdgeRestApplication` is pinned to `listener: 'native'` and installs **no `express`/`cors`** (fetch/Workers/Bun/Deno). The neutral middleware-chain machinery lives in `@agentback/middleware`; `express`/`cors`/`multer` are **optional peer deps** of `rest` (only the Express host / uploads need them). Deploy with `@agentback/cli` (`agentback deploy vercel|cloudflare`).
   - `mcp` — decorator-driven MCP server (`@mcpServer`, `@tool` with `input`/`output` Zod schemas) on top of the official `@modelcontextprotocol/sdk`. Runs stdio transport by default. `@tool({ui})` links a `ui://` widget for **MCP Apps** (SEP-1865) — see the MCP Apps note below.
   - `mcp-inspector` — small in-process inspector UI at `/mcp-inspector`; the official `@modelcontextprotocol/inspector` is a CLI, not embeddable.
   - `rest-explorer` — mounts Swagger UI 5.x at `/explorer`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ninemindai/agentback](https://github.com/ninemindai/agentback) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
