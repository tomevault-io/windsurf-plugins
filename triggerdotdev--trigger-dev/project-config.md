---
trigger: always_on
description: This file provides guidance to Claude Code when working with this repository. Subdirectory CLAUDE.md files provide deeper context when you navigate into specific areas.
---

# AGENTS.md

This file provides guidance to Claude Code when working with this repository. Subdirectory CLAUDE.md files provide deeper context when you navigate into specific areas.

## Build and Development Commands

This is a pnpm 10.33.2 monorepo using Turborepo. Run commands from root with `pnpm run`.

**Adding dependencies:** Edit `package.json` directly instead of using `pnpm add`, then run `pnpm i` from the repo root. See `.claude/rules/package-installation.md` for the full process.

```bash
pnpm run docker              # Core dev services (Postgres, Redis, Electric, MinIO, ClickHouse, s2-lite)
# pnpm run docker:full       # Same + observability stack (Prometheus, Grafana, OTEL) and chaos tooling
pnpm run db:migrate           # Run database migrations
pnpm run db:seed              # Seed the database (required for reference projects)

# Build packages (required before running)
pnpm run build --filter webapp && pnpm run build --filter trigger.dev && pnpm run build --filter @trigger.dev/sdk

pnpm run dev --filter webapp  # Run webapp (http://localhost:3030)
pnpm run dev --filter trigger.dev --filter "@trigger.dev/*"  # Watch CLI and packages
```

### Verifying Changes

The verification command depends on where the change lives:

- **Apps and internal packages** (`apps/*`, `internal-packages/*`): Use `typecheck`. **Never use `build`** for these — building proves almost nothing about correctness.
- **Public packages** (`packages/*`): Use `build`.

```bash
# Apps and internal packages — use typecheck
pnpm run typecheck --filter webapp                  # ~1-2 minutes
pnpm run typecheck --filter @internal/run-engine

# Public packages — use build
pnpm run build --filter @trigger.dev/sdk
pnpm run build --filter @trigger.dev/core
```

Only run typecheck/build after major changes (new files, significant refactors, schema changes). For small edits, trust the types and let CI catch issues.

## Testing

We use vitest exclusively. **Never mock anything** - use testcontainers instead.

```bash
pnpm run test --filter webapp                          # All tests for a package
cd internal-packages/run-engine
pnpm run test ./src/engine/tests/ttl.test.ts --run     # Single test file
pnpm run build --filter @internal/run-engine           # May need to build deps first
```

Test files go next to source files (e.g., `MyService.ts` -> `MyService.test.ts`).

### Testcontainers for Redis/PostgreSQL

```typescript
import { redisTest, postgresTest, containerTest } from "@internal/testcontainers";

redisTest("should use redis", async ({ redisOptions }) => {
  /* ... */
});
postgresTest("should use postgres", async ({ prisma }) => {
  /* ... */
});
containerTest("should use both", async ({ prisma, redisOptions }) => {
  /* ... */
});
```

## Code Style

### Formatting and linting

Format and lint are enforced by CI (`code-quality` check). Run before committing:

```bash
pnpm run format      # oxfmt — auto-fixes formatting
pnpm run lint:fix    # oxlint — auto-fixes lint violations
pnpm run lint        # oxlint — check only (no fixes)
```

### Imports

**Prefer static imports over dynamic imports.** Only use dynamic `import()` when:
- Circular dependencies cannot be resolved otherwise
- Code splitting is genuinely needed for performance
- The module must be loaded conditionally at runtime

Dynamic imports add unnecessary overhead in hot paths and make code harder to analyze. If you find yourself using `await import()`, ask if a regular `import` statement would work instead.

## Changesets and Server Changes

When modifying any public package (`packages/*` or `integrations/*`), add a changeset:

```bash
pnpm run changeset:add
```

- Default to **patch** for bug fixes and minor changes
- Confirm with maintainers before selecting **minor** (new features)
- **Never** select major without explicit approval

When modifying only server components (`apps/webapp/`, `apps/supervisor/`, etc.) with no package changes, add a `.server-changes/` file instead. See `.server-changes/README.md` for format and documentation.

**Write the description for users, not maintainers.** Both changesets and `.server-changes/` notes ship verbatim in user-visible release notes. Lead with what changed *for the user* - one plain sentence describing behavior, not implementation, and never naming internal tools or infra. The full writing guidance in `.server-changes/README.md` applies to changesets too.

## Dependency Pinning

Zod is pinned to a single version across the entire monorepo (currently `3.25.76`). When adding zod to a new or existing package, use the **exact same version** as the rest of the repo - never a different version or a range. Mismatched zod versions cause runtime type incompatibilities (e.g., schemas from one package can't be used as body validators in another).

## Architecture Overview

### Request Flow

User API call -> Webapp routes -> Services -> RunEngine -> Redis Queue -> Supervisor -> Container execution -> Results back through RunEngine -> ClickHouse (analytics) + PostgreSQL (state)

### Apps

- **apps/webapp**: Remix 2.17.4 app - main API, dashboard, orchestration. Uses Express server.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [triggerdotdev/trigger.dev](https://github.com/triggerdotdev/trigger.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
