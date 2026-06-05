---
trigger: always_on
description: > **This repo is `seismic-revm`, a fork of [REVM](https://github.com/bluealloy/revm).**
---

# CLAUDE.md

> **This repo is `seismic-revm`, a fork of [REVM](https://github.com/bluealloy/revm).**
> The first part of this file is the upstream CLAUDE.md, preserved verbatim so upstream merges apply cleanly.
> The [Seismic Fork Extensions](#seismic-fork-extensions) section below contains all Seismic-specific context and **takes precedence** where it overlaps with upstream (build commands, test commands, lint, architecture, etc.).

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

REVM is a highly efficient Rust implementation of the Ethereum Virtual Machine (EVM). It serves both as:
1. A standard EVM for executing Ethereum transactions
2. A framework for building custom EVM variants (like Optimism's op-revm)

The project is used by major Ethereum infrastructure including Reth, Foundry, Hardhat, Optimism, Scroll, and many zkVMs.

## Build and Development Commands

### Essential Commands
```bash
# Build the project
cargo build
cargo build --release

# Run all tests
cargo nexttest run --workspace

# Lint and format
cargo clippy --workspace --all-targets --all-features
cargo fmt --all

# Check no_std compatibility
cargo check --target riscv32imac-unknown-none-elf --no-default-features
cargo check --target riscv64imac-unknown-none-elf --no-default-features

# Run Ethereum state tests
cargo run -p revme statetest legacytests/Cancun/GeneralStateTests
```

### Test Scripts
```bash
# Download and run ethereum tests
./scripts/run-tests.sh

# Clean test fixtures and re-run
./scripts/run-tests.sh clean

# Run with specific profile
./scripts/run-tests.sh release
```

## Architecture

The workspace consists of these core crates:

- **revm**: Main crate that re-exports all others
- **revm-primitives**: Constants, primitive types, and core data structures
- **revm-interpreter**: EVM opcode implementations and execution engine
- **revm-context**: Execution context, environment, and journaled state
- **revm-handler**: Execution flow control and call frame management
- **revm-database**: State database traits and implementations
- **revm-precompile**: Ethereum precompiled contracts
- **revm-inspector**: Tracing and debugging framework
- **op-revm**: Example of custom EVM variant (Optimism)

### Key Design Patterns

1. **Trait-based Architecture**: Core functionality is defined through traits, allowing custom implementations
2. **Handler Pattern**: Execution flow is controlled through customizable handlers
3. **no_std Support**: All core crates support no_std environments
4. **Feature Flags**: Extensive use of feature flags for optional functionality

### Important Interfaces

1. **Database Trait** (`revm-database`): Defines how state is accessed
2. **Inspector Trait** (`revm-inspector`): Hooks for transaction tracing
3. **Handler Interface** (`revm-handler`): Customizable execution logic
4. **Context** (`revm-context`): Manages execution state and environment

## Current Development Context

When working on the `frame_stack` branch, note that significant refactoring is happening around:
- Frame and FrameData structures (moved from handler to context)
- Execution loop simplification
- Inspector trait cleanup

## Testing Strategy

1. Unit tests in each crate
2. Integration tests using Ethereum official test suite
3. Example projects demonstrating features
4. Benchmarking with CodSpeed

When adding new features:
- Ensure no_std compatibility
- Add appropriate feature flags
- Include tests for new functionality
- Update relevant examples if needed

---

# Seismic Fork Extensions

> **This is a Seismic fork of REVM.** The sections above are from upstream REVM and are preserved verbatim for merge compatibility. The sections below are specific to this Seismic fork and **take precedence** where they overlap (e.g., build commands, test commands, lint configuration).

## What This Does

Standard EVM storage is publicly readable. Seismic extends REVM with:

- **CLOAD/CSTORE opcodes** (0xB0/0xB1) — load/store to confidential storage slots. Each slot is a tuple `(value, is_private)` with strict access rules: SLOAD on a private slot halts, CSTORE on a non-zero public slot halts.
- **Privacy-preserving precompiles** — RNG (0x64), ECDH (0x65), AES-GCM encrypt/decrypt (0x66/0x67), HKDF (0x68), secp256k1 sign (0x69).
- **Flat gas costs** for confidential ops to prevent gas-based side-channel leaks.
- **Semantic test runner** in `revme` for testing Seismic Solidity (`ssolc`) compiler output against this VM.

## Build

Rust workspace using Cargo. MSRV: **1.88.0**. Output binary: `target/debug/revme` (or `target/release/revme`).

### Prerequisites (all platforms)

- Rust toolchain >= 1.88.0 (`rustup update stable`)
- C compiler (for native crypto deps: `blst`, `c-kzg`, `secp256k1-sys`, `gmp-mpfr-sys`)
- GMP library (for `rug`/`gmp-mpfr-sys` crate used by modexp precompile)
- Git (workspace has git dependencies on `seismic-enclave` and `seismic-alloy-core`)

### macOS

```bash
# Install system deps
brew install gmp

# Build (debug)
cargo build --workspace

# Build (release)
cargo build --workspace --release
```

### Linux (Ubuntu/Debian)

```bash
# Install system deps
sudo apt-get update

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SeismicSystems/seismic-revm](https://github.com/SeismicSystems/seismic-revm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
