---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Development Commands

**Note:** Build times can exceed 10 minutes, especially for full workspace builds with all features or when building the FHIR generator due to large generated files.

### Building
```bash
# Build default (R4 only)
cargo build

# Build with all FHIR versions
cargo build --features R4,R4B,R5,R6

# Build specific crate
cargo build -p helios-sof
cargo build -p helios-fhirpath
cargo build -p helios-rest
cargo build -p helios-persistence --features postgres,elasticsearch

# Build pysof (excluded from default workspace build, requires Python)
cd crates/pysof && uv run maturin develop --release
```

### Running Binaries
```bash
# HFS FHIR server (default: R4, SQLite, port 8080)
cargo run --bin hfs

# FHIRPath CLI
cargo run --bin fhirpath-cli -- -e "Patient.name.family" -r patient.json

# FHIRPath HTTP server (default port 3000)
cargo run --bin fhirpath-server

# SQL-on-FHIR CLI
cargo run --bin sof-cli -- --view view.json --bundle data.json --format csv

# SQL-on-FHIR HTTP server (default port 8080)
cargo run --bin sof-server

# Persistence config advisor
cargo run --bin config-advisor
```

### Testing
```bash
# Run all tests (default R4)
cargo test

# Test with all FHIR versions
cargo test --features R4,R4B,R5,R6

# Test specific crate
cargo test -p helios-sof
cargo test -p helios-fhirpath
cargo test -p helios-persistence

# Run single test
cargo test test_name_pattern

# Run tests in specific file
cargo test --test test_file_name

# Show test output
cargo test -- --nocapture

# pysof Python tests (from crates/pysof/)
cd crates/pysof && uv run pytest python-tests/ -v
```

### Linting and Formatting
```bash
# Format code
cargo fmt --all

# Lint code (with CI-compatible flags)
# Note: --all-features is safe here — it enables the `skip-r6-download` feature
# in all crates (fhir, fhir-gen, hfs), preventing R6 spec downloads from build.fhir.org.
cargo clippy --all-targets --all-features -- -D warnings \
  -A clippy::items_after_test_module \
  -A clippy::large_enum_variant \
  -A clippy::question_mark \
  -A clippy::collapsible_match \
  -A clippy::collapsible_if \
  -A clippy::field_reassign_with_default \
  -A clippy::doc-overindented-list-items \
  -A clippy::doc-lazy-continuation

# Check types without building
cargo check
```

### Before Completing Code Changes
Before declaring a plan complete after significant code changes, always run:
1. `cargo fmt --all` - Format all code
2. `cargo clippy` with the CI flags shown above - Fix any linting issues
3. `cargo test` for affected crates - Ensure tests pass

### Documentation
```bash
# Generate and view docs
cargo doc --no-deps --open
```

### FHIR Code Generation
```bash
# Generate FHIR models for all versions
cargo build -p helios-fhir-gen --features R6
./target/debug/fhir_gen --all

# Note: R6 specification files are auto-downloaded from HL7 build server
# Note: Building fhir-gen can take 5-10 minutes due to large generated files
```

## Architecture Overview

### Workspace Structure

The project is a Rust workspace with 12 crates (`pysof` excluded from default-members):

| Crate | Description |
|-------|-------------|
| **`helios-fhir`** | Core FHIR data models (auto-generated). Supports R4, R4B, R5, R6 via feature flags. |
| **`helios-fhir-gen`** | Code generator — produces Rust structs from FHIR JSON schemas. R6 specs auto-downloaded. |
| **`helios-fhir-macro`** | Procedural macros for FHIR functionality. |
| **`helios-fhirpath`** | FHIRPath expression language — parser (chumsky), evaluator, CLI tool, and HTTP server. |
| **`helios-fhirpath-support`** | Shared support utilities for FHIRPath. |
| **`helios-serde`** | JSON and XML serialization for FHIR resources (`xml` feature flag). |
| **`helios-serde-support`** | Shared serde helpers. |
| **`helios-rest`** | FHIR RESTful API layer (Axum) — handlers, middleware, extractors, multi-tenancy routing. |
| **`helios-persistence`** | Polyglot persistence — backends (SQLite, PostgreSQL, Elasticsearch, MongoDB), composite storage, search registry, tenant isolation. |
| **`helios-hfs`** | Main FHIR server binary. Combines `helios-rest` with storage backends. |
| **`helios-sof`** | SQL-on-FHIR implementation — ViewDefinition processing, CLI and HTTP server. |
| **`pysof`** | Python bindings (PyO3/maturin) for SQL-on-FHIR. Excluded from default workspace build. |

### Binaries

| Binary | Crate | Description |
|--------|-------|-------------|
| `hfs` | helios-hfs | FHIR server |
| `fhirpath-cli` | helios-fhirpath | FHIRPath expression evaluator CLI |
| `fhirpath-server` | helios-fhirpath | FHIRPath HTTP evaluation server |
| `sof-cli` | helios-sof | SQL-on-FHIR CLI tool |
| `sof-server` | helios-sof | SQL-on-FHIR HTTP server |
| `config-advisor` | helios-persistence | Storage configuration advisor |
| `hts` | helios-hts | FHIR Terminology Server (HTS) |

### Key Design Patterns

#### Version-Agnostic Abstraction
The codebase uses enum wrappers and traits to handle multiple FHIR versions:

```rust
// Example from sof crate
pub enum SofViewDefinition {
    R4(fhir::r4::ViewDefinition),
    R4B(fhir::r4b::ViewDefinition),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HeliosSoftware/hfs](https://github.com/HeliosSoftware/hfs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
