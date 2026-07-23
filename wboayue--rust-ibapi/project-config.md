---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Quick Start

The rust-ibapi crate is a Rust implementation of the Interactive Brokers TWS API with both synchronous and asynchronous support.

**Important:** The async client is enabled by default. You can opt into the blocking client with `--features sync`, and the two features may be combined:
- `cargo build` (default features) exposes the async client as `client::Client`
- `cargo build --no-default-features --features sync` enables only the blocking client
- `cargo build --no-default-features --features "sync async"` enables both; the blocking API lives under `client::blocking::Client`

## Documentation Index

### Getting Started
- [**Quick Start Guide**](docs/quick-start.md) - Get up and running in minutes
- [**Examples Guide**](docs/examples.md) - Running and writing examples
- [**Troubleshooting**](docs/troubleshooting.md) - Common issues and solutions

### Core Concepts
- [**Architecture Overview**](docs/architecture.md) - System design, components, and module organization
- [**Feature Flags**](docs/feature-flags.md) - Sync vs async modes and feature guards
- [**API Patterns**](docs/api-patterns.md) - Builder patterns, protocol versions, and common patterns

### Development
- [**Code Style Guidelines**](docs/code-style.md) - Coding standards and conventions
- [**Build and Test**](docs/build-and-test.md) - Build commands, testing patterns, and CI
- [**Testing Patterns**](docs/testing-patterns.md) - Test fixture stratification: `MessageBusStub` / `MemoryStream` / handshake-replay listener
- [**Integration Tests**](docs/integration-tests.md) - Writing tests against a live gateway
- [**Extending the API**](docs/extending-api.md) - Adding new TWS API functionality

## Version 3.0 Philosophy

Version 3.0 is a breaking release. Fix API inconsistencies even when it means breaking changes — consistent naming, idiomatic Rust patterns, and a clean public API take priority over backward compatibility.

## Branches

- **`main`** — 3.x development and releases. **This is the only actively maintained branch.**
- **`v2-stable`** — 2.x maintenance, frozen. Only touch it when a **specific bug** must be backported there, and only when explicitly asked.

Default all work — features and fixes — to `main` alone. Do **not** open v2-stable PRs by default; only backport a specific named bug on explicit request. Changes to either branch go through pull requests.

## Key Points to Remember

1. **Feature coverage**: default-async, sync-only, and all-features builds must compile **and** pass tests when touched
2. **Follow module structure**: Client methods live as `impl Client` blocks in domain modules (e.g., `news/sync.rs`), not in `client/sync.rs` or `client/async.rs`. Prefer the flat `<domain>/sync.rs` + `<domain>/async.rs` layout over `<domain>/sync/mod.rs` — the project minimizes `mod.rs` files. Use `common/` for shared logic between sync/async. Protobuf decoders live in each domain's `common/decoders.rs`; shared proto→domain converters live in `proto/decoders.rs`
3. **Run quality checks before committing**: every command in [Quick Commands](#quick-commands) below — formatter, all three clippy configs, all three rustdoc configs. `cargo test --doc` only validates doc-test compilation, not intra-doc link resolution; that's why the rustdoc step is separate
4. **Design principles**: see [docs/code-style.md](docs/code-style.md#design-principles) for SRP, composition, and avoiding repetition. Project-specific constraint: max 3 params per function — use a builder for 4+. The receiver (`self` / `&self` / `&mut self`) does not count toward this budget; `pub fn foo(&self, a, b, c)` is at the limit, `pub fn foo(&self, a, b, c, d)` needs a builder. **Builder rationale matters**: the rule's goal is to spare callers from positional-argument noise when there are optional / defaultable fields. A 4+ param function where *every* arg is required with no reasonable default gains little from a builder — `client.foo(a, b, c, d, e)` is no worse than `client.foo(a).b(b).c(c).d(d).e(e).run()`. For all-required signatures, prefer grouping related args into a struct (e.g. `DateRange { start, end }`) or accept the violation with a comment, rather than mechanical builder conversion
5. **Never use `block_on` in async code**: Do not use `futures::executor::block_on()` inside async contexts — it blocks tokio worker threads and risks deadlocks. Use atomics (`AtomicI32`, etc.) for lock-free access to rarely-written values, or make the function `async` and `.await` the lock
6. **Every new function needs a test**: Before opening a PR, verify every new `pub`/`pub(crate)` function has a corresponding unit test. Review test coverage as a final step — missing tests should block the PR. **Overall line coverage target: 90%+** — run `just cover` and check the report; if a touched module drops below 90%, add tests before opening the PR

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wboayue/rust-ibapi](https://github.com/wboayue/rust-ibapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
