---
trigger: always_on
description: Guidelines for contributors and coding agents working on `faust-rs`.
---

# AGENTS

Guidelines for contributors and coding agents working on `faust-rs`.

## 1. Project Goal

- Port the Faust compiler from C++ to Rust.
- Keep semantic parity with the C++ compiler as the default objective.
- Prefer explicit, testable behavior over speculative refactors.
- C++ reference branch used for baseline analysis: `master-dev-ocpp-od-fir-2-FIR19` (`8eebea429`) in /Users/letz/Developpements/RUST/faust folder.

## 2. Workspace Rules

- This repository is a Cargo workspace with many crates under `crates/`.
- Respect crate boundaries; avoid circular dependencies.
- Add new code in the most specific crate first, then expose upward through public APIs.
- Keep `crates/compiler` as the top-level orchestration crate (lib + CLI entry point).
- Use `clap` as the default command-line argument parser for user-facing binaries; use another parser only with an explicit documented reason in `porting/` or `JOURNAL.md`.
- Keep crate responsibilities aligned with `porting/faust-rust-porting-plan-en.md` section 2/4.
- Preserve key integrations recommended by the plan:
  - `patternmatcher` logic merged into `eval`.
  - `extended` math nodes integrated into `signals`.
  - `parallelize` integrated into `transform`.

## 3. Code Quality

- Rust edition and toolchain are controlled by workspace files.
- Before committing, run:
  - `cargo fmt --all`
  - `cargo clippy --workspace --all-targets -- -D warnings`
  - `cargo test --workspace --all-targets`
- Avoid introducing `unsafe` unless strictly required and documented.
- Tests must be self-contained: they must not depend on a locally installed
  Faust (e.g. `/usr/local/share/faust`), and copies of the Faust standard
  libraries must not be committed to the repository. When a test needs
  library-style DSP behavior, write a compact test-local Faust definition
  inline and compile it with the `compile_source_to_*` APIs (see
  `crates/compiler/tests/signal_fir_lane.rs` for the pattern).

## 4. CI Expectations

- CI runs on Linux, macOS, and Windows.
- CI stages include `cargo check`, formatting, clippy, and tests.
- CI also runs golden parity guardrails via `cargo run -p xtask -- golden-check`.
- CI also runs the compilation-cost gate via
  `cargo run --release -p xtask -- compile-budget-check`.
- A change is not considered ready unless CI is green.
- Code that constructs, normalizes, displays, or compares filesystem paths must
  be checked for cross-platform behavior. Prefer `Path`/`PathBuf` operations,
  components, or explicit display-normalization helpers over ad hoc string
  concatenation, hardcoded separators, or Unix-only assumptions.
- For filesystem path assertions in tests, compare `Path`/`PathBuf` values (or
  components) instead of stringified paths; avoid hardcoded `/` separators
  because CI runs on Windows.
- In versioned documentation, generated reports, and stored test artifacts,
  prefer repository-relative paths over absolute local checkout paths so the
  content stays portable on GitHub and across contributor machines.

## 5. Porting Discipline

- Use the `porting/` documents as source of truth for scope and phases.
- Preserve behavior first, optimize later.
- Treat local quality gates as mandatory for each porting step:
  - `cargo fmt --all`
  - `cargo clippy --workspace --all-targets -- -D warnings`
  - `cargo test --workspace --all-targets`
  - `cargo run --release -p xtask -- compile-budget-check` before any
    commit that touches the compilation pipeline (`parser`, `eval`, `propagate`,
    `normalize`, `sigtype`, `transform`, `fir`, `codegen`, `compiler`) — see
    "Compilation-cost discipline" below.
- Add or update unit tests in the touched crate(s) as part of each porting change; if tests cannot be added immediately, record the reason, owner, and planned follow-up in `JOURNAL.md`.
- Document migrated source provenance as you port: add Rustdoc comments (`///` or `//!`) that reference the corresponding C++ source files/functions and capture key invariants/semantic notes needed to maintain parity.
- Public API migration is parity-driven, not blindly signature-driven:
  - internal Rust crate APIs may be adapted for idiomatic ownership/types/error handling,
  - external compatibility surfaces (CLI + C/C++ API tiers) target stable behavior and compatibility contracts.
- When a backend also exists in C++ Faust, the **generated code must expose the
  same public contract** as the C++ Faust output for that language, so existing
  architectures and projects keep working unchanged. Example: the Rust backend
  emits the host-supplied `F32`/`F64`/`FaustFloat` types, `ParamIndex`-based
  parameter access, and the `FaustDsp` trait expected by
  `faust2jackrust -source` / `faust2portaudiorust -source` projects (contract
  documented in the `crates/codegen/src/backends/rust/mod.rs` module header).
  Validate contract-affecting emitter changes by building generated output
  inside such projects.
- For each touched public API, document mapping status (`1:1`, `adapted`, or `deferred`) with rationale and compatibility impact in the relevant `porting/` phase document or `JOURNAL.md`.
- For representation-level adaptations (`adapted`) versus C++ data layout:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grame-cncm/faust-rs](https://github.com/grame-cncm/faust-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
