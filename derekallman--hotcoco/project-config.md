---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

hotcoco is a pure Rust port of [pycocotools](https://github.com/ppwwyyxx/cocoapi) with PyO3 Python bindings. It provides 13-23x speedups over pycocotools for bbox, segmentation, and keypoint evaluation.

- **Primary language:** Rust. All core logic lives in `hotcoco`.
- **Python bindings:** PyO3/maturin in `hotcoco-pyo3`, exposed as the `hotcoco` Python package.
- **CLI:** `hotcoco-cli` binary wrapping the Rust library.

### Key Architecture

- Single root `pyproject.toml` acts as both maturin build config and Python package definition; `manifest-path = "crates/hotcoco-pyo3/Cargo.toml"` points maturin at the cdylib. `[tool.uv] package = false` means uv won't auto-build — always use `just build` explicitly.
- `hotcoco-pyo3` uses `hotcoco-core` as the Cargo dependency alias for `hotcoco` to avoid name collision with the `hotcoco` Python module name
- Python bindings return plain dicts (not wrapped Rust structs) matching pycocotools conventions
- Mask operations handle numpy row-major <-> Rust column-major transposition in the PyO3 layer
- `cargo build --workspace` will fail at link time for hotcoco-pyo3 (expected — cdylib needs Python). Use `cargo check` instead, or build via maturin.
- **Type stubs:** `python/hotcoco/__init__.pyi` is hand-written and must be updated whenever the Python API changes (new methods, renamed parameters, changed return types). Run `uv run pytest scripts/test_stubs.py` to check for drift. The test catches missing names but not signature mismatches — review the stub manually when changing signatures.

## Metric Parity

All COCO evaluation metrics must match pycocotools: 12 for bbox/segm, 10 for keypoints (no small area range), 13 for LVIS (adds APr/APc/APf/AR@300).

- **Always ensure exact parity when modifying evaluation logic.** Run `cargo test` after Rust changes.
- Verified on val2017: keypoints exact, bbox within 0.0001, segm within 0.0002.
- When in doubt, run differential tests against pycocotools on real COCO data before declaring a task complete.
- After any change to evaluation logic, run `/parity` — it holds the full verification sequence and the expected tolerances.

## Testing

- Run `cargo test` after any Rust code changes and verify all tests pass before committing.
- For Python binding changes: `just build` as a smoke test, then `just parity` to verify metrics.
- `just test` runs `cargo test` + fast Python regression tests (`scripts/test_parity.py`) — safe for CI, completes in under 30s.
- `just fuzz` runs the hypothesis-based fuzzer (`scripts/fuzz_parity.py`) — use to hunt for parity bugs, not in CI. Takes several minutes.
- Model: use the fuzzer to *find* bugs, then prove fixes with Rust integration tests in `crates/hotcoco/tests/`.

## Tool Preferences

- **Always use `uv run python` — never bare `python` or `python3`.** This project uses uv-managed Python; the OS Python is not the project environment.
- For library/crate documentation, prefer a docs MCP server (e.g. context7) when one is connected. None is configured by default, so WebFetch against docs.rs, python.org, and PyPI is the working fallback — those domains are already in the project allowlist.

## Build Gotchas

`just` is the task runner — run `just --list` for the current recipes. The non-obvious parts:

- **`uv sync` alone is not enough.** Without `--all-extras` it skips `maturin`, and `just build` then fails. Always use `just setup` for first-time setup.
- `uv run python` works from anywhere in the repo (no need to cd first).
- The `coco` CLI is installed into `.venv/bin/coco` by `just build`. Run it as `uv run coco <subcommand>` (or activate the venv with `source .venv/bin/activate` for bare `coco`).

## Documentation

- This project targets Python users first, Rust users second. Documentation, README, and examples should lead with Python usage in a Python-first tone similar to Polars. Do not be Rust-centric.
- Before making large-scale changes (docs revamps, major refactors), present a concrete preview or small example for approval first. Do not rewrite everything at once. For small additions (a single new page, a new section), just write it directly.

Docs are built with Zensical (config: `zensical.toml`). Preview locally with `zensical serve`.

When updating documentation (`docs/`) or `README.md`, always ensure both reflect the same information. Any change to one must be checked against the other — benchmark numbers, API examples, CLI flags, installation instructions, and feature descriptions must stay consistent across both.

## Design System ("Cold Brew")

All visual surfaces (browse UI, docs site, matplotlib, Plotly dashboard) share the **Cold Brew** theme. The canonical spec — colors, fonts, the 10-color chart palette, and which file owns each token — is the `cold-brew` skill. Consult it before changing any colors, fonts, or chart palettes, and when adding a new visual surface. Never eyeball new values.

## Pre-Commit Checks

A git pre-commit hook in `.github/hooks/pre-commit` runs formatting, clippy, and tests. All must pass or the commit is rejected.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [derekallman/hotcoco](https://github.com/derekallman/hotcoco) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
