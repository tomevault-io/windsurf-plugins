---
trigger: always_on
description: This file provides guidance for Claude Code when working with this codebase.
---

# CLAUDE.md - Artemis Solana

This file provides guidance for Claude Code when working with this codebase.

## Project Overview

Artemis Solana is a Rust-based blockchain data collection and execution engine, forked from [Paradigm's Artemis](https://github.com/paradigmxyz/artemis) and adapted for Solana. It provides a modular architecture for:

- Real-time data collection from Solana blockchain
- Event processing through customizable strategies
- Transaction execution through pluggable executors

The project is designed for MEV/arbitrage bots and applications requiring blockchain event monitoring and transaction execution.

## Architecture

The engine follows a three-stage pipeline pattern:

```
Collectors → Strategies → Executors
  (Events)    (Logic)     (Actions)
```

1. **Collectors**: Subscribe to blockchain events and emit event streams
2. **Strategies**: Process events, maintain state, decide on actions
3. **Executors**: Execute actions (send transactions, simulate, etc.)

Components communicate via Tokio broadcast channels, orchestrated by the `Engine` struct in `artemis-core`.

## Project Structure

```
artemis-sol/
├── crates/
│   ├── artemis-core/          # Core traits and engine orchestration
│   │   └── src/
│   │       ├── engine.rs      # Main Engine<E, A> orchestrator
│   │       ├── types.rs       # Collector, Strategy, Executor traits
│   │       └── lib.rs
│   ├── jito-geyser-client/    # High-level Jito Geyser gRPC client
│   │   └── src/
│   │       ├── geyser_consumer.rs  # GeyserConsumer with sequence tracking
│   │       ├── interceptor.rs      # gRPC auth interceptor
│   │       ├── types.rs
│   │       └── lib.rs
│   └── jito-geyser-protos/    # Protobuf definitions for Jito Geyser
│       ├── proto/             # .proto files
│       ├── src/
│       │   ├── convert.rs     # Serde conversions
│       │   └── lib.rs
│       └── build.rs           # tonic-build for proto compilation
└── lib/                       # Concrete Solana implementations
    └── src/
        ├── collectors/
        │   ├── pubsub/        # Solana PubSub collectors
        │   ├── jito/          # Jito Geyser collectors
        │   ├── yellowstone/   # Yellowstone gRPC collectors
        │   └── helius/        # Helius SDK collectors
        ├── executors/
        │   ├── simulate_executor.rs
        │   └── transaction_executor.rs
        └── lib.rs
```

## Key Crates

### artemis-core
Core abstractions defining the pipeline:
- `Collector<E>` trait: Async streams of events
- `Strategy<E, A>` trait: Stateful event processors with `sync_state()` for initialization
- `Executor<A>` trait: Action executors
- `Engine<E, A>`: Orchestrates the pipeline with configurable channel capacities

### jito-geyser-client
High-performance gRPC client for Jito Geyser:
- `GeyserConsumer`: Consumes account/slot updates with monotonic ordering validation
- LRU-based sequence tracking (5000 slot capacity)
- Heartbeat monitoring and stale update detection

### lib (artemis-sol)
Concrete implementations:

**PubSub Collectors** (via `solana-client::PubsubClient`):
- AccountCollector, ProgramCollector, SlotCollector, VoteCollector
- BlockCollector, RootCollector, SignatureCollector, LogsCollector

**Jito Geyser Collectors**:
- AccountCollector, PartialAccountCollector, ProgramCollector
- BlockCollector, SlotCollector

**Yellowstone gRPC Collectors** (via `yellowstone-grpc-client`):
- AccountCollector: Monitor specific accounts or program-owned accounts
- SlotCollector: Stream slot updates with commitment filtering
- BlockCollector: Full block data with transactions/accounts/entries
- TransactionCollector: Transaction monitoring with vote/failed/account filters

**Helius Collectors**:
- TransactionCollector (enhanced transaction monitoring)

**Executors**:
- SimulateExecutor, TransactionExecutor

## Build Commands

```bash
# Build entire workspace
cargo build

# Build in release mode
cargo build --release

# Check for compilation errors
cargo check

# Run tests
cargo test

# Format code
cargo fmt

# Lint with clippy
cargo clippy
```

## Dependencies

Key dependencies (Solana 2.0 ecosystem):
- `solana-sdk`, `solana-client`, `solana-program` (2.0)
- `tokio` (1.37) with full async runtime
- `tonic` (0.10) for gRPC with TLS
- `yellowstone-grpc-client`, `yellowstone-grpc-proto` (5.1) for Yellowstone gRPC
- `helius` (dev branch from GitHub)
- `anyhow`, `thiserror` for error handling
- `tracing` for structured logging

## Code Style

Formatting rules (`rustfmt.toml`):
- Edition 2021
- Crate-level import granularity
- StdExternalCrate import grouping
- Reorder imports enabled
- Error on unformatted code

Conventions:
- Use `#[async_trait]` for async traits
- Error handling with `anyhow::Result<T>` or custom `thiserror` enums
- `Arc<>` for shared ownership, atomics for concurrent state
- Structured logging with `info!()`, `debug!()`, `error!()`

## Core Traits

When implementing new components:

```rust
// Collector: Source of events
#[async_trait]
pub trait Collector<E>: Send + Sync {
    async fn get_event_stream(&mut self) -> Result<CollectorStream<'_, E>>;
}

// Strategy: Event processor
#[async_trait]
pub trait Strategy<E, A>: Send + Sync {
    async fn sync_state(&mut self) -> Result<()>;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sarmatdev/artemis-sol](https://github.com/sarmatdev/artemis-sol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
