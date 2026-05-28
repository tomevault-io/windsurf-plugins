---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Rust crate (`hamming-bitwise-fast`) providing fast bitwise Hamming distance computation for byte arrays/slices. The implementation uses auto-vectorization techniques that enable SIMD optimizations (AVX-512 VPOPCNTDQ on x86, NEON on ARM) without explicit intrinsics.

## Build Commands

```sh
# Build
cargo build
cargo build --release

# Build with x86 multiversion support (runtime CPU dispatch, enabled by default)
cargo build --features multiversion_x86

# Build with native CPU optimizations
RUSTFLAGS="-C target-cpu=native" cargo build --release

# Run tests
cargo test

# Run a specific test
cargo test slice_distance_all_bits_different

# Run benchmarks (uses criterion via cargo-criterion for better output and HTML reports)
# Install: cargo install cargo-criterion
cargo criterion

# Run a specific benchmark file
cargo criterion --bench competitors

# Filter benchmarks by name pattern
cargo criterion -- single
```

## Development Guidelines

**Always verify both assembly AND benchmarks after changes.** This crate's performance comes entirely from the compiler generating optimal SIMD instructions. Seemingly minor code changes (e.g., loop structure, iterator patterns, type annotations) can cause the compiler to emit dramatically slower code.

For any performance-related change:
1. **Inspect assembly** on x86 (with AVX-512) to verify expected instructions are generated. See [Inspecting Generated Assembly](#inspecting-generated-assembly).
2. **Run benchmarks on both ARM and x86** to confirm performance hypotheses. Assembly that "looks right" can still be slow, and vice versa.
3. **Understand why** the results are what they are. If benchmarks don't match expectations, investigate until you understand the cause—don't just accept surprising results.

Development is typically done on ARM Mac, but x86 benchmarks require a remote server (the project uses Linode and fly.io).

## Architecture

### Public API

Organized into two modules (`src/array.rs` and `src/slice.rs`):

**`array` module** — fixed-size `[u8; N]` (recommended when size is known):
1. **`array::distance(&[u8; N], &[u8; N]) -> u32`** - Single comparison
2. **`array::batch(&[u8; N], &[[u8; N]], &mut [u32])`** - One-to-many (fastest for bulk)

**`slice` module** — variable-length `&[u8]`:
3. **`slice::distance(&[u8], &[u8]) -> u32`** - Single comparison
4. **`slice::batch(&[u8], &[&[u8]], &mut [u32])`** - One-to-many

### Code Structure

- **`src/lib.rs`** — Two `cfg`-gated versions of `distance_impl()`: x86 uses u64 chunks via `chunks_exact(8)`, non-x86 uses simple byte iteration. Also re-exports `array`/`slice` modules and provides a `hamming_bitwise_fast()` convenience alias.
- **`src/array.rs`** — `distance()` and `batch()` for `[u8; N]`. Contains the `opaque_ptr` asm barrier for gather avoidance (see below). Each function has a `#[cfg_attr(... multiversion::multiversion(...))]` attribute for runtime CPU dispatch on x86.
- **`src/slice.rs`** — `distance()` and `batch()` for `&[u8]`. Same multiversion attribute pattern. No gather barrier needed (pointer-indirect layout).
- **`src/tests.rs`** — Parameterized tests using `test_case` crate.

### Platform Dispatch Strategy

Each public function uses `#[cfg_attr]` to conditionally apply `#[multiversion::multiversion]`:

- **x86/x86_64 with `multiversion_x86` feature** (default): The `multiversion` attribute generates multiple copies targeting AVX-512, AVX2, and SSE4.2, with runtime CPU dispatch via CPUID.
- **x86/x86_64 without feature**: Falls through to `distance_impl` which uses u64-chunked processing with `chunks_exact(8)` for auto-vectorization.
- **ARM/other platforms**: `distance_impl` uses simple byte iteration that auto-vectorizes well with NEON.

### Core Algorithm

The x86 `distance_impl` processes bytes as u64 chunks:
```rust
a.chunks_exact(8).zip(b.chunks_exact(8))
    .map(|(a_chunk, b_chunk)| {
        let a_val = u64::from_ne_bytes(a_chunk.try_into().unwrap());
        let b_val = u64::from_ne_bytes(b_chunk.try_into().unwrap());
        (a_val ^ b_val).count_ones()
    })
    .sum()
```

This pattern enables the compiler to use VPOPCNTDQ on AVX-512 CPUs.

**LTO requirement:** Auto-vectorization depends on the compiler seeing the full loop body. Without LTO, cross-crate MIR inlining doesn't give LLVM enough visibility — it emits scalar POPCNT instead of VPOPCNTDQ. The `[profile.bench] lto = true` in Cargo.toml ensures benchmarks get full optimization. Users should enable `lto = true` in their release profile for best single-call performance.

### Gather Avoidance in Batch Functions

**PERFORMANCE INVARIANT:** `array::batch` uses an `asm!` barrier (`opaque_ptr` in `src/array.rs`) on target references to prevent LLVM from generating slow AVX-512 VPGATHERQQ gather instructions across iterations of the contiguous `&[[u8; N]]` layout.

The barrier's effect depends on LTO:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emschwartz/hamming-bitwise-fast](https://github.com/emschwartz/hamming-bitwise-fast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
