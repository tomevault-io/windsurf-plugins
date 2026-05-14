---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

zodvex is a TypeScript library that provides Zod v4 → Convex validator mapping with correct optional and nullable semantics. It acts as a thin, practical glue around `convex-helpers` that preserves Convex's notion of optional fields while offering ergonomic wrappers and codecs for working with Zod schemas in Convex applications.

## Monorepo Structure

This is a bun workspaces monorepo:

- `packages/zodvex/` — the publishable library (source, tests, build config)
- `examples/task-manager/` — full example app using zodvex via `workspace:*`
- `examples/task-manager-mini/` — same app using `zod/mini` to verify mini compatibility
- `examples/quickstart/` — minimal getting-started example
- `examples/stress-test/` — performance/edge-case testing
- Root `package.json` — workspace root (private, not published)

All commands can be run from the repo root — they delegate to `packages/zodvex/`.

## Key Commands

### Development

- `bun run dev` - Run tsup in watch mode for continuous builds
- `bun run build` - Build the library with tsup
- `bun run type-check` - Type check with TypeScript (no emit)

### Testing

- `bun run test` - Run tests via vitest (NEVER use `bun test` — it invokes Bun's built-in runner which fails on vitest APIs)

### Code Quality

- `bun run lint` - Check code with Biome (linting and formatting)
- `bun run lint:fix` - Fix code issues with Biome
- `bun run format` - Format code with Biome

### Validation

- `bun run validate` - **Full pre-release validation.** Runs lint → type-check → test → verify:consumer-declarations → build → verify:examples (local) → verify:examples:network (deploys task-manager + task-manager-mini + quickstart to their Convex dev instances and runs real HTTP smoke tests) → full stress-test ceiling search. Requires `CONVEX_DEPLOYMENT` configured in each example's `.env.local` (one-time `npx convex dev --configure` per example). Run locally before trialing a release in downstream projects — CI can't do the Convex deploy step.
- `bun run verify:examples` - Local-only subset (no network). Typechecks + runs vitest + regenerates codegen in both task-manager apps.
- `bun run verify:examples:network` - Deploys schemas to real Convex and runs smoke tests. Standalone script if you want the Convex portion without the whole pipeline.

### Releasing

**Beta releases** (manual, fast):
- `bin/release-beta` — auto-increments prerelease number, builds, tests, tags, pushes
- `bin/release-beta 0.7.0-beta.0` — explicit version
- Tag push triggers `.github/workflows/release.yml` → npm publish with `--tag beta`
- No GitHub Release created for betas

**Stable releases**: No automated stable release workflow yet. Stable releases are cut manually.

**PR titles** must follow conventional commit format (`feat:`, `fix:`, `chore:`, etc.) — enforced by `.github/workflows/pr-title.yml`

## Architecture

### Core Modules

The library is organized into focused modules in `packages/zodvex/src/`:

- **zod-core.ts** - Central re-export of `zod/v4/core` types and functions. All instanceof checks and type constraints use these core types so zodvex works with both `zod` and `zod/mini`.

- **model.ts** - `defineZodModel()` — the primary API for defining Convex table schemas with codec support. Client-safe.

- **init.ts** - `initZodvex()` — one-time project setup that returns pre-configured function builders (`zq`, `zm`, `za`) with codec-wrapped `ctx.db`.

- **mapping/** - Core Zod to Convex validator conversion logic (`mapping/core.ts`) with type-specific handlers in `mapping/handlers/`.

- **codec.ts** - `convexCodec` / `zodvexCodec` for encoding/decoding between Zod-shaped data and Convex-safe JSON.

- **wrappers.ts** - Function wrappers (`zQuery`, `zMutation`, `zAction` and their internal variants) that add Zod validation to Convex functions.

- **custom.ts** - Custom function builders (`zCustomQuery`, `zCustomMutation`, `zCustomAction`). Supports convex-helpers' `onSuccess` callback convention.

- **codegen/** - `zodvex generate` CLI. Runtime discovery (`discover.ts`), schema-to-source serialization (`zodToSource.ts`), and file generation (`generate.ts`).

- **tables.ts** - `zodTable` for defining Convex tables from Zod schemas (legacy — prefer `defineZodModel`).

- **types.ts** - TypeScript type definitions and utility types.

- **utils.ts** - Shared utility functions.

### Entrypoints

- `zodvex` — everything (re-exports core + server)
- `zodvex/core` — client-safe: validators, codecs, model definitions, registry
- `zodvex/mini` — same as core but with `zx` typed for `zod/mini` compatibility
- `zodvex/server` — server-only: `initZodvex`, function builders, DB wrappers
- `zodvex/react` — React hooks (`useZodQuery`, `useZodMutation`)
- `zodvex/client` — vanilla JS client
- `zodvex/codegen` — CLI and generation utilities

### Key Design Principles

1. **Semantic Preservation**: The library carefully preserves the distinction between optional and nullable fields:
   - `.optional()` → `v.optional(T)`
   - `.nullable()` → `v.union(T, v.null())`
   - Both → `v.optional(v.union(T, v.null()))`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [panzacoder/zodvex](https://github.com/panzacoder/zodvex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
