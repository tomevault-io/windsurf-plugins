---
trigger: always_on
description: High-performance YAML 1.2.2 parser with Rust core and Python/Node.js bindings.
---

# fast-yaml Copilot Instructions

High-performance YAML 1.2.2 parser with Rust core and Python/Node.js bindings.

## Architecture

**Workspace structure** with 3 Rust crates + 2 binding packages:
- `crates/fast-yaml-core/` — Core parser/emitter wrapping saphyr
- `crates/fast-yaml-linter/` — Linting engine with pluggable rules and rich diagnostics
- `crates/fast-yaml-parallel/` — Rayon-based multi-threaded document processing
- `python/` — PyO3 bindings (maturin build)
- `nodejs/` — NAPI-RS bindings

**Data flow**: Python/Node.js → FFI bindings → core crates → saphyr

## Critical Commands

```bash
# Format (requires nightly for Edition 2024)
cargo +nightly fmt --all

# Lint (exclude python extension to avoid pyo3 issues)
cargo clippy --workspace --all-targets --exclude fast-yaml -- -D warnings

# Test (ALWAYS use nextest, not cargo test)
cargo nextest run --workspace

# Python development
uv sync && uv run maturin develop
uv run pytest tests/ -v

# Coverage
cargo llvm-cov nextest --workspace --html
```

## Code Conventions

**Error handling**: Use `thiserror` in library crates, `anyhow` in bindings.

**Unsafe code is denied by default**: `unsafe_code = "deny"` in workspace lints. Use `#[allow(unsafe_code)]` only for FFI boundaries (NAPI-RS, memory-mapping) with mandatory SAFETY documentation.

**YAML 1.2.2 compliance**: Unlike PyYAML (1.1), `yes/no/on/off` are strings, not booleans. Octal requires `0o` prefix.

**Type conversions** follow patterns in [python/src/conversion.rs](../python/src/conversion.rs) — convert between Rust `Value` types and Python/JS objects via FFI traits.

## Adding Features

**New lint rule**: Create in `crates/fast-yaml-linter/src/rules/`, implement `LintRule` trait, register in `Linter::with_all_rules()`.

**Python API**: Add function in `python/src/lib.rs` with `#[pyfunction]`, export in `#[pymodule]`, add type stub in `python/fast_yaml/_core.pyi`.

**New crate dependency**: Add to `[workspace.dependencies]` in root `Cargo.toml`, then reference with `dep.workspace = true`.

## Testing Patterns

- Unit tests in `#[cfg(test)]` modules within source files
- Integration tests in `crates/*/tests/` and `tests/`
- YAML spec fixtures in `tests/fixtures/yaml-spec/`
- Python tests use pytest in `tests/test_fast_yaml.py`

## Performance Notes

- Release builds use LTO + `codegen-units=1` for maximum optimization
- Python GIL released during CPU-intensive Rust operations
- Parallel processing splits at `---` document boundaries
- Input size capped at 100MB (`MAX_INPUT_SIZE`) for DoS protection

## Code Review Guidelines

### Review Checklist for PRs

**Pre-merge requirements:**
- [ ] All CI checks passing (format, clippy, tests, coverage, security)
- [ ] Code coverage maintained or improved (minimum 60% overall)
- [ ] Documentation updated (API docs, README, CHANGELOG)
- [ ] Tests added for new functionality
- [ ] No clippy warnings introduced
- [ ] Commit messages follow conventional commits format
- [ ] Breaking changes documented and justified

**Code quality:**
- [ ] Code follows Rust idioms and patterns
- [ ] Error handling is comprehensive and informative
- [ ] No `unwrap()` or `expect()` in library code (only in tests)
- [ ] Public APIs have documentation comments with examples
- [ ] Complex logic has explanatory comments
- [ ] No dead code or unused imports

**Performance:**
- [ ] No unnecessary allocations in hot paths
- [ ] Appropriate use of `&str` vs `String`, `&[T]` vs `Vec<T>`
- [ ] Clone operations are justified
- [ ] Large data structures use references when possible

### Rust-Specific Review Points

**Ownership and lifetimes:**
- Check for unnecessary clones that could be borrows
- Verify lifetime annotations are correct and minimal
- Ensure `'static` lifetimes are truly necessary
- Look for potential iterator chaining instead of collecting intermediate results

**Error handling:**
- Library crates use `thiserror` for custom error types
- Binding crates use `anyhow` with context chains
- Errors include source location information (file, line, column) for diagnostics
- Error messages are actionable and user-friendly
- `Result` types are properly propagated with `?` operator

**Clippy compliance:**
- Run `cargo clippy --workspace --all-targets -- -D warnings -W clippy::pedantic`
- Address all warnings or explicitly allow with justification
- Common issues to watch:
  - `clippy::missing_errors_doc` — document error conditions
  - `clippy::must_use_candidate` — add `#[must_use]` where appropriate
  - `clippy::redundant_closure` — use method references
  - `clippy::unnecessary_wraps` — remove `Result` if error path is impossible

**Memory safety:**
- Minimize `unsafe` code (denied by default in workspace lints)
- FFI requires unsafe (NAPI-RS, memory-mapping) - ensure proper encapsulation and SAFETY comments
- All unsafe code must have `#[allow(unsafe_code)]` and SAFETY documentation
- Verify bounds checking on slice operations
- Check for potential panics (`unwrap`, `expect`, indexing)

**Concurrency:**
- Parallel code uses Rayon work-stealing correctly
- No data races (Rust prevents at compile-time, but check logic)
- Shared state is properly synchronized (Mutex, RwLock, atomic types)
- Thread pool sizes are configurable or auto-detected


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bug-ops/fast-yaml](https://github.com/bug-ops/fast-yaml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
