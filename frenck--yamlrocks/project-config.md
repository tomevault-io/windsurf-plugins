---
trigger: always_on
description: Guidance for AI coding agents (and humans) working in this repository. This file
---

# AGENTS.md

Guidance for AI coding agents (and humans) working in this repository. This file
follows the [agents.md](https://agents.md) convention.

## What this project is

YAMLRocks is a fast, correct YAML library for Python, implemented in Rust with PyO3
and built by maturin. The Rust crate lives in `src/`; the Python package and
type stubs live in `pysrc/yamlrocks/`.

## Project layout

| Path               | Purpose                                                                               |
| ------------------ | ------------------------------------------------------------------------------------- |
| `src/scanner/`     | Byte reader, tokenizer, comment extraction                                            |
| `src/parser/`      | Tokens to events                                                                      |
| `src/resolver/`    | Scalar typing (YAML 1.1 and 1.2 schemas)                                              |
| `src/decode/`      | Fast path: events to a `Value` tree (+ merge keys)                                    |
| `src/encode/`      | Fast path: `Value` tree to YAML bytes                                                 |
| `src/roundtrip/`   | Rich AST, composer, emitter, `YAMLRocksDocument`, upgrade                             |
| `src/include/`     | `!include`/`!secret`/`!env_var`/`!input` resolution                                   |
| `src/schema/`      | JSON Schema validation against the AST                                                |
| `src/ffi/`         | PyO3 module: functions, types, conversions                                            |
| `pysrc/yamlrocks/` | Python package, stubs, PyYAML compat shim                                             |
| `tests/`           | pytest suite, grouped by capability (see `tests/README.md`); data under `tests/data/` |
| `docs/`            | Astro Starlight documentation site                                                    |
| `adr/`             | Architecture decision records (the why behind major choices)                          |

## The `just` task runner

This project uses [uv](https://docs.astral.sh/uv/) for the Python side and
[just](https://github.com/casey/just) as the task runner. **`just` is the primary
interface: every common workflow (build, test, lint, type-check, docs) has a
recipe, and the recipes wrap the exact `uv`/`cargo` commands CI runs.** Run
`just --list` to see them all, or `just` on its own for the same list.

Run a recipe either by activating the venv once (`uv sync && source
.venv/bin/activate`, then `just <recipe>`) or without activating by prefixing
`uv run --no-sync just <recipe>`. Recipes assume `just setup` (or `just develop`)
has run at least once.

| Recipe                        | What it does                                                                    |
| ----------------------------- | ------------------------------------------------------------------------------- |
| `just setup`                  | Install all dev dependencies into the uv-managed venv                           |
| `just develop`                | Build + install the extension in **release** mode (rerun after Rust changes)    |
| `just develop-debug`          | Faster **debug** build for tight iteration loops                                |
| `just test [args]`            | Run the Python suite (`pytest`); args pass through, e.g. `just test -k anchors` |
| `just test-rust [args]`       | Run the Rust unit tests (`cargo test --lib`)                                    |
| `just lint`                   | Lint and format-check Python (read-only)                                        |
| `just fmt`                    | Auto-format Python **and** Rust in place                                        |
| `just typecheck`              | Type-check with both `mypy` and `ty`                                            |
| `just clippy`                 | Lint the Rust crate (`-D warnings`)                                             |
| `just spellcheck`             | Spell-check with `codespell`                                                    |
| `just precommit [hook]`       | Run every pre-commit hook via `prek` (the exact set CI runs)                    |
| `just check`                  | **Full local gate**: build, all hooks, Rust + Python suites, doc examples       |
| `just examples`               | Run every documented Python example and verify its output comments              |
| `just docs` / `just docs-dev` | Build the docs site / serve it with live reload                                 |
| `just bench [args]`           | Comparison benchmarks vs PyYAML, ruamel.yaml, and yamlium                       |
| `just audit`                  | Audit every dependency ecosystem for advisories                                 |
| `just licenses`               | Regenerate `THIRD_PARTY_LICENSES.md` from the crate graph                       |
| `just coverage`               | Rust line coverage measured through the Python suite                            |
| `just fuzz [seconds]`         | Fuzz the parser, optionally time-boxed                                          |
| `just clean`                  | Remove build and fuzzing artifacts                                              |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frenck/YAMLRocks](https://github.com/frenck/YAMLRocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
