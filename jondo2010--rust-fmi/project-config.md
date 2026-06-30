---
trigger: always_on
description: Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.
---

# rust-fmi: A Rust FMI (Functional Mockup Interface) Library

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## MCP / rust-analyzer prompt snippet

Use this in the system or preamble when working with the rust-analyzer MCP server:

```
Always prefer MCP tools over guessing. For any codebase question, call `search` (use regex/includePattern) and then `read` the relevant files; do not manually grep or speculate. Batch related queries when possible. If a tool fails, adjust the query and retry before concluding data is missing. Only cite files/lines that were fetched via tools.
```

## Working Effectively

- **CRITICAL**: Initialize git submodules FIRST before any build operation:
  - `git submodule update --init --recursive` -- REQUIRED: FMI standard headers are in submodules
- Bootstrap and build the project:
  - Ensure C compiler is available: `gcc --version` or `clang --version`
  - `cargo check --all` -- takes ~75 seconds. NEVER CANCEL. Set timeout to 120+ seconds.
  - `cargo build --all` -- takes ~49 seconds. NEVER CANCEL. Set timeout to 90+ seconds.
- **CRITICAL**: Unit tests that work offline:
  - `cargo test --package fmi-schema --lib` -- takes ~12 seconds including compilation. Tests XML schema parsing (34 tests pass)
  - `cargo test --package fmi-sim --lib` -- takes ~7 seconds. Tests simulation internals (3 tests pass)
- **WARNING**: Full test suite requires internet access:
  - `cargo test --all` -- FAILS in restricted environments due to TLS certificate issues downloading Reference-FMUs
  - Integration tests download test data from GitHub and will fail offline
- **IMPORTANT**: When using types from the `fmi-schema` or `fmi-sys` crates in other crates, prefer to use them through the re-exports in the `fmi` crate: `use fmi::fmi3::{schema, binding}`, and with prefix.

## Validation

- ALWAYS manually validate any FMI-related code changes by running unit tests for the specific crate.
- ALWAYS run `cargo check --all` before committing to ensure compilation succeeds.
- Test the fmi-sim CLI tool to validate simulation functionality:
  - `cargo run -p fmi-sim -- --help` -- verify CLI interface works
  - `cargo run -p fmi-sim -- --model /nonexistent/file.fmu co-simulation --help` -- test subcommand help
- Build documentation to check for doc issues:
  - `cargo doc --package fmi-schema --no-deps` -- takes ~4 seconds, generates docs with warnings
- **Validation Scenarios After Changes**:
  - For FMI schema changes: Run `cargo test --package fmi-schema --lib` to validate XML parsing
  - For FMI core changes: Run `cargo test --package fmi --lib` (if unit tests exist)
  - For simulation changes: Run `cargo test --package fmi-sim --lib` to validate solver/interpolation logic
  - For bindings changes: Run `cargo build --package fmi-sys` to validate C bindings compilation
- **DO NOT** attempt to run examples or integration tests without internet access - they will fail

## Repository Structure

This is a Rust workspace with 6 main crates:

| Crate           | Purpose                                        | Key Features                                |
| --------------- | ---------------------------------------------- | ------------------------------------------- |
| `fmi`           | Core FMI library for importing/executing FMUs | FMI 2.0/3.0 support, model importing       |
| `fmi-sys`       | Raw Rust bindings to FMI C API               | Uses bindgen, requires C compiler          |
| `fmi-schema`    | XML parsing of FMU Model Description         | Handles FMI 2.0/3.0 XML schemas            |
| `fmi-sim`       | FMU simulation CLI tool                       | ME/CS/SE simulation modes                   |
| `fmi-test-data` | Reference FMUs for testing                    | Downloads test data from GitHub             |
| `xtask`         | Development automation for FMU building      | Bundle, inspect, multi-platform builds     |

## Common Tasks

The following are tested commands and expected behaviors:

### Build Commands (with timing)
```bash
# Essential preparation
git submodule update --init --recursive  # ~30 seconds, downloads FMI headers

# Core build commands
cargo check --all        # ~75 seconds - NEVER CANCEL. Set timeout to 120+ seconds.
cargo build --all        # ~49 seconds - NEVER CANCEL. Set timeout to 90+ seconds.
cargo build --all --release  # ~163 seconds (2m 42s) - NEVER CANCEL. Set timeout to 300+ seconds.

# Working unit tests
cargo test --package fmi-schema --lib  # ~12 seconds including compilation, 34 tests pass
cargo test --package fmi-sim --lib     # ~7 seconds, 3 tests pass
```

### CLI Tool Usage
```bash
# FMI simulation CLI
cargo run -p fmi-sim -- --help                    # Show main help
cargo run -p fmi-sim -- --model file.fmu --help   # Show model-specific options

# Subcommands available:
# - model-exchange: Perform ModelExchange simulation
# - co-simulation: Perform CoSimulation simulation
```

### XTask Development Tool
The `xtask` crate provides development automation for FMU building and inspection:

```bash
# Show available commands

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jondo2010/rust-fmi](https://github.com/jondo2010/rust-fmi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
