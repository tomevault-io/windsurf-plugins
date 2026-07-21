---
trigger: always_on
description: This file provides guidance for AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance for AI agents when working with code in this repository.

## Overview

FutureSDR is an experimental asynchronous SDR (Software Defined Radio) runtime for heterogeneous architectures. It supports stream-based and message-based data processing through **Blocks** that combine into **Flowgraphs** executed by a **Runtime** and **Scheduler**.

FutureSDR implements a flexible pipeline computation model that forms a directed acyclic graph. Every block can have an arbitrary number of inputs and outputs. The runtime can be extended with custom buffer and scheduler implementations.

FutureSDR borrows ideas from the actor model, where a block is an actor that reads from a mpsc channel. Through the channel, the block is either notified that it should run again or gets BlockMessages that provide more metadata to the message, e.g., call a message handler with this data argument.

## Project Goals

- FutureSDR is meant to experiment with novel concepts, API stability is not a goal right now. The main question is: how could an SDR framework in Rust look like?
- FutureSDR code should be minimal, elegant, performant, and follow Rust best-practices.
- It is more important that the user facing API (instantiate blocks, create a flowgraph, run the flowgraph) is ergonomic than the developer API (implement blocks, custom schedulers, or custom buffers).
- It is important that the core runtime code (in `src/runtime`) that glues everything together is easy to comprehend and minimal.
- Specific implementations that contain complexity (e.g., a buffer implementation, a scheduler implementation) can be highly optimized and complexity is ok.

## Requirements

- Stream connections for a directed acyclic graph. Circles or self-connections are not supported.
- Message connections can have arbitrary connection topologies. Circles and self-connections are possible.

## Build & Test Commands

The root crate uses Rust 2024 edition, currently declares `rust-version = "1.89"`, and requires the nightly Rust channel. The repository includes `rust-toolchain.toml`, so cargo/rustup automatically select nightly inside the checkout. The `rust-version` value is only the minimum compiler version; it does not imply stable support.

The root workspace contains `.`, `crates/futuredsp`, `crates/macros`, and `crates/types`. `crates/prophecy`, `crates/remote`, every directory under `examples/`, and every directory under `perf/` are independent Cargo workspaces.

```sh
# Build main crate (default features)
cargo build

# Build with specific features
cargo build --features=burn,vulkan,zeromq,audio,flow_scheduler,seify_dummy,wgpu

# Run all tests (main workspace)
cargo test --all-targets --workspace --features=vulkan,zeromq,audio,flow_scheduler,seify_dummy,soapy,wgpu,zynq

# Run a single test
cargo test --test flowgraph
cargo test --test apply

# Run tests for a sub-crate
cargo test --all-targets --manifest-path=crates/futuredsp/Cargo.toml
cargo test --all-targets --all-features --manifest-path=crates/types/Cargo.toml

# Lint (matches the root check script)
cargo clippy --all-targets --workspace --features=burn,vulkan,zeromq,audio,flow_scheduler,soapy,zynq,wgpu,seify_dummy -- -D warnings

# Format (repository convention uses nightly rustfmt)
cargo fmt --all

# Format check
cargo fmt --all -- --check
```

### Examples, Perf, and independent crates

`crates/prophecy`, `crates/remote`, and each directory under `examples/` and `perf/` is an independent Cargo workspace. Build/test them with `--manifest-path`:

```sh
cargo build --manifest-path=examples/wlan/Cargo.toml
cargo test --all-targets --manifest-path=examples/wlan/Cargo.toml
cargo clippy --all-targets --manifest-path=perf/burn/Cargo.toml -- -D warnings
cargo test --all-targets --manifest-path=crates/remote/Cargo.toml
```

### WASM builds

```sh
rustup target add wasm32-unknown-unknown
cargo clippy --lib --workspace --features=burn,audio,seify_dummy,wgpu --target wasm32-unknown-unknown -- -D warnings
```

### Prophecy GUI (web frontend)

Located at `crates/prophecy/`, built with [Trunk](https://trunkrs.dev):

```sh
cd crates/prophecy
trunk build --release   # output in dist/
trunk serve             # dev server
```

Served automatically at `http://localhost:1337/` when running any FutureSDR application.

## Architecture

### Core Concepts

**Block** — the fundamental processing unit. Every block implements the `Kernel` trait (`src/runtime/kernel.rs`):
- `init()` — called once at startup
- `work()` — called repeatedly to process stream data; sets `WorkIo` flags to signal state
- `deinit()` — called once at shutdown

Blocks are created using the `#[derive(Block)]` proc macro (from `crates/macros/`) which generates the `KernelInterface` impl. The macro derives stream/message port declarations from annotated struct fields.

**Flowgraph** (`src/runtime/flowgraph.rs`) — a directed graph of blocks connected via stream ports or message ports. Built using the `connect!` macro:
```rust
connect!(fg,
    src > head > snk;               // stream connection (default "out"/"in" ports)
    src."custom out" > snk;         // named ports
    producer | consumer;            // message connection
);
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FutureSDR/FutureSDR](https://github.com/FutureSDR/FutureSDR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
