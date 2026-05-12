---
trigger: always_on
description: jepa-rs is a Rust Cargo workspace for JEPA primitives, strict image and video reference paths, checkpoint and ONNX interoperability, plus CLI/TUI and browser demo surfaces built on burn.
---

<identity>
jepa-rs is a Rust Cargo workspace for JEPA primitives, strict image and video reference paths, checkpoint and ONNX interoperability, plus CLI/TUI and browser demo surfaces built on burn.
</identity>

<stack>

| Layer | Technology | Version | Notes |
|-------|------------|---------|-------|
| Runtime | Rust toolchain | stable, MSRV 1.85 | CI uses stable; nightly only for `fuzz/` |
| Language | Rust | 2021 edition | `rust-version = 1.85` in workspace manifests |
| Workspace | Cargo workspace | resolver 2 | 7 member crates plus nested fuzz workspace |
| ML framework | burn | 0.20.1 | `autodiff` enabled workspace-wide |
| CPU backend | burn-ndarray | 0.20.1 | Default test and demo backend |
| GPU/WebGPU backend | burn-wgpu | 0.20.1 | Present as workspace dependency |
| Serialization | serde, serde_json | 1.x | Configs and metadata |
| Errors | thiserror | 2.x | Library crates |
| Checkpoint format | safetensors | 0.7.0 | `jepa-compat` |
| ONNX runtime | tract-onnx | 0.23.0-dev.2 | `jepa-compat` runtime path |
| CLI | clap | 4.x | `crates/jepa` |
| TUI | ratatui, crossterm | 0.29, 0.28 | `crates/jepa` |
| Testing | cargo test, proptest | 1.x | Unit, integration, doc, property tests |
| Benchmarks | criterion | 0.8.2 | Per-crate benches |
| Fuzzing | cargo-fuzz, libfuzzer-sys | nightly, 0.4.12 | Separate `fuzz/` workspace |

</stack>

<structure>
```text
crates/
├── jepa-core/      # Shared contracts, semantic tensor wrappers, masking, energy, config [agent: gated for public API files]
├── jepa-vision/    # ViT, IJepa, VJepa, strict image/video reference paths [agent: create/modify]
├── jepa-train/     # Generic training orchestration and schedules [agent: create/modify]
├── jepa-world/     # Planning, memory, hierarchical world-model helpers [agent: create/modify]
├── jepa-compat/    # safetensors, key mapping, ONNX metadata/runtime [agent: create/modify]
├── jepa/           # CLI binary, demos, and ratatui dashboard [agent: create/modify]
└── jepa-web/       # Browser demo crate with JS assets and WASM exports [agent: create/modify]
.github/workflows/  # CI gates and release smoke checks [agent: gated]
docs/               # Project docs [agent: create/modify]
docs/agentic/       # Living agent memory: decisions and lessons learned [agent: create/modify]
fuzz/               # Separate cargo-fuzz workspace and fuzz targets [agent: modify with care]
scripts/            # Parity and export scripts [agent: gated]
specs/differential/ # Strict I-JEPA parity fixtures and fixture tooling [agent: gated]
target/             # Generated build outputs and demo artifacts [agent: never edit]
README.md           # Public project overview [agent: create/modify]
CONTRIBUTING.md     # Dev workflow and commit convention [agent: create/modify]
CHANGELOG.md        # Release history [agent: create/modify]
docs/ARCHITECTURE.md    # Architecture map, invariants, and boundary notes [agent: create/modify]
docs/QUALITY_GATES.md   # Exact verification commands and escalation notes [agent: create/modify]
docs/RELEASE.md         # Release checklist and current publish constraints [agent: create/modify]
docs/ROADMAP.md         # Near-term milestones with exit criteria [agent: create/modify]
docs/PRODUCTION_GAPS.md # Honest blocker register for production readiness [agent: create/modify]
Cargo.toml          # Workspace membership and shared dependency versions [agent: gated]
Cargo.lock          # Locked dependency graph [agent: gated]
```

Module boundaries:
- `jepa-core` is the contract layer. All library crates depend on it.
- `jepa-vision`, `jepa-train`, `jepa-world`, and `jepa-compat` are library layers on top of `jepa-core`.
- `crates/jepa` is the only binary crate and depends on all other workspace crates.
- `crates/jepa-web` is a browser demo crate that reuses the model crates; the exported path currently runs on the CPU-backed WASM backend and keeps the WebGPU path internal.
- `fuzz/` is not a normal workspace member. Treat it as a separate nightly-only validation surface.
</structure>

<commands>

| Task | Command | Notes |
|------|---------|-------|
| Workspace check | `cargo check --workspace --all-targets` | Fastest whole-repo compile gate |
| Workspace tests | `cargo test --workspace` | Runs unit, integration, and doc tests |
| Core-only tests | `cargo test -p jepa-core` | Use before wider workspace runs |
| Vision tests | `cargo test -p jepa-vision` | Includes strict-path unit tests |
| Compat tests | `cargo test -p jepa-compat` | safetensors, ONNX, keymap coverage |
| CLI/TUI tests | `cargo test -p jepa` | Clap parsing and demo helpers |
| Browser demo tests | `cargo test -p jepa-web` | WASM-facing config and inference boundary coverage |
| Clippy | `cargo clippy --workspace --all-targets -- -D warnings` | Warnings are errors in CI |
| Format check | `cargo fmt -- --check` | Use `cargo fmt` only to apply formatting |
| Strict parity | `scripts/run_parity_suite.sh` | Runs all bundled strict image fixtures |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbdelStark/jepa-rs](https://github.com/AbdelStark/jepa-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
