---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

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
- [**Rule Nodes**](docs/rules/README.md) - Project conventions as linked nodes; indexed below under [Rule index](#rule-index)
- [**Code Style Guidelines**](docs/code-style.md) - Coding standards and conventions
- [**Build and Test**](docs/build-and-test.md) - Build commands, testing patterns, and CI
- [**Testing Patterns**](docs/testing-patterns.md) - Test fixture stratification: `MessageBusStub` / `MemoryStream` / handshake-replay listener
- [**Integration Tests**](docs/integration-tests.md) - Writing tests against a live gateway
- [**Extending the API**](docs/extending-api.md) - Adding new TWS API functionality
- [**Blocked Issues**](docs/block-issues.md) - Work waiting on a gateway capture or paper-account setup

## Version 3.0 Philosophy

Version 3.0 is a breaking release. Fix API inconsistencies even when it means breaking changes — consistent naming, idiomatic Rust patterns, and a clean public API take priority over backward compatibility.

## Branches

- **`main`** — 3.x development and releases. **This is the only actively maintained branch.**
- **`v2-stable`** — 2.x maintenance, frozen. Only touch it when a **specific bug** must be backported there, and only when explicitly asked.

Default all work — features and fixes — to `main` alone. Do **not** open v2-stable PRs by default; only backport a specific named bug on explicit request. Changes to either branch go through pull requests.

## Rule index

Detail lives in [`docs/rules/`](docs/rules/README.md), one directive per file. Follow the
link when the situation matches — the node carries the mechanics, the bug class, and the
precedents. Every rule now lives in a node; nothing is inline.

### Wire protocol — protobuf-only at `server_versions::PROTOBUF_REST_MESSAGES_3` (213)

- **Writing or modifying a domain decoder** → [proto-only decoding](docs/rules/wire/proto-only-decoding.md)
- **Adding a `ResponseMessage` accessor, or a public API on a proto inbound message type** →
  [proto-aware accessors](docs/rules/wire/proto-aware-accessors.md)
- **Typing a `String` field as an enum** → [wire enum typing](docs/rules/wire/enum-typing.md)
- **Accepting an order id, minting a request id, or routing by an inbound id** →
  [id partition](docs/rules/wire/id-partition.md)
- **Adding a one-shot client method, or passing a processor to a `one_shot_*` helper** →
  [one-shot narrowing](docs/rules/wire/one-shot-narrowing.md) — the `(message type, decoder)`
  pair lives on the payload's `ProtoPayload` impl, so the call site names neither

### Code structure and style

- **Adding a client method, or creating a domain module** →
  [domain module layout](docs/rules/style/domain-module-layout.md)
- **Writing a function that takes four or more parameters** →
  [param budget](docs/rules/style/param-budget.md) — clippy's `too_many_arguments` only fires
  at eight, so nothing gates this
- **Exposing one or two items from an otherwise-private module** →
  [narrow re-exports](docs/rules/style/narrow-reexports.md)
- **About to write a `macro_rules!`, or reviewing one** →
  [macros last resort](docs/rules/style/macros-last-resort.md)
- **Adding a named builder method that sets an enum, or adding a variant to such an enum** →
  [builder enum coverage](docs/rules/style/builder-enum-coverage.md) — exhaustiveness checking
  does not reach "one setter per variant"

### Testing

- **Adding a `pub` / `pub(crate)` fn, or checking coverage before opening a PR** →
  [coverage floor](docs/rules/testing/coverage-floor.md)
- **Reviewing a new test, or asserting on captured request bytes** →
  [exercise production code](docs/rules/testing/exercise-production-code.md)
- **Building a response test fixture** → [fixture builders](docs/rules/testing/fixture-builders.md)
- **Writing a `#[cfg(test)] mod tests` block** → [sibling test files](docs/rules/testing/sibling-test-files.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wboayue/rust-ibapi](https://github.com/wboayue/rust-ibapi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
