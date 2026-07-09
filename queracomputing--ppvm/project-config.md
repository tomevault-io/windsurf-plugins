---
trigger: always_on
description: > **Read the Developer Guide first.** The canonical contributor reference for
---

# AGENTS.md

> **Read the Developer Guide first.** The canonical contributor reference for
> this repository — for both human and AI contributors — is the Astro page
> at [`docs/src/pages/develop.astro`](docs/src/pages/develop.astro) (rendered
> at `/develop/` on the live site). It covers project layout, build/test
> commands for the Rust workspace, the Python package, **and the docs site
> itself** (`§ 2 Build & test`), architecture, conventions, the Python
> binding pipeline, extension recipes, and the file-by-file "where to look
> for X" table.
>
> This file is a short pointer so agents can find that guide quickly. Do
> not add content here that belongs in the Developer Guide — keep the
> guide as the single source of truth.

## Install the ppvm-usage skill

If you have [ion](https://ion.rogerluo.dev) installed, install the
`ppvm-usage` skill before writing any ppvm code:

```bash
ion add QuEraComputing/ppvm/skills/ppvm-usage
```

The skill (`skills/ppvm-usage/SKILL.md` in this repo) covers the Heisenberg /
Schrödinger gate-order trap, `Config`-generic `PauliSum` usage, truncation
strategies, and Python / Rust call sites for both backends. Read it before
the Developer Guide if your task is *using* ppvm rather than modifying its
internals.

## TL;DR for agents

If you are an AI agent picking up a task in this repository:

1. Open [`docs/src/pages/develop.astro`](docs/src/pages/develop.astro) and
   read the sections relevant to your task. The "For AI agents" callout at
   the top tells you which sections are load-bearing.
2. Use `uv` for anything Python; never `pip`.
3. Use Conventional Commits: `<type>(<scope>): <description>`.
4. Build & test the relevant target:
   - **Rust workspace**: `cargo test --workspace`
   - **Python package**: `uv run --project ppvm-python --group dev pytest …`
   - **Docs site**: `cd docs && npm run build` (chains `extract:rust`,
     `extract:python`, `extract:notebooks`, then `astro build`).
     Per-step commands and the `astro:dev` / `astro:build` escape
     hatches are documented under `§ 2 → "This docs site"` in the
     Developer Guide and in [`docs/README.md`](docs/README.md).
   - **Docs notebooks (`docs/notebooks/*.py`)**: executed at build time
     by [`docs/scripts/build-notebooks.py`](docs/scripts/build-notebooks.py)
     and embedded into `/examples/<slug>`. Outputs are content-addressed
     cached under `docs/.notebook-cache/`, fingerprinted by
     `CACHE_SCHEMA_VERSION` + the extractor script itself + the
     notebook source + every `Cargo.toml` + `Cargo.lock` +
     `ppvm-python/uv.lock`. (The extractor is in the fingerprint so
     rendering/sanitiser edits invalidate cached outputs automatically.)
     CI persists the directory via `actions/cache` (see the "Restore
     executed-notebook cache" step in
     [`.github/workflows/docs.yml`](.github/workflows/docs.yml)).
     **If you change the fingerprint inputs**, update both
     `_shared_fingerprint_files()` in the script *and* the
     `hashFiles(...)` call in the workflow — they must stay in sync
     (both already list `docs/scripts/build-notebooks.py`). Force a
     clean re-execution with `PPVM_NOTEBOOK_CACHE=0`, or bump
     `CACHE_SCHEMA_VERSION` for a global invalidation that survives
     in the cache. Full rationale lives under `§ 2.1 Notebook
     execution & caching` in the Developer Guide.
5. Respect the `Config`-trait generics in `ppvm-traits`; do not introduce
   runtime dispatch where a compile-time bound suffices.
6. Pauli propagation runs **backwards** (Heisenberg picture). Reverse the
   gate order accordingly when writing tests.
7. **Python docstring cross-references** use Markdown backtick spans, not
   RST syntax. The docs pipeline (griffe → `marked`) renders docstrings as
   Markdown, so `:meth:` / `:func:` / `:class:` are never parsed as links —
   they appear as literal text. Use plain backticks instead:
   - ✅ `` `fork` `` or `` `GeneralizedTableau.sample` ``
   - ❌ `` :meth:`fork` `` or `` :func:`ppvm.sample_stim` ``

## Workspace at a glance

```
crates/ppvm-traits          # Trait system, Config bundle, Pauli alphabet, map impls
crates/ppvm-pauli-word      # Packed Pauli strings: PauliWord, phased, lossy, pattern
crates/ppvm-pauli-sum       # PauliSum engine, truncation strategy, concrete configs
crates/ppvm-tableau         # Stabilizer + generalized-tableau simulator
crates/ppvm-sym             # Symbolic (parametric) Pauli propagation
crates/ppvm-stim            # Stim program execution against the tableau
crates/stim-parser          # Standalone Stim parser
crates/ppvm-python-native   # PyO3 cdylib, compiled into the `ppvm` wheel as `ppvm._core`
ppvm-python/                # Python package `ppvm` (maturin: Python wrapper + the compiled `ppvm._core`)
docs/                       # Astro docs site — includes the Developer Guide
```

Everything else — design patterns, extension recipes, the file-by-file
"where to look for X" table — is in the Developer Guide.

## Docs-site visual language

If you're modifying anything under `docs/`, respect these conventions. The
site is meant to read as research-grade documentation, not a marketing
landing — that intent guides every other call.

### Design vocabulary


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QuEraComputing/ppvm](https://github.com/QuEraComputing/ppvm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
