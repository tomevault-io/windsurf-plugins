---
trigger: always_on
description: Oscen is a Rust library for audio software: a graph-based DSP engine where
---

# AGENTS.md

Oscen is a Rust library for audio software: a graph-based DSP engine where
nodes (oscillators, filters, envelopes, effects) connect through typed
endpoints, plus a `graph!` macro DSL that compiles declarative graph
definitions into static Rust code.

## Workspace layout

- `oscen-lib/` — the main `oscen` crate: DSP nodes, graph runtime,
  MIDI, voice allocation, frames, assets, offline rendering, resampling,
  etc. Uses `#![feature(inherent_associated_types)]` and the nightly pinned
  in `rust-toolchain.toml` (currently **nightly-2026-01-22**).
- `oscen-graph-compiler/` — the `graph!` DSL compiler (parse → AST → IR →
  codegen), a plain library crate so tooling besides the proc macro can use
  it. Diagnostics are accumulated, not fail-fast. `manifest.rs` handles
  endpoint manifests and wildcard-hoist expansion.
- `oscen-macros/` — proc macros: `graph!`, `#[derive(Node)]`,
  `oversample_variants!`, plus the hidden wildcard-hoist continuation.
  Delegates graph compilation to `oscen-graph-compiler`; node derive lives
  here. `tests/ui/` contains trybuild fixtures and `.stderr` snapshots.
- `examples/` — `oscen-examples` bin crate (`examples/src/bin/*.rs`, run with
  `cargo run -p oscen-examples --bin <name>`), plus separate workspace crates:
  `fm-synth` (NIH-plug + Slint, also a standalone bin), `nih-twin-peaks`
  (package `twin-peak-nih`) and `simple-echo` (NIH-plug), `electric-piano`
  and `pivot` (Slint apps), `oversampled-saturator` (CPAL app).
  Read the README in an example dir before editing it, if present.
- `xtask/` — NIH-plug bundler: `cargo xtask bundle <plugin> --release`
  produces VST3/CLAP in `target/bundled/`.
- `docs/COOKBOOK.md` — practical graph, parameter, event, and plugin idioms.
  Other files in `docs/` are plans/review notes, not authoritative API docs.
  Check implementation and tests before relying on their status claims:
  `ERGONOMICS_PLAN.md` describes `poly::<N>` as shipped, but this checkout
  uses explicit allocator/handler/voice-array wiring; `BLOCK_FISSION_SPEC.md`
  is a draft, not the current block scheduler.

## Build & test

```bash
cargo build --workspace                 # all crates, including plugin/UI examples
cargo test -p oscen                      # library unit, integration, and doc tests
cargo test -p oscen --test golden_render  # bit-exact render regression tests
cargo test -p oscen --test realtime_safety # allocation checks (keep debug mode)
cargo test -p oscen-graph-compiler        # parser/IR/codegen/manifest tests
cargo test -p oscen-macros               # macro integration + trybuild UI tests
cargo test -p oscen-macros --features nih-plug --test nih_params_test
cargo xtask bundle fm-synth --release    # build a plugin
cargo bench -p oscen                     # per_sample, graph_blocks, synth_app
```

- The toolchain is pinned to stabilize trybuild diagnostics. Bump it
  deliberately and review regenerated `.stderr` snapshots.
- Plugin examples, macro dev-dependencies, and `xtask` pull NIH-plug from
  GitHub (both upstream and a fork); first dependency fetch needs network.
- Codegen snapshots: `oscen-graph-compiler/tests/snapshots/*.tokens`.
  After an intentional codegen change, re-bless with
  `OSCEN_UPDATE_SNAPSHOTS=1 cargo test -p oscen-graph-compiler --test codegen_snapshot`,
  then review the snapshot diff before committing.
- Compile-fail snapshots: after intentional diagnostic/toolchain changes,
  run `TRYBUILD=overwrite cargo test -p oscen-macros --test parse_rate_test`,
  review `oscen-macros/tests/ui/*.stderr`, then rerun without the variable.
- Golden render hashes: run
  `OSCEN_PRINT_GOLDEN=1 cargo test -p oscen --test golden_render -- --nocapture`
  after intentional DSP changes, then **manually update the constants** in
  `oscen-lib/tests/golden_render.rs` and justify the audio change in the
  commit message. Printing hashes bypasses assertions; rerun normally.
- Criterion baselines: save with
  `cargo bench -p oscen --bench synth_app -- --save-baseline <name>` and
  compare with `cargo bench -p oscen --bench synth_app -- --baseline <name>`.
  Bench results are only trustworthy when nothing else compiles concurrently.
  Whole-app graphs shared by benches and golden tests live in
  `oscen-lib/benches/support/poly_synth.rs`.

## Conventions & invariants

- **Real-time safety is a hard requirement.** No heap allocation, locking, or
  blocking in the audio path (`process()`, `process_block()`, and anything
  they call). `oscen-lib/tests/realtime_safety.rs` checks allocations with
  `assert_no_alloc` in debug mode; add coverage when touching hot-path code.
  Prefer `arrayvec`, `rtrb`, `arc-swap`, and preallocated buffers over `Vec`
  growth/`Mutex`. Asset preparation and publishing belong off the audio thread.
- Graph endpoints are typed (`stream`, `value`, `event`, plus `asset` bindings
  from `external` declarations). Connection type, rate, and array-size
  mismatches must be reported as useful compiler diagnostics, not panics.
  When changing the DSL, update parser recovery (`parse_recovery.rs`),
  diagnostic accumulation tests, and relevant trybuild fixtures alongside.
- Stream fan-in sums; multiple sources into a value endpoint are an error.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [reedrosenbluth/oscen](https://github.com/reedrosenbluth/oscen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
