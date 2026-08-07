---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Mosaic is a slide package for Typst (0.15+). The `mosaic/` directory is the package itself; everything else (tests, scripts, docs, Makefile) is repository tooling around it. Python tooling runs through `uv run python`.

## Commands

- `make install` — copy `mosaic/` into Typst's package index as `@preview/mosaic:0.0.1`. Tests, docs, and example decks all import that published spelling, so the working tree shadows the Universe copy while developing; run this after any package source change and before compiling anything by hand. `make uninstall` removes the shadow.
- `make check` — the whole test suite in one target: the facade export contract (`cd tests && uv run python -m unittest test_check_api_exports test_theme_architecture test_palettes`, plus `scripts/check-api-exports.py`), the `core` and `layout` and `negative` manifest groups, and `scripts/check-doc-assets.py`.
- One manifest group, without a make target: `uv run python scripts/run-tests.py core|layout|negative --typst typst`.
- Single positive test: `make install`, then `typst compile --root . tests/<name>.typ /tmp/out.pdf` from the repo root. Many core tests also have output assertions in `scripts/run-tests.py` (pdftotext/SVG greps), so a clean compile is necessary but not always sufficient.
- Single negative test: `typst compile --root . tests/invalid/<name>.typ /tmp/out.pdf` must fail with the exact diagnostic listed in `tests/invalid/expected-diagnostics.txt`.
- `make website` — render `docs-src/` into `docs/` from the committed example artifacts; `make build` = doctor + install + check + website. `make doctor` checks prerequisites. Calepin owns `docs/` and wipes it on every build, so never edit anything there and never add a file to it by hand.
- `make publish-docs` — `docs/` is gitignored so routine commits never carry a rebuild. This target rebuilds the site and force-stages it (`git add -f`, minus Calepin's manifest and Syncthing conflict copies) for a deliberate GitHub Pages commit; it stages only, and never commits or pushes.
- `make artifacts` — re-render the committed example artifacts (embedded PDFs and SVGs, deck PDFs and covers, the showcase reel). The website build no longer does this, so run it deliberately after changing an example or the package's visual output.
- `make release-stage` — sync the README and its illustrations into the package, then stage the Typst Universe file set in `dist/packages/preview/mosaic/{version}/`, ready to copy into a `typst/packages` fork.
- `make clean` removes every generated file: build stamps, staged API modules, Calepin's caches, the rendered site, and all example artifacts. The artifacts are committed, so a clean shows up as deletions in `git status` until `make artifacts` re-renders them (minutes of work); restore them from git if all you wanted was a fresh cache. `make help` lists all eleven commands.

Test manifests are exhaustive and enforced: every `tests/*.typ` must appear in `tests/positive-manifest.json` (groups `core`, `layout`, `responsive`), and every `tests/invalid/*.typ` needs a `stem|expected diagnostic` line in `tests/invalid/expected-diagnostics.txt`. Adding or renaming a fixture without updating the manifest fails the run.

## Conventions

`CONVENTIONS.md` is authoritative for all `.typ` code: kebab-case everywhere, the `is-`/`validate-`/`require-`/`resolve-`/`render-` verb families, the one-word-per-concept vocabulary (node/cell/track/grid, fields/options/settings/record, gap vs gutter vs spacing vs inset), and signature shape (one positional subject, named booleans, native-superset shadows). Read it before writing package code.

Two project-specific rules worth restating:
- Hard-coded visual constants belong in a theme's `apply` function as `set`/`show` rules, not as new token records.
- Themes deliberately repeat structure; do not factor a shared base `apply` across themes.

Documentation prose is never hard-wrapped. Write each paragraph as one long line and let the editor soft-wrap it; do not insert artificial line breaks at a column limit. This applies to prose in `docs-src/`, `skills/`, and Markdown files. Code blocks and code comments wrap normally.

## Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vincentarelbundock/mosaic](https://github.com/vincentarelbundock/mosaic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
