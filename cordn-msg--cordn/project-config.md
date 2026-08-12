---
trigger: always_on
description: [`cordn`](package.json) is a TypeScript MLS delivery service and ContextVM server adapter. It provides:
---

# AGENTS.md

## Project overview

[`cordn`](package.json) is a TypeScript MLS delivery service and ContextVM server adapter. It provides:

- a ContextVM/MCP server wrapper around that coordinator
- a small CLI client for local and integration testing

This is a **pnpm workspace monorepo**. Packages import each other by `@cordn/*`
name; `pnpm install` symlinks each workspace package into its consumers'
`node_modules`, and the root [`tsconfig.json`](tsconfig.json) `paths` map each
name to its source so `tsc`/esbuild/editor resolve the same way. Node 22 runs
the `.ts` entry points directly via built-in type-stripping — there is **no
build step for dev** (only `pnpm run build` bundles the server for deployment).

Packages (`packages/*`):

- [`@cordn/core`](packages/core/src/) — shared foundation: wire contracts
  (zod schemas + types + method names), leaf codecs (base64, MLS framing,
  last-resort key-package extension), and consumed-ref value types.
  Transport-agnostic, dependency-light. Everything depends on this.
- [`@cordn/coordinator`](packages/coordinator/src/) — core delivery-service
  state and behavior (key-package store, welcome/join-request queues, per-group
  message cursors, storage backends). Server-only; clients never embed it.
- [`@cordn/server`](packages/server/src/) — ContextVM/MCP server bindings over
  the coordinator and the runnable server entrypoint.
- [`@cordn/cli`](packages/cli/src/) — terminal client (REPL + session model) and
  integration helpers. A leaf consumer; depends on `@cordn/core` (and on
  `@cordn/server` + `@cordn/test-utils` in its integration tests).
- [`@cordn/test-utils`](packages/test-utils/src/) — shared test fixtures: MLS
  artifact builders (`testUtils.ts`) and the mock Nostr relay (`mockRelay.ts`).
  A `devDependency` of every package whose tests need fixtures.

Primary implementation areas (by package):

- [`packages/coordinator/src/`](packages/coordinator/src/) — delivery-service state and behavior
- [`packages/server/src/`](packages/server/src/) — ContextVM server bindings and runnable server entrypoint
- [`packages/cli/src/`](packages/cli/src/) — terminal client and integration helpers
- [`packages/core/src/contracts.ts`](packages/core/src/contracts.ts) — shared request/response wire contracts

Coordinator contract notes:

- Group delivery cursors are monotonic per group, never global across all groups.
- [`fetchGroupMessages({ groupId, afterCursor })`](packages/coordinator/src/coordinator.ts:160) must treat `afterCursor` as a cursor within that group only.
- [`FetchManyGroupMessages`](packages/cli/src/coordinatorClient.ts:342) must preserve bounded catch-up semantics with independent per-group cursors.
- [`subscribeGroupMessages({ groupId, afterCursor })`](packages/coordinator/src/coordinator.ts:265) must preserve the same cursor semantics while replaying backlog before live delivery.
- [`SubscribeManyGroupMessages`](packages/cli/src/coordinatorClient.ts:371) must remain backwards-compatible with the single-group subscription stream shape while preserving independent per-group cursor semantics.
- Keep storage backends behaviorally aligned; changes in [`packages/coordinator/src/storage/inMemoryStorage.ts`](packages/coordinator/src/storage/inMemoryStorage.ts) and [`packages/coordinator/src/storage/sqliteStorage.ts`](packages/coordinator/src/storage/sqliteStorage.ts) require parity coverage in [`packages/coordinator/src/storage/storage.test.ts`](packages/coordinator/src/storage/storage.test.ts).

## Setup commands

- Install dependencies: `pnpm install`
- Start the server: `pnpm run dev`
- Start the CLI client: `pnpm run client:cli`
- Type-check: `pnpm run typecheck`
- Build the server bundle: `pnpm run build`
- Run all tests: `pnpm run test`
- Format source files: `pnpm run format`

## Development workflow

- Package manager: `pnpm` (workspace root in [`package.json`](package.json); members declared in [`pnpm-workspace.yaml`](pnpm-workspace.yaml))
- Runtime: Node.js (≥20) — runs `.ts` source directly via type-stripping
- Main server entrypoint: [`packages/server/src/main.ts`](packages/server/src/main.ts)
- Public exports: each package's `src/index.ts` barrel (e.g. [`packages/core/src/index.ts`](packages/core/src/index.ts)); there is no single root barrel
- Build output: `dist/` (server bundle only)

Environment notes for the server:

- Optional `.env` and `.env.local` files are loaded by the server and CLI entrypoints via Node's native `process.loadEnvFile` (first write wins; missing files are ignored). Requires Node >= 20.12.
- Relevant variables include `CORDN_SERVER_PRIVATE_KEY`, `CORDN_RELAY_URLS`, `CORDN_SERVER_NAME`, `CORDN_SERVER_ABOUT`, `CORDN_SERVER_WEBSITE`, `CORDN_ANNOUNCED`, `CORDN_STORAGE_BACKEND`, and `CORDN_SQLITE_PATH`

## Testing instructions

- Test runner: Vitest configured in [`vitest.config.ts`](vitest.config.ts)
- Run full suite: `pnpm run test`
- Run a single test file: `pnpm exec vitest run packages/coordinator/src/coordinator.test.ts`
- Run matching tests by name: `pnpm exec vitest run -t "key package"`

Test locations:

- unit/integration tests live beside source files under each `packages/*/src/`
- file naming follows `*.test.ts` and `*.integration.test.ts`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cordn-msg/cordn](https://github.com/Cordn-msg/cordn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
