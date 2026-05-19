---
trigger: always_on
description: - Use `ai-slop-guard` for every task that produces user-facing prose.
---

# Repository Guidelines

## Mandatory Skills
- Use `ai-slop-guard` for every task that produces user-facing prose.
- Use `crimee-audit` after any task that creates or modifies code.
- Use `semantic-authority-audit` for macros, code generation, introspection, schema or policy extraction, docs generation, static analysis, or any work claiming exact, authoritative, exhaustive, or source-of-truth semantics.

## Semantic Authority Rule
Treat words like `exact`, `source of truth`, `authoritative`, `exhaustive`, `no drift`, and `exact branch alternatives` as proof obligations.
- Before shipping such a feature, state the observation point it actually uses: raw source, parsed AST, cfg-pruned AST, expanded items, type-checked items, runtime values, or persisted state.
- If the observation point is weaker than the public claim, fail closed or narrow the claim.
- Happy-path tests are not sufficient. Add adversarial cases that differ across observation stages: `#[cfg]`, macro-generated items, `include!`, duplicate-id pressure.
- Closeout must state: claimed authority surface, actual observation point, unsupported cases rejected or still open, adversarial tests added.

## Workspace Layout
Rust workspace (`resolver = "2"`) with four published-or-internal crates:
- `statum/` - public API facade. Re-exports `statum-core` types and `statum-macros` proc macros. Has `strict-introspection` feature that toggles `statum-macros/strict-introspection`.
- `statum-core/` - runtime types (`Error`/`Result`, projection, introspection registry via `linkme`).
- `statum-macros/` - proc-macro crate (`#[state]`, `#[machine]`, `#[transition]`, `#[validators]`). See `statum-macros/ARCHITECTURE.md` for required subsystem layout and dependency direction (`source/` must not depend on `state`/`machine`/`transition`/`validators`).
- `statum-examples/` - `publish = false`. Non-obvious split:
  - `src/toy_demos/` and `src/showcases/` are library modules.
  - `src/bin/*.rs` are runnable binaries that call into showcases.
  - There are **no `[[example]]` targets**; do not use `cargo run --example`.

`benchmarks/compile/{plain,statum}-fixture` are deliberately `exclude`d from the workspace - do not treat them as workspace members or add them to CI paths.

## Toolchain
- `rust-toolchain.toml` pins `stable`. Workspace `rust-version = "1.93"` is the real MSRV and is enforced in CI via a `1.93.1` job.
- Edition varies per crate: `statum` and `statum-core` use `edition = "2021"`, `statum-macros` and `statum-examples` use `edition = "2024"`. Keep that when adding new code.
- `Cargo.lock` is gitignored (this repo ships only libraries).

## Commands
Run locally the same gates CI runs, in this order:

```
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
bash scripts/check_readme_links.sh
cargo test --workspace --all-features
bash scripts/check_workspace_hygiene.sh
RUSTDOCFLAGS="-D warnings" cargo doc --workspace --all-features --no-deps
```

Focused variants:
- Macro UI/compile tests only: `cargo test -p statum-macros`.
- Single example binary: `cargo run -p statum-examples --bin axum-sqlite-review` (see `statum-examples/src/bin/` for the list).
- Compile-time benchmark: `bash scripts/benchmark_compile.sh [--mode cold|warm|both]`.
- Runtime benchmark: `cargo bench -p statum --bench runtime`.

CI also runs `cargo modum check --root . --mode warn` (requires `modum` installed; see `[package.metadata.modum]` in the per-crate `Cargo.toml`s) and `cargo deny check` / `cargo audit`.

## `strict-introspection` Feature
`statum-macros/tests/macro_errors.rs` gates different trybuild fixture sets on `cfg(feature = "strict-introspection")`. When adding a UI test:
- Register the `.rs` path explicitly in `macro_errors.rs` under the right `#[cfg]` arm. Files under `tests/ui/` are **not** auto-picked.
- Match the filename prefix to intent: `invalid_*` -> `compile_fail`, `valid_*` -> `pass`, `strict_*` -> strict-only arms.
- `.stderr` expectations are required for `compile_fail` cases. Regenerate with `TRYBUILD=overwrite cargo test -p statum-macros`.
- Run the suite once per feature flag combination you touch: plain and `--features strict-introspection`.

## Macro Development Rules
- Respect the dependency direction in `statum-macros/ARCHITECTURE.md`: emission code must not perform fresh source scans; diagnostics must not broaden fail-closed behavior.
- Prefer type-driven validation (state data type) over function-body inspection.
- Macro diagnostics must name the relevant state enum / machine so errors are attributable (e.g. "TaskState", not "this enum").
- Favor deterministic macro behavior even if it costs extra scanning or caching.
- `statum-macros/wip/` is scratch space. `scripts/check_workspace_hygiene.sh` fails CI if anything other than `.gitignore` lives there; clean it before committing.

## Publishing
`scripts/check_publish_dry_run.sh` enforces publish order: `statum-core -> statum-macros -> statum`. All three share `version.workspace = true` from root `Cargo.toml`; bump there, not per-crate. Use `scripts/update_version.rs` (via `cargo script`).

## Testing Notes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eboody/statum](https://github.com/eboody/statum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
