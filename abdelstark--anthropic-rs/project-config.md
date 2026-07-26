---
trigger: always_on
description: Operational notes for Claude Code (and other AI agents) working on this
---

# CLAUDE.md

Operational notes for Claude Code (and other AI agents) working on this
repository. Read this in full before making changes.

## Project identity

`anthropic-rs` is a typed, async Rust client for Anthropic's Messages API.
It is a **library crate** intended to be consumed via crates.io. There is no
binary and no long-running service. The user is a Rust developer building an
application that talks to `https://api.anthropic.com`.

## Workspace layout

This is a Cargo workspace with **three** independent workspaces:

```
/                           ← top-level workspace (publishes the SDK)
├── Cargo.toml              ← workspace = ["anthropic"]
├── anthropic/              ← the published `anthropic` crate
│   ├── Cargo.toml
│   ├── README.md           ← also surfaces on crates.io / docs.rs
│   ├── src/
│   │   ├── lib.rs          ← public re-exports + `__fuzz` entry points
│   │   ├── client.rs       ← Client / ClientBuilder / retries / streaming transport
│   │   ├── error.rs        ← AnthropicError + ApiError payload
│   │   ├── types.rs        ← Messages API request / response / content blocks / RetryPolicy
│   │   ├── stream.rs       ← StreamAccumulator + collect helpers
│   │   ├── tool_loop.rs    ← run_tool_loop agentic helper
│   │   ├── batches.rs      ← Message Batches API
│   │   ├── count_tokens.rs ← count_tokens endpoint
│   │   └── models.rs       ← list_models / get_model
│   └── tests/              ← wiremock-backed integration tests
├── examples/               ← SECOND workspace, never built by `cargo test` at root
│   ├── Cargo.toml          ← workspace = ["basic-messages", "streaming-messages", "tool-loop"]
│   ├── basic-messages/
│   ├── streaming-messages/
│   └── tool-loop/
├── fuzz/                   ← THIRD workspace — `cargo fuzz` targets (requires nightly)
│   ├── Cargo.toml          ← excluded from the root workspace so stable builds never pull libfuzzer-sys
│   └── fuzz_targets/
│       ├── parse_error.rs
│       └── parse_results_jsonl.rs
└── deny.toml               ← cargo-deny policy (advisories, licenses, sources, bans)
```

Both `examples/` and `fuzz/` are **separate workspaces** so the SDK can
be published without dragging the example crates along and so a top-level
`cargo test` does not try to resolve nightly fuzz dependencies. To build
an example you must `cd examples` first; to run a fuzz target you must
`cd fuzz` and use `cargo +nightly fuzz run <target>`.

## Tech stack

- **Language**: Rust 2021 edition. MSRV pinned to 1.82 via
  `package.rust-version`; an MSRV job in CI reads that value and builds +
  tests the workspace on the declared toolchain.
- **Async runtime**: `tokio` (multi-thread, macros).
- **HTTP**: `reqwest` 0.12 with `json` + `stream` features.
- **SSE**: `reqwest-eventsource` 0.6.
- **TLS**: `rustls` (default) or `native-tls` via Cargo features.
- **Retries**: `backoff` 0.4 (`ExponentialBackoff` for 429s, honoring `Retry-After`).
  Per-call overrides live on `RetryPolicy` and are plumbed through the
  request builders.
- **Tracing**: optional `tracing` Cargo feature — compiled out entirely
  when disabled. Spans wrap `execute_bytes` on the transport critical path
  and carry `method`, `path`, `status`, `attempts`, and `duration_ms`.
- **Errors**: `thiserror`.
- **Tests**: `wiremock` 0.6 + `dotenvy` (dev only).
- **Supply chain**: `cargo audit` + `cargo deny` run on every PR via
  `.github/workflows/supply-chain.yml`.
- **Fuzzing**: `fuzz/` sub-crate with `cargo-fuzz` harnesses for
  `parse_error` and `parse_results_jsonl` (the two parsers on the
  transport critical path that run on untrusted bytes).

## Build / test / lint commands

These commands match what CI runs (`.github/workflows/ci.yml`). They must
all be green before a PR can merge:

```bash
# Format check (uses rustfmt.toml at the repo root)
cargo fmt --all -- --check

# Lint — warnings are errors, run on every target with all features
cargo clippy --workspace --all-targets --all-features -- -D warnings

# Tests — unit + integration + doctests
cargo test --workspace --all-features

# Docs build — no broken intra-doc links
cargo doc --workspace --no-deps --all-features
```

The example workspace is built separately:

```bash
(cd examples && cargo build)
```

Fuzz targets live in their own workspace (`fuzz/`) and require nightly
Rust and `cargo-fuzz`:

```bash
(cd fuzz && cargo +nightly fuzz run parse_error -- -max_total_time=30)
(cd fuzz && cargo +nightly fuzz run parse_results_jsonl -- -max_total_time=30)
```

CI runs a short smoke version of the same commands on every PR via
`.github/workflows/fuzz.yml`.

## Conventions

### Module / API design

- Public types live in `types.rs` and the per-feature modules
  (`batches.rs`, `count_tokens.rs`, `models.rs`).
- Every request type has a builder (`MessagesRequestBuilder`,
  `CountTokensRequestBuilder`) that validates inputs locally before they
  reach the network. **Add validation to the builder, not to `Client`.**
- HTTP transport, retries, and header construction live in `client.rs` and
  must stay there. Per-endpoint methods on `Client` should be thin wrappers
  that call into `post` / `get` / `delete` / `post_stream`.
- Use the existing `AnthropicError` variants. Avoid stuffing transport

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbdelStark/anthropic-rs](https://github.com/AbdelStark/anthropic-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
