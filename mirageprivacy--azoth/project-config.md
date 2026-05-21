---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Azoth is a research-grade toolchain for Ethereum smart-contract obfuscation. The project follows a three-stage pipeline:

**Repository**: https://github.com/MiragePrivacy/obfuscator

1. **Pre-processing/Analysis**: Isolate runtime bytecode and measure structure
2. **Obfuscation Passes**: Apply reversible transforms to raise analyst effort
3. **Re-assembly & Validation**: Splice segments back and validate equivalence

## Architecture

This is a Rust workspace with six main crates:

- **`crates/core/`**: Core functionality including bytecode decoder, encoder, section detector, bytecode stripper, and CFG-IR generation. The detection module includes section isolation (`detection/sections.rs`) and dispatcher pattern detection (`detection/dispatcher.rs`)
- **`crates/analysis/`**: Analysis utilities for measuring bytecode complexity and obfuscation quality through comprehensive metrics
- **`crates/transforms/`**: Obfuscation passes including opaque predicates, control flow shuffling, function dispatcher obfuscation, and jump address transformation. The `obfuscator.rs` module orchestrates all transforms
- **`crates/verification/`**: Formal verification engine providing mathematical proofs of semantic equivalence using Z3 SMT solver, plus practical testing with REVM. Supports multiple verification levels and generates cryptographic certificates
- **`crates/utils/`**: Shared utilities including deterministic seed generation and error types
- **`crates/cli/`**: Command-line interface (`azoth` binary) with subcommands for decode, strip, cfg, and obfuscate

### Additional Workspace Members
- **`tests/`**: Workspace-level integration and end-to-end tests organized by component (core, transforms, analysis, verification, e2e)
- **`examples/`**: Example contracts including escrow bytecode for testing and benchmarking

## Development Requirements

**Important**: This project uses Rust edition 2021 and standard stable toolchain:
```bash
cargo build
```

## Common Commands

### Build and Test

The project uses custom cargo aliases defined in `.cargo/config.toml`:

```bash
# Build all packages with release optimizations
cargo bb

# Run clippy on all packages (warnings treated as errors)
cargo cc

# Run all tests using cargo-nextest
cargo tt

# Standard cargo commands also work
cargo build
cargo test
cargo fmt
cargo clippy
```

**Note**: This project uses `cargo-nextest` as the test runner. Install it with:
```bash
cargo install cargo-nextest
```

### CLI Usage
```bash
# Build CLI binary
cargo build --bin azoth

# Decode bytecode to annotated assembly
cargo run --bin azoth -- decode <bytecode>

# Strip init/auxdata, output runtime hex
cargo run --bin azoth -- strip <bytecode>

# Generate and visualize CFG-IR
cargo run --bin azoth -- cfg <bytecode>

# Apply obfuscation transforms
cargo run --bin azoth -- obfuscate <bytecode>
```

### Input Formats
- Hex string: `0x608060405234801561001057600080fd5b50...`
- File path: `path/to/bytecode.hex`

## Code Style and Documentation Standards

### Rust Code Standards
The project follows strict Rust coding standards to ensure maintainability and consistency across the codebase.

**Formatting and Structure:**
- Standard rustfmt configuration
- Maximum line width of 100 characters
- Rust edition 2021
- Consistent indentation and spacing enforced by automated tools

**Documentation Requirements:**
All public APIs must include comprehensive rustdoc documentation. This is enforced by the `missing-docs` lint at the workspace level.

**Required Documentation Elements:**
- Module-level documentation explaining the purpose and scope of functionality
- Comprehensive documentation for all public functions, structs, enums, and traits
- Parameter descriptions with type constraints and expected ranges
- Return value documentation with error conditions
- Usage examples demonstrating typical API interactions
- Cross-references to related functionality where appropriate

**Documentation Style Guidelines:**
Write documentation in clear, professional prose rather than fragmented bullet points. The documentation should provide sufficient context for developers to understand not just how to use the API, but why certain design decisions were made and how the component fits into the larger system architecture.

**Example Documentation Pattern:**
```rust
/// Constructs a Control Flow Graph with Intermediate Representation from EVM bytecode.
///
/// This function performs the core transformation from linear bytecode instructions
/// into a structured graph representation that enables sophisticated analysis and
/// obfuscation transforms. The resulting CFG maintains semantic equivalence while
/// providing the structural information necessary for advanced code analysis.
///
/// The construction process involves several phases: basic block identification
/// through control flow analysis, edge creation based on jump target resolution,
/// and SSA form generation for stack operation tracking. Each phase validates
/// the bytecode structure to ensure the resulting CFG accurately represents
/// the original program semantics.
///

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MiragePrivacy/azoth](https://github.com/MiragePrivacy/azoth) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
