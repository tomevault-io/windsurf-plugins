---
trigger: always_on
description: This file provides guidance to coding agents working in this repository. It
---

# Agent Instructions

This file provides guidance to coding agents working in this repository. It
carries the things that are true everywhere: what fatou is, the tenets, the
commands, and the cross-cutting invariants.

**Per-subsystem directives live in `.claude/rules/*.md`**, path-scoped in
frontmatter so each loads only when you read that subsystem's files: `parser`,
`oracle` (JuliaSyntax parity + the Julia version pin), `formatter`, `linter`,
`lsp`, `semantic` (semantic, resolution, project, salsa), `index` (package
index + Julia environment), `config`, `docs` (docs site + benchmarks), and
`release` (packaging, workflows, versioning).

Keep each rule file terse and under 200 lines: a rule, the one clause that keeps
it from looking arbitrary, and a pointer. A rule that must hold *before* any
file is read belongs here instead — path-scoped rules only load once a matching
file is read, and they are not re-injected after a compaction.

Worked examples and issue archaeology belong in neither file: they live in the
issue tracker, in `git log`, and above all in named tests and fixtures, which
are what fails when a rule is violated. **`TODO.md` is the live roadmap** and
records known issues and follow-ups; when in doubt about scope or priority, it
is the source of truth.

## What this project is

Fatou is a Rust CLI providing a language server, formatter, and linter for the
Julia language. It is a Cargo workspace (edition 2024) whose root package,
`fatou` (binary *and* library), hosts the CLI, LSP, linter, semantic model,
resolution, project projections, and package index, and builds on two
independently published member crates:

- **`crates/fatou-parser`** — `syntax` (SyntaxKind, node pointers), `ast` (typed
  wrappers), `parser` (lossless CST parser + incremental reparse).
- **`crates/fatou-formatter`** — the formatting engine, for embedders such as a
  dprint plugin.

The root crate re-exports the parser crate's modules, and `src/formatter.rs`
bridges the engine while hosting the CLI-side batch `check` API — so
`fatou::parser`, `fatou::formatter`, etc. stay the paths everything uses.

**Both member crates must stay `wasm32-unknown-unknown`-clean** (no filesystem,
process, thread, or clock use); a dedicated CI job is what enforces it.

Beyond the crate the repo ships the distribution surfaces: a VS Code extension
(`editors/code`), npm packages (`npm/`), a PyPI package (via maturin), an AUR
package (`packaging/aur`), the docs site (`docs/`), and benchmarks (`bench/`).

The design follows rust-analyzer, and the author's R tool **`arity`**, on which
fatou is modeled directly: lossless `rowan` CST trees, `salsa` for the
incremental database, `lsp-server` for the language-server transport, and an
event-pipeline parser built for incremental reparse.

## Tenets

1. **Deterministic, canonical formatting.** Output is decided solely by the
   formatter's rules and the layout engine, never by how the input happens to be
   written. Semantically-equivalent inputs **must** format identically: the
   input's line breaks, whitespace, operator spelling (`in` vs `∈`, `a*b` vs
   `a * b`), and numeric-literal form never influence the result. Fatou does
   **not** honor "persistent line breaks"; it **fully reflows**, laying out each
   construct from scratch under `line_width` and breaking only where width or
   semantics require it. Push back against hard-coding special cases for
   specific constructs. Any deviation from full reflow is a deliberate, recorded
   choice, never silent non-determinism.
2. **Incremental parsing is first-class**, not an afterthought. Parser/CST work
   must keep the salsa-based reparse path (`src/incremental.rs`) viable.
3. **Parsing is the parser's job.** Never paper over parser mistakes in the
   formatter, and never let parsing logic creep into the formatter. If the
   formatter hits something the parser handled wrong, fix it in the parser.
4. **Losslessness is the parser's job.** The parser preserves all text
   (whitespace, comments, and the rest) so that `reconstruct(text) == text`
   always. The formatter may assume a lossless CST and focus on layout.

Because the formatter is the **sole authority on layout**, a lint fix is not a
formatter: `lint --fix` applies each fix as a byte-range replacement and never
runs the formatter. The pipeline is fix-then-format. Full policy in
`.claude/rules/linter.md`.

**Semantics stay static** everywhere: no Julia runtime, no evaluation, at any
point in the pipeline — including the package index, which reads Julia's on-disk
layout with fatou's own parser.

## Commands

```sh
cargo build --workspace           # dev build
cargo test --workspace            # all tests (bare `cargo test` runs only the root crate!)
cargo test --workspace <substring>                   # tests matching a name
cargo test -p fatou-parser --test parser_snapshots   # one member-crate test file
cargo test --test linter_rules                       # one root-crate test file (`ls tests/*.rs`)
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo fmt --all -- --check        # keep changes rustfmt-clean
```

Subcommands are `parse`, `format`, `lint`, `lsp`, and `debug`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolars/fatou](https://github.com/jolars/fatou) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
