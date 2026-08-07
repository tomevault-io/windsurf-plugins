---
trigger: always_on
description: Guidance for AI agents working with Badness, a formatter, linter, and language
---

# AGENTS.md

Guidance for AI agents working with Badness, a formatter, linter, and language
server for LaTeX.

This file is the **rules** you work under: the tenets, the load-bearing
architectural decisions (stated tersely), and the invariants and conventions to
respect. The **why**—worked examples, issue provenance, and the full catalog of
statically-recognized patterns—lives in the book's Development section, which is the
source of truth for design detail:

- Architecture (`docs/src/development/architecture.md`)
- Parser & lexer modes (`docs/src/development/parser.md`)
- Formatter (`docs/src/development/formatter.md`)
- Linter (`docs/src/development/linter.md`)
- LSP & environment awareness (`docs/src/development/lsp.md`)

When a decision below changes, update both this file (the rule) and the relevant
Development page (the detail). Extended roadmap rationale is threaded through TODO.md.

## What this project is

Badness follows **rust-analyzer's** architecture: a generic, error-tolerant,
hand-written parser produces a **lossless concrete syntax tree (CST)**; semantics are
layered on top as a separate concern; recomputation is incremental via salsa. On the
CST sit a **formatter** (`badness format`), a **linter** (diagnostics), and a
**language server** (LSP). (We were also inspired by
[arity](https://github.com/jolars/arity), the same kind of tool for R.)

It is a **four-crate Cargo workspace** (edition 2024) whose root package is the
CLI/LSP/linter crate `badness`, with two publishable, **wasm-clean** library
crates and one non-published wasm shim under `crates/`:

- **`badness-parser`** — `syntax`, `ast`, `parser`, `semantic` (minus the
  disk/salsa-backed `load`), the BibTeX parsing+semantic layers, the `data/`
  signature artifacts, and the phf codegen `build.rs`.
- **`badness-formatter`** — the layout engine (`core`, `ir`, `printer`, `style`,
  `context`, `colspec`, `sentence`, `perturb`) and the `.bib` formatter.
  Embedded by the dprint Wasm plugin (`jolars/dprint-plugin-badness`, its own
  repo so `plugin.wasm` stays out of this one's `v*` release stream), so it (and
  the parser it depends on) must keep building for `wasm32-unknown-unknown` — a
  CI job guards this (and covers `badness-wasm` too). Anything touching the
  filesystem, threads, or processes stays in the root crate. The plugin is
  sandboxed with no filesystem, so it passes an empty `SignatureDb` where the
  CLI folds in signatures scanned from sibling `.sty`/`.cls` files — the one
  sanctioned divergence from `badness format`.
- **`badness-wasm`** — `publish = false` wasm-bindgen shim over the two library
  crates, powering the docs playground (`docs/src/playground/index.html`); built
  with `wasm-pack` via `task playground:wasm`, never released.

The root crate keeps `linter/`, `lsp/`, `project/`, `incremental` (salsa),
`completion`, `config`, `file_discovery`, `text/`, and the CLI, and re-exports
the member crates at the old module paths via **shim modules** (`src/parser.rs`
is `pub use badness_parser::parser::*;`, etc.), so intra-repo consumers write
`crate::parser::…` as before. Bridge modules host the CLI-side halves of split
concerns: `formatter::check` + the disk-backed `format_file_with_packages`
entries, and `semantic::load`. The CLI
processes `.tex`, `.sty`/`.cls`, `.dtx`, `.ins`, and `.bib`; `badness.toml` is
local project config consumed only by the CLI. See the Architecture page for the full
tour.

## Tenets

1. **Deterministic, rule-based formatting.** Layout is decided solely by the
   formatter's rules and the layout engine—the formatter is the **sole authority on
   layout**. Push back against hard-coding special cases. Autofixes are textual edits
   that never invoke the formatter: a fix decides *what* to rewrite, never *how to lay
   it out*, and owes only correctness (the result still parses and is still lossless),
   never line-width. When a fix can't meet that bar for some shape, make it correct by
   construction or withhold it for that shape (still report the finding). The pipeline
   is fix-then-format; don't run the formatter inside `--fix`—and, mirrored, content
   rewrites never run inside `format`: the layout engine changes only trivia (see
   Invariants).
2. **Incremental parsing is first-class.** Parser/CST work must keep the salsa-based
   reparse path (`incremental.rs`) viable.
3. **Parsing is the parser's job.** Never paper over parser mistakes in the formatter,
   and never let parsing logic creep into the formatter. If the formatter hits
   something the parser got wrong, fix it in the parser.
4. **Losslessness is the parser's job.** `reconstruct(text) == text`, always. The
   formatter may assume a lossless CST.

## Core architectural decisions

Load-bearing. If a change pushes against one, raise it explicitly. Each rule below
links to the Development page carrying its full rationale, examples, and provenance.

1. **The parser treats input as generic TeX surface syntax and always produces a
   lossless tree.** It never *requires* resolving macros or catcodes; we do **not**
   implement macro expansion or a TeX evaluator. Anything we cannot statically resolve
   degrades to generic nodes (plus a diagnostic where useful), never a crash or

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolars/badness](https://github.com/jolars/badness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
