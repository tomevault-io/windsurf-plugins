---
trigger: always_on
description: This file provides guidance to coding agents when working with code in this
---

# Agent Instructions

This file provides guidance to coding agents when working with code in this
repository.

## Project

Arity is a Rust CLI providing a language server, formatter, and linter for the R
language. Single-crate Cargo package (published to crates.io as `arity`, edition
2024; the binary and library crate are both named `arity`), not a workspace.

**Strategy (see `TODO.md`):** bring the parser + formatter foundation to
near-completion *first*; the linter and LSP are deferred to later phases. When
in doubt about scope/priority, `TODO.md` is the live roadmap and records known
issues and follow-ups.

The dev environment is provided via `devenv`/Nix (`devenv.nix`, `.envrc`) and
includes `R`.

## Tenets

1. **Deterministic, rule-based formatting.** Output is decided solely by the
   formatter's rules and the layout engine. Push back against attempts to
   hard-code special cases or exceptions for specific constructs. Unlike air
   (arity's closest relative), arity does **not** honor "persistent line
   breaks"—the input's existing line breaks never influence the result. Because the
   formatter is the **sole authority on layout**, autofixes are textual edits
   that never invoke it: a fix decides *what* to rewrite, never *how to lay it
   out*. Producing well-formatted output after a fix is a separate format pass's
   job, not the fixer's (see the autofix-correctness note under the linter).
2. **Incremental parsing is first-class**, not an afterthought. Parser/CST work
   must keep the `salsa`-based incremental reparse path (`src/incremental.rs`)
   viable.
3. **Parsing is the parser's job.** Never paper over parser mistakes in the
   formatter, and never let parsing logic creep into the formatter. If the
   formatter hits something the parser handled wrong, fix it in the parser.
4. **Losslessness is the parser's job.** The parser must preserve all text
   (whitespace, comments, etc.) so that `reconstruct(text)` is always `text`.
   The formatter can assume the CST is lossless and focus on formatting logic.

## Air compatibility (soft target)

Arity tracks a **soft, one-directional compatibility target** with the `air`
formatter (a la ruff's "% Black-compatible" number)—but this is **strictly
subordinate to Tenet 1** and is **never a quality gate**. We do not match air;
we measure how often air would leave arity's output unchanged, and treat air's
maturity as a free differential oracle for finding our own inconsistencies.

- The gauge lives in `tests/air_compat.rs` (`#[ignore]`d, so it never runs in
  `cargo test` and cannot fail CI). Run it with `task air-compat`; it
  regenerates `AIR_COMPAT.md`.
- It measures the *fixed point* `air(arity(x)) == arity(x)`, not a head-to-head
  diff—this cancels out the persistent-line-break difference (which is the
  whole point of arity) by construction, leaving only genuine rule divergences.
- Divergences are triaged into two buckets. **Adopt** when air's output is
  simply more idiomatic and arity is being inconsistent (fix the rule).
  **Record** when the divergence is a deliberate arity choice—add it to
  `tests/air_compat_allowlist.toml` with a rationale.
- Diverging from air is allowed, but should **raise tension**: it is a
  conscious, documented decision (an allowlist entry), not a silent one. An
  unexplained divergence in `AIR_COMPAT.md` is an open question, never an excuse
  to fail a build.

## Commands

```sh
cargo build                       # dev build
cargo build --release
cargo test                        # all tests (CI: cargo test --verbose)
cargo test <substring>            # run tests matching a name
cargo test --test parser_snapshots   # one integration test file (also: formatter, lint, ast_wrappers, salsa_incremental, line_endings, air_parser_harness)
cargo clippy --all-targets --all-features -- -D warnings   # lint; warnings are errors
cargo fmt -- --check              # rustfmt check (keep changes rustfmt-clean)
```

CLI usage:

```sh
cargo run -- parse <file.R>                  # print CST; stdin if no file
cat file.R | cargo run -- parse --verify --quiet   # losslessness round-trip check
cargo run -- format <file.R>                 # format to stdout (stdin if omitted)
cargo run -- format --check <path>           # check without writing (multi-path requires --check)
cargo run -- format --verify <file.R>        # check idempotence; does not write
cargo run -- lint <path>                     # lint (stdin if no path); exits 1 on findings
```

The documentation site (`book/`) is an mdBook. Its reference pages are
generated: `build.rs` writes `book/src/reference/cli.md` from the clap CLI, and
`cargo run --example docgen` renders the per-rule pages (and `version.md`) by
running the real linter on each rule's examples. `mdbook build book` then builds
the site; `.github/workflows/docs.yml` deploys it to GitHub Pages. The rendered
rule docs are pinned by `tests/rule_docs.rs` so they can't drift from behavior.

Snapshot tests use `insta`: review/accept with `cargo insta review` or
`cargo insta accept`. Logging honors `RUST_LOG` (e.g.
`RUST_LOG=debug cargo test`) via `env_logger`. `task <name>` (Taskfile.yml)
wraps the above: `lint`, `format`, `test`, `test-debug`, `audit`, `deny`,
`docs-gen`, `docs-build`, `docs-preview`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jolars/arity](https://github.com/jolars/arity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
