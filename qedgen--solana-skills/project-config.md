---
trigger: always_on
description: Guidance for Claude Code when working in this repository. This file is loaded into **every** session — keep it lean. Deep material lives in `references/` and `docs/design/`; this file orients and points.
---

# CLAUDE.md

Guidance for Claude Code when working in this repository. This file is loaded into **every** session — keep it lean. Deep material lives in `references/` and `docs/design/`; this file orients and points.

## What this is

QEDGen is a Claude Code skill for spec-driven verification of Solana programs. The `.qedspec` is the single source of truth: `qedgen check` validates it (lint + proptest + Lean), `qedgen codegen` generates downstream artifacts (Rust scaffold, Kani/proptest harnesses, Lean proofs, CI), and `#[qed(verified)]` stamps verified code. Leanstral and Aristotle fill hard proof sub-goals when escalated.

**Core loop:** intent → write `.qedspec` → `qedgen check` → iterate → `qedgen codegen --all` → fill `todo!()` → `qedgen verify`.

The UX is **agent-first**: the user interacts with the SKILL (`SKILL.md`) and agents; the `qedgen` CLI is glue between agents and artifacts, not a user-facing tool. Full CLI reference: [`references/cli.md`](references/cli.md).

## How to think about this codebase

**Escalation ladders — mechanize first, escalate only after you've tried.**

- *Proofs:* (1) mechanical → codegen template (`lean_gen_mir.rs`); (2) tractable → write the Lean directly in-session (most real proof bodies: case analysis, Mathlib lemma selection, per-handler structural proofs); (3) hard → Leanstral (`qedgen fill-sorry`); (4) last resort → Aristotle (`qedgen aristotle submit`).
- *Code/tests:* (1) mechanical → codegen template (`codegen_mir` + `codegen_shared::mechanize_effect`); (2) tractable → fill `todo!()` in-session (events, transfers, CPI wiring, complex effect RHS); (3) last resort → refine the spec (under-specification is the real bug).

**Design principles:**
- A DSL feature that *structurally eliminates* a proof obligation beats a new proof template or a shelled-out sorry.
- Codegen mechanizes only deterministic translation. Supported direct CPIs with transaction signers, Anchor/Quasar lifecycle account creation, and Anchor builder-shape CPIs with assemblable PDA signer seeds are complete; transfer sugar, events, unsupported calls, Pinocchio PDA creation, and every other PDA-signed CPI stay agent-filled `todo!()`. The per-target contract lives in `docs/framework-support.md`.
- When a template can't close a case, emit `sorry` with a comment — never bury it in tactics that might spuriously close.
- Don't pre-shell to Leanstral/Aristotle for what a local LLM can do. Escalation is for *after* you've tried, not when you expect to need to.
- The typed MIR (`mir.rs`) exists for bug-class elimination, not LoC: matches over the closed `Stmt` enum are exhaustive by discipline (no `_` arms — see the enum doc in `mir.rs`), so a new statement kind is a compile error at every `Stmt` consumer (Lean codegen, Rust scaffold, and the Kani/proptest effect lowering via `rust_codegen_util::stmt_effect_triple`), not silent drift. Conditional `effect { match … }` bodies lower to `Stmt::Branch` (Phase 5): Rust renders a `match`; both the flat and ADT Lean transitions apply exactly one arm with per-arm bound guards (ADT via `emit_adt_branch`). Caveat: the ADT abort/overflow emitters (`adt_effect_map` / `adt_bound_conds`) treat `Stmt::Branch` as a no-op, so ADT abort/overflow obligations ignore branch-arm effects (honest today only because those bodies are `by sorry`). Measure intrinsics by bugs eliminated, not lines saved.
- The closed-enum discipline applies to small enums too: never tally or gate on an enum with open-coded `.filter(|x| x.variant == …)` chains — count through an exhaustive `match` (e.g. `SeverityCounts::of`) so a new variant is a compile error at every accounting site, not a silently dropped case (#260/#270: `Severity::Error` vanished from the check summary and exit code for ten releases this way).

## Build and test

```bash
cargo build --release && cp target/release/qedgen bin/qedgen   # always copy to bin/
cargo test                                                      # Rust unit + snapshot tests
cd lean_solana && lake build                                    # Lean support library
```

Snapshot suites (`tests/{mir,kani,codegen,proptest}_snapshot.rs`) gate every fixture against checked-in references; the shared harness lives in `tests/common/mod.rs` and rebuilds `qedgen` before driving it (no stale-binary footgun). Regenerate with `UPDATE_SNAPSHOTS=1 cargo test --test <suite>`. Snapshots prove output stability, not correctness: the executable artifact gate (`tests/generated_artifact_gate.rs`, `-- --ignored`, own CI job) regenerates every bundled Anchor example, compiles all generated Rust artifacts, runs the generated unit tests and proptests, and type-checks the Kani harness via `crates/kani-compile-stub` (#294). Full command + flag reference: [`references/cli.md`](references/cli.md).

## Dogfooding → toolchain backlog (dev-mode loop)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QEDGen/solana-skills](https://github.com/QEDGen/solana-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
