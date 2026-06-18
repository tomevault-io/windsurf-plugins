---
trigger: always_on
description: Fast jq-compatible JSON processor using SIMD parsing (C++ simdjson via FFI), parallel NDJSON processing, and streaming architecture.
---

# qj — a fast, jq-compatible JSON processor

Fast jq-compatible JSON processor using SIMD parsing (C++ simdjson via FFI), parallel NDJSON processing, and streaming architecture.

## After writing Rust code
```
cargo fmt
cargo clippy --release -- -D warnings
cargo test
```

## After writing shell scripts
```
shellcheck <file>.sh
```

## Testing
`cargo test` runs the fast suite (unit + e2e + ndjson + ffi, ~5s).
Compat suites are `#[ignore]` — run them with `--release` after adding features.

```
cargo test                                                              # fast: unit + e2e (~5s)
cargo test --release -- --ignored --nocapture                           # all tests including compat (~50s)
cargo test --release jq_conformance -- --ignored                        # jq.test pass rate (summary on stderr)
cargo test --release jq_conformance_ndjson -- --ignored --nocapture     # jq.test via NDJSON path (single vs NDJSON diff)
cargo test --release jq_conformance_verbose -- --ignored --nocapture    # jq.test with failure details
cargo test --release conformance_gaps -- --ignored                      # gap tests by category
cargo test --release gap_label_break -- --ignored                       # run one category
cargo test --release jq_compat -- --ignored --nocapture                 # cross-tool comparison
cargo test --release feature_compat -- --ignored --nocapture            # feature matrix
cargo test --release jq_differential -- --ignored --nocapture          # proptest differential vs jq
cargo test --release differential_filter -- --ignored --nocapture      # differential: random filters
cargo test --release differential_arithmetic -- --ignored --nocapture  # differential: arithmetic focus
cargo test --release differential_builtins -- --ignored --nocapture    # differential: builtins focus
cargo test --release differential_formats -- --ignored --nocapture     # differential: format strings
```

**Note:** The conformance test prints its summary to stderr (visible without `--nocapture`).
Never pipe `--nocapture` output through `tail` — the verbose test produces 500+ lines which
can OOM `tail` on macOS. Use `grep` to filter if needed, or run the non-verbose test.

- **Unit tests:** `#[cfg(test)]` modules alongside code.
- **Integration tests:** `tests/e2e.rs` — runs the `qj` binary against known JSON inputs.
  - Includes **jq conformance tests** (`assert_jq_compat`) that run both qj and jq and
    compare output. These run automatically when jq is installed, and are skipped otherwise.
  - **Zero divergence policy:** Every e2e test that exercises jq-compatible behavior MUST
    use `assert_jq_compat` to verify output matches jq exactly. Never write tests that
    accept output differing from jq — if a fast path (passthrough, NDJSON, etc.) would
    produce different results, it must fall back to the normal evaluator.
  - Includes **number literal preservation tests** — verifies trailing zeros, scientific
    notation, and raw text are preserved from JSON input through output.
- **NDJSON tests:** `tests/ndjson.rs` — parallel NDJSON processing integration tests.
- **FFI tests:** `tests/simdjson_ffi.rs` — low-level simdjson bridge tests.
- **jq conformance suite** (`#[ignore]`): `tests/jq_conformance.rs` — runs jq's official test
  suite (`tests/jq_compat/jq.test`, vendored from jqlang/jq) against qj and reports pass rate.
  Also includes `jq_conformance_ndjson` which runs each object/array test case through both
  single-doc and NDJSON paths, asserting identical output (catches NDJSON path divergences).
- **Conformance gap tests** (`#[ignore]`): `tests/conformance_gaps.rs` — 9 tests for
  jq.test bignum/precision edge cases. All pass with `QJ_JQ_COMPAT=1` (497/497).
  See `docs/COMPATIBILITY.md` for analysis.
- **Cross-tool compat comparison** (`#[ignore]`): `tests/jq_compat_runner.rs` — runs jq.test
  against qj, jq, jaq, and gojq. Writes `tests/jq_compat/results.md`.
- **Feature compatibility suite** (`#[ignore]`): `tests/jq_compat/features.toml` — TOML-defined
  tests, per-feature Y/~/N matrix. Writes `docs/COMPATIBILITY.md` (appends below marker).
- **Differential testing** (`#[ignore]`): `tests/jq_differential.rs` — property-based tests using
  `proptest` that generate random (filter, input) pairs and compare qj vs jq output. Four focused
  tests: general filters, arithmetic, builtins, and format strings. 2000 cases each. Catches
  behavioral divergences that hand-written tests miss. Run iteratively: fix or exclude each
  failure, re-run to find the next.
- **Updating the vendored test suite:** `tests/jq_compat/update_test_suite.sh` — downloads
  `jq.test` and test modules from a jq release tag and updates `mise.toml`.
  ```
  bash tests/jq_compat/update_test_suite.sh          # uses version from mise.toml
  bash tests/jq_compat/update_test_suite.sh 1.9.0    # upgrade to new version
  ```
- **When adding new jq builtins or language features**, always:
  1. Add corresponding e2e tests in `tests/e2e.rs` and `assert_jq_compat` checks
  2. Run `cargo test --release jq_compat -- --ignored --nocapture` and update jq compat % in `README.md`
- **When adding or modifying NDJSON fast-path variants** (`NdjsonFastPath` enum in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [6/qj](https://github.com/6/qj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
