---
trigger: always_on
description: Guidance for working in this repository. See `README.md` for the project overview,
---

# CLAUDE.md

Guidance for working in this repository. See `README.md` for the project overview,
`docs/architecture-analysis.md` for the design rationale, `docs/graph-sharded-datastore.md`
for the storage design as built, `docs/future-work.md` for candidate directions that are
analyzed but not committed, and `docs/scalability-program.md` for the measurement-gated
performance program (nothing there is scheduled; triggers decide).

## What this is

A SpiceDB-compatible **rearchitecture** of [SpiceDB](https://github.com/authzed/spicedb)
(Google Zanzibar) on .NET 10 + Microsoft Orleans — the graph-evaluation engine and schema
compiler are ported faithfully; the system around them (dispatch, caching, storage,
distribution) is re-founded on virtual actors rather than translated mechanism-for-mechanism.
The recursive permission-check dispatch runs on Orleans virtual actors; the gRPC surface is
`authzed.api.v1`-compatible (the real `zed` CLI works against it).

## Build & test

```bash
dotnet build                                   # whole solution
dotnet test                                    # all tests
dotnet test tests/Spiceport.Conformance.Tests  # the SpiceDB conformance corpus (fast)
```

- Target framework is `net10.0`; nullable + implicit usings are ON (see `Directory.Build.props`).
- **Use the dotnet CLI for package/reference/project changes** (`dotnet add package`,
  `dotnet add reference`, `dotnet sln add`). Do not hand-edit `<PackageReference>`/
  `<ProjectReference>` items. Editing build items like `<Protobuf>` is fine.
- The grain-storage **durability tests** (`tests/Spiceport.Grains.Tests/Durability`) use
  Testcontainers and require Docker running; they spin up and dispose their own `postgres`
  container (and skip when Docker is unavailable).
- The solution file is `Spiceport.slnx` (the .NET 10 XML solution format).

## Architecture (the load-bearing ideas)

- **Storage is not *dispatch*-grain state.** Evaluation is a pure function of
  `(schema@revision, tuples@revision, request)`; dispatch grains never hold relationship data.
  The MVCC mechanics (visibility at a revision, the per-revision diff) live in `Spiceport.Datastore`
  and are reused everywhere — the `ReferenceDatastore` reference model and the per-key graph-shard
  folds (`ShardFold`, a key-restriction of the same fold) share one set of semantics.
- **Storage is an event-sourced grain (the log is the storage/compute seam).** All
  relationship/schema/counter state lives behind a single cluster-singleton `DatastoreGrain`, a
  **journaled grain whose append-only `LogEvent` log is the source of truth**; the materialized state
  is the fold. A commit is a version-checked **append** (the CAS serialization point), not a
  whole-state rewrite; the single non-reentrant activation makes the minted revision the cluster-wide
  global order. Persistence is the grain's own via `ICustomStorageInterface` over an Orleans
  grain-storage provider — **no application SQL** (per-version log entries + periodic snapshots +
  compaction). Engine reads go through **per-key `GraphShardGrain`s** (forward shards keyed by
  object, reverse shards keyed by subject — resolved via `ShardedGraphReader` behind the
  `IGraphReaderSource` seam): each shard's activation state is the per-key restriction of the fold
  (`ShardFold`), hydrated once from `ReadShard` and advanced by tailing the log, so activation *is*
  the hot-set cache — cold keys never activate, silo memory is O(hot shards), not O(graph). The
  per-shard `AppliedRevision` watermark is the closed-timestamp gate: exact/at-least-as-fresh reads
  block until the shard's watermark covers the pinned revision. Writes are **declarative
  `DatastoreGrain.Commit`** — preconditions/updates/deleteByFilter/schema (with `ExpectedSchemaHash`)
  /counters evaluated and executed on the sequencer's single non-reentrant activation, rejections
  returned as typed `CommitReply` failures with nothing applied; `ReadWriteTx` survives only as the
  `ExpectedHead` compatibility path over the same wire contract. Broad/admin scans (loose-filter
  ReadRelationships, bulk export, counters) and schema-at-revision bypass the shard mesh and fetch
  the sequencer snapshot (`ISnapshotScanner` / `ISchemaSource`). The sequencer grain still
  materializes the whole fold; slimming it (per-shard durable snapshots + tail trim) is the recorded
  next step (`docs/graph-sharded-datastore.md` §7). The same log feed
  drives **Watch** (one per-silo `LogWatchHub` notifier, no per-stream polling) and an on-by-default
  (opt-out via `MembershipWalkOptions`) **Leopard membership-walk grain mesh** (`IMembershipWalkGrain`,
  sharded as addressable per-subject walk grains — sibling recursion across grain boundaries, cold sets
  deactivate, revision-exact by construction because each hop reads a pinned MVCC snapshot) for
  `LookupResources` (**candidates, never verdicts**: Check confirmation guarantees soundness — no false
  member can survive; completeness is pinned by the walk-on==walk-off equivalence gates, because a
  silently *missing* candidate is the failure Check confirmation cannot see). See
  `docs/architecture-analysis.md` §3.5.
- **The dispatcher seam is the core mechanism.** `Spiceport.Engine`'s `CheckEngine` never

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dave-hillier-co/spiceport](https://github.com/dave-hillier-co/spiceport) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
