---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Kitsune2 is a peer-to-peer / DHT communication framework written in Rust, used by Holochain. It is structured as a Cargo workspace of crates that together provide a node implementation, plus a bootstrap server for WAN peer discovery.

## Common commands

This project uses `cargo-make`, configured via `Makefile.toml`, for the standard developer workflow.

- `cargo make verify` — the default task, which runs the full check: format, clippy, doc-check, taplo, and tests.
- `cargo make static` — all static checks, without running tests.
- `cargo make test` — runs `cargo test` across the workspace.
- `cargo make fix` — applies `cargo fmt`, regenerates protobuf code, and runs `taplo format`.
- `cargo make clippy` — runs `cargo clippy --all-targets -- --deny=warnings`, so warnings are treated as errors.
- `cargo make doc-check` — runs `cargo +nightly doc --all-features --no-deps` with `RUSTDOCFLAGS=--cfg docsrs --deny=warnings`. Requires a nightly toolchain; expected to be available from rustup.
- `cargo make proto` — regenerates protobuf-derived Rust from `.proto` files via the `tool_proto_build` crate. Run after editing any `.proto`.
- Run a single test with `cargo test -p <crate> <test_name>`, for example `cargo test -p kitsune2_gossip gossip_round`.

The protobuf wire-protocol code in `crates/api` is generated; do not hand-edit generated files. Edit the `.proto` sources and run `cargo make proto`, or run `cargo make fix` which includes it.

### Pre-commit / pre-push checks

Before committing and pushing, run in order:

1. `cargo fmt`.
2. `cargo make static` — must pass before going further.
3. Tests for the changed crate(s) **and** any downstream crates that depend on them. Scope this by what was touched, so for example:
   - **API change implemented in `core` and/or a transport**: just run `cargo make test`, since the blast radius is wide enough that running the full suite is simpler than reasoning about it.
   - **`transport_iroh` only**: run tests for `kitsune2_transport_iroh` and for `kitsune2`, which is the top-level integration crate that consumes the transport.
   - **Other localised changes**: test the changed crate plus its direct downstream consumers, following the dependency direction described in *Workspace architecture* below.

   Run a single crate's tests with `cargo test -p <crate>`.

## Workspace architecture

The workspace is intentionally split so that the API surface, the production implementations, and the integrations are independently versioned and replaceable. Most crates depend on `kitsune2_api`.

- **`crates/api`**, published as `kitsune2_api` — the trait and type surface that defines a Kitsune2 node. It defines traits such as `Kitsune`, `Space`, `Transport`, `PeerStore`, `OpStore`, `Bootstrap`, `Gossip`, `Fetch`, `Publish`, and `LocalAgentStore`, along with the prost-generated wire `protocol`, config plumbing, and the `Builder` pattern used to assemble a node from pluggable modules. Anything reusable across implementations lives here.
- **`crates/core`**, published as `kitsune2_core` — production-ready and test default implementations of the API traits, such as in-memory stores. Consumers typically reuse some of these and override others; in particular, `OpStore` usually needs persistence.
- **`crates/kitsune2`** — the top-level integration crate exposing a default `Builder` wired together from `core`, `dht`, `gossip`, and a transport. This is the entry point for applications embedding Kitsune2.
- **`crates/dht`**, published as `kitsune2_dht` — the DHT data model, organised as sector, ring, and disc structures. It is used by the gossip crate to drive diff exchange and has no networking of its own.
- **`crates/gossip`**, published as `kitsune2_gossip` — gossip protocol implementation that uses the `dht` model to compare state between peers and exchange missing ops and agents. The state machine is documented as a mermaid diagram in `crates/gossip/README.md`, flowing from init → accept → diff exchange → hashes/agents → terminate. Read it before changing protocol flow.
- **`crates/transport_tx5`** — `Transport` implementation built on the `tx5` WebRTC stack, which is the historical default.
- **`crates/transport_iroh`** — alternative `Transport` implementation built on `iroh`. The two transports are interchangeable behind the API trait.
- **`crates/bootstrap_srv`** — standalone HTTP server, built on axum, that helps nodes discover each other on a WAN. It uses tempfiles instead of RAM for storage and embeds either an SBD signal server for the tx5 transport or a relay service for the Iroh transport. It ships with its own CLI.
- **`crates/bootstrap_client`** — client used by nodes to talk to a `bootstrap_srv`.
- **`crates/test_utils`** — shared test fixtures and helpers, used only as a dev-dependency.
- **`crates/tool_proto_build`** — build tool that runs `prost-build` over the `.proto` files; invoked via `cargo make proto`.
- **`crates/kitsune2_showcase`** — interactive CLI demo app showing how the pieces fit together.

### Key architectural points


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [holochain/kitsune2](https://github.com/holochain/kitsune2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
