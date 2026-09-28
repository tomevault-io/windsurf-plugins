---
trigger: always_on
description: This file provides guidance to [OpenAI codex](https://github.com/openai/codex) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to [OpenAI codex](https://github.com/openai/codex) when working with code in this repository.

## 🛠️ PROJECT TECHNICAL DETAILS

### Project Overview

Cribo is a Python source bundler written in Rust that produces a single .py file from a multi-module Python project by inlining first-party source files. It's available as a CLI tool.

Key features:

- Tree-shaking to include only needed modules
- Unused import detection and trimming
- Requirements.txt generation
- Configurable import classification

### Build Commands

#### Rust Binary

```bash
# Development build
cargo build

# Release build
cargo build --release

# Run the tool directly
cargo run -- --entry path/to/main.py --output bundle.py

# Output to stdout for debugging (no temporary files)
cargo run -- --entry path/to/main.py --stdout

# Run with verbose output for debugging
cargo run -- --entry path/to/main.py --output bundle.py -vv

# Run with trace-level output for detailed debugging
cargo run -- --entry path/to/main.py --output bundle.py -vvv

# Combine stdout output with verbose logging for development
cargo run -- --entry path/to/main.py --stdout -vv
```

### CLI Usage

```bash
cribo --entry src/main.py --output bundle.py [options]

# Output to stdout instead of file (ideal for debugging)
cribo --entry src/main.py --stdout [options]

# Common options
--emit-requirements    # Generate requirements.txt with third-party dependencies
-v, --verbose...       # Increase verbosity (can be repeated: -v, -vv, -vvv)
                       # No flag: warnings/errors only
                       # -v: informational messages  
                       # -vv: debug messages
                       # -vvv: trace messages
--config               # Specify custom config file path
--target-version       # Target Python version (e.g., py38, py39, py310, py311, py312, py313)
--stdout               # Output bundled code to stdout instead of a file
```

The verbose flag is particularly useful for debugging bundling issues. Each level provides progressively more detail about the bundling process, import resolution, and dependency graph construction.

The `--stdout` flag is especially valuable for debugging workflows as it avoids creating temporary files and allows direct inspection of the bundled output. All log messages are properly separated to stderr, making it perfect for piping to other tools or quick inspection.

### Testing Commands

```bash
# Run all tests
cargo nextest run --workspace

# Run with code coverage
cargo llvm-cov nextest --workspace --json
```

#### Snapshot Testing with Insta

Accept new or updated snapshots using:

```bash
cargo insta accept
```

### Architecture Overview

The project is organized as a Rust workspace with the main crate in `crates/cribo`. The architecture follows a clear separation of concerns with dedicated modules for analysis, code generation, and AST traversal.

#### 🔍 Core Components & Navigation Guide

**THE REAL CRITICAL PATH: How Modules Get Bundled**

1. **CLI Entry Point** → `main.rs`
   - [`main()` in `main.rs`](crates/cribo/src/main.rs#L85) is the entry point.
   - Creates [`BundleOrchestrator`](crates/cribo/src/orchestrator.rs#L176) and calls `bundle()` or `bundle_to_string()`.

2. **Application Orchestration** → `orchestrator.rs::BundleOrchestrator`
   - [`bundle()` / `bundle_to_string()`](crates/cribo/src/orchestrator.rs#L636) → Entry points
   - [`bundle_core()`](crates/cribo/src/orchestrator.rs#L356) → Module discovery, parsing, dependency resolution
   - [`emit_static_bundle()`](crates/cribo/src/orchestrator.rs#L1850) → Delegates to PhaseOrchestrator

3. **🔥 Phase-Based Bundling** → [`code_generator/phases/orchestrator.rs::PhaseOrchestrator`](crates/cribo/src/code_generator/phases/orchestrator.rs#L31)
   Coordinates 9 bundling phases:
   - [`PhaseOrchestrator::bundle()`](crates/cribo/src/code_generator/phases/orchestrator.rs#L47) → Main entry
   - `InitializationPhase` → Setup, future imports
   - `PreparationPhase` → Trim imports, index ASTs
   - `ClassificationPhase` → Inlinable vs wrapper decision
   - `ProcessingPhase` → Module emission in dependency order
   - `EntryModulePhase` → Special entry module handling
   - `PostProcessingPhase` → Namespace attachments, proxies

4. **Module Classification** → `analyzers/module_classifier.rs`
   - [`classify_modules()` in `module_classifier.rs`](crates/cribo/src/analyzers/module_classifier.rs#L126) is where the decision to inline or wrap a module is made.

   ```rust
   // THE decision that determines bundle structure:
   if has_side_effects || has_invalid_identifier || needs_wrapping_for_circular:
       → wrapper_modules.push()  // Becomes init function with circular import guards
   else:
       → inlinable_modules.push() // Directly inserted into bundle
   ```

5. **Side Effect Detection** → `visitors/side_effect_detector.rs`
   - Detects side effects that force wrapping: `Expr::Call(_)`, `Expr::Lambda(_)`, class metaclasses, non-literal expressions.

6. **Module Processing** → `code_generator/phases/processing.rs::ProcessingPhase`
   - Processes modules in topological order
   - Two-phase emission for circular dependencies

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ophi-dev/cribo](https://github.com/ophi-dev/cribo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
