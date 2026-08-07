---
trigger: always_on
description: <!-- Maintainer notes. Block-level HTML comments are stripped before this file enters an agent's context:
---

<!-- Maintainer notes. Block-level HTML comments are stripped before this file enters an agent's context:

- Claude Code reads CLAUDE.md, not AGENTS.md; CLAUDE.md pulls this file in with `@AGENTS.md`.
- Everything here loads into every session. Keep it short and keep it INSTRUCTIONS; explain a rule's why only when the why changes what you do.
- Material needed only inside one area belongs in .claude/skills/ or docs/, never here.
-->

# AGENTS.md

Agent contract for dewasm. Project docs are written in English; `tests/spec` is an upstream submodule — never edit it.

## Development environment

Rust toolchain is pinned by `rust-toolchain.toml` (stable); plain `cargo` commands pick it up. Required tools/setup for the test suite (ruby >= 3.4, bash >= 5, the spec submodule, the `apps` cache) and the fail-loud-not-skip policy behind it (ADR-15) are documented in [`docs/testing.md`](docs/testing.md) — read it before wondering why a test panics with a setup instruction instead of skipping.

## Common commands

| Command | What it does |
| --- | --- |
| `cargo test` | **The baseline check for every change**: unit + e2e + curated spec harness. The spec harness is a libtest-mimic test (one trial per `.wast` file); each backend crate owns its own conformance suites (ADR-27), only cross-backend tests live in `dewasm-cli`. |
| `cargo fmt --check` | Verify Rust code formatting. |
| `cargo clippy --all-targets -- -D warnings` | Run linter on all targets, failing on any warnings. |
| `cargo test -p dewasm-backend-ruby --test spec i32` | Spec harness on `.wast` files whose name matches (cargo's built-in test-name filter — substring, add `--exact` for one file). Swap the crate (`-p dewasm-backend-bash`) to switch language. |
| `cargo test -p dewasm-backend-bash --test spec -- --include-ignored` | Full-testsuite run for bash (curated files are the default; the rest are `#[ignore]`d trials); trials run in parallel. Use `-- --ignored` to run only the non-curated files. |
| `cargo test -p dewasm-backend-ruby --test convert` | Whole-cache convert suite (ADR-54): converts every cached app with the backend, no run. One trial per app (name = cache stem); `ruby`/`cpython` are `slow_test`-conditional. Swap the crate to switch language. |
| `cargo run -p dewasm-cli -- input.wasm --mode standalone -o out.rb` | Convert; `.wat` input works too, `-o -` for stdout. |
| `cargo xtask bench [filter]` | Run the cross-runtime benchmark suite: every backend, wasmtime and the other native runtimes (wasmer, wasmedge, wazero, wasm3), and the pywasm/wardite interpreters, over the `benchmarks/wat/` and `benchmarks/c/` microbenchmarks and the app cases. Writes a dated result file to `benchmarks/results/` and regenerates `docs/benchmarks/results.md` together with the SVG charts it embeds (`docs/benchmarks/figs/`, one per workload; charts the record no longer covers are pruned) — a measurement, not a snapshot, so no freshness test guards it. `--list` prints the matrix and each runner's availability without running anything; `--reps`/`--target-ms`/`--timeout` tune the measurement; `--render benchmarks/results/<file>.json` regenerates the doc from a stored record, since a full benchmark run takes tens of minutes and a wording change must not require re-measuring. Needs `wasmtime` plus `benchmarks/wat/build.sh`, `benchmarks/c/build.sh` and `benchmarks/setup.sh`; anything missing is reported as skipped-with-reason. |
| `examples/apps/setup.sh` | Fetch/build the pinned real-world apps (cowsay, QuickJS, the three sqlite3 shapes, minigzip, ripgrep, CPython, CRuby plus its wasi-vfs-packed shape) into the gitignored cache; needs a few build tools on PATH (`zig`, the `wasm32-wasip1` rustup target, `wasi-vfs` — see docs/testing.md). Enables the `apps` cases of the `e2e` test. |

## Verification


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dewasm/dewasm](https://github.com/dewasm/dewasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
