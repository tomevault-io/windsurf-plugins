---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
cargo build                          # build the whole workspace
cargo test                           # run all tests
cargo test -p autosar-data <name>    # run a single test by (partial) name
cargo clippy -- -D warnings          # lint; CI fails on any clippy warning
cargo bench -p autosar-data          # criterion benchmarks
cargo run --example demo             # examples: demo, businfo, generate_files, sort
```

The `load_file` and `build_model` benchmarks work on arxml files given by the caller. `load_file`
measures parsing (the file content is read into memory before the measurement, so it measures
parsing, not disk IO); `build_model` loads a file as a setup step and then measures rebuilding an
identical model element by element through the public creation API (`create_sub_element` etc.,
deliberately not `duplicate`/`create_copied_sub_element`), which shows the hotspots of building a
large model from scratch. Both take their input files in the `AUTOSAR_BENCH_FILES` environment
variable, separated by the platform path separator, because criterion consumes the command line of
the benchmark binary itself:

```sh
AUTOSAR_BENCH_FILES=/path/to/big.arxml cargo bench -p autosar-data --bench load_file
AUTOSAR_BENCH_FILES=/path/to/big.arxml cargo bench -p autosar-data --bench build_model
```

`AUTOSAR_BENCH_STRICT` (default: false) and `AUTOSAR_BENCH_SAMPLES` (default: 10) can also be set;
the shared input handling lives in `benches/common/mod.rs`. Without `AUTOSAR_BENCH_FILES` these
benchmarks print a usage note and do nothing.

`cargo bench -p autosar-data-specification --bench regex` is self-contained (no input files) and
does two things. Before measuring anything it **verifies** every validator in `regex.rs` against the
pattern in its doc comment: `benches/common/cases.rs` lists the patterns, and `verify()` compiles
them with the `regex` crate (a dev-dependency) and compares over ~1.3 million generated inputs - all
strings over a per-pattern alphabet up to a given length, every single character edit of every
sample, random strings, and random splices of samples. Any change to `regex.rs`, including a
regeneration, should be run through it. Then it benchmarks every validator (group `patterns`) and
compares the hot identifier and path validators against alternative implementations (groups
`regex_7`, `regex_8`, `regex_22`, `regex_24` and `mixed`, from `benches/common/candidates.rs`). The
validators are `pub(crate)`, so `benches/regex.rs` includes `src/regex.rs` with `#[path]` instead of
going through the public API. `AUTOSAR_REGEX_SKIP_VERIFY=1` skips the verification, which is needed
to benchmark an older revision that the check would reject.

`cargo run --example generate_files` (run it in a scratch directory - it writes to the cwd) produces
one arxml file per Autosar version that covers nearly the whole metamodel; these are useful inputs
for both benchmarks when no real arxml file is at hand.

Coverage (as run in CI): `cargo llvm-cov --all-features --workspace`

Tests live inline in the source files as `#[cfg(test)]` modules, not in separate `tests/` directories.

## Workspace structure

Two crates; `[patch.crates-io]` in the root `Cargo.toml` makes `autosar-data` build against the local `autosar-data-specification` instead of the published version.

### autosar-data-specification

A `no_std` crate containing the combined AUTOSAR metamodel of all 22 AUTOSAR 4 XSD revisions (4.0.1 through R25-11) as static Rust data tables.

- `specification.rs`, `elementname.rs`, `enumitem.rs`, `attributename.rs`, and `regex.rs` are **generated code** (from parsing the AUTOSAR .xsd files, by an external generator not in this repo). Do not hand-edit them except when regeneration is impossible; changes to the metamodel come from regeneration. Exception: every validator in `regex.rs` is hand-written (the generated DFA state tables were removed) and the generator is expected to emit these forms - preserve them across a regeneration, and check the result with the `regex` benchmark, which compares each validator against its pattern.
- The central type is `ElementType`, an index into the static tables. `ElementType::ROOT` is the type of the `<AUTOSAR>` root element.
- Version applicability is encoded as `u32` bitmasks, one bit per `AutosarVersion`. APIs like `find_sub_element(name, version_mask)` filter by these masks.
- `ElementName` ↔ `ElementType` is an n:m mapping — never assume a name determines a type or vice versa. Name lookup uses a generated perfect hash (`hashfunc`).
- Feature `docstrings` (off by default) compiles in element documentation strings.

### autosar-data

The user-facing API for reading, modifying, and writing arxml files. All data is validated against the specification crate on load and on every mutation.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DanielT/autosar-data](https://github.com/DanielT/autosar-data) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
