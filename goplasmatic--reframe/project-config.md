---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Reframe** is an enterprise-grade, open-source bidirectional SWIFT MT ↔ ISO 20022 transformation service built in Rust. It provides REST API endpoints for converting between legacy SWIFT MT messages and modern ISO 20022 XML format in both directions.

## Development Commands

### Building and Running

```bash
# Build in debug mode (faster compilation, with debug symbols)
cargo build

# Build in release mode (optimized for performance)
cargo build --release

# Run the application
cargo run

# Run with debug logging to see detailed transformation steps
RUST_LOG=debug cargo run

# Run with info logging (recommended for development)
RUST_LOG=info cargo run

# Kill existing process on port 3000 and restart
lsof -i :3000 | grep LISTEN | awk '{print $2}' | xargs kill -9 2>/dev/null; RUST_LOG=info cargo run

# Run with custom Tokio thread count
TOKIO_WORKER_THREADS=8 cargo run --release

# Run benchmark to find optimal configuration
python3 test/simple_benchmark.py
```

### Performance Configuration

Reframe uses an async Engine with Tokio runtime for high-performance message processing. Performance settings can be configured via `reframe.config.json` or environment variables.

#### Runtime Architecture

- **Async I/O**: Non-blocking operations for network and file handling
- **CPU optimization**: Tokio worker threads configurable (defaults to all CPU cores)
- **Efficient concurrency**: Handles thousands of concurrent requests efficiently
- **Memory efficiency**: Async operations reduce memory overhead

#### Configuration Methods

Performance can be configured in three ways (priority order):

1. **Environment Variables** (highest priority)
   ```bash
   TOKIO_WORKER_THREADS=8 cargo run
   ```

2. **Configuration File** (`reframe.config.json`)
   ```json
   {
     "server": {
       "runtime": {
         "tokio_worker_threads": "8"
       }
     }
   }
   ```

3. **Auto-detection** (fallback)
   - Set `tokio_worker_threads: "auto"` in config
   - Automatically uses all available CPU cores

#### Performance Examples

```bash
# Use configuration from reframe.config.json
cargo run --release

# Override with environment variable
TOKIO_WORKER_THREADS=4 cargo run --release

# Conservative (low resources)
TOKIO_WORKER_THREADS=1 cargo run

# High performance (specific core count)
TOKIO_WORKER_THREADS=16 cargo run --release
```

#### Runtime Characteristics

- **Automatic scaling**: Configurable to match available resources
- **Efficient concurrency**: Handles multiple concurrent transformations
- **Resource awareness**: Can be tuned for resource-constrained environments
- **Hot-reload**: Workflow changes reload without restart

### Testing

```bash
# Run all tests
cargo test

# Run tests with output visible
cargo test -- --nocapture

# Run tests with debug logging
RUST_LOG=debug cargo test -- --nocapture

# Run a specific test
cargo test test_name -- --nocapture

# Test scenario generation for specific message types
python3 test/test_scenarios.py -m MT103 -d
python3 test/test_scenarios.py -m pacs.008 -d
python3 test/test_scenarios.py -m camt.052 -d

# Test all scenarios with verbose output
python3 test/test_scenarios.py --all -v

# Generate sample messages
python3 test/generate_sample.py MT103 -s standard
python3 test/generate_sample.py pacs.008 -s cbpr_standard -p

# Validate sample messages
python3 test/validate_sample.py MT103 -s standard -d
```

### Code Quality

```bash
# Format code
cargo fmt

# Check formatting without changes
cargo fmt -- --check

# Run clippy linter
cargo clippy

# Run clippy with all warnings as errors
cargo clippy -- -D warnings
```

### Docker Operations

```bash
# Build container (downloads package automatically)
docker build -t reframe .

# Build with specific package URL
docker build --build-arg PACKAGE_URL=https://github.com/GoPlasmatic/reframe-package-swift-cbpr/releases/download/v2.1.2/reframe-swift-cbpr-v2.1.2.zip -t reframe .

# Run container (package is baked into image)
docker run -p 3000:3000 reframe

# Build and run with docker-compose (recommended)
docker-compose up -d
```

## Architecture and Code Structure

### Core Components

The application follows a unified package-based architecture with clear separation of concerns:

1. **Main Server** (`src/main.rs`):
   - Axum HTTP server setup
   - Route configuration with unified API endpoints
   - Engine initialization from external packages
   - Package-based workflow loading

2. **Transformation Engines** (`src/engine.rs`):
   - **Transform Engine**: Unified bidirectional MT ↔ ISO 20022 transformations
   - **Generation Engine**: Sample message generation for both MT and MX
   - **Validation Engine**: Message validation for both MT and MX
   - All engines use dataflow-rs for workflow orchestration
   - Engines are initialized once from package configuration and reused across requests
   - Package loading system supports `REFRAME_PACKAGE_PATH` environment variable

3. **Message Parsing**:
   - `src/parse_mt.rs`: SWIFT MT parsing with custom parser
   - `src/parse_mx.rs`: ISO 20022 XML parsing and validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GoPlasmatic/Reframe](https://github.com/GoPlasmatic/Reframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
