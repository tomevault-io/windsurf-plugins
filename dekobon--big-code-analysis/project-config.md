---
trigger: always_on
description: Universal project instructions for AI coding assistants.
---

# AGENTS.md

Universal project instructions for AI coding assistants.

## Project Overview

`big-code-analysis` is a Rust library that extracts maintainability
metrics from source code in many languages. It is a hard fork of
Mozilla's
[rust-code-analysis](https://github.com/mozilla/rust-code-analysis),
maintained in this repository. It is built on
[tree-sitter](https://tree-sitter.github.io/tree-sitter/) and is published on
crates.io as a library plus two binaries.

The repository is a Cargo workspace:

| Crate | Path | Purpose |
|-------|------|---------|
| `big-code-analysis` | `./` (root) | Library: parsers, AST traversal, metric computation |
| `big-code-analysis-cli` | `big-code-analysis-cli/` | CLI for invoking the library on files / trees |
| `big-code-analysis-py` | `big-code-analysis-py/` (excluded from default-members; needs Python headers + maturin) | PyO3 Python bindings |
| `big-code-analysis-web` | `big-code-analysis-web/` | REST API server wrapping the library |
| `xtask` | `xtask/` (excluded from default-members) | Build-time helper that renders man pages from the live clap definitions (see `man/`) |
| `enums` | `enums/` (excluded from default workspace) | Code-generation helper for language enums |

Vendored / path-dependent grammar crates also live in the repo:
`tree-sitter-ccomment`, `tree-sitter-mozcpp`, `tree-sitter-mozjs`,
`tree-sitter-preproc`, `tree-sitter-tcl`. External grammar crates are
pinned with `=X.Y.Z` versions in the root `Cargo.toml`.

The default branch is **`main`**.

The CLI binary is **`bca`** (package `big-code-analysis-cli`); the
web-server binary is **`bca-web`** (package `big-code-analysis-web`).
From a checkout, run them via `cargo run -p big-code-analysis-cli --`
and `cargo run -p big-code-analysis-web --`.

## Project layout

- `src/lib.rs` — public re-exports; this is the published API surface.
- `src/languages/` — one `language_<lang>.rs` per supported language. These
  modules deliberately mirror each other; macros under
  `src/c_langs_macros/`, `src/macros/`, and `src/c_macro.rs` generate the
  shared structure. A
  bug in one language module typically exists in several — fix all
  affected siblings together.
- `src/metrics/` — individual metric implementations: `abc.rs`,
  `cognitive.rs`, `cyclomatic.rs`, `nexits.rs`, `halstead.rs`, `loc.rs`,
  `mi.rs`, `nargs.rs`, `nom.rs`, `npa.rs`, `npm.rs`, `tokens.rs`,
  `wmc.rs`.
- `src/output/` — JSON / YAML / TOML / CBOR serializers for metric output.
- `src/parser.rs`, `src/node.rs`, `src/spaces.rs`, `src/checker.rs`,
  `src/getter.rs`, `src/alterator.rs`, `src/traits.rs` — core AST plumbing.
- `tests/` — integration tests, including `insta` snapshot tests
  (`*.snap` / `*.snap.new`).
- `big-code-analysis-book/` — mdBook documentation source.
- `enums/` — separate workspace member (excluded from the root workspace)
  that generates language enum tables.
- Helper scripts: `check-grammar-crate.py`, `check-grammars-crates.sh`,
  `recreate-grammars.sh`, `generate-grammars/`. (The grammar-bump diff
  step now uses the native `bca diff`; the former
  `split-minimal-tests.py` + external `json-minimal-tests` chain was
  retired in #487.)

## Editing principles

- This is a published `2.x` library (`big-code-analysis` on crates.io)
  with a written stability contract in [`STABILITY.md`](./STABILITY.md).
  Treat `lib.rs` re-exports, public traits (`ParserTrait`,
  `LanguageInfo`, etc.), and public types (`Metrics`, `FuncSpace`,
  language enums) as a stable API surface. Within the current major
  line, additive changes belong under a minor bump; breaking shape
  changes are reserved for the next major (`3.0`) and must be planned
  deliberately — never slip a SemVer break into a patch or minor
  release. Public-API changes must be cross-referenced against
  `STABILITY.md` and recorded in the `## [Unreleased]` section of
  `CHANGELOG.md`; if the change is a source-level break that must
  wait for the next major, mark the entry **(breaking)** and note
  that it is deferred to the next major bump (the release-prep
  commit then moves the entry into the appropriate version section).
- For code files: prefer LSP / symbol-level editing
  (`replace_symbol_body`, `insert_before/after_symbol`) over line-based
  edits when available. Read the file (or use a symbol overview) before
  editing.
- For non-code files (Markdown, TOML, YAML, JSON): use targeted edits with
  scoped `old_string` / `new_string` pairs. Avoid `sed` for multi-line
  edits.
- Never rewrite an entire test file to add or fix one test. Modify only
  the specific tests that need changing.
- Verify previously passing tests still pass before committing
  (`cargo test --workspace --all-features`).
- When fixing a bug, add a regression test that would catch the exact bug
  if reintroduced.
- Default to writing no comments. Only add one when the *why* is
  non-obvious.
- **MANDATORY** before any public API change: enumerate every call site
  (`find_referencing_symbols` if an LSP tool is available, otherwise a
  workspace-wide search). Cross-crate breakage is silent until CI.
- When a change touches metric computation, AST traversal, or anything
  under `src/languages/`, exercise **every** language affected — passing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dekobon/big-code-analysis](https://github.com/dekobon/big-code-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-14 -->
