---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

### Building
```bash
cargo build --release          # Build optimized release binary
cargo build                    # Build debug binary
```

The release profile in `Cargo.toml` is optimized for performance:
- `codegen-units = 1`: Better optimization at cost of slower builds
- `lto = true`: Link-time optimization across crates
- `opt-level = 3`: Maximum optimization level

### Testing
```bash
cargo test                     # Run all tests
cargo test --verbose           # Run tests with verbose output
cargo test --package <name>    # Run tests for a specific package
```

### Running
```bash
./target/release/arkflow --config config.yaml          # Run with config
./target/release/arkflow --config config.yaml --validate  # Validate config only
```

The binary supports configuration validation via the `--validate` flag.

### CI Requirements
The CI pipeline requires protobuf compiler:
```bash
sudo apt-get install protobuf-compiler  # Linux
export PROTOC=$(which protoc)
```

## Project Architecture

ArkFlow is a high-performance Rust stream processing engine built on Tokio with a plugin-based architecture.

### Workspace Dependency Management

The project uses Cargo workspace with centralized dependency management in the root `Cargo.toml`. All workspace members share versions through `[workspace.package]` and dependencies through `[workspace.dependencies]`. When adding dependencies, add them to the workspace section and reference with `workspace = true` in crate `Cargo.toml` files.

**Important**: The project requires Rust 1.88 or later (specified in `rust-version`).

### Workspace Structure

This is a Cargo workspace with three crates:

- **`arkflow-core`** (`crates/arkflow-core/`) - Core engine abstractions and interfaces
  - `Engine`: Main orchestrator managing streams and health checks
  - `Stream`: Complete data processing unit (input → pipeline → output)
  - `Pipeline`: Ordered collection of processors
  - `MessageBatch`: Columnar data using Apache Arrow `RecordBatch`
  - Abstract traits for `Input`, `Output`, `Processor`, `Buffer`, `Codec`

- **`arkflow-plugin`** (`crates/arkflow-plugin/`) - Extensible plugin implementations
  - Input plugins: Generate, File, HTTP, Kafka, Memory, Modbus, MQTT, Multiple Inputs, NATS, Pulsar, Redis, SQL, WebSocket
  - Output plugins: Drop, HTTP, InfluxDB, Kafka, MQTT, NATS, Pulsar, Redis, SQL, Stdout
  - Processor plugins: Batch, JSON, Protobuf, Python UDF, SQL, VRL
  - Buffer plugins: Memory, Session Window, Sliding Window, Tumbling Window, Join
  - Codec plugins: JSON, Protobuf

- **`arkflow`** (`crates/arkflow/`) - Main binary executable

### Key Architectural Patterns

#### Plugin Registration System
Uses `lazy_static` with `RwLock<HashMap>` for dynamic component registration. Each plugin implements a builder trait and registers itself via `register_*_builder()` functions. All plugins are initialized through `*_init()` functions (e.g., `input::init()`, `processor::init()`).

When adding a new plugin:
1. Implement the appropriate builder trait (`InputBuilder`, `ProcessorBuilder`, etc.)
2. Create an `init()` function that calls `register_*_builder()`
3. Call the plugin's `init()` from the module's `init()` function

#### Stream Processing Flow
Each `Stream` runs concurrently with:
- **Input worker**: Reads data from source
- **Processor workers**: Multiple threads (configurable via `thread_num`) process batches
- **Output worker**: Writes to sink with ordered delivery using sequence numbers
- **Buffer layer**: Handles backpressure (threshold: 1024 messages in channel)

Data flow: `Input → Buffer → [Processor1 → Processor2 → ...] → Output`
Errors are routed to `error_output` if configured.

**Backpressure Mechanism**: When the channel between input and processor contains 1024+ messages, the input worker blocks until space is available, preventing memory overflow from fast inputs/slow processors.

#### Data Model
Uses Apache Arrow's `RecordBatch` for efficient columnar storage. The `MessageBatch` wrapper includes:
- `record_batch`: Arrow RecordBatch
- `input_name`: Optional source identifier

Configuration is YAML-driven and supports dynamic component loading.

#### Metadata System
Inputs can attach metadata to messages using standardized columns (prefixed with `__meta_`):
- `__meta_source`: Source identifier
- `__meta_partition`: Partition number (for partitioned sources like Kafka)
- `__meta_offset`: Offset/position within partition
- `__meta_key`: Message key
- `__meta_timestamp`: Message timestamp from source
- `__meta_ingest_time`: When the message was ingested
- `__meta_ext`: Extended metadata as MapArray for flexible key-value pairs

These metadata columns are accessible in SQL queries within processors.

#### Actor-like Concurrency
Each stream is an independent concurrent task using:
- `Tokio` async runtime with multi-threaded executor
- `CancellationToken` for graceful shutdown coordination
- `flume` channels for message passing between stages
- `TaskTracker` for managing concurrent tasks

#### Ordered Output Delivery
The output worker ensures ordered delivery using:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [arkflow-rs/arkflow](https://github.com/arkflow-rs/arkflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
