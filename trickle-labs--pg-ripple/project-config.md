---
trigger: always_on
description: **pg_ripple** is a PostgreSQL 18 extension written in Rust (pgrx 0.18) that implements a high-performance RDF triple store with native SPARQL query execution. See [plans/implementation_plan.md](plans/implementation_plan.md) for the full architecture and [ROADMAP.md](ROADMAP.md) for the phased delivery plan.
---

# pg_ripple — Agent Guidelines

**pg_ripple** is a PostgreSQL 18 extension written in Rust (pgrx 0.18) that implements a high-performance RDF triple store with native SPARQL query execution. See [plans/implementation_plan.md](plans/implementation_plan.md) for the full architecture and [ROADMAP.md](ROADMAP.md) for the phased delivery plan.

> **Implementation status** (as of 2026-05-22): v0.128.0 is released and all pg_regress tests pass. The full SPARQL 1.1 stack, SHACL (including `sh:equals`/`sh:disjoint`/`sh:SPARQLRule`), Datalog (including aggregation, magic sets, `owl:sameAs` canonicalization, demand-filtered inference, well-founded semantics, tabling, DRed, parallel stratum evaluation, and worst-case optimal joins), HTAP storage, parallel merge workers, cost-based federation, live CDC subscriptions, streaming SPARQL cursors, explain/observability, JSON-LD framing, CONSTRUCT/DESCRIBE/ASK views, vector + SPARQL hybrid search, GraphRAG export, Datalog-native PageRank (with IVM, WCOJ, Prometheus gauges), probabilistic reasoning (noisy-OR, confidence propagation), bidirectional relay, and the `pg_ripple_http` companion service are all implemented. The JSON mapping relational writeback feature (v0.128.0 JSON-WRITEBACK-01) is also implemented, completing the full round-trip between JSON payloads, RDF graphs, and relational tables. One release remains: v1.0.0 (production hardening, stress testing, security audit). All four conformance suites run in CI: W3C SPARQL 1.1 (smoke subset required; full suite informational), Apache Jena (~1,000 tests; non-blocking until ≥95% pass rate), WatDiv (100 query templates; non-blocking, correctness + performance), LUBM (14 OWL RL queries; required), and OWL 2 RL (informational until ≥95% pass rate).

## Tech Stack

| Concern | Technology |
|---|---|
| Language | Rust, Edition 2024 |
| PG binding | pgrx 0.18 (`pg18` feature flag) |
| PostgreSQL target | 18.x only |
| SPARQL parser | `spargebra` |
| SPARQL optimizer | `sparopt` (first-pass algebra optimizer) |
| RDF parsers | `rio_turtle`, `rio_xml`; `oxrdf` (v0.3, direct dep since v0.25.0) for RDF-star graph model |
| Hashing | `xxhash-rust` (XXH3-128) |
| Serialization | `serde` + `serde_json` |
| Tests | `#[pg_test]`, `cargo pgrx regress`, `pgbench` via `pgrx-bench` |

## Architecture

```
src/lib.rs                 — pgrx entry points, _PG_init, GUC parameters
src/bidi/                  — Bidirectional relay for pg-trickle CDC
src/citus/                 — Citus sharding integration, shard pruning
src/construct_rules/       — SPARQL CONSTRUCT writeback rules
  src/construct_rules/catalog.rs   — ensure_catalog bootstrap
  src/construct_rules/scheduler.rs — topological sort + source-graph parse helpers
  src/construct_rules/delta.rs     — compile_construct_to_inserts + run_full_recompute
  src/construct_rules/retract.rs   — Delete-Rederive retraction
src/datalog/               — Datalog rule parser, stratifier, SQL compiler, built-in RDFS/OWL RL
src/dictionary/            — IRI/blank-node/literal → i64 encoder (XXH3-128 + LRU cache)
src/export/                — Turtle / N-Triples / JSON-LD serialization
src/framing/               — JSON-LD framing logic
src/gucs/                  — GUC parameter definitions and registration
src/llm/                   — LLM/RAG integration, vector hybrid search
src/pagerank/              — Datalog-native PageRank, IVM, WCOJ, centrality measures
src/schema/                — Internal schema management (_pg_ripple namespace)
src/shacl/                 — SHACL shapes → DDL constraints + async validation pipeline
src/sparql/                — SPARQL text → spargebra algebra → SQL → SPI execution → decode
  src/sparql/parse.rs      — query complexity checks + ARQ aggregate preprocessing
  src/sparql/plan.rs       — SPARQL algebra → SQL translation + plan cache
  src/sparql/decode.rs     — batch dictionary decode for SPARQL results
  src/sparql/execute.rs    — SPI execution, CONSTRUCT/DESCRIBE/UPDATE, explain
src/storage/               — VP tables, HTAP delta/main partitions, merge background worker
  src/storage/promote.rs   — VP promotion helpers (promote_predicate, promote_rare_predicates)
src/uncertain_knowledge_api/ — Probabilistic reasoning, noisy-OR, confidence propagation
src/views/                 — CONSTRUCT/DESCRIBE/ASK view management
```

`pg_ripple_http/src/` layout (v0.91.0):
```
pg_ripple_http/src/main.rs          — startup code + main(); includes COMPAT-01 extension version check
pg_ripple_http/src/common.rs        — AppState, check_auth, env_or, redacted_error
pg_ripple_http/src/spi_bridge.rs    — execute_sparql_with_traceparent + execute_select/ask/construct/describe
pg_ripple_http/src/arrow_encode.rs  — Arrow Flight bulk-export endpoint (streams via Body::from_stream)
pg_ripple_http/src/stream.rs        — SSE/chunked-transfer streaming
pg_ripple_http/src/datalog.rs       — Datalog REST API handlers
pg_ripple_http/src/metrics.rs       — Prometheus metrics
pg_ripple_http/src/routing/         — HTTP routing module (split in v0.91.0)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [trickle-labs/pg-ripple](https://github.com/trickle-labs/pg-ripple) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
