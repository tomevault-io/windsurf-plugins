---
trigger: always_on
description: Effect-native SDKs with exhaustive error typing. We use a TDD-driven approach to discover, document, and patch missing API behavior from vendor specifications.
---

# Distilled

Effect-native SDKs with exhaustive error typing. We use a TDD-driven approach to discover, document, and patch missing API behavior from vendor specifications.

## Directory Structure

```
distilled/
├── packages/
│   ├── core/             # @distilled.cloud/sdk-core — shared client, traits, errors, categories
│   ├── aws/              # @distilled.cloud/aws — AWS SDK from Smithy models
│   ├── cloudflare/       # @distilled.cloud/cloudflare — Cloudflare SDK from TypeScript SDK
│   ├── coinbase/         # @distilled.cloud/coinbase — Coinbase CDP SDK from OpenAPI spec
│   ├── fly-io/           # @distilled.cloud/fly-io — Fly.io SDK from OpenAPI spec
│   ├── gcp/              # @distilled.cloud/gcp — GCP SDK from Discovery Documents
│   ├── mongodb-atlas/    # @distilled.cloud/mongodb-atlas — MongoDB Atlas SDK from OpenAPI spec
│   ├── neon/             # @distilled.cloud/neon — Neon SDK from OpenAPI spec
│   ├── planetscale/      # @distilled.cloud/planetscale — PlanetScale SDK from OpenAPI spec
│   ├── prisma-postgres/  # @distilled.cloud/prisma-postgres — Prisma Postgres SDK from OpenAPI spec
│   ├── stripe/           # @distilled.cloud/stripe — Stripe SDK from OpenAPI spec
│   ├── supabase/         # @distilled.cloud/supabase — Supabase SDK from OpenAPI spec
│   └── turso/            # @distilled.cloud/turso — Turso SDK from OpenAPI spec
├── scripts/              # Root-level scripts (create-sdk.ts, bump.ts, etc.)
├── .github/workflows/    # CI (test.yml) and preview publishing (pkg-pr.yml)
└── AGENTS.md             # This file
```

### Package Layout (each package)

```
packages/{name}/
├── src/                  # Source code (generated + hand-written)
│   ├── client.ts         # API.make/makePaginated factory
│   ├── traits.ts         # HTTP trait annotations (PathParam, Http, etc.)
│   ├── errors.ts         # Typed error classes
│   ├── category.ts       # Error categories (re-exported from core)
│   └── operations/       # Generated operations (DO NOT HAND-EDIT)
├── tests/                # Tests
├── specs/                # Git submodules with vendor specs (see below)
├── scripts/              # Code generation scripts
├── lib/                  # Build output (gitignored) — .js, .d.ts, .js.map, .d.ts.map
├── package.json
└── tsconfig.json
```

### Core Package

`packages/core` (`@distilled.cloud/sdk-core`) contains shared infrastructure used by all SDKs:

- `client.ts` — `API.make()` and `API.makePaginated()` factories that create Effect operations from annotated schemas
- `traits.ts` — Schema annotations for HTTP bindings (`T.Http`, `T.PathParam`, `T.HttpHeader`, `T.JsonName`, etc.)
- `errors.ts` — Base error classes (`NotFound`, `Unauthorized`, `Forbidden`, etc.) with error code matching
- `category.ts` — Error categories (`AuthError`, `ThrottlingError`, `ServerError`, etc.) for retry logic and semantic grouping
- `pagination.ts` — `paginatePages`/`paginateItems` stream utilities
- `retry.ts` — Retry policy configuration
- `sensitive.ts` — Sensitive data schemas
- `json-patch.ts` — JSON Patch (RFC 6902) implementation for spec patching

All other packages depend on core via `@distilled.cloud/sdk-core` workspace dependency.

## Tools

| Tool | Purpose | Command |
|------|---------|---------|
| **Bun** | Runtime, package manager, test runner | `bun install`, `bun run ...` |
| **tsc** | Type checking (native TypeScript compiler, `typescript@7`) | `tsc` (check), `tsc -b` (build) |
| **oxlint** | Linter | `oxlint --fix src` |
| **oxfmt** | Formatter | `oxfmt --write src`, `oxfmt --check src` |
| **vitest** | Test framework | `bunx vitest run test` |
| **Effect** | Core framework | All operations return `Effect<A, E, R>` |

### Per-Package Scripts

Every package has these scripts:

| Script | Command | Description |
|--------|---------|-------------|
| `typecheck` | `tsc` | Type check only (no emit) |
| `build` | `tsc -b` | Build to `lib/` (.js + .d.ts + source maps) |
| `check` | `tsc && oxlint src && oxfmt --check src` | Full check (types + lint + format) |
| `fmt` | `oxfmt --write src` | Format source |
| `lint` | `oxlint --fix src` | Lint + autofix |
| `test` | `bunx vitest run test` | Run tests |
| `generate` | `bun run scripts/generate.ts && oxlint --fix src && oxfmt --write src && oxfmt --write src` | Regenerate from spec |

### Root Build

```bash
bun run build  # Builds core first, then all packages in parallel
```

Core must be built before other packages because they resolve `@distilled.cloud/sdk-core/*` via the `types` export condition pointing to `lib/*.d.ts`.

### Scaffolding a New SDK

Use `create-sdk` to scaffold a new SDK package with all boilerplate, submodule setup, and code generation:

```bash
bun run create-sdk <name> --specs <url-or-repo>... [--register-package]
```

Examples:
```bash
# OpenAPI spec URL → creates distilled-spec-* mirror repo
bun run create-sdk stripe --specs https://raw.githubusercontent.com/stripe/openapi/master/openapi/spec3.json

# Git repo → adds as direct submodule
bun run create-sdk stripe --specs https://github.com/stripe/openapi.git

# Publish a 0.0.0 placeholder to npm

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alchemy-run/distilled](https://github.com/alchemy-run/distilled) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
