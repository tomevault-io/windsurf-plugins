---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

`ibig` is a pure-Rust arbitrary-precision integer library (`no_std` compatible). The public types are `UBig` (unsigned) and `IBig` (signed). The library aims for good performance, so much of the code is concerned with algorithm selection by operand size and per-word-size primitives.

The repository is currently a **ground-up rewrite**. The new crates (`ibig-core`, `ibig`) are early-stage; the previous full implementation lives in `ibig-old` and is the reference for the algorithms still to be ported.

## Tools

- **Reading/editing files**: use the Read/Edit/Write tools, not shell commands (`cat`, `sed`, `head`, `echo`, etc.).

## Repository layout

A Cargo workspace (`Cargo.toml`) with two active members plus two excluded reference directories:

- **`ibig-core/`** — low-level arithmetic algorithms operating on slices of `Digit`s (addition, subtraction, multiplication, division). `Digit` is defined here (`src/lib.rs`) as `unative::UNative`, the platform's native unsigned integer. `no_std`.
- **`ibig/`** — the user-facing crate with `UBig`/`IBig` and their trait impls. Depends on `ibig-core`. `no_std`. This is the published `ibig` crate (version 0.4.0+).
- **`ibig-old/`** — the previous implementation, published as `ibig` 0.3.6. **Excluded from the workspace.** Keep it as a reference when porting algorithms; do not edit it as part of new work.
- **`dev-tools-old/`** — offline code-generation utilities for the old crate (e.g. NTT prime constants). **Excluded from the workspace.**

## Commands

```bash
cargo test --workspace --all-features            # run all tests (64-bit, native)
cargo test --workspace --all-features <name>     # run tests matching <name>

cargo fmt --all -- --check                        # formatting check (CI-enforced)
cargo clippy --workspace --all-features --tests -- -D warnings   # lint (CI-enforced; warnings are errors)
```

### Testing across word sizes

`Digit` width (16/32/64 bits) is chosen by `unative` from the target — there is no `force_bits` override in the new crates. CI therefore exercises each width with a real target (see `.github/workflows/ci.yml`):

```bash
# 64-bit: native
cargo test --workspace --all-features

# 32-bit: i686 (needs the 32-bit C runtime, e.g. `gcc-multilib`)
cargo test --workspace --all-features --target i686-unknown-linux-gnu

# 16-bit: msp430, build only — no prebuilt std, so build core/alloc from source on nightly
cargo +nightly build --workspace -Z build-std=core,alloc --target msp430-none-elf
```

Always consider whether a change behaves correctly at all three word sizes, since algorithm thresholds and NTT primes differ per size.

### MSRV and dependency versions

- MSRV for the new crates is rustc **1.95** (`rust-version` in `ibig/Cargo.toml` and `ibig-core/Cargo.toml`). CI has a job pinned to 1.95; keep the manifests and that job in sync. (`ibig-old` is separately 1.93.)
- A CI job runs `cargo +nightly update -Z direct-minimal-versions` to verify the declared lower bounds of direct dependencies actually build. If you raise a dependency requirement, make sure that minimum still compiles.

## Architecture

### Number representation (`ibig`)

- `UBig` (`src/ubig/repr.rs`) and `IBig` (`src/ibig/repr.rs`) each wrap a private `Digits` — a `SmallVec<[Digit; INLINE_DIGITS]>` (`INLINE_DIGITS = 4`, in `src/lib.rs`) holding the little-endian digits inline up to 4 digits and spilling to the heap beyond that.
- The representation is **canonical** — every value has exactly one representation, so derived `Eq`/`Hash` are correct. The buffer is never empty (zero is the single digit `[0]`); for `UBig` the most-significant digit is nonzero (except zero), and for `IBig` the digits are two's complement with a most-significant digit that is not a redundant sign-extension of the one below it (the sign lives in its top bit).
- Construct via `from_digits`, which normalizes (trims to the canonical length, keeps small values inline, shrinks heavily over-allocated heap buffers) — preserve these invariants when building `Digits` directly. `from_digit` / `const_from_digits` build small values (usable in `const`). Read the digits with `as_digits()` (`&[Digit]`) or `into_digits()` (owned `Digits`); `try_to_digit()` returns the value as a single `Digit`/`IDigit` when it fits, which is the basis of the single-digit fast path.
- The digit count is capped at `MAX_DIGITS` (chosen so the total bit length fits in `usize`); `from_digits` panics beyond it.
- `Digit` (`ibig-core`) is the architecture's native unsigned integer (`UNative`, 16/32/64 bits) and `IDigit` is its signed counterpart (`INative`); numbers are little-endian `Digit` slices.

### Core algorithms (`ibig-core`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tczajka/ibig](https://github.com/tczajka/ibig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
