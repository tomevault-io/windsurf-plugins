---
trigger: always_on
description: FokosDB is a globally strongly-consistent key-value database on Cloudflare Durable Objects. Its API and transaction model follow DynamoDB. It ships as the `fokosdb` npm package.
---

# FokosDB

FokosDB is a globally strongly-consistent key-value database on Cloudflare Durable Objects. Its API and transaction model follow DynamoDB. It ships as the `fokosdb` npm package.

## Rules

- Write in Simplified Technical English (ASD-STE100). `.claude/skills/spec-write/references/ste-rules.md` has the rules.
- Correctness and reliability come first. Write as little code as the task needs.
- A code comment must stand alone. Never name a discussion, a report, a plan, or a feature that the codebase does not contain.
- Run `pnpm test` in a subagent. Its output is long.
- Your knowledge of the Workers platform can be out of date. Read the current [Workers](https://developers.cloudflare.com/workers/) and [Durable Objects](https://developers.cloudflare.com/durable-objects/best-practices/rules-of-durable-objects/) documentation before you change either, and read a limit from the product's `/platform/limits/` page.
- Do not add a production hook for a test.

## Commands

This is a pnpm workspace. Run the scripts of the root `package.json` from the repository root.

- `pnpm build`, `pnpm test`, `pnpm check` (build, lint, typecheck, format), `pnpm fmt`.
- `pnpm cf-typegen` after you change a binding. Each wrangler project keeps its own `worker-configuration.d.ts` and its own `.wrangler/` state.
- The examples import the built `dist/`, so a source change needs a build. `pnpm test` and `pnpm dev` build first.
- There are two wrangler projects: `packages/fokosdb/wrangler.jsonc` gives vitest an entrypoint and is never deployed, and `examples/http-api/wrangler.jsonc` is the deployable example.
- `.github/workflows/preview-release.yml` publishes a preview build through pkg.pr.new. Keep it to ONE `pkg-pr-new publish` call and pass extra packages as extra arguments, because a second call counts as spam.

## Package layout

`packages/fokosdb/src` has three parts. Convention keeps them apart, not the module system.

- `client/` — `db.ts` and the entry barrel. Published as `fokosdb/client`.
- `server/` — the two Durable Object classes. Published as `fokosdb/server`.
- `shared/` — what both sides use. tsdown inlines it into whichever entry reaches it.

**The client must never import a Durable Object class as a value.** That pulls the whole server implementation into `dist/client`. Use the type-only helpers in `shared/do-stubs.ts` and keep every class import `import type`. `pnpm build` enforces the rule, pins the packages the client may import, and holds the client bundle under a size budget.

A cohesive folder stays whole inside `shared/` even when only one side uses it. An entry pulls in only the modules it names.

## Architecture

- **`PartitionDO`** (`src/server/do-partition.ts`) — holds items in SQLite, one DO per partition shard. It serves single-item reads and writes, acts as a resource manager in 2PC, and splits itself when it grows past its cap. It hosts `FokosShardingRuntime` (`src/sharding/runtime.ts`): the runtime owns identity, routing, the route caches, the repartition flow, the read-through, and the alarm; the class owns its SQLite schema, its operations, its admission and split policy, its migration pages, and its TTL timer.
- **`TransactionCoordinatorDO`** (`src/server/do-transaction-coordinator.ts`) — the second host of the sharding runtime. It drives 2PC for a write transaction. The idempotency token is its route key, and its shard group is `fokos.tc.<shardGroup>`: `coordinatorRootsN` roots that split by hash when they grow past `hashSplitConditions.maxSizeMb`. A read transaction runs in the Worker instead.
- **`FokosDB`** (`src/client/db.ts`) — the client entry point. It routes with `FokosRouter`, sends a multi-partition write to a coordinator, and drives a multi-partition read itself.

Every partition RPC answers a `FokosEnvelope<T>`: `value` is the result and `routing` is the route evidence. `FokosRouter.unwrap` opens it, and `client/partition-info.ts` builds the public `PartitionInfo` from `routing.servedBy` and `routing.forwardCount`. An error a partition raises carries its `routing` as an own property, and `withFokosErrors` in `db.ts` turns it into the same public `meta` and drops the routing.

An item has a `hashKey`, an optional `sortKey` (default `""`), data as `Uint8Array | string`, a `version` that every write increments, and an optional TTL.

**`PartitionContext` travels in every RPC.** Workers RPC cannot configure a DO at instantiation, so the topology configuration goes with each request and the DO compares it with the one it stored. Never read `env[ctx.ns]` outside `shared/do-stubs.ts`: use `partitionNamespace`, `txCoordinatorNamespace`, or the stub helpers, because each one applies the configured jurisdiction.

## Partitions

- `rootTreesN` root partitions exist at startup, and a hash of the hash key selects one. A partition ID is opaque: read it only through `PartitionIdHelper`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lambrospetrou/fokosdb](https://github.com/lambrospetrou/fokosdb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
