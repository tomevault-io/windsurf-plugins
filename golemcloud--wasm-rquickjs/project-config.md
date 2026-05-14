---
trigger: always_on
description: **wasm-rquickjs** is a command-line tool and library that wraps JavaScript code into WebAssembly Components using the QuickJS engine. The tool generates self-contained Rust crates that compile to WASM components.
---

# AGENTS.md - wasm-rquickjs Development Guide

## Project Overview

**wasm-rquickjs** is a command-line tool and library that wraps JavaScript code into WebAssembly Components using the QuickJS engine. The tool generates self-contained Rust crates that compile to WASM components.

## Project Structure

```
wasm-rquickjs/
├── crates/
│   ├── wasm-rquickjs/          # Main library crate
│   │   ├── skeleton/           # Embedded skeleton crate (separate project)
│   │   └── src/
│   └── wasi-logging/           # WASI logging support crate
├── examples/
│   ├── runtime/                # Examples used by runtime tests
│   └── compilation/            # Examples only tested via compilation
├── src/                        # CLI binary source (main.rs)
├── tests/                      # Integration tests
├── Cargo.toml                  # Workspace root
├── cleanup-skeleton.sh         # Script to clean skeleton build artifacts
└── README.md                   # Main documentation
```

## Skeleton Crate

The `skeleton` crate (`crates/wasm-rquickjs/skeleton/`) is a **separate project** embedded via `include_dir!`. It has special build requirements — **load the `skeleton-development` skill** for the full workflow (Cargo.toml_ convention, cleanup, test rules).

## Build Commands

### Build the CLI binary
```bash
cargo build --release
```

### Run tests
```bash
cargo test
```

### ⚠️ CRITICAL TEST RULES

**DO NOT run `cargo test` without arguments** — it runs everything and takes too long. **ALWAYS filter** to a specific test harness and module. Load the `skeleton-development` skill for full test rules and examples.

### Generate code for a JavaScript module
```bash
./target/release/wasm-rquickjs generate-wrapper-crate \
  --js <path/to/module.js> \
  --wit <path/to/wit/root> \
  --output <output/directory>
```

### Generate TypeScript definitions
```bash
./target/release/wasm-rquickjs generate-dts \
  --wit <path/to/wit/root> \
  --output <output/directory>
```

## Code Quality

### Formatting

The project uses `rustfmt` for code formatting. Before committing, ensure code is formatted:

```bash
cargo fmt
```

To check formatting without making changes:

```bash
cargo fmt -- --check
```

### Clippy Linting

The project uses Clippy with strict warnings enabled. All warnings must be fixed:

```bash
cargo clippy -- -Dwarnings
```

To fix common issues automatically:

```bash
cargo clippy --fix -- -Dwarnings
```

### Pre-commit Checks

Run all quality checks before committing:

```bash
cargo fmt
cargo clippy -- -Dwarnings
cargo test
```

## Workspace Configuration

The workspace is configured in the root `Cargo.toml` with the following members:
- `crates/wasi-logging` - WASI logging support
- `crates/wasm-rquickjs` - Main library crate

**Excluded from workspace:**
- `crates/wasm-rquickjs/skeleton` - Separate project, compiled independently
- `tmp/` - Temporary build artifacts

## Development Workflow

### When working on the skeleton:

Load the `skeleton-development` skill. For Node.js compat test work, also load `fixing-node-compat-test`. For adding new modules, load `adding-builtin-module`.

### When working on code generation:

1. Modify the main `crates/wasm-rquickjs` crate
2. Run tests with `cargo test`
3. Changes to the skeleton will be embedded via `include_dir!` at compile time

### When working on the CLI:

1. Modify `src/main.rs`
2. Build with `cargo build --release`
3. Test with `cargo test --test compilation` or `cargo test --test runtime`

### Updating the Supported APIs Documentation

When adding a new built-in API — such as a new Node.js-compatible module or a new exported function in an existing module — **always update the list of supported APIs in `README.md`** to reflect the change.

## Regenerating DTS Goldenfiles

Load the `regenerating-goldenfiles` skill for the workflow.

## Node.js Compatibility Tests

The `tests/node_compat/` directory contains vendored Node.js test files used to verify our Node.js API compatibility. Important rules:

- **Never modify vendored test files** in `tests/node_compat/suite/`. These are upstream Node.js tests fetched via `vendor.sh` and must remain unmodified.
- **We only implement the public Node.js API.** Tests that exercise Node.js internals (internal modules, private APIs, implementation details) are out of scope. Only tests for the public-facing Node.js API surface are relevant.
- The `config.jsonc` allowlist controls which tests are run. Add or remove entries there rather than modifying test files.
- Tests are **dynamically generated** by `tests/node_compat.rs` using test-r's `#[test_gen]`: one test case per entry in `config.jsonc`. A shared `PreparedComponent` (compiled WASM) is created once as a test dependency and reused across all tests.
- Tests with `"skip": true` in `config.jsonc` are marked as `is_ignored` and reported as `IGNORED` by the test runner.
- Test names follow the pattern `gen_node_compat_tests::<suite>__<test_file>` (e.g., `parallel__test_btoa_atob_js`).

Load the `fixing-node-compat-test` skill for the full workflow when making a test pass.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [golemcloud/wasm-rquickjs](https://github.com/golemcloud/wasm-rquickjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
