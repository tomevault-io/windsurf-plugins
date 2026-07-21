---
trigger: always_on
description: Lance is a modern columnar data format optimized for ML workflows and datasets, providing high-performance random access, vector search, zero-copy automatic versioning, and ecosystem integrations. The vision is to become the de facto standard columnar data format for machine learning and large language models.
---

# AGENTS.md

Lance is a modern columnar data format optimized for ML workflows and datasets, providing high-performance random access, vector search, zero-copy automatic versioning, and ecosystem integrations. The vision is to become the de facto standard columnar data format for machine learning and large language models.

Also see directory-specific guidelines: [rust/](rust/AGENTS.md) | [python/](python/AGENTS.md) | [java/](java/AGENTS.md) | [protos/](protos/AGENTS.md) | [docs/src/format/](docs/src/format/AGENTS.md)

## Architecture

Rust workspace with Python and Java bindings:

- `rust/lance/` - Main library implementing the columnar format
- `rust/lance-core/` - Core types, traits, and utilities
- `rust/lance-arrow/` - Apache Arrow integration layer
- `rust/lance-encoding/` - Data encoding and compression algorithms
- `rust/lance-file/` - File format reading/writing
- `rust/lance-index/` - Vector and scalar indexing
- `rust/lance-io/` - I/O operations and object store integration
- `rust/lance-linalg/` - Linear algebra for vector search
- `rust/lance-table/` - Table format and operations
- `rust/lance-geo/` - Geospatial data support
- `rust/lance-datagen/` - Data generation for tests and benchmarks
- `rust/lance-namespace/` / `rust/lance-namespace-impls/` - Namespace/catalog interfaces
- `rust/lance-test-macros/` / `rust/lance-testing/` - Test infrastructure
- `rust/lance-tools/` - CLI and developer tooling
- `rust/examples/` - Sample binaries and demonstrations
- `rust/compression/bitpacking/` / `rust/compression/fsst/` - Compression codecs
- `rust/lance-datafusion/` - DataFusion integration (built separately)
- `python/` - Python bindings (PyO3/maturin)
- `java/` - Java bindings (JNI)

Key technical traits: async-first (tokio), Arrow-native, versioned writes with manifest tracking, custom ML-optimized encodings, unified object store interface (local/S3/Azure/GCS).

## File Format Stability and Compatibility

- Treat every file format marked stable as a durable compatibility contract. All changes to a stable format must preserve both backward and forward compatibility.
- Treat every file format marked unstable as disposable. It may change freely; do not add compatibility code, migrations, fallbacks, or tests for files written by earlier unstable revisions.
- Evaluate compatibility against the latest released stable version while continuing to honor all stable format contracts. Changes that exist only on the current branch or `main` are not compatibility constraints; do not compromise a cleaner or more complete design to preserve those intermediate states.

## Development Commands

### Rust

* Check: `cargo check --workspace --tests --benches`
* Test: `cargo test --workspace` or `cargo test -p <package> <test_name>`
* Lint: `cargo clippy --all --tests --benches -- -D warnings`
* Format: `cargo fmt --all`
* Coverage: `cargo +nightly llvm-cov -q -p <crate> --branch`
* Coverage HTML: `cargo +nightly llvm-cov -q -p <crate> --branch --html`
* Coverage for file: `python ci/coverage.py -p <crate> -f <file_path>`
* Use repository-defined Cargo profiles instead of ad hoc LTO overrides.
* Use `release-with-debug` for benchmarks and profiling so optimized builds keep debug symbols without a rebuild.
* Use `release-no-lto` only for local debugging, IO-bound benchmarks, or compile-time-sensitive performance investigation where LTO would not affect the measured bottleneck.

### Python / Java

See [python/AGENTS.md](python/AGENTS.md) and [java/AGENTS.md](java/AGENTS.md).

## Language-Specific Environment Contract

- For language-specific tasks, always follow the environment and command rules in the corresponding subdirectory guide before running build, test, lint, format, or tooling commands.
- Do not substitute a different environment manager or toolchain just because a command appears missing, unavailable, or slow.
- If a language-specific command fails outside the documented workflow, treat that as an environment usage mistake first. Fix the environment usage, rerun with the prescribed commands, and only then conclude that a dependency or tool is unavailable.

### Integration Testing

```bash
cd test_data && docker compose up -d
AWS_DEFAULT_REGION=us-east-1 pytest --run-integration python/tests/test_s3_ddb.py
```

## Coding Standards

### General

- Always use English in code, examples, and comments.
- Code is for readability, not just execution. Only add meaningful comments and tests.
- Comments should explain non-obvious "why" reasoning, not restate what the code does.
- Remove debug prints (`println!`, `dbg!`, `print()`) before merging — use `tracing` or logging frameworks.
- Extract logic repeated in 2+ places into a shared helper; inline single-use logic at its call site.
- Think carefully before adding a helper: only introduce one when it materially reduces cognitive load or eliminates substantial duplication, and do not add thin wrappers that only rename or forward existing calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lance-format/lance](https://github.com/lance-format/lance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
