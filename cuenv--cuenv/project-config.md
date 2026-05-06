---
trigger: always_on
description: **ALWAYS follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**
---

# cuenv - CUE-Powered Environment Management & Build Toolchain

**ALWAYS follow these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

cuenv is a Rust + Go FFI application that provides type-safe environment management and task orchestration using CUE configuration language. It consists of a core CUE evaluation engine (cuengine), shared utilities (cuenv-core), and CLI interface (cuenv-cli).

## Working Effectively

### Bootstrap and Build

Run these commands in order to get a working development environment:

**CRITICAL BUILD TIMING:** Build takes 1.5-2 minutes for debug, 45+ seconds for release. Tests take 45-60 seconds. **NEVER CANCEL** these operations. Set timeouts to 120+ minutes for builds, 60+ minutes for tests.

```bash
# Build the entire workspace (NEVER CANCEL - takes 90+ seconds)
cargo build --workspace --all-features

# Build release version (NEVER CANCEL - takes 45+ seconds)
cargo build --release --workspace

# Run all tests (NEVER CANCEL - takes 45-60 seconds)
cargo test --workspace

# Run tests with library-only (faster, 30+ seconds)
cargo test --lib --workspace
```

### Code Quality and Formatting

```bash
# Format code (required before commits)
cargo fmt

# Check formatting without changes
cargo fmt --check

# Run clippy linting (takes 15-20 seconds)
cargo clippy --workspace --all-targets --all-features -- -D warnings

# Format Go code in cuengine directory
cd crates/cuengine && gofmt -w .
```

### Development Workflow with Nix Flake

This project uses a Nix flake for reproducible development environments:

```bash
# Enter the development shell
nix develop

# Or run commands directly:
nix develop --command cargo build --workspace --all-features
nix develop --command cargo test --workspace
nix develop --command cargo clippy --workspace --all-targets --all-features -- -D warnings
nix develop --command treefmt --fail-on-change  # Format all files
```

**If Nix is NOT available:** Use standard cargo commands directly (as shown above).

## Validation Scenarios

### Always Test These Scenarios After Making Changes:

1. **Build validation**: Ensure both debug and release builds succeed
2. **Basic functionality**: Test the CLI with example CUE files
3. **Error handling**: Test with invalid inputs to verify error messages

```bash
# Test basic CLI functionality
cargo run -- version
cargo run -- env print --path examples/env-basic --package examples

# Test JSON output format
cargo run -- env print --path examples/env-basic --package examples --output-format json

# Test error handling with invalid path
cargo run -- env print --path /nonexistent
```

Expected outputs:

- Version command shows version info with correlation ID
- env print shows environment variables in KEY=VALUE format
- JSON format outputs valid JSON structure
- Invalid path shows helpful error message with exit code 3

## Timing Expectations and Timeouts

**CRITICAL: NEVER CANCEL long-running operations. Always use appropriate timeouts:**

| Operation                 | Expected Time | Minimum Timeout |
| ------------------------- | ------------- | --------------- |
| `cargo build --workspace` | 90 seconds    | 180 seconds     |
| `cargo build --release`   | 45 seconds    | 120 seconds     |
| `cargo test --workspace`  | 50 seconds    | 120 seconds     |
| `cargo clippy`            | 18 seconds    | 60 seconds      |
| `cargo bench`             | 60+ seconds   | 300 seconds     |

### Build Process Details

- **Debug build**: Downloads dependencies first (~30s), then compiles (~60s)
- **Release build**: Longer optimization phase, but fewer total dependencies
- **Tests**: Runs 75+ tests across all crates (cuengine: 25, cuenv-cli: 50+, cuenv-core: 17)
- **Go FFI tests**: Require CGO and may fail in some CI environments

## Repository Structure

```
cuenv/
├── crates/
│   ├── cuengine/          # Core CUE evaluation engine (Rust + Go FFI)
│   │   ├── bridge.go      # Go bridge for CUE language integration
│   │   ├── build.rs       # Rust build script for Go compilation
│   │   └── src/           # Rust FFI wrapper and caching
│   ├── cuenv-core/        # Shared types, errors, validation
│   └── cuenv-cli/         # CLI interface with TUI support
├── examples/
│   └── env-basic/         # Example CUE configuration files
├── .github/workflows/     # CI/CD configuration
└── flake.nix             # Nix flake configuration
```

## Common Tasks

### Working with CUE Files

cuenv evaluates CUE configuration files to extract environment variables:

```bash
# List what's in the examples directory
ls examples/env-basic/

# View example CUE file
cat examples/env-basic/env.cue

# Test with the example
cargo run -- env print --path examples/env-basic --package examples
```

### Testing Changes to Core Engine

```bash
# Test only the core engine
cd crates/cuengine && cargo test

# Run integration tests
cargo test --test integration_tests

# Test FFI edge cases
cargo test --test ffi_edge_cases
```

### Performance Testing

```bash
# Run benchmarks (NEVER CANCEL - takes 60+ seconds)
cargo bench --workspace --no-fail-fast
```

## CI/CD Integration


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cuenv/cuenv](https://github.com/cuenv/cuenv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
