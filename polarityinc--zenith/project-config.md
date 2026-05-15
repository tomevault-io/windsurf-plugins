---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

ZenithDB — a Rust columnar database engine purpose-built for AI agent traces. 18-crate workspace, one `cargo build`. Engine name `zenithdb`, CLI binary `zen`, all crates prefixed `zen_`.

## Toolchain

- Rust **1.87 stable**, pinned via `rust-toolchain.toml` (rustup will pick it up automatically).
- `protoc` 3.21+ is required to build (`brew install protobuf` / `apt-get install protobuf-compiler`). Several crates run `prost-build` / `tonic-build` in their `build.rs`.

## Common commands

CI runs four gates and rejects on any of them. Match them locally before pushing:

```bash
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings   # -D warnings is enforced
cargo test --workspace --all-features
cargo deny check                                        # licenses + advisories
```

`cargo audit --deny warnings` runs in CI as a fifth gate.

### Running a single test

```bash
cargo test -p <crate> <test_name>                       # unit/integration test in a crate
cargo test -p zen_format encoder::tests::roundtrip      # by full path
cargo test -p integration auth::                        # the integration suite lives in tests/integration
cargo test -p integration -- --nocapture                # see println! / tracing output
```

The integration / perf / chaos suites are **separate crates** under `tests/` (not `cargo test` default targets — they're listed in the workspace `members`). Always pass `-p integration` / `-p perf` / `-p chaos` to target them.

### Run the server

```bash
# Default dev profile — SQLite catalog + local-FS object store under ./data/. Zero external services.
cargo run --release -p zen_cli -- serve --config examples/zenithdb.dev.toml
```

`zen_cli` ships **two binaries**: `zen` (the user-facing CLI with `serve`, `bench`, `admin-backup`, `admin-restore`, etc.) and `zenithdb` (server-only entrypoint). Both live in `crates/zen_cli/src/`.

For a Postgres + MinIO stack:

```bash
docker compose -f deploy/docker/docker-compose.dev.yml up -d
ZEN_PROFILE=prod-like cargo run --release -p zen_cli -- serve
```

### Benchmarks

The bench suite is first-class. If you touch storage, query, or compaction, run a before/after.

```bash
cargo run --release -p zen_cli -- bench gen --rows 4000000 --output /tmp/corpus.bin
cargo run --release -p zen_cli -- bench load --input /tmp/corpus.bin --target http://localhost:8080
cargo run --release -p zen_cli -- bench run --suite all --warmup 30s --duration 60s \
  --output bench-results/$(date +%Y%m%d-%H%M%S).json
cargo run --release -p zen_cli -- bench compare \
  --baseline bench-results/baseline.json \
  --candidate bench-results/<your-run>.json
```

`bench-results/baseline.json` is committed and is the reference. Only update it as a deliberate, separate change.

### Fuzzing

Targets in `fuzz/fuzz_targets/`: `fuzz_segment_parse`, `fuzz_wal_parse`, `fuzz_zenithql_parse`. Run with `cargo +nightly fuzz run <target>` from `fuzz/`. Re-run the relevant target whenever you change a parser, codec, or wire format.

## Architecture (the big picture)

### Request flow

```
Clients (REST / gRPC / OTLP)
        │
        ▼
   Gateway (axum + tonic)        — crates/zen_server
   ├── ingest  → Writer (memtable + WAL)        — zen_memtable, zen_wal
   └── query   → Querier (planner + executor)   — zen_query, zen_ql
                       │
                       ▼
              Catalog (sqlite / postgres)        — zen_catalog
                       │
                       ▼
              Object storage (fs / S3 / GCS / Azure) — zen_storage
              ├── WAL (.wal)
              └── Segments (.zseg)               — zen_format
```

### The five "moat" crates

These contain the engine's defining work. Changes here need close review and usually a CHANGELOG entry:

| Crate            | Responsibility |
|------------------|----------------|
| `zen_format`     | PAX segment encoder/decoder; FSST/ZSTD/Gorilla/FoR/RLE/dict codecs; segment footer + offset directory layout. **On-disk format = stability surface.** |
| `zen_compactor`  | Streaming k-way merge compactor. Enforces row-group-level **trace-locality**: every span of a trace lands in one row group. |
| `zen_query`      | Vectorized scan operator with **late materialization** — wide columns are never decoded for rows that didn't survive every other filter. Predicate pushdown lives here. |
| `zen_fts`        | Tantivy embedded **inline in segments** (not a sidecar index). |
| `zen_wal`        | WAL on object storage with conditional PUT, queryable on PUT-ack and merged with compacted segments at query time. |

### Supporting crates


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Polarityinc/zenith](https://github.com/Polarityinc/zenith) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
