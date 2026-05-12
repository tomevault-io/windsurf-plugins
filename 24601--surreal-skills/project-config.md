---
trigger: always_on
description: This skill was built from the following upstream sources. Use `check_upstream.py`
---

# Agent Briefing: surrealdb

Structured reference for AI agents. Minimal prose, maximum signal.

## Quick Start

```bash
# Check if configured
uv run {baseDir}/scripts/onboard.py --check

# Get full capabilities manifest (JSON)
uv run {baseDir}/scripts/onboard.py --agent
```

## Capabilities

| Command | Script | What It Does | When to Use |
|---------|--------|-------------|-------------|
| `doctor` | `scripts/doctor.py` | Health check: CLI, connectivity, version, storage | First run, troubleshooting, verifying environment |
| `doctor --check` | `scripts/doctor.py` | Quick pass/fail (exit code only) | CI pipelines, pre-flight checks |
| `schema introspect` | `scripts/schema.py` | Full schema dump (tables, fields, indexes, events) | Understanding existing database structure |
| `schema tables` | `scripts/schema.py` | List tables with field counts and indexes | Quick overview of database contents |
| `schema table <name>` | `scripts/schema.py` | Inspect a single table in detail | Debugging a specific table definition |
| `schema export` | `scripts/schema.py` | Export schema as SurrealQL or JSON | Reproducible deployments, version control |
| `onboard --agent` | `scripts/onboard.py` | JSON capabilities manifest | Agent self-discovery, integration setup |
| `onboard --check` | `scripts/onboard.py` | Verify prerequisites | First run, CI |

## Decision Trees

### "User wants to create a SurrealDB project"

```
1. Run doctor to verify environment:
   uv run {baseDir}/scripts/doctor.py

2. Is surreal CLI installed?
   NO  -> brew install surrealdb/tap/surreal (or see https://surrealdb.com/install)
   YES -> Continue

3. Choose storage engine (LOCAL DEV -- use scoped credentials in production):
   Development     -> surreal start memory --user root --pass root
   Single-node     -> surreal start rocksdb://data/mydb.db --user root --pass root
   Time-travel     -> surreal start surrealkv://data/mydb --user root --pass root
   Distributed     -> surreal start tikv://... --user root --pass root

4. Design schema:
   -> Reference rules/data-modeling.md for table/field patterns
   -> Reference rules/graph-queries.md if domain has relationships
   -> Reference rules/vector-search.md if semantic search needed

5. Apply schema (local dev credentials -- use scoped users in production):
   surreal import --endpoint $SURREAL_ENDPOINT --user root --pass root \
     --ns <ns> --db <db> schema.surql

6. Verify:
   uv run {baseDir}/scripts/schema.py introspect
```

### "User has a data modeling question"

```
What kind of data?
  Documents/records    -> rules/data-modeling.md (record IDs, field types, schema modes)
  Relationships/graphs -> rules/graph-queries.md (RELATE, edge tables, traversal)
  Vector embeddings    -> rules/vector-search.md (vector fields, HNSW indexes, similarity)
  Time-series          -> rules/data-modeling.md (datetime fields, range queries, aggregations)
  Geospatial           -> rules/data-modeling.md (geometry types, geo functions, spatial indexes)
  Mixed/multi-model    -> rules/data-modeling.md + relevant specialized rules

Need schema validation?
  YES -> DEFINE TABLE ... SCHEMAFULL (strict, all fields must be defined)
  PARTIAL -> DEFINE TABLE ... SCHEMALESS (flexible, defined fields are validated)
```

### "User needs to optimize performance"

```
1. Check current schema and indexes:
   uv run {baseDir}/scripts/schema.py introspect

2. Identify bottleneck type:
   Slow queries        -> rules/performance.md (EXPLAIN, index strategies)
   High write latency  -> rules/performance.md (batch operations, storage engine)
   Memory pressure     -> rules/deployment.md (resource limits, storage engine selection)
   Connection issues   -> rules/sdks.md (connection pooling, WebSocket vs HTTP)

3. Index audit:
   Missing index on filtered field  -> DEFINE INDEX ... ON TABLE ... FIELDS ...
   Full-text search slow             -> DEFINE INDEX ... SEARCH ANALYZER ...
   Vector search slow                -> DEFINE INDEX ... HNSW DIMENSION ... DIST ...

4. Storage engine review:
   Memory       -> Fast but volatile, development only
   RocksDB      -> General purpose, good read/write balance
   SurrealKV    -> Time-travel queries, versioned data
   TiKV         -> Distributed, horizontal scaling
```

### "User wants to write WASM extensions"

```
1. Reference rules/surrealism.md for Surrealism module system
2. Prerequisites: Rust toolchain, wasm32-unknown-unknown target
3. Workflow:
   a. Create Rust project with surrealism SDK
   b. Implement custom functions/analyzers
   c. Compile to WASM
   d. Deploy to SurrealDB instance
   e. Use in SurrealQL queries via custom function syntax
```

### "User wants to deploy / serve an ML model"

```
1. Reference rules/surrealml.md
2. Prerequisites: `surrealml` PyPI package 0.0.4 (extras: [sklearn] / [torch] /
   [tensorflow]); SurrealML is preview-stage and the SurrealQL invocation
   surface is unstable as of 2026-05-05.
3. The v1.4.0 documentation for `DEFINE MODEL`, `INFO FOR MODEL`,
   `ml::name<version>(...)`, `surreal ml import`, `db.upload_ml(...)`, and
   `SurMlFile.from_<framework>(...)` was retracted in v1.4.1 -- those
   surfaces were not present in current upstream. Treat anything beyond the

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [24601/surreal-skills](https://github.com/24601/surreal-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
