---
trigger: always_on
description: `fslc` is the verifier for FSL, an AI-native formal specification language. The native Rust
---

# Repository Guidelines

## Project and authority

`fslc` is the verifier for FSL, an AI-native formal specification language. The native Rust
workspace under `rust/` is the authoritative implementation and distribution surface, including
the `fslc-lsp` language server. The Python package under `src/fslc/` is a frozen compatibility
reference; do not add product behavior there unless a compatibility change explicitly requires it.

Interpret evidence in this order:

1. Language and CLI contracts in `docs/LANGUAGE.md`, accepted `docs/DESIGN-*.md`, tests, and CI.
2. The native Rust implementation and its public Kernel/JSON contracts.
3. Observable behavior of the frozen Python reference where a parity contract applies.
4. Proposals, task notes, conversation history, and agent memory.

When sources disagree, stop and resolve the contract conflict instead of silently choosing one.

## Project structure

- `rust/fsl-syntax`: lexer, parsers, source locations, and surface AST.
- `rust/fsl-core`: typed kernel model, validation, resolution, and dialect lowering.
- `rust/fsl-runtime`: solver-independent Monitor and explicit-state/BFS behavior.
- `rust/fsl-solver*`: backend-neutral solver boundary plus native and browser Z3 backends.
- `rust/fsl-verifier`: BMC, induction, refinement, liveness, and scenarios.
- `rust/fsl-tools`: analysis, mutation, report, typestate, and test generation tools.
- `rust/fslc`: native CLI and JSON/process contract.
- `rust/fsl-wasm`: browser Worker surface.
- `rust/fsl-lsp`: native language server and document index.
- `src/fslc`: frozen Python compatibility reference.
- `tests`: Python-driven Rust contract, parity, and compatibility tests.
- `specs` and `examples`: FSL corpus and reproducing cases.
- `skills`: canonical agent skills; `.claude/skills/fsl*` and `.agents/skills/fsl*` symlink here.

## Build and verification commands

Run the native CLI from the working tree:

```bash
cargo run --manifest-path rust/Cargo.toml -p fslc-rust --bin fslc -- check specs/cart_v1.fsl
cargo run --manifest-path rust/Cargo.toml -p fslc-rust --bin fslc -- verify specs/cart_v1.fsl --depth 8
cargo run --manifest-path rust/Cargo.toml -p fslc-rust --bin fslc -- verify specs/cart_v1.fsl --engine induction
```

Run the narrowest relevant check first. The Rust CI-equivalent gate is:

```bash
cargo fmt --manifest-path rust/Cargo.toml --all -- --check
cargo clippy --manifest-path rust/Cargo.toml --workspace --all-targets --locked -- -D warnings
cargo test --manifest-path rust/Cargo.toml --workspace --locked
cargo build --manifest-path rust/Cargo.toml --workspace --locked
```

The complete required product gate has one Rust-native entrypoint and does not execute Python:

```bash
./tools/check-native-integration.sh
```

Pull requests into `main` may use the bounded `merge readiness` gate defined in
`docs/DESIGN-ci.md`; it is not product verification. Every merged `main` state and every
production/release promotion must still receive the complete product evidence. Do not hide a
post-merge product-gate failure or treat its automatically created issue as a waiver.

Python is optional and is used only for changes explicitly scoped to the frozen compatibility
reference or Python-based repository hooks. Native solver changes should also run
the focused `fsl-solver-z3`, `fsl-verifier`, and `fslc-rust` tests.
Changes to concrete/symbolic semantics additionally run
`./tools/check-native-integration.sh fsl-logic pr`; generator, comparator,
inventory, and promotion changes run the `scheduled` tier.

## Correctness invariants

- `fsl-runtime` must remain independent of `fsl-solver`, Z3, and JavaScript solver bridges.
- Symbolic verification, the concrete Monitor, and solver-free BFS must agree. A confidently green
  false negative is more dangerous than a crash.
- Native CLI and Worker output must preserve the JSON envelope, exit codes, locations, and replayable
  evidence contract. Do not allowlist verdict, location, assurance, or exit-code differences.
- A language feature moves with its grammar/lowering, typed model, symbolic and concrete semantics,
  regression cases, `docs/LANGUAGE.md`, `docs/LANGUAGE.ja.md`, `skills/fsl/reference.md`, a design
  note, and a `changelog.d/` fragment (see `changelog.d/README.md`; `CHANGELOG.md`'s `[Unreleased]`
  body itself is aggregated from fragments at release time and must not be hand-edited). A new
  declaration, binder, or reference form additionally moves with
  `rust/fsl-lsp/src/index.rs` and a targeted role/scope test, or it silently loses
  definition/references/rename/documentSymbol with no parse failure to surface the gap;
  `rust/fsl-lsp/tests/corpus.rs` only asserts that every identifier is indexed as something.
  `docs/LANGUAGE.ja.md` is a second canonical source kept section-aligned
  1:1 with `docs/LANGUAGE.md` (same count/order of `## ` sections) — `tools/build_site_reference.py`
  fails loudly on drift; see `docs/DESIGN-docs-site.md` D7 (this one *is* a required CI check:
  `.github/workflows/site-reference-freshness.yml`). A new dialect's top-level construct (and any new
  `examples/`/`specs/` directory) additionally moves with `tests/dialect_registry.py` (`DIALECTS`,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ymm-oss/fsl](https://github.com/ymm-oss/fsl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-10 -->
