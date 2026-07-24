---
trigger: always_on
description: **Last Updated**: 2026-02-11 (based on v0.45.0, Rust 1.92)
---

# DiskANN Repository - Agent Onboarding Guide

**Last Updated**: 2026-02-11 (based on v0.45.0, Rust 1.92)

This guide helps coding agents understand how to work efficiently with the DiskANN repository.

---

## Table of Contents

1. [Repository Overview](#repository-overview)
2. [Repository Structure](#repository-structure)
3. [Testing](#testing)
4. [Code Quality & Linting](#code-quality--linting)

---

## Repository Overview

**DiskANN** is a Rust implementation of scalable approximate nearest neighbor (ANN) search algorithms. The project is a rewrite from C++ to Rust.

- **Language**: Rust (Edition 2021), toolchain version in [`rust-toolchain.toml`](rust-toolchain.toml)
- **License**: MIT (see [`LICENSE.txt`](LICENSE.txt))
- **Version**: See [`Cargo.toml`](Cargo.toml)
- **Architecture**: Cargo workspace with 15+ crates
- **Legacy Code**: Older C++ code is on the `cpp_main` branch (not maintained)

### Key Resources
- **Contributing**: See [`CONTRIBUTING.md`](CONTRIBUTING.md) (requires CLA)
- **Code of Conduct**: See [`CODE_OF_CONDUCT.md`](CODE_OF_CONDUCT.md)

---

## Repository Structure

The repository uses a Cargo workspace with crates organized into functional tiers. See [`Cargo.toml`](Cargo.toml) for:
- Workspace members and their dependencies
- Shared dependency versions
- Build profiles (release, test)
- Workspace-level lints

### Crate Organization

**Tier 1: Foundation**
- `diskann-wide/` - Low-level SIMD, bit manipulation, type width abstractions
- `diskann-vector/` - Vector primitives and operations

**Tier 2: Core Libraries**
- `diskann-linalg/` - Linear algebra operations
- `diskann-utils/` - Shared utilities (Reborrow, MatrixView traits)
- `diskann-quantization/` - Vector quantization (PQ, SQ)

**Tier 3: Algorithm & Storage**
- `diskann/` - Core ANN graph algorithm and in-memory indexing (CENTRAL crate)
- `diskann-providers/` - Storage abstraction layer
- `diskann-disk/` - Disk-based indexing with io_uring support
- `diskann-label-filter/` - Inverted index for filtered search

**Tier 4: Infrastructure & Tools**
- `diskann-benchmark-runner/` - Test runner infrastructure
- `diskann-benchmark-core/` - Benchmark framework
- `diskann-benchmark-simd/` - SIMD-specific benchmarks
- `diskann-benchmark/` - Benchmark definitions and runners
- `diskann-tools/` - CLI utilities (autotuner, etc.)

---

## Dependencies

### Internal

- Tier 1 and Tier 2 crates may be added as dependencies of any internal crate
- `diskann` may be added as a dependency of any equal or higher tier internal crate except those below
- Do not add Tier 3 crates as dependencies of these Tier 4 crates:
  - `diskann-benchmark-runner`
  - `diskann-benchmark-core` (`diskann` is allowed)
  - `diskann-benchmark-simd`

---

## Testing

### Test Execution

```bash
# Run all tests
cargo test

# Run tests for specific crate
cargo test -p diskann

# Run specific test
cargo test -p diskann -- --exact test_name

# Run doc tests
cargo test --doc
```

**Note**: CI uses `cargo-nextest` for running tests. See [`.cargo/nextest.toml`](.cargo/nextest.toml) for test configuration (timeouts, retries, etc.).

### Test Baseline Caching System

DiskANN uses a baseline caching system for regression detection. See [`diskann/README.md`](diskann/README.md) for a high-level overview of how the baseline system works. For implementation and API details, refer directly to:
- [`diskann/src/test/cache.rs`](diskann/src/test/cache.rs) — core baseline caching APIs
- [`diskann/src/test/cmp.rs`](diskann/src/test/cmp.rs) — `VerboseEq` and related helpers for better test error messages

### AVX-512, Aarch64, and multi-platform

When touching architecture-specific intrinsics, run cross-platform validation per `diskann-wide/README.md`:

- Testing AVX-512 code on non-AVX-512 capable x86-64 machines.
- Testing Aarch64 code on x86-64 machines.
- Testing code compiled for and running on the `x86-64` CPU (no AVX/AVX2) does not execute unsupported instructions.

---

## Code Quality & Linting

### Error Handling

There are three regimes of error handling and the strategy to use depends on the regime.

#### Low-Level

Low-level crates should use bespoke, precise, non-allocating error types. Use `thiserror` for boilerplate. Chain with `std::error::Error::source`.

`diskann::ANNError` is not a suitable low-level error type.

#### Mid-Level (diskann algorithms)

Use `diskann::ANNError` and its context machinery. This type:

- Has a small size and `Drop` implementation, so is efficient in function ABIs.
- Records stack trace of its first creation under `RUST_BACKTRACE=1`.
- Precisely records line numbers of creation.
- Has a context layering machinery to add additional information as an error is passed up the stack.

When converting to `ANNError`, use `#[track_caller]` for better source reporting.

Traits with associated error types should consider constraining with `diskann::error::ToRanked` instead of `Into<ANNError>` if non-critical errors should be supported.
`diskann::ANNError` should be used only for unrecoverable errors.

#### High Level (tooling)

At this level `anyhow::Error` is an appropriate type to use.

#### Do Not

Do not use a single crate-level error enum. Problems:

- Provides no documentation on how an individual function could fail

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/DiskANN](https://github.com/microsoft/DiskANN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
