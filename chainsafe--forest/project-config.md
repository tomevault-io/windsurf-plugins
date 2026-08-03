---
trigger: always_on
description: This file provides guidance to AI coding assistants (such as Claude Code, Cursor, Copilot, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants (such as Claude Code, Cursor, Copilot, etc.) when working with code in this repository.

## Project Overview

Forest is a Rust implementation of a Filecoin node that can transfer FIL, host a high-performance RPC API, validate the Filecoin blockchain, and generate blockchain snapshots. It aims to be faster and easier-to-use than the canonical Filecoin node (Lotus).

## Development Commands

### Building and Installing

```bash
# Install Forest binaries with release profile (recommended)
mise run install

# Install with different profiles
mise run install quick          # Faster build, less optimization
mise run install release-lto-fat # Maximum optimization (slower build)
mise run install dev            # Debug build

# Install slim version (minimal features)
mise run install --slim

# Build without installing
cargo build --release
cargo build --profile quick  # Faster compile time

# Run binaries directly (for development)
cargo daemon --chain calibnet  # Alias for: cargo run --bin forest --
cargo cli info show            # Alias for: cargo run --bin forest-cli --
cargo forest-tool --help       # Alias for: cargo run --bin forest-tool --release --
```

### Testing

```bash
# Run all tests (requires cargo-nextest: cargo install cargo-nextest --locked)
mise test           # Uses 'quick' profile by default
mise test release   # Run with release profile
mise test dev       # Run with dev profile

# Run only Rust tests (no doctests)
mise test:rust
mise test:rust release

# Run only doctests
mise test:docs
mise test:docs release

# Run specific test
cargo nextest run --cargo-profile quick <test_name>

# Run tests in a specific file/module
cargo nextest run --cargo-profile quick state_manager::

# Run single test with full output
cargo nextest run --cargo-profile quick --no-capture <test_name>

# Run doctests for private items
cargo test --doc --profile quick --features doctest-private
```

### Linting and Formatting

```bash
# Install all linting tools
mise install-lint-tools

# Run all linters
mise lint

# Run specific linters
mise lint:rust-fmt    # Rust formatting check
mise lint:clippy      # Rust linter
mise lint:toml        # TOML formatting/linting
mise lint:spellcheck  # Spell checking
mise lint:deny        # Check licenses and security
mise lint:unused-deps # Check for unused dependencies
mise lint:dockerfile  # Lint Dockerfiles
mise lint:shellcheck  # Lint shell scripts
mise lint:golang      # Lint Go code (F3 sidecar)

# Format code
mise fmt              # Format Rust, TOML, markdown, YAML

# Check specific issues
cargo fmt --all -- --check
cargo clippy --all-targets --no-deps -- --deny=warnings
taplo fmt --check && taplo lint
```

### Code Coverage

```bash
# Generate coverage report (requires cargo-llvm-cov: cargo install cargo-llvm-cov)
mise codecov
```

### Cleaning

```bash
# Clean all build artifacts and dependencies
mise clean
```

## High-Level Architecture

### Core Modules

- **`daemon/`** - Node startup, initialization, and service orchestration
- **`chain/`** - Blockchain storage (`ChainStore`) and index management
- **`chain_sync/`** - Chain synchronization, consensus (`ChainFollower`, `ChainMuxer`)
- **`state_manager/`** - State tree management and FVM execution coordinator
- **`rpc/`** - JSON-RPC API server with authentication and filtering middleware
- **`libp2p/`** - P2P networking (peer discovery, chain exchange, `gossipsub`)
- **`message_pool/`** - Transaction pool for pending messages
- **`db/`** - Database abstraction layer (ParityDb, `MemoryDB`, CAR format)
- **`interpreter/`** - Filecoin Virtual Machine (FVM) integration (multi-version)
- **`blocks/`** - Block and tipset structures
- **`shim/`** - Filecoin protocol primitives (actors, crypto, addresses, state tree)
- **`eth/`** - Ethereum compatibility layer (EVM transactions, address mapping)
- **`wallet/`** - Key management and transaction signing
- **`networks/`** - Network configurations (mainnet, calibnet, devnet)

### Key Architectural Patterns

**Blockstore Pattern**: Generic database trait (`fvm_ipld_blockstore::Blockstore`) allows swapping storage implementations. Most core structures are generic over `DB: Blockstore`:

```rust
pub struct StateManager<DB> where DB: Blockstore { ... }
pub struct ChainStore<DB> where DB: Blockstore { ... }
```

**Publisher/Subscriber**: `HeadChange` events (new tipsets, reorg reverts) are broadcast via `Publisher` to multiple subscribers (RPC, message pool, chain indexer).

**State Management**: `StateManager` is the central coordinator for state transitions, actor queries, and FVM execution. It caches state roots and receipts in `TipsetStateCache`.

**Multi-version FVM**: Supports `FVM2`, `FVM3`, and `FVM4` for different network versions. Stack size management with `stacker::grow()` is required for WASM execution.

### Daemon Startup Flow

1. `startup_init()` - Increase file descriptor limits, initialize proof cache
2. `AppContext::init()` - Initialize database, state manager, keystore, JWT
3. `create_p2p_service()` - Start libp2p networking stack
4. `create_mpool()` - Initialize message pool

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChainSafe/forest](https://github.com/ChainSafe/forest) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
