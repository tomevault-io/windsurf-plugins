---
trigger: always_on
description: Four rules that bias toward caution over speed. For trivial mechanical edits, use judgment.
---

# Repository Guidelines

## Working Discipline

Four rules that bias toward caution over speed. For trivial mechanical edits, use judgment.

**Think before coding.** State assumptions explicitly instead of building on them silently. When a task admits several readings, name them and choose openly; when a simpler approach exists than the one requested, say so before implementing. Confusion is a finding, not an obstacle: working interactively, ask; working unattended, record the open question instead of guessing. For platform behavior this rule has hard machinery — 1C semantics are measured, never inferred, and an unmeasured decision carries a `НЕ ИЗМЕРЕНО` marker with its registry entry and measure line (see "Measuring on the 1C platform" below). Some divergences from the platform are deliberate decisions; check the comments and history at the site before "fixing" one back.

**Simplicity first.** Write the minimum code that solves the problem: no speculative features, no configurability nobody asked for, no abstractions for single-use code, no handling of impossible errors. If the diff comes out several times larger than the task suggests, rewrite it before submitting. The dependency rule below is this principle applied to external crates, and it cuts both ways — reuse what the workspace already has rather than writing a second copy; the measured inline budget around the VM dispatch loop (see the comments on `step_cold` in `bsl-vm`) is the same principle applied to hot code, where size itself is a cost.

**Surgical changes.** Every changed line should trace to the task at hand. No drive-by reformatting, renaming, or "improving" of adjacent code — match the surrounding style, including the Russian comment prose, even where you would write it differently. Remove imports and helpers your change orphaned; leave pre-existing dead code in place and mention it instead. Files owned by machinery are not edited as a side effect: `НЕ ИЗМЕРЕНО` markers move only together with their measurement.

**Goal-driven execution.** Before implementing, turn the task into a check that can fail: a bug fix starts from a test or fixture that reproduces it, a compatibility change from measured platform output, a refactor from the suite that must stay green — plus `the_optimizing_passes_agree_with_the_plain_run_on_every_script` for compiler optimization work and an alternating A/B run against the baseline binary for any performance claim. For multi-step work, state a short plan with a verification per step. A task whose success criterion cannot be named is not understood yet; resolving that comes first.

## Project Structure & Module Organization

This repository is a Rust 2024 workspace implementing a BSL interpreter. Crates follow the execution pipeline:

- `crates/bsl-syntax`: lexer, parser, AST, and diagnostics.
- `crates/bsl-sema`: name resolution and semantic representation.
- `crates/bsl-bytecode`: bytecode instructions and the textual bytecode format (`text.rs`) used by both `--emit-bytecode` and `--run-bytecode` — printing and parsing share one format, so adding an instruction means touching `write_instr`, `parse_instr`, `OPCODES`, and the round-trip corpus (now `crates/bsl-compiler/tests/text_round_trip.rs`) together, and bumping `FORMAT_VERSION` if the encoding changes. The crate holds the representation only: it depends on neither `bsl-syntax` nor `bsl-sema`, in normal *or* dev dependencies, so its own tests build programs by hand (`tests/support`) instead of compiling BSL. `bundle.rs` marks VLIW bundles — runs of mutually independent neighbor instructions (no RAW/WAW inside a bundle, WAR allowed) that the VM executes in one dispatch; the classification it runs on lives in `analysis.rs` (see below). `Chunk::bundle_len` is a derived table and is never serialized: the parser recomputes it (listings only show it as `; бандл N` comments), and `crates/bsl-cli/tests/bundles.rs` re-verifies the invariants over the whole conformance corpus with `bundle::verify`.
- `crates/bsl-compiler`: code generation from `bsl-sema`'s representation into `Program`, plus `compile_dynamic_snippet` — the whole front end of `Выполнить`/`Вычислить` behind the neutral `bsl_bytecode::DynamicCompiler` contract. It sits between the front end and the representation so that `bsl-vm` can depend on the representation alone; `cargo tree -p bsl-vm -e normal` must not show `bsl-syntax` or `bsl-sema`.
- `crates/bsl-rt`, `bsl-number`, and `bsl-format`: runtime values, decimal arithmetic, and BSL formatting. `BslValue::Display` is debug-only and does not reproduce 1C formatting — use `bsl_format::format_value` for any user-visible or conformance-checked text (it backs `Строка`/`Формат` and the CLI).
- `crates/bsl-vm`: bytecode execution; examples live in `examples/`.
- `crates/bsl-cli`: script runner, REPL (syntax highlighting in `highlight.rs`, Tab completion in `complete.rs`), and end-to-end conformance runner.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dobpilot/open-bsl](https://github.com/dobpilot/open-bsl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
