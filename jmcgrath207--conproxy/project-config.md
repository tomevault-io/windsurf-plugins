---
trigger: always_on
description: Cache proxy for heterogeneous RAG/vector search backends (Elasticsearch, OpenSearch, Qdrant, pgvector, Meilisearch, Pinecone, Milvus). Rust 2021, Axum + tonic, multi-process: lib + `conproxy` CLI + `test_runner` + `generate_embeddings` + `perf_summarize` + `hitrate_bench` + `console_snap` + `corpus_seed` + Python SDK.
---

# conproxy Agent Instructions

Cache proxy for heterogeneous RAG/vector search backends (Elasticsearch, OpenSearch, Qdrant, pgvector, Meilisearch, Pinecone, Milvus). Rust 2021, Axum + tonic, multi-process: lib + `conproxy` CLI + `test_runner` + `generate_embeddings` + `perf_summarize` + `hitrate_bench` + `console_snap` + `corpus_seed` + Python SDK.

## Product framing (for PR copy, README, and any external docs you write)

- **What:** retrieval-leg cache for **agentic RAG** (embed + upstream search). **Not** an LLM-answer cache.
- **Pitch:** cost + faster search on **hits**; agents re-query (retries, fanout, tool storms).
- **Not:** GPTCache / RedisVL SemanticCache territory; not "faster chat RAG" as the headline alone.
- **Proof:** `docs/benchmarks.md` + `make bench-hitrate*`; BYO with `make bench-hitrate-replay QUERIES=…`.
- **User-facing decision docs:** `README.md` (consider/skip + vs table), `docs/benchmarks.md`. This file is ops-focused.
- **Stability:** pinecone / milvus experimental; peer = trusted network, no mTLS.

## Fast Feedback Tiers

Three tiers — Tier 1 & 2 for the per-PR loop, Tier 3 for release publishing. Run the tier that matches your stage. Vertical-specific commands live in the `contributing` skill (Feature Test Matrix + 14 verticals).

## Local iteration loop

Default save-and-run:

```bash
make t                # cargo-nextest dev profile, falls back to cargo test --lib
make test-slow        # top-25 slowest tests
make test-filter PAT=foo
make target-prune     # drop conproxy-only build artifacts
```

`make t` warms to ~3-4 s on a 16-core host thanks to `cargo-nextest`, `lld` (`.cargo/config.toml`), and the dev-profile settings in `Cargo.toml` (`debug = "line-tables-only"`, `split-debuginfo = "unpacked"`). One-time: `make nextest-install`. Full rationale + measurements in `docs/dev-loop.md`.

### Tier 1: Smoke (every save, <60s)

```bash
cargo fmt -- --check
cargo clippy -- -D warnings
cargo test --lib
cargo test --features "embed-api" --lib
# optional lead proofs / gates:
# make proof-cascade
# make cov-scope-tune
# make sdk-smoke   # needs maturin + python3-dev
```

### Tier 2: Pre-PR Gate (before opening PR, ~8 min cold)

```bash
# 1. Format
cargo fmt -- --check

# 2-7. Lint (all feature surfaces)
cargo clippy -- -D warnings
cargo clippy --features "embed-api" --lib -- -D warnings
cargo clippy --features persistence --lib -- -D warnings
cargo clippy --features mcp --lib -- -D warnings
cargo clippy --features pgvector --lib -- -D warnings
cargo clippy --features linux-sandbox --lib -- -D warnings
cargo clippy --bin perf_summarize -- -D warnings
cargo clippy --features tokio-console --bin conproxy -- -D warnings

# 8-9. Test (lib unit tests)
cargo test --lib
cargo test --features "embed-api" --lib

# 10. Build (workspace — catches SDK proto breakage)
cargo build --workspace

# 11. Build (embed linker — ort deps, hard gate)
cargo build --features embed --lib

# 12-13. Check + test (e2e compile + pure-Rust e2e units)
cargo check --features e2e --tests
cargo test --features e2e --tests

# 14. Build (binary sanity)
cargo build --bin conproxy

# 15. MCP integration tests (conproxy mcp spawn + JSON-RPC)
cargo test --features mcp --test mcp_test
# optional Docker (label run-integration / nightly):
# make test-integration
# make e2e-smoke-core   # or e2e-cascade / e2e-federated
```

**Prerequisites:** `python3-dev` and Python headers required for `cargo build --workspace` (builds `sdk/python` cdylib). ONNX runtime libs auto-downloaded via `download-binaries` feature (`cargo build --features embed --lib`).

### Tier 2.5: PR CI gate (every PR + default branch)

`ci.yml` runs **six jobs** on every PR. `unit`, `integration`,
`integration-experimental`, `security`, and `fuzz` start in parallel;
`e2e` is gated on `unit` + `integration` so the heavy compose +
load + ignored suite never burns minutes if the cheaper gates would
block the merge.

| Job | Needs | Steps (summary) |
|-----|-------|-----------------|
| `unit` | — | fmt, clippy (all feature surfaces + bins), lib tests (default / embed-api / mcp / release), mcp_test, build (workspace + embed), release binary smoke, install-sim |
| `integration` | — | `make test-integration` (testcontainers: qdrant, ES, OS, meili, pgvector, cascade, peer, circuit, batch, metrics, context_config, singleflight) |
| `integration-experimental` | — | `make test-integration-experimental` (pinecone + milvus mocks, no real backends, ~2 min) |
| `security` | — | `cargo audit` (RustSec), `cargo deny check` (supply chain + license), `cargo clippy` with security lints (unwrap/expect/panic/indexing_slicing/arithmetic_side_effects) |
| `fuzz` | — | `cargo fuzz run` on all 5 targets, 60s each, `continue-on-error: true`. Crashes uploaded as `fuzz-artifacts` artifact. |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jmcgrath207/conproxy](https://github.com/jmcgrath207/conproxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
