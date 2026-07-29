---
trigger: always_on
description: This is the official Rust client for [NATS.io](https://nats.io), a high-performance messaging system.
---

# AGENTS.md — AI Agent Instructions for nats.rs

## Project Overview

This is the official Rust client for [NATS.io](https://nats.io), a high-performance messaging system.
The active crate is **`async-nats`** — a fully async, Tokio-based client.

> **The `nats` crate (synchronous client) is deprecated.** It receives security fixes only. All new
> work happens in `async-nats`. Do not modify the `nats/` directory unless explicitly asked.

## Repository Structure

```
nats.rs/
├── async-nats/          # Primary crate — async NATS client (this is where you work)
│   ├── src/
│   │   ├── lib.rs       # Entry point, core types (Command, ServerOp, ClientOp, connect())
│   │   ├── client.rs    # Client handle — publish, subscribe, request, flush
│   │   ├── connection.rs# Low-level I/O — protocol parsing, read/write buffers
│   │   ├── connector.rs # Connection establishment, reconnection, auth handshake
│   │   ├── options.rs   # ConnectOptions builder
│   │   ├── error.rs     # Generic Error<Kind> pattern
│   │   ├── header.rs    # HeaderMap — NATS message headers
│   │   ├── subject.rs   # Subject type, ToSubject trait
│   │   ├── status.rs    # StatusCode (100-999 NATS protocol codes)
│   │   ├── message.rs   # Message types
│   │   ├── tls.rs       # TLS configuration
│   │   ├── auth.rs      # Auth trait definitions
│   │   ├── auth_utils.rs# Credential file parsing
│   │   ├── crypto.rs    # Crypto feature support
│   │   ├── id_generator.rs
│   │   ├── jetstream/   # JetStream API (feature-gated)
│   │   │   ├── context.rs    # JetStream context — streams, publishing
│   │   │   ├── stream.rs     # Stream management, consumer creation
│   │   │   ├── consumer/     # Pull, Push, Ordered consumers
│   │   │   ├── kv/           # Key-Value store (feature: "kv")
│   │   │   ├── object_store/ # Object store (feature: "object-store")
│   │   │   ├── errors.rs     # JetStream error codes
│   │   │   ├── message.rs    # JetStream message types
│   │   │   ├── publish.rs    # PublishAck
│   │   │   └── response.rs   # Response wrapper
│   │   └── service/     # Service API (feature: "service")
│   │       ├── mod.rs        # Service, ServiceBuilder
│   │       ├── endpoint.rs   # Endpoint handling
│   │       └── error.rs      # Service errors
│   ├── tests/           # Integration tests (require running nats-server)
│   │   ├── configs/     # NATS server config files for tests
│   │   ├── client_tests.rs
│   │   ├── jetstream_tests.rs
│   │   ├── kv_tests.rs
│   │   ├── object_store.rs
│   │   ├── service_tests.rs
│   │   └── ...
│   ├── examples/        # Runnable examples
│   └── benches/         # Criterion benchmarks
├── nats-server/         # Test harness — launches real nats-server instances
├── nats/                # DEPRECATED sync client — do not modify
└── nats-core/           # Experimental embedded/no_std client (separate)
```

## Build & Test Commands

```bash
# Format (required: nightly toolchain)
cargo +nightly fmt

# Lint — CI denies all clippy warnings
cargo clippy --benches --tests --examples --all-features -- --deny clippy::all

# Build
cargo build --all-targets

# Test (standard — requires nats-server binary in PATH)
cargo test --features slow_tests,websockets -- --nocapture

# Test specific TLS backend
cargo test tls --no-default-features \
  --features jetstream,kv,object-store,service,nkeys,nuid,crypto,websockets,ring

# Test feature combinations (thorough)
bash .cargo-hack-check.sh

# Build docs
cargo doc --no-deps --all-features

# Check MSRV (1.88.0)
cargo +1.88.0 check

# Check licenses
cargo deny check licenses
```

**nats-server**: Tests require the `nats-server` binary. Install via Go:
```bash
go install github.com/nats-io/nats-server/v2@main
```

## CI Requirements

All of these must pass before a PR is merged:

| Check | Command |
|-------|---------|
| Format | `cargo +nightly fmt -- --check` |
| Clippy | `cargo clippy --benches --tests --examples --all-features -- --deny clippy::all` |
| Tests | `cargo test --features slow_tests,websockets` |
| Docs | `cargo doc --no-deps --all-features` |
| MSRV | Build with Rust 1.88.0 |
| Licenses | `cargo deny check licenses` |
| Spelling | `cargo spellcheck --code 1` |
| Examples | `cargo check --examples` |
| Min versions | `cargo check --locked --all-features --all-targets` (with `-Zminimal-versions`) |
| TLS backends | Tests run separately with `ring`, `aws-lc-rs`, and `fips` |
| Platforms | Ubuntu, macOS, Windows |

**Environment**: CI sets `RUSTFLAGS="-D warnings"` — all warnings are errors.

## Formatting Rules

Defined in `.rustfmt.toml`:
- `max_width = 100`
- `reorder_imports = true`
- `format_code_in_doc_comments = true`
- Edition: 2018 (rustfmt setting — the crate itself is edition 2021)

Always run `cargo +nightly fmt` before committing.

## Architecture

### Internal Communication

```
Client (cloneable handle)
  │  sends Command variants via mpsc channel
  ▼
ConnectionHandler (single task)
  │  manages subscriptions, multiplexer, ping/pong
  │  drives reconnection via Connector
  ▼
Connection (protocol I/O)
  │  read_buf (BytesMut) / write_buf (VecDeque<Bytes>)
  │  parses ServerOp, serializes ClientOp
  ▼
NATS Server (TCP / TLS / WebSocket)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nats-io/nats.rs](https://github.com/nats-io/nats.rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
