---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

froGQL — a Rust graph database implementing ISO GQL path pattern matching with single-file storage. Distributed as a CLI binary (`frogql`, modelled on `sqlite3`), an embeddable library, and PyO3 Python bindings published to PyPI as `frogql`. Part of an academic research project (test names occasionally reference paper sections, e.g. `test_paper_example_s2_4`); a separate Python reference interpreter exists in a sister project but is not required to develop or use this crate.

The crate package is still named `gqlrust` for legacy reasons; the user-facing binary, Python module, and PyPI package are all `frogql`, and every environment variable is prefixed `FROGQL_`. Some internal identifiers and doc-comments still say "gqlite"; they are not part of any user surface and should be renamed opportunistically.

## Commands

```bash
# Local dev shortcuts (see `justfile`; these wrap the commands below).
# Install the runner once: `cargo install just` (or `winget install Casey.Just`).
# `just` with no args lists every recipe.
just lint        # fmt --check + cargo check + clippy -D warnings (CI parity)
just lint-fix    # rewrite formatting + apply machine-applicable clippy fixes
just fmt         # format only, no compile
just t [targets] # fast: lib unit tests + the targets you touched (~5 s)
just test        # full sweep, warmed in parallel — the pre-commit gate
just repl movies.gdb [--import-csv dir/ | --no-typecheck]   # rebuild + open REPL

# Underlying commands (what the recipes wrap):
# Full test sweep — lib unit tests + every integration target. Use the
# unqualified form so the set never drifts as tests are added:
cargo test                       # everything (lib + all tests/*.rs)
cargo test --lib                 # just the in-crate unit tests (fast)
# The sweep's wall clock is a first-execution cost macOS charges per
# freshly linked test binary (~5-8 s each, at 0% CPU, once per file), not
# the tests themselves (~5.5 s total). `cargo test` runs targets one at a
# time, so 108 of them serialise. `just test` links everything, warms all
# the binaries 16-wide in parallel, then tests — ~30 s of warming instead
# of ~30 min of waiting. Do not hand-roll `cargo test` for a full sweep;
# use the recipe, which is where that warming lives.
#
# Do NOT re-diagnose this as "macOS rescanning" without measuring — an
# earlier version of this note claimed that and was wrong.
cargo test --test runtime_test --test typecheck_test --test parser_test

# Single test / single file
cargo test --test runtime_test test_join_star_any_label -- --exact
cargo test --test parallel_edge_test          # one integration file

# Strict clippy (run before every commit)
cargo clippy --workspace --all-targets -- -D clippy::all

# Build all binaries (defaults: `repl` + `bench` features on)
cargo build --release

# Interactive REPL
./target/release/frogql movies.gdb --import-csv path/to/csv_dir/   # create + open
./target/release/frogql movies.gdb                                 # open existing
./target/release/frogql movies.gdb --no-typecheck                  # skip typecheck for the session
./target/release/frogql movies.gdb --no-auto-indexes               # skip the secondary-index auto-build at open

# Python bindings (builds cdylib, installs `frogql` into the active venv)
cd python && source <your-venv>/bin/activate && pip install maturin && maturin develop --release
# For wheels to ship to other machines:
cd python && maturin build --release   # output in target/wheels/

# Browser WASM bindings (frogql-wasm)
rustup target add wasm32-unknown-unknown            # one-time
cargo test -p frogql-wasm                           # host tests of the engine core (query_json/dm_json)
cargo build -p frogql-wasm --target wasm32-unknown-unknown
# Generate the publishable npm package (web target); release-wasm.yml runs the same:
cargo install wasm-pack && wasm-pack build wasm --target web --out-dir pkg
```

### Diagnostic env vars

Runtime/store toggles for A/B testing and tracing (all read at query/open time; set to `1` unless noted). Each optimization has a kill switch so a differential test can pin "optimized ≡ baseline". User-facing tutorial with measured costs per mode: `docs/modes-options.md`.

| Var | Effect |
|---|---|
| `FROGQL_LTJ_SOURCE=build` | ignore `<db>.ltj` and rebuild the LTJ index from the graph (the kill switch the persistence differential test A/Bs against) |
| `FROGQL_DISABLE_LTJ_DELTA` | drop the cached LTJ index after every DML instead of maintaining a delta beside it — the kill switch `tests/ltj_delta_test.rs` A/Bs against |
| `FROGQL_LTJ_PERSIST=0` | build the LTJ index but do not write `<db>.ltj`. The index is written automatically the first time a database is opened without a usable sidecar; this declines the file, not the index |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pleiad/frogql](https://github.com/pleiad/frogql) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
