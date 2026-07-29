---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Mandatory Pre-Completion Checks

**Before declaring any code change complete, you MUST run the following and resolve ALL errors and warnings:**

```bash
# Format all code (fix in place)
cargo fmt --all

# Run clippy with strict settings — ALL warnings must be resolved
cargo clippy --all -- -D clippy::all -D warnings
```

Do NOT skip these steps. Do NOT declare a task done if either command produces errors or warnings. Fix any issues and re-run until clean.

## Overview

Ractor is a pure-Rust actor framework inspired by Erlang's `gen_server`, providing lightweight actors with single-threaded message processing, supervision trees, and optional distributed clustering capabilities.

## Workspace Structure

This is a Cargo workspace with multiple crates:
- **ractor**: Core actor framework library
- **ractor_cluster**: Distributed cluster support (experimental)
- **ractor_cluster_derive**: Procedural macros for cluster message serialization
- **ractor_cluster_integration_tests**: Integration tests for clustering
- **ractor_example_entry_proc**: Example entry point procedures
- **ractor_playground**: Playground/experimentation crate
- **xtask**: Build/development automation tasks

## Build and Test Commands

### Running Tests

```bash
# Run all tests
cargo test

# Run tests for a specific package
cargo test --package ractor
cargo test --package ractor_cluster

# Run tests with specific feature flags
cargo test --package ractor --features cluster
cargo test --package ractor --features async-std,message_span_propogation --no-default-features
cargo test --package ractor --features monitors
cargo test --package ractor --features blanket_serde

# Run a specific test
cargo test --package ractor --test <test_name>
cargo test --package ractor <test_name>
```

### CI Feature Flag Matrix

To reproduce a CI failure locally, find the matching row and run `cargo test --package <package> <flags>`:

| CI Job Name | Package | Flags |
|---|---|---|
| Run the default tests | ractor | *(none)* |
| Test ractor with async-trait | ractor | `-F async-trait` |
| Test ractor without span propogation | ractor | `--no-default-features -F tokio_runtime` |
| Test ractor with the `cluster` feature | ractor | `-F cluster` |
| Test ractor with the `blanket_serde` feature | ractor | `-F blanket_serde` |
| Test ractor with async-std runtime | ractor | `--no-default-features -F async-std,message_span_propogation` |
| Test ractor with async-std runtime but no span propagation | ractor | `--no-default-features -F async-std` |
| Test ractor with async-std runtime and async-trait | ractor | `--no-default-features -F async-std,async-trait` |
| Test ractor_cluster with native async traits | ractor_cluster | *(none)* |
| Test ractor_cluster with async_trait | ractor_cluster | `-F async-trait` |
| Test ractor with the monitor API | ractor | `-F monitors` |
| Test ractor with output-port-v2 feature | ractor | `-F output-port-v2` |
| Test ractor without output-port-v2 feature (explicit) | ractor | `--no-default-features -F tokio_runtime,message_span_propogation` |

CI also runs: `clippy`, `rustfmt --check`, `cargo doc --lib -r`, and `cargo bench --no-run`.

### Running Benchmarks

```bash
# Run benchmarks
cargo bench -p ractor

# Compile benchmarks without running
cargo bench --no-run
```

### Linting and Formatting

```bash
# Run clippy
cargo clippy --all -- -D clippy::all -D warnings

# Run rustfmt
cargo fmt --all

# Check formatting without modifying
cargo fmt --all -- --check
```

### Documentation

```bash
# Build documentation
cargo doc --lib

# Build documentation in release mode
cargo doc --lib -r
```

## Architecture

### Core Actor Model

Ractor implements the actor model with these key components:

1. **Actor**: The trait defining actor behavior (`Actor` trait for `Send` actors, `ThreadLocalActor` for `!Send` actors)
   - `pre_start`: Initialize actor state (fallible, errors returned to spawner)
   - `post_start`: Post-initialization hook
   - `handle`: Process incoming messages
   - `handle_supervisor_evt`: Handle supervision events from children
   - `post_stop`: Cleanup on shutdown

2. **ActorCell**: Reference-counted primitive representing an actor's communication channels
   - Contains message ports (signal, stop, supervision, message)
   - Manages actor lifecycle status (Unstarted, Starting, Running, Upgrading, Draining, Stopping, Stopped)

3. **ActorRef<TMessage>**: Strongly-typed wrapper over ActorCell for type-safe communication
   - Primary interface for sending messages to actors
   - Derefs to ActorCell for access to lower-level operations

### Message Types

Without `cluster` feature: blanket `impl<T: Any + Send + Sized + 'static> Message for T` — any `Send + 'static` type works as a message with zero boilerplate.

With `cluster` feature: the blanket impl is replaced by one requiring `BytesConvertable`. For enum message types, use derive macros:
- `#[derive(RactorMessage)]` — local-only actors (marks as non-serializable)
- `#[derive(RactorClusterMessage)]` — remote-capable actors (generates serialization). Mark RPC variants with `#[rpc]`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [slawlor/ractor](https://github.com/slawlor/ractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
