---
trigger: always_on
description: Kontrol is the Foundry integration for KEVM: it compiles a Foundry project's Solidity into K and runs symbolic-execution proofs over its `test*` functions.
---

# Kontrol — Project Conventions

Kontrol is the Foundry integration for KEVM: it compiles a Foundry project's Solidity into K and runs symbolic-execution proofs over its `test*` functions.
It is a thin Python layer (`src/kontrol/`) on top of two upstream dependencies — `kevm-pyk` (the EVM semantics + K frontend) and `pyk` (K's Python toolkit: option system, proof/KCFG infrastructure, RPC clients).
Most of the heavy lifting lives upstream; Kontrol adds Foundry-specific semantics (cheatcodes, storage layout), the CLI, and the proof orchestration.

## Architecture: the `src/kontrol` package

The package is small; each module has a clear role:

- `__main__.py` — entry point.
  `main()` parses args, builds the typed options object, then dispatches by convention: command `foo-bar` runs the `exec_foo_bar()` function in this module.
  Each `exec_*` loads a `Foundry` object and delegates to a `foundry_*` domain function.
- `cli.py` — `KontrolCLIArgs` (subclass of `kevm_pyk`'s `KEVMCLIArgs`) and `_create_argument_parser()` build the argparse tree; `generate_options()` maps parsed args to the right Options class.
- `options.py` — one `Options` dataclass per command (`ProveOptions`, `ShowOptions`, `BuildOptions`, …) plus shared mixins like `RpcOptions`.
  This is the load-bearing pattern: each class defines `default()`, `from_option_string()`, and `get_argument_type()` static methods, and pyk's `Options.__init__` walks the MRO to merge defaults from every parent with TOML config and CLI args by field name.
  Because the same option can arrive from a CLI flag, a `kontrol.toml` key, or a default, **a new flag must be wired in all three places** — the argparse arg in `cli.py`, the field + `default()` entry in `options.py`, and (if its CLI name differs from the field) `from_option_string()`/`get_argument_type()`.
- `foundry.py` — the `Foundry` class (wraps a project root, `foundry.toml`, the `KEVM` instance, and the `out/` artifacts) plus every `foundry_*` domain function for the non-prove commands.
  `FoundryKEVM(KEVM)` and `KontrolSemantics(KEVMSemantics)` live here; `KontrolSemantics` is where Foundry cheatcodes (FFI, `console.log`, symbolic-storage helpers) are handled as custom KCFG steps.
- `prove.py` — `foundry_prove()` orchestrates proofs: spins up the Kore/Booster RPC server, builds the test list from `--match-test`, and for each test drives a `pyk` `KCFGExplore` (over a `CTermSymbolic` RPC client) to extend the KCFG to completion.
- `display.py` — `foundry_show()` / `foundry_view()` render a proof's KCFG as text or in the interactive TUI.
- `kompile.py` — `foundry_kompile()` generates `foundry.k` from solc output and calls into `kevm_pyk`'s kompile.
- `solc_to_k.py` — parses solc JSON output into a `Contract` model and emits the per-contract K.
- `kdist/` — the K semantics Kontrol owns: `kontrol.md` (entry), `cheatcodes.md`, `foundry.md`, `assert.md`, `kontrol_lemmas.md`, etc., compiled via the kdist plugin (see below).

The dependency boundary: EVM-layer semantics and lemmas live **upstream** in `evm-semantics`/`kevm-pyk` (pinned by the `kevm-pyk@git+…@vX.Y.Z` ref in `pyproject.toml`); only Foundry-specific semantics live in `src/kontrol/kdist/`.
A change needed in the EVM layer is an upstream change — see the cross-repo rules.

## Development environment & checks

`uv` is the package manager (version pinned in `deps/uv_release`); Python `~=3.10`.
`uv sync` installs the dev dependency group; every tool runs under `uv run`.
The Makefile is the source of truth for how to invoke everything (`UV_RUN := uv run --`).

Code-quality workflow before any PR — `make format` then `make check`:

- `make format` = `autoflake` (remove unused imports/vars, in place) → `isort` → `black`, all over `src`.
- `make check` = `check-flake8 check-mypy check-autoflake check-isort check-black` — the read-only counterparts; this is exactly what CI's Code Quality job runs, plus `make pyupgrade`.
- Tool configs: `.flake8` (line length 120, bugbear + strict type-checking + quotes + pep8-naming plugins, `__init__.py` exempt from F401, `test-data` excluded); `pyproject.toml` for mypy (`disallow_untyped_defs = true`), isort (`profile = black`), black (line length 120, `skip-string-normalization`), and autoflake.
- `make pyupgrade` (`--py310-plus`) is optional, not part of `make check`, but CI runs it as a separate step.

Note: `black --check` prints a Python-3.10-vs-target-version parse warning on this machine; it is harmless and the files still pass.

## Building the K definition (kdist)

Kontrol's K semantics are compiled through pyk's kdist system; targets are declared in `src/kontrol/kdist/plugin.py` (registered via the `[project.entry-points.kdist] kontrol = "kontrol.kdist.plugin"` entry point).
Four Haskell-backend targets, all rooted at `kontrol.md`, differ only by which lemmas the main module pulls in:

- `kontrol.base` → module `KONTROL-BASE` (no extra lemmas).
- `kontrol.keccak` → `KONTROL-KECCAK` (Keccak assumptions).
- `kontrol.aux` → `KONTROL-AUX` (auxiliary helper lemmas).
- `kontrol.full` → `KONTROL-FULL` (everything).

Build them with `uv run kdist --verbose build -jN 'kontrol.*'`; on Linux prefix `CXX=clang++-14` (LLVM 14).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [runtimeverification/kontrol](https://github.com/runtimeverification/kontrol) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
