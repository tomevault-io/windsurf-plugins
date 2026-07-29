---
trigger: always_on
description: Spice is a SQL query, search, and LLM-inference engine in Rust for data apps and agents: federated SQL, data acceleration/materialization, vector/keyword/full-text search, and AI inference via industry-standard APIs. Rust CLI (`bin/spice`) + runtime daemon (`bin/spiced`), built on Apache DataFusion, Arrow, and DuckDB; configured via Spicepods (YAML). Core principle: developer experience first — bring data and AI to the application, not the other way around.
---

# Spice.ai Agent Instructions

Spice is a SQL query, search, and LLM-inference engine in Rust for data apps and agents: federated SQL, data acceleration/materialization, vector/keyword/full-text search, and AI inference via industry-standard APIs. Rust CLI (`bin/spice`) + runtime daemon (`bin/spiced`), built on Apache DataFusion, Arrow, and DuckDB; configured via Spicepods (YAML). Core principle: developer experience first — bring data and AI to the application, not the other way around.

## Data correctness — absolute top priority

As an AI-native database, query results can NEVER be wrong. Correctness supersedes performance, developer experience, and feature velocity. Verify transformations preserve integrity (row counts, key values); rigorously test NULLs, empty sets, boundaries, type coercions, and overflow; when uncertain, return a structured error instead of possibly-wrong data. Never corrupt data or drop errors silently.

## Build, test, lint (expensive — read first)

Full workspace and release builds take 20–35 minutes. Minimize large builds:

- **Batch all related edits first, then run one build/lint/test pass at the end.** Never build after each edit.
- **Scope cargo to touched crates**: `cargo check -p <crate>`, `cargo test -p <crate> --lib <filter>`, `cargo clippy -p <crate> --no-deps`. Validate compilation with incremental `cargo check`; run `cargo build --release -p spiced` once, only after everything is green.
- **New crates must opt into the workspace lints**: clippy lint levels live in `[workspace.lints.clippy]` in the root `Cargo.toml` (pedantic + `unwrap_used`/`expect_used`/`clone_on_ref_ptr`/…). Every member crate inherits them via `[lints]\nworkspace = true` in its `Cargo.toml` — add this to any new crate, or scoped `cargo clippy -p <crate>` and rust-analyzer will silently under-lint it. Additional shared lint config belongs in `[workspace.lints.*]` in the root `Cargo.toml` (a crate can’t inherit with `workspace = true` and also define per-crate `[lints.*]` overrides). `make lint-rust` re-applies the flags over `--workspace` as a backstop, so a forgotten opt-in fails CI, not silently ships.
- **Fix the feature set for the whole session.** Cargo re-fingerprints incremental artifacts on the exact `--features` (and profile, and `RUSTFLAGS`/wrapper), and features flow through the entire dependency graph — so alternating flag sets between `check`/`test`/`clippy`, or diverging from the `make lint-rust` gate's `--features adbc,…,release,…`, silently forces full recompiles. At the start of a branch, identify the features your touched crates need, reuse the *same* `--features` on every `cargo` invocation, and scope the gate to match: `make lint-rust-fix PACKAGES="<crates>" FEATURES="<same set>"`. Keep the profile and the sccache-bypass env constant for the same reason. (`clippy` and `check` still recompile the workspace crates when alternated even with matching features — deps are reused, so the win is still large.)
- **One cargo invocation at a time**: a second blocks on the target-dir lock, and concurrent heavy builds contaminate bench timings.
- **Lint covers tests too**: `make lint-rust` runs a second clippy pass over `--tests` with pedantic lints, so test code and its doc comments must pass (e.g. `doc_markdown`: backtick product names like `PostgreSQL`, `DuckDB`). Green tests ≠ lint green; the scoped clippy above now inherits the same lint *levels*, but `make lint-rust` is still the gate (it adds the full release feature set + the `--tests` pass) — run `make lint-rust-fix` before pushing.
- **Sign off after pushing** (the script attests your pushed HEAD and refuses an unpushed branch): `make signoff` first target-lints crates touched by the branch (`make lint-rust PACKAGES=…`), then runs the full gate (`make lint-rust` + `make build-cli nextest`) and records the attestation that gates merge-queue entry — a single `Attestation` PR check; the full suite then runs once in the merge queue (see `docs/dev/ci_signoff.md`). Use `make signoff-remote` to dispatch the same sequence on a self-hosted runner. These steps span profiles (lint/dev vs release), so expect separate compiles; keep the *same* `--features`/profile as the `make lint-rust` gate across your other local `cargo` commands to avoid re-fingerprinting and full rebuilds (see the fixed-feature-set bullet above).
- **If sccache fails** (unwritable volume breaks the `aws-lc-sys` C compile): `env -u RUSTC_WRAPPER -u RUSTC_WORKSPACE_WRAPPER CC=cc CXX=c++ cargo …`.

```bash
make install-dev        # Dev build (faster); release: make install
SPICED_CUSTOM_FEATURES="postgres sqlite" make build-runtime
make test               # Unit tests
make test-integration   # Needs credentials (.env or `spice login`)
make lint-rust-fix      # Auto-fix lint issues
```

## Git & PRs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [spiceai/spiceai](https://github.com/spiceai/spiceai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
