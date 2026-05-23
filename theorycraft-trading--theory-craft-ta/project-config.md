---
trigger: always_on
description: Provides an interactive UI for configuring and executing
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TheoryCraftTA is an Elixir library that extends TheoryCraft with technical analysis indicators and tools. It is designed to work within the TheoryCraft streaming pipeline architecture, providing processors for calculating technical indicators from market data.

This project is in early development stages. It depends on the parent `theory_craft` project located at `../theorycraft`.

## Common Commands

### Testing
```bash
# Run all tests
mix test

# Run a specific test file
mix test test/theory_craft_ta_test.exs

# Run tests with specific line number
mix test test/theory_craft_ta_test.exs:5
```

### Development
```bash
# Get dependencies
mix deps.get

# Compile the project
mix compile

# Format code
mix format

# Run in interactive shell
iex -S mix

# Start Tidewave MCP server (for Claude Code integration)
mix tidewave
```

### Code Quality
```bash
# Format all Elixir files
mix format

# Format specific file
mix format lib/theory_craft_ta.ex

# Run Credo code analysis
mix credo

# Get detailed explanation for a specific Credo issue
mix credo explain lib/theory_craft_ta/overlap/wma.ex:34:24

# Run full CI check (includes tests, compilation, and Credo)
mix ci

# Run CI check only (no tests, just compilation and Credo)
mix ci.check
```

**Note**: When you complete a feature and run `mix ci`, Credo checks will be performed automatically as part of the CI pipeline.

### Benchmarking
```bash
# Run SMA benchmark (compares Native vs Elixir backends)
mix run benchmarks/sma_benchmark.exs

# Create a benchmark file (example structure)
# benchmarks/sma_benchmark.exs
```

### Rust/Native Development
```bash
# Build Rust NIF
mix rust.build

# Clean Rust build artifacts
mix rust.clean

# Run Rust tests
mix rust.test

# Format Rust code
mix rust.fmt

# Run Rust linter
mix rust.clippy
```

**Note**: ta-lib is built automatically by the Rust build script (`build.rs`) when compiling the NIF. The build scripts are located in `native/theory_craft_ta/tools/` and are invoked automatically during compilation.

**Windows Disk Space Issue**: If you encounter "Espace insuffisant sur le disque" errors during Rust compilation, set `TMPDIR` to a drive with more space:
```bash
set TMPDIR=D:\temp
```

**CRITICAL - Windows Command Execution**:
- **NEVER** use `cmd /c` when running commands on Windows
- **ALWAYS** use `.tools/run_ci.cmd &&` prefix for mix commands that need environment setup
- **ALWAYS** use forward slashes `/` in paths
- Examples:
  - ❌ BAD: `cmd /c ".tools\run_ci.cmd" && mix test`
  - ✅ GOOD: `.tools/run_ci.cmd && mix test`
  - ❌ BAD: `cmd /c .tools\run_ci.cmd && mix compile`
  - ✅ GOOD: `.tools/run_ci.cmd && mix compile --force`

**CRITICAL - Cargo Clean Policy**:
- **NEVER** run `cargo clean` during normal development
- `cargo clean` should ONLY be used when explicitly troubleshooting compilation issues
- Running cargo clean unnecessarily slows down development by forcing full recompilation (4+ minutes)
- Cargo's incremental compilation is very reliable - trust it
- Examples:
  - ❌ BAD: `cargo clean --manifest-path=native/theory_craft_ta/Cargo.toml && mix compile`
  - ✅ GOOD: `mix compile` (cargo will incrementally rebuild only what changed)
  - ❌ BAD: `cargo clean && .tools/run_ci.cmd`
  - ✅ GOOD: `.tools/run_ci.cmd` (let cargo handle incremental builds)
- Only use cargo clean when:
  - User explicitly requests it
  - Encountering unexplainable Rust compilation errors
  - Switching between major Rust/dependency versions

## Architecture

### Relationship to TheoryCraft

TheoryCraftTA extends TheoryCraft's streaming pipeline architecture by providing:
- **Technical Analysis Processors**: Implementations of the `TheoryCraft.Processor` behaviour for TA indicators
- **Native Performance**: Uses Rustler for performance-critical indicator calculations (planned)

### Integration with TheoryCraft Pipeline

Technical analysis indicators fit into TheoryCraft's data flow:

**Data Source** → **DataFeed** → **MarketEvent Stream** → **TA Processors** → **Strategy/Output**

Each TA indicator is implemented as a `Processor` that:
1. Receives `MarketEvent` structs containing `Tick` or `Bar` data
2. Calculates indicator values (e.g., SMA, RSI, MACD)
3. Enriches the `MarketEvent` with calculated indicator values
4. Passes the enriched event downstream

### Native Components (Rustler)

TheoryCraftTA uses Rustler NIFs for performance-critical calculations with ta-lib:

**Architecture**:
- **Dual Backend System**: Both Native (Rust NIF) and Pure Elixir implementations
- **Backend Selection**: Configured at compile-time via `config/*.exs`
- **Rust NIF**: Wraps ta-lib C library for high performance
- **Pure Elixir**: Fallback implementation, useful for testing and development

**TA-Lib Integration**:
- ta-lib is built from source (not a system dependency)
- Build script downloads ta-lib 0.6.4 from GitHub
- Uses CMake to configure and build static library
- Conditional compilation: NIF compiles with or without ta-lib
- If ta-lib is missing, Native backend returns error suggesting Elixir backend

**Key Files**:
- `native/theory_craft_ta/src/overlap.rs` - Rust NIF implementations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theorycraft-trading/theory_craft_ta](https://github.com/theorycraft-trading/theory_craft_ta) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-23 -->
