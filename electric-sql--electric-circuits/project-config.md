---
trigger: always_on
description: Guidance for AI agents working in **electric-circuits** — an Electric-style reactive sync engine. App
---

# AGENTS.md

Guidance for AI agents working in **electric-circuits** — an Electric-style reactive sync engine. App
writes go to **Postgres**; a Rust engine turns logical-replication changes into **live shapes**
(incrementally maintained, fully de-duplicated); **durable streams** is the log between them. Two
client surfaces: the Electric-compatible `GET /v1/shape` (works with the ElectricSQL TS client) and
the extended `@electric-circuits/client` API (shapes + subset queries + live aggregations — the surface
the project is growing toward).

## Layout

| Path | What |
|---|---|
| `apps/engine` | Rust engine. Key files: `engine/` (the engine module — `sequencer.rs` the LSN-ordered sequencer, `lifecycle.rs` shape creation/sharing/retention, `circuit_serving.rs` circuit-tier serving, `executors.rs` routers/filters/folds, `planning.rs` circuit placement, `catalog.rs` durable catalog, `introspection.rs` graph/state, `membership.rs` the shared membership kernel (flips, query-backs), `output.rs` envelope codec, `mod.rs` the `Engine` handle), `arrangements.rs` (the circuit: in-memory counts pipelines, group-aggregated boot seeding), `subquery.rs` (cross-table registry: shared inner-set nodes, flips, absolute emission), `replication.rs` (streaming pgoutput ingestor) + `pgoutput.rs` (message decoder), `pg.rs` (backfill + `SnapshotGate`), `electric.rs` (`/v1/shape`), `where_sql.rs`/`sql.rs` (SQL⇄predicate), `ds.rs` (streams client incl. `append_reliable`). |
| `apps/api` | tRPC API (`router.ts`) over the engine + durable-streams (`core.ts`). |
| `packages/protocol` | Shared types + the change-event envelope (`types.ts`, `envelope.ts`). |
| `packages/client` | Browser client: `shape()`, `subset()` (see `subset.ts` — LSN watermarks + tombstones), `aggregate()`. All lifecycles tracked; `close()` is one-shot and deletes server-side with retry. |
| `packages/conformance` | The real test suite — engine vs oracle, incl. live replication, fuzz, NULLs, concurrency, shape sharing. |
| `packages/oracle` | Reference implementation shapes are checked against. |
| `packages/bench` | Benchmarks incl. the **benchmarking-fleet runner** (`electric-bench-runner.ts`, `pnpm bench:fleet` — auto-clones electric-sql/benchmarking-fleet). |
| `packages/loadgen` | Headless load generator (state-machine users; memory/CPU/disk sampling; Docker-scalable clients). |
| `electric-conformance/` | Electric's own oracle/property/integration tests pointed at our `/v1/shape`. |
| `docker/` | Containerized stack: `compose.yaml` (postgres + ds + engine + api), `Dockerfile.engine`, `Dockerfile.node`. `pnpm docker:up`. |
| `apps/pipeline-viz` | Live pipeline explorer (shapes, shared families/nodes, reactive per-node state + index dumps) over `GET /graph` + `/state` + `/trace`. |
| `examples/linearlite` | The flagship demo. `scripts/linearlite.sh start <size>` boots everything. |

## Docs (read these before designing)

- `README.md` — the system in one page + the consistency model summary.
- `docs/ARCHITECTURE.md` — the as-built architecture: ingest, `SnapshotGate` fencing, sharing,
  subquery registry, reliability model, Electric adapter, client layer.
- `docs/ivm-engine-internals.md` — engine execution strategies + the analytical cost model,
  including the three-tier serving model (circuit/routing/fallback): see
  [`docs/ivm-engine-internals.md#serving-tiers-compiled-routed-fallback`](docs/ivm-engine-internals.md#serving-tiers-compiled-routed-fallback).
- `docs/live-queries-guide.md` — user/integrator guide.
- `docs/deployment-postgres.md` — Postgres-as-source-of-record setup.
- Each package has its own `README.md` (surface, commands, env knobs).

## Designing dbsp circuits: pipelines vs shapes

The load-bearing mental model: **pipelines are few and fixed; shapes are many and dynamic —
and the fan-out between them lives outside the circuit.** A pipeline's output is keyed by
*cohort groups* (project, (project, status), aggregate group, …). A shape is a selection or
union over those groups, materialized as a per-shape stream at the delivery edge. Shape
cardinality can vastly exceed pipeline cardinality: a subquery shape filtering issues exists
per *combination* of projects a client asks for, yet every combination is fed from the same
`issues_by_project` pipeline — the circuit never grows with shape count, only the routing
table does. If a design makes the circuit's structure scale with shapes, users, or parameter
combinations, it is wrong (the circuit-per-shape trap: structure must never scale with
subscriptions).

The recipe for capturing an app's query set in one circuit:

1. **Enumerate call sites → collapse to templates.** Parameters become *data* (keys in the
   output index, rows in an input relation) — never circuit structure.
2. **Find the access cohort** (LinearLite: the project) and key every pipeline output by it,
   never by user or shape. Per-shape work happens only at the fan-out edge: a shape = the set
   of cohort groups its parameters select, unioned by delivery. The union is correct only when
   the cohort key **partitions** the table (a row lives in exactly one group) — overlapping
   groups would double-emit and need dedup at the edge. Genuinely per-user predicates

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [electric-sql/electric-circuits](https://github.com/electric-sql/electric-circuits) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
