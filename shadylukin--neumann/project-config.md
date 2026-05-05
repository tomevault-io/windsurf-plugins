---
trigger: always_on
description: Neumann is a unified tensor-based runtime that stores relational data, graph
---

# CLAUDE.md

Neumann is a unified tensor-based runtime that stores relational data, graph
relationships, and vector embeddings in a single mathematical structure.
19 crates, ~400K+ lines of Rust (with tests), built on macOS (MacBook Air M4).

## Quality Gates -- YOU MUST FOLLOW THESE EVERY TIME

All code MUST pass these checks. Do not commit, do not mark work as done, do not move on until all four pass cleanly:

```bash
cargo fmt --check
cargo clippy --package <crate> -- -D warnings -D clippy::pedantic
cargo nextest run --package <crate>
cargo doc --no-deps
```

IMPORTANT: When clippy or pedantic reports warnings, you MUST fix them
properly -- not suppress them with `#[allow(...)]` unless there is a
genuine, documented reason (e.g., a false positive on a trait impl).
Fixing means rewriting the code to satisfy the lint. If you are unsure
whether an allow is justified, ask.

Pre-commit hook auto-runs `cargo fmt` on staged files.

### Coverage -- 95% Minimum

After making changes, YOU MUST check coverage and ensure it meets the threshold:

```bash
cargo llvm-cov --package <crate> --lcov --output-path /tmp/coverage.lcov
lcov --extract /tmp/coverage.lcov '*/<crate>/src/*' --output-file /tmp/crate_only.lcov
lcov --summary /tmp/crate_only.lcov
```

Thresholds: 95% default, neumann_shell 88%, tensor_blob 91%, query_router 92%.

If coverage drops below the threshold, you MUST add tests to bring it
back up before finishing. This is non-negotiable. Write targeted tests
for uncovered paths -- do not write dummy tests that inflate numbers
without testing real behavior.

## IMPORTANT: Things That Go Wrong

- NEVER use `unsafe` code unless absolutely necessary and well-justified
- tensor_chain's `raft.rs` is 7,684 lines -- always use targeted edits, never rewrite large sections
- When adding new engine types or query kinds, you MUST update `query_router`'s match arms
- When modifying `TensorValue` or `ScalarValue` variants, update ALL match arms across crates -- these are used everywhere
- The `tensor_store` slab types (relational_slab, graph_tensor, embedding_slab, etc.)
  each have their own locking -- do not wrap them in additional locks
- Snapshot format has versions (V2/V3) in `tensor_store/src/snapshot.rs` -- new formats must be backward-compatible
- `tensor_chain` TCP transport uses length-prefixed framing -- any protocol changes must update both encode and decode paths
- `BlobStore` is async-first -- do not use blocking I/O in tensor_blob

## Workflow -- ALWAYS FOLLOW THIS SEQUENCE

After EVERY set of changes, run these steps in order. Do not skip any step:

1. `cargo fmt` -- format the code
2. `cargo clippy --package <crate> -- -D warnings -D clippy::pedantic` -- fix ALL warnings properly (no blanket allows)
3. `cargo nextest run --package <crate>` -- all tests must pass
4. Check coverage (see above) -- must meet threshold, add tests if it doesn't
5. For cross-crate changes, also run `cargo nextest run --package integration_tests`
6. Commit with clear imperative messages, no emoji, reference issues when applicable

Do NOT tell me "all checks pass" unless you have actually run them and
they passed. If a clippy lint is hard to fix, show me the lint and your
proposed fix before applying an allow attribute.

## Workspace Structure

19 crates organized in dependency tiers:

### Foundation Layer (no workspace dependencies)

| Crate | Purpose |
| ----- | ------- |
| `tensor_store` | Core key-value storage with HNSW, sparse vectors, tiered storage |
| `tensor_compress` | Tensor Train decomposition, delta encoding, RLE compression |
| `neumann_parser` | Hand-written recursive descent parser for query language |

### Engine Layer (depends on tensor_store)

| Crate | Purpose |
| ----- | ------- |
| `relational_engine` | SQL-like tables with SIMD filtering, indexes, columnar scans |
| `graph_engine` | Directed graphs with BFS traversal, shortest path, properties |
| `vector_engine` | k-NN similarity search via HNSW with multiple distance metrics |

### Specialized Storage Layer

| Crate | Purpose |
| ----- | ------- |
| `tensor_vault` | AES-256-GCM encrypted secrets with graph-based access control |
| `tensor_cache` | Multi-layer LLM response cache (exact + semantic + embedding) |
| `tensor_blob` | S3-style content-addressable blob storage with streaming (async) |
| `tensor_checkpoint` | Atomic snapshot/restore with retention and confirmation |
| `tensor_unified` | Cross-engine unified entity operations |

### Distributed Layer

| Crate | Purpose |
| ----- | ------- |
| `tensor_chain` | Tensor-native blockchain with Raft consensus and 2PC |

### Query Execution Layer

| Crate | Purpose |
| ----- | ------- |
| `query_router` | Unified query routing across all engines |
| `neumann_shell` | Interactive CLI with readline, WAL, snapshots |
| `neumann_server` | gRPC server exposing QueryRouter |

### Testing

| Crate | Purpose |
| ----- | ------- |
| `integration_tests` | Cross-crate integration tests (267+ tests) |
| `stress_tests` | Performance and concurrency stress tests |
| `experiments` | Research and experimental features |
| `seed_model` | Geometric intelligence model implementation |

## Core Types

These are the load-bearing types across the workspace:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shadylukin/Neumann](https://github.com/Shadylukin/Neumann) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
