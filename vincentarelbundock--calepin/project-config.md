---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Calepin is a Rust CLI that turns `.typ` files into computational notebooks. It is Typst-native: executable code chunks live directly inside Typst documents (no Markdown layer). The CLI scans a document, executes its code chunks, and lets the real `typst` binary render the results in place. Native engines: `r`, `python`. Diagram engines: `mermaid`, `tikz`, `dot`, `d2` (stateless, always emit SVG). Any other fence language, including `julia` and `sh`, is treated as a Jupyter kernel name and routed through the Jupyter bridge; there is no `bash` alias for `sh`, and the kernel name must match (or version-prefix-match) an installed Jupyter kernel or the chunk is marked unavailable, with a warning on the terminal naming the chunk, engine, and reason (`--strict` or `CALEPIN_STRICT=1` turns that into an error).

The Typst runtime is not one embedded file. `build.rs` bundles every file under `src/assets/typst-runtime/` into the binary at compile time; at preprocess/compile time Calepin writes that bundle to `.calepin/runtime/`, plus a generated facade at `.calepin/calepin.typ` (imports the bundle and re-exports the public API) and a per-notebook binding that points the facade at that notebook's results and config. There is no separate Typst Universe package.

## Commands

The binary crate is nested at `calepin/`, so direct cargo invocations need `--manifest-path calepin/Cargo.toml`. The `Makefile` is the canonical entry point and wraps this for you.

- `make build` / `make build-release` / `make install` (installs to `~/.cargo/bin`)
- `make test` runs the suite: `cargo test --manifest-path calepin/Cargo.toml` plus the `calepin-docs` crate's tests
- Single test: `cargo test --manifest-path calepin/Cargo.toml <test_name>`
- `make check` for a fast `cargo check`
- `cargo clippy --manifest-path calepin/Cargo.toml` for lints
- `make docs-check` runs the generated-docs-fragment test (`cargo test --manifest-path calepin/Cargo.toml generated_docs_fragment_matches_source`); rerun it with `CALEPIN_UPDATE_DOCS=1` to regenerate `docs-src/reference/generated.md` after a change to the facts it tracks (see Conventions below)
- `make cli-reference` regenerates `docs-src/reference/cli.typ` from clap `--help` output
- `make website` / `make serve` build the docs site via `calepin compile docs-src docs` into `docs/` (website config auto-discovered at `docs-src/calepin.toml`)
- `make bump VERSION=x.y.z` then `make release` cuts a release (tags + pushes, which fires the cargo-dist and crates.io workflows). `make release` refuses a dirty tree.
- `make linux-packages` builds the `.deb`, `.rpm` and Arch `.pkg.tar.zst` into `dist/` from `packaging/linux/nfpm.yaml` (needs `nfpm` on `PATH`, Linux only)
- `make editors` builds the extension from `editors/vscode/`, installs it in VS Code, and installs it in Positron when the Positron CLI is available

Integration tests in `calepin/tests/typst_preprocess.rs` (there is no root `tests/` directory) shell out to the built binary plus real `typst`/`python3`/`pdftotext`. They return early (skip, not fail) when a required tool is absent, so a green run on a machine without `typst` may have skipped the meaningful tests. Set `CALEPIN_TEST_REQUIRE_TOOLS=1` (as the CI test job does) to turn every such skip into a failure naming the missing tool.

## Architecture

### Two-pass model around the real `typst` binary

Calepin never renders Typst itself. It wraps the user's `typst` binary and drives it twice over the same source file. The mode is selected via Typst CLI inputs that the runtime reads from `sys.inputs`:

1. **Query pass** (`--input calepin-mode=query`): `typst::preprocess` runs `typst eval` with internal `query(...)` expressions to extract metadata as JSON: `<calepin-config>` (setup defaults, themes) and `<calepin-chunk>` (one entry per chunk). `typst::query` parses these into `ChunkSpec`s.
2. **Render pass** (`--input calepin-mode=render`): `typst::compile` invokes the real `typst compile` or `typst watch`, passing `calepin-results=<path>` and `calepin-target=paged|html`. The runtime reads `results.json` and splices computed output back into the document.

Between the two passes, `typst::execute` runs every chunk and writes `results.json` (schema version 2; see `docs-src/reference/generated.md` for the exact number, generated from `typst/model.rs`).

So the data flow is: `preprocess` (write the runtime bundle -> query metadata -> execute chunks -> write results.json) then `compile_with_typst` (render with results spliced in). `handle_compile` in `typst/cli.rs` chains these; `watch` does the same once, then keeps both processes alive (see below).

### Reserved inputs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentarelbundock/calepin](https://github.com/vincentarelbundock/calepin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
