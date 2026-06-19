---
trigger: always_on
description: This file defines conventions for AI agents and human contributors working in this repository.
---

# otto Project - AI Agent & Contributor Guidelines

This file defines conventions for AI agents and human contributors working in this repository.

## Formatting and Linting

- Use Biome for linting/formatting: `bun lint` (or `bun lint --fix` to auto-fix issues)
- Do not disable rules globally
- If an exception is required, limit scope and add rationale in PR/commit message
- Keep imports sorted and remove unused code

## Modular Structure

- Prefer many small, focused modules over large files
- One route module per endpoint group (or per endpoint if it grows)
- One schema/table per file under `packages/database/src/schema/`, re-exported via index
- Keep `apps/tui` focused on interactive terminal UX and use `@ottocode/api` for server calls
- Avoid circular dependencies
- If a module grows beyond ~200–300 lines, consider refactoring

## Frontend Performance Boundaries

- Keep React parent/layout components focused on structure, not feature-specific state.
- A component should only subscribe to stores, queries, and hooks needed to render its own immediate output.
- If a dependency is only needed by a child panel, modal, list row, or controller, move that dependency into that child.
- Do not run expensive hooks before visibility gates. Use lightweight wrappers for hidden panels/modals and mount the heavy content only when visible.
- Prefer narrow Zustand selectors for exact values/actions; avoid subscribing to broad objects such as full store slices or all panel widths when only one value is needed.
- Avoid per-row global store subscriptions in large lists. Compute shared state once in the parent and pass stable props to memoized rows.
- Gate closed modals instead of always rendering them with `isOpen={false}` when the modal wrapper does non-trivial work.
- For frontend performance work, follow the plan in [docs/plans/react-performance-optimization-plan.md](docs/plans/react-performance-optimization-plan.md) and verify changes with React Scan where possible.

## Monorepo Package Imports

Use workspace package imports for cross-package dependencies:

- `@ottocode/api` - Type-safe API client
- `@ottocode/database` - SQLite + Drizzle ORM
- `@ottocode/install` - npm installer package
- `@ottocode/sdk` - Core SDK (tools, streaming, agents, auth, config, providers, prompts)
- `@ottocode/server` - HTTP server
- `@ottocode/web-sdk` - React components, hooks, and utilities
- `@ottocode/web-ui` - Pre-built static web UI assets

**Import Rules:**

- Use workspace imports (`@ottocode/...`) for cross-package dependencies
- Use relative imports (`./`, `../`) within the same package only
- **Never use `@/` path aliases** (removed during monorepo migration)

## Runtime and Tooling

- Use Bun for everything: scripts, running, building, testing, linting
- Do not use npm/yarn/pnpm commands
- Tests must use `bun:test` and live in `tests/`

## Database and Migrations

- SQLite via Drizzle ORM
- Schema lives under `packages/database/src/schema/`
- Migrations generated with Drizzle Kit into `packages/database/drizzle/`
- Server ensures database exists and runs migrations on startup

**Migration Workflow:**

When you need schema/database changes:

1. Update the schema files in `packages/database/src/schema/`
2. Generate migrations: `bunx drizzle-kit generate`
3. Update `packages/database/src/migrations-bundled.ts` to include the new migration file
4. Test the migration locally before committing

**Never manually create migration files** - always use `bunx drizzle-kit generate`

## API and Server

- Hono-based app
- Each endpoint belongs in its own module under `packages/server/src/routes/`
- Endpoint contracts must be Zod-first: define request params/query/body and response schemas with `@hono/zod-openapi`/Zod in server route/schema modules, then derive OpenAPI from those schemas.
- Register documented endpoints with `zodOpenApiRoute(...)`; do not reintroduce `openApiRoute(...)`, hand-written `OperationObject` route specs, or hardcoded OpenAPI component registries.
- Avoid broad `z.any()` endpoint schemas. Prefer explicit Zod objects/enums/unions; use `z.unknown()` only for genuinely opaque payloads such as binary multipart/file content, and document the reason in the route module.
- Do not hand-write OpenAPI schema objects as the source of truth for normal JSON endpoints. If an endpoint truly cannot be represented by Zod/OpenAPI (for example raw WebSocket upgrade handling, SSE helpers, binary file responses, or multipart edge cases), keep the exception narrow and document why in the route module.
- Expose OpenAPI at `/openapi.json` from registered server routes; do not maintain a separate hardcoded spec file.
- Generate clients from the OpenAPI output with hey (`bun run --filter @ottocode/api generate`/`build`) and have first-party clients call the generated SDK instead of duplicating endpoint URLs or response types.
- Streaming uses SSE; prefer AI SDK helpers for stream responses
- For API changes, follow this order:
  1. Implement/update route methods in `packages/server/src/routes/`
  2. Add/update Zod OpenAPI schemas alongside the route
  3. Regenerate OpenAPI JSON + SDK: `bun run --filter @ottocode/api generate`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nitishxyz/otto](https://github.com/nitishxyz/otto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
