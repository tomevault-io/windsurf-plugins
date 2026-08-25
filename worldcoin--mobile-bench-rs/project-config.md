---
trigger: always_on
description: This file provides guidance to Codex and other coding agents when working in
---

# AGENTS.md

This file provides guidance to Codex and other coding agents when working in
this repository.

## Project Overview

mobile-bench-rs, now published as **mobench**, is a Rust mobile benchmarking
toolkit. It lets benchmark authors define Rust benchmarks once, build generated
Android/iOS runners, run host-only/local/BrowserStack benchmark jobs, produce
stable CI artifacts, and run local native profiling sessions.

Current workspace and crates.io release line: **v0.2.0**.

Published crates:

- [`mobench`](https://crates.io/crates/mobench): CLI and programmatic
  orchestration API.
- [`mobench-sdk`](https://crates.io/crates/mobench-sdk): timing harness,
  benchmark registry, generated runner support, Android/iOS builders, profiling
  helpers, UniFFI compatibility, and native C ABI support.
- [`mobench-macros`](https://crates.io/crates/mobench-macros): `#[benchmark]`
  proc macro.
- [`mobench-runtime`](https://crates.io/crates/mobench-runtime): bounded counts,
  distributions, and resource aggregation.
- [`mobench-domain`](https://crates.io/crates/mobench-domain): strict report
  envelopes and mobile framing.
- [`mobench-process`](https://crates.io/crates/mobench-process): subprocess
  supervision and provenance.
- [`mobench-artifacts`](https://crates.io/crates/mobench-artifacts): isolated,
  immutable artifact publication.
- [`mobench-provider`](https://crates.io/crates/mobench-provider): provider
  lifecycle and execution state.
- [`mobench-report`](https://crates.io/crates/mobench-report): context-safe
  Markdown, CSV, and GitHub report rendering.

All packages are MIT licensed by World Foundation, 2026.

## Commit Guidelines

Do not add `Co-Authored-By` lines to commit messages.

## Workspace Structure

```text
crates/mobench/          CLI, BrowserStack, reports, profiling
crates/mobench-sdk/      timing, registry, builders, codegen, templates
crates/mobench-macros/   #[benchmark] proc macro
crates/sample-fns/       repository demo benchmark crate
examples/basic-benchmark minimal SDK example
examples/ffi-benchmark   full generated FFI example
android/                 checked-in Android runner/demo app
ios/                     checked-in iOS runner/demo app
templates/               editable template sources
docs/                    guides, specs, schemas, diagrams, codebase reference
```

The workspace uses Rust 2024 with MSRV Rust 1.85.

## Main Product Surfaces

### Benchmark Execution

- Build Android/iOS artifacts.
- Run benchmarks host-only, locally, or on BrowserStack.
- Write result JSON, `summary.json`, `summary.md`, `results.csv`, optional
  `plots/*.svg`, PR comments, and check-run summaries.

### Local Native Profiling

- `android-native`: local Android `simpleperf` capture and symbolization.
- `ios-instruments`: local simulator-host `sample` capture.
- `rust-tracing`: planned manifest/trace contract.
- BrowserStack native stack/flamegraph profiling is explicitly unsupported in
  this release.

## Quick Start

```bash
cargo install mobench
cargo add mobench-sdk inventory
```

```rust
use mobench_sdk::benchmark;

#[benchmark]
pub fn my_benchmark() {
    let result = expensive_operation();
    std::hint::black_box(result);
}
```

Run a host-only smoke benchmark:

```bash
cargo mobench run \
  --target android \
  --function my_crate::my_benchmark \
  --local-only \
  --iterations 20 \
  --warmup 5 \
  --output target/mobench/results.json
```

## Common Commands

```bash
# Rust tests
cargo test --workspace

# CLI help and version
cargo run -q -p mobench --bin mobench -- --version
cargo run -q -p mobench --bin mobench -- --help

# Prerequisite checks
cargo mobench check --target android
cargo mobench check --target ios
cargo mobench doctor --target both --browserstack false

# Build mobile artifacts
cargo mobench build --target android --progress
cargo mobench build --target ios --progress
cargo mobench build --target both --progress

# List and verify benchmarks
cargo mobench list --crate-path examples/basic-benchmark
cargo mobench verify --target android --check-artifacts

# BrowserStack device resolution
cargo mobench devices --platform android
cargo mobench devices resolve \
  --platform android \
  --profile default \
  --device-matrix device-matrix.yaml

# CI contract output
cargo mobench ci run \
  --target android \
  --function sample_fns::fibonacci \
  --local-only \
  --plots auto \
  --output-dir target/mobench/ci

# Fetch BrowserStack artifacts
cargo mobench fetch \
  --target android \
  --build-id <browserstack-build-id> \
  --output-dir target/browserstack \
  --wait

# Local native profiling
cargo mobench profile run \
  --target android \
  --provider local \
  --backend android-native \
  --function sample_fns::fibonacci
```

## Benchmark Authoring

Simple benchmarks take no parameters and return `()`:

```rust
use mobench_sdk::benchmark;

#[benchmark]
pub fn checksum_bench() {
    let data = [1u8; 1024];
    let sum: u64 = data.iter().map(|b| *b as u64).sum();
    std::hint::black_box(sum);
}
```

Setup runs outside measured iterations:

```rust
fn create_input() -> Vec<u8> {
    vec![42; 1024 * 1024]
}

#[benchmark(setup = create_input)]
pub fn checksum(input: &Vec<u8>) {
    let sum: u64 = input.iter().map(|b| *b as u64).sum();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [worldcoin/mobile-bench-rs](https://github.com/worldcoin/mobile-bench-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
