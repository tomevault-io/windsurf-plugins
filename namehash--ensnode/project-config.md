---
trigger: always_on
description: ENSNode is a multichain ENS indexer monorepo. It indexes ENS names across multiple chains (mainnet, Basenames, Lineanames, 3DNS) and exposes them via GraphQL and REST APIs.
---

# ENSNode

ENSNode is a multichain ENS indexer monorepo. It indexes ENS names across multiple chains (mainnet, Basenames, Lineanames, 3DNS) and exposes them via GraphQL and REST APIs.

## Monorepo Structure

- `apps/ensindexer` — Blockchain indexer powered by Ponder
- `apps/ensapi` — ENS API server (GraphQL and REST, Hono)
- `apps/ensadmin` — Dashboard for navigating indexed ENS state (Next.js)
- `apps/ensrainbow` — Label healing service: recovers labels from labelHashes (Hono)
- `apps/fallback-ensapi` — AWS Lambda fallback that proxies ENS Subgraph requests when ENSApi is unhealthy
- `packages/ensdb-sdk` — SDK for interacting with data in ENSDb
- `packages/ensnode-sdk` — SDK for interacting with ENSNode
- `packages/ensrainbow-sdk` — SDK for interacting with ENSRainbow
- `packages/datasources` — Catalog of chain datasources (contracts, start blocks, event filters)
- `packages/ponder-subgraph` — Hono middleware for Subgraph-compatible GraphQL
- `packages/ponder-sdk` — Utility library for interacting with Ponder apps and data
- `packages/ens-referrals` — Utilities for ENS Referrals
- `packages/namehash-ui` — UI components for NameHash Labs apps
- `packages/shared-configs` — Shared TypeScript configurations
- `docs/ensnode.io` — Documentation site (Astro/Starlight)

## Tech Stack

- **Language:** TypeScript
- **Package manager:** pnpm (workspaces with catalog for dependency versioning)
- **API framework:** Hono
- **Indexer framework:** Ponder
- **Validation:** Zod
- **ORM:** Drizzle
- **Linting/formatting:** Biome
- **Testing:** Vitest
- **Build:** tsup, tsx

## Commands

Runnable commands for validating changes; lint and format with Biome.

- Install dependencies: `pnpm install`
- Run all tests: `pnpm test`
  - Run tests for a single project: `pnpm test --project <project>` (e.g. `pnpm test --project ensapi`)
  - Run tests for a single file: `pnpm test <path>`
- Lint and format: `pnpm lint` (fixes where applicable); CI lint: `pnpm lint:ci`
- Type checking: `pnpm typecheck` (runs typecheck in all workspaces)
  - Always use `pnpm -F <package-name> typecheck`, never call `tsc` or `tsgo` directly
- Omnigraph example sample responses (docs): after changing SDK Omnigraph example queries/variables in `packages/ensnode-sdk` or when refreshing live JSON shown in the docs Omnigraph examples, run `pnpm -F @docs/ensnode omnigraph-examples:refresh-responses` (requires `curl`, network). Updates `docs/ensnode.io/src/data/omnigraph-examples/responses.json`.

## Testing

- Tests are colocated with source files (e.g. `foo.test.ts` next to `foo.ts`).
- Use the `*.test.ts` naming convention. Do not use `*.spec.ts`.
- Use `describe`/`it` blocks with `expect` assertions.
- Use `vi.mock()` for module mocking and `vi.fn()` for function stubs.
- Each app and package has its own `vitest.config.ts`.
- Prefer the `await expect(...).resolves.*` format over await-then-expect.
- Prefer `await expect(...).resolves.toMatchObject({})` over expecting individual properties, if it is more concise.

## Documentation & DRY

- Do not duplicate definitions across multiple locations. Duplication creates a significant maintenance burden.
- Ensure documentation resides at the correct place and the correct layer of responsibility.
- Use type aliases to document invariants. Each invariant MUST be documented exactly once, on its type alias.

## Code Comments

- Do not add JSDoc `@returns` tags that merely restate the method summary (e.g. "returns the X" when the description already says "Get the X"). Remove such redundancy during PR review.
- Maintain comment consistency within a file: if most types, schemas, or declarations lack comments, do not add a comment to a single one. Address the inconsistency during PR review.

## Error Handling

Fail fast and loudly on invalid inputs.

- **Validation — API requests:** Use the existing Hono validation middleware (Zod schemas + `validate()` from `apps/ensapi/src/lib/handlers/validate.ts`). Failed validation becomes a 400 response with structured details via `errorResponse`; handlers receive already-validated data. Do not manually call `zod.parse`/`safeParse` in route handlers for request body/params/query when this middleware is in use.
- **Validation — non-API code (config, SDK, scripts):** Use `zod.parse(...)` when invalid input should throw immediately; use `zod.safeParse(...)` when you need a non-throwing branch (e.g. optional or fallback). Prefer `parse` for fail-fast.
- **Error types:** Use plain `Error` (or `ZodError` when propagating Zod validation errors). The codebase does not define a custom hierarchy (e.g. `AppError`/`ValidationError`); do not introduce one unless the project adopts it. Use `throw new Error("message")` from application code.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [namehash/ensnode](https://github.com/namehash/ensnode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
