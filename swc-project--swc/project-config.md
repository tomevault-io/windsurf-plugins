---
trigger: always_on
description: When working in a specific directory, apply the rules from that directory and all parent directories up to the root.
---

# AI Agent Rules

When working in a specific directory, apply the rules from that directory and all parent directories up to the root.

## While working on `.`

*Source: `AGENTS.md`*

# Instructions

## General rule

-   Read the codebase - the codebase is source of truth, and you should prefer reading lots of source code over searching.
-   Do not search web unless explicitly asked to do so. Web search does not help in general for our project.
-   Write performant code. Always prefer performance over other things.
-   Use `gh` CLI tool for fetching data from `github.com`.

## Code style

-   Write comments and documentations in English.
-   Write documentation for your code.
-   Commit your work as frequent as possible using git. Do NOT use `--no-verify` flag.
-   Prefer multiple small files over single large file.

---

-   When creating Atom instances, it's better to use `Cow<str>` or `&str` instead of `String`. Note that `&str` is better than `Cow<str>` here.

## Testing

-   Write unit tests for your code.
-   Prefer fixture tests over inline (`#[test]`) tests.
-   You can do `UPDATE=1 cargo test` to get test outputs updated for fixture tests.
-   When instructed to fix tests, do not remove or modify existing tests.

### Fixture Test Common Guide

- Add new coverage by extending existing fixture suites instead of adding ad-hoc inline tests.
- Find the exact fixture harness before adding files with: `rg -n "#\[(testing::)?fixture\(" tests src --glob "*.rs"`.
- Keep each suite's naming conventions (for example: input.*, output.*, exec.*, .ans).
- For snapshot-style tests, update expected outputs with `UPDATE=1 cargo test -p ...` using the exact crate command documented in each crate's AGENTS.md.
- Always rerun the same crate tests without `UPDATE` before finishing.

## Verification

-   Before finishing a task, always run this baseline locally.
    -   `cargo fmt --all`
    -   `cargo clippy --all --all-targets -- -D warnings`
-   For each touched Rust crate, run crate-level verification locally.
    -   `cargo test -p <crate>`
-   If wasm binding packages are touched, run:
    -   `(cd bindings/binding_core_wasm && ./scripts/test.sh)`
    -   `(cd bindings/binding_minifier_wasm && ./scripts/test.sh)`
    -   `(cd bindings/binding_typescript_wasm && ./scripts/test.sh)`
    -   `(cd bindings/binding_es_ast_viewer && ./scripts/test.sh)`
-   If node bindings or integration paths are touched, run:
    -   `(cd packages/core && pnpm build:dev && pnpm test)`

## Compatibility rule

-   Do not use unstable, nightly only features of rustc.


---

## While working on `crates/jsdoc`

*Source: `crates/jsdoc/AGENTS.md`*

### Fixture Test Addition Guide

- Preferred fixture roots in this crate: tests/fixtures.
- Update generated fixture outputs with: UPDATE=1 cargo test -p jsdoc.
- Verify without UPDATE before finishing: cargo test -p jsdoc.


---

## While working on `crates/swc`

*Source: `crates/swc/AGENTS.md`*

### Fixture Test Addition Guide

- Preferred fixture roots in this crate: tests/fixture, tests/errors, tests/exec, tests/minify, tests/typescript, tests/vercel, tests/stacktrace, tests/babel-exec.
- Update generated fixture outputs with: UPDATE=1 cargo test -p swc.
- Verify without UPDATE before finishing: cargo test -p swc.


---

## While working on `crates/swc_bundler`

*Source: `crates/swc_bundler/AGENTS.md`*

### Fixture Test Addition Guide

- Preferred fixture roots in this crate: tests/fixture, tests/deno-exec.
- Update generated fixture outputs with: UPDATE=1 cargo test -p swc_bundler.
- Verify without UPDATE before finishing: cargo test -p swc_bundler.


---

## While working on `crates/swc_core`

*Source: `crates/swc_core/AGENTS.md`*

### Fixture Test Addition Guide

- Preferred fixture roots in this crate: tests/fixture.
- Update generated fixture outputs with: UPDATE=1 cargo test -p swc_core.
- Verify without UPDATE before finishing: cargo test -p swc_core.


---

## While working on `crates/swc_css_codegen`

*Source: `crates/swc_css_codegen/AGENTS.md`*

### Fixture Test Addition Guide

- Preferred fixture roots in this crate: tests/fixture, tests/options, ../swc_css_parser/tests/fixture.
- Update generated fixture outputs with: UPDATE=1 cargo test -p swc_css_codegen.
- Verify without UPDATE before finishing: cargo test -p swc_css_codegen.


---

## While working on `crates/swc_css_compat`

*Source: `crates/swc_css_compat/AGENTS.md`*

### Fixture Test Addition Guide

- Preferred fixture roots in this crate: tests/nesting, tests/custom-media-query, tests/media-query-ranges, tests/color-hex-alpha, tests/color-legacy, tests/selector-not, tests/color-hwb, tests/all.
- Update generated fixture outputs with: UPDATE=1 cargo test -p swc_css_compat.
- Verify without UPDATE before finishing: cargo test -p swc_css_compat.


---

## While working on `crates/swc_css_lints`

*Source: `crates/swc_css_lints/AGENTS.md`*

### Fixture Test Addition Guide

- Preferred fixture roots in this crate: tests/rules/pass, tests/rules/fail.
- Update generated fixture outputs with: UPDATE=1 cargo test -p swc_css_lints.
- Verify without UPDATE before finishing: cargo test -p swc_css_lints.


---

## While working on `crates/swc_css_minifier`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swc-project/swc](https://github.com/swc-project/swc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
