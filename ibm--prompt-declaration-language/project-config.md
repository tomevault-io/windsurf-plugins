---
trigger: always_on
description: Guidance for coding agents working in the Prompt Declaration Language (PDL) repository.
---

# AGENTS.md

Guidance for coding agents working in the Prompt Declaration Language (PDL) repository.

## What this project is

PDL is a declarative YAML-based language for writing LLM prompts and programs. The repo contains:

- A **Python interpreter** (`src/pdl/`) — the reference implementation, published to PyPI as `prompt-declaration-language`.
- A **Rust/Tauri + React viewer** (`pdl-live-react/`) — the desktop GUI, plus a partial Rust interpreter and a `compile` subcommand.
- **Examples**, **docs**, and a test suite that runs the examples end to end.

The upstream project is `IBM/prompt-declaration-language`; documentation is published at
<https://ibm.github.io/prompt-declaration-language/>.

## Repository layout

| Path | What lives there |
| --- | --- |
| `src/pdl/pdl_ast.py` | The Pydantic model of the language. Every block type (`text`, `model`, `if`, `repeat`, `call`, …) is a class here. Start here to understand or extend the language. |
| `src/pdl/pdl_interpreter.py` | The evaluator. `process_block` dispatches to `process_leaf_block` / `process_structured_block`. Largest file in the repo. |
| `src/pdl/pdl.py` | CLI entry point (`pdl`) and the Python API: `exec_program`, `exec_file`, `exec_str`, `exec_dict`. |
| `src/pdl/pdl_parser.py` | YAML → AST parsing, raises `PDLParseError`. |
| `src/pdl/pdl_dumper.py` | AST → YAML/JSON, used for traces. |
| `src/pdl/pdl_linter.py` | `pdl-lint` CLI; configured via `[tool.pdl-lint]` in `pyproject.toml`. |
| `src/pdl/pdl_llms.py`, `pdl_openai.py`, `pdl_granite_io.py` | Model backends (LiteLLM, OpenAI, granite-io). |
| `src/pdl/optimize/` | AutoPDL prompt optimizer (`pdl-optimize`). |
| `src/pdl/pdl_compilers/` | Compilation targets (e.g. `to_regex`). |
| `src/pdl/pdl-schema.json` | **Generated** JSON Schema — see "Generated files" below. |
| `src/pdl/pdl_stdlib.pdl`, `pdl_stdlib.py` | PDL standard library. |
| `tests/` | Pytest suite. `tests/data/` holds fixtures, `tests/results/` holds expected example outputs. |
| `examples/` | ~130 `.pdl` programs, all exercised by CI. |
| `docs/` | MkDocs site (`docs/tutorial.md` is the main language reference). |
| `pdl-live-react/` | Viewer: React/TypeScript front end, `src-tauri/` Rust back end. |
| `contrib/prompt_library/` | Community PDL snippets. |

## Setup

Python 3.11+ is required (CI matrix: 3.11, 3.12, 3.14; mypy targets 3.12).

```bash
python -m venv .venv && source .venv/bin/activate
pip install -e ".[all]"     # or ".[dev]" for just the test/lint tooling
pre-commit install
```

Verify the install:

```bash
pdl examples/demo/1-hello.pdl
```

## Common commands

```bash
# Unit tests (fast; excludes the example-running suite)
pytest tests --ignore=tests/test_examples_run.py

# All static checks: isort, black, flake8, pylint, bandit, mypy, pyright
pre-commit run --all-files

# Run a PDL program, with a trace for the viewer
pdl --trace out.json path/to/program.pdl

# Docs preview
mkdocs serve

# Viewer (from pdl-live-react/)
npm ci
npm test            # lint + typecheck + prettier + playwright
npm start           # Tauri dev window
npm run test:interpreter   # cargo test for the Rust interpreter
```

Style is enforced by tooling, not by hand: black formatting, isort with the black profile,
flake8 (`max-line-length = 89`, E203/E501 ignored), pylint with `pylintrc`, mypy, and pyright
over `src`, `tests`, `examples`, `docs`. Run `pre-commit run --all-files` before committing —
CI runs exactly this.

## Generated files — keep them in sync

- **`src/pdl/pdl-schema.json`** is derived from `pdl_ast.py`. `tests/test_schema.py` asserts they
  match, so **any change to the AST models requires regenerating it**:

  ```bash
  python -m src.pdl.pdl --schema > src/pdl/pdl-schema.json
  ```

  **Regenerate with Python > 3.11** (3.12 or later). Python 3.11 emits a different schema, so a
  file generated there will not match. CI reflects this: `build.yml` passes
  `--ignore=tests/test_schema.py` on the 3.11 matrix entry and only checks the committed schema on
  the newer versions.

- **`pdl-live-react/src/pdl_ast.d.ts`** is generated from that schema. Regenerate with
  `npm run types` in `pdl-live-react/` when the AST changes.

- **`src/pdl/_version.py`** is written by setuptools-scm. Never edit it.

## Tests and examples

The suite has two halves:

1. **Unit tests** (`tests/test_*.py`) — no network, run everywhere. Most build a program as a YAML
   string or dict and assert on `exec_str(...)` / `exec_dict(...)`. Follow that pattern for new
   language features.

2. **Example runs** (`tests/test_examples_run.py`) — executes every `.pdl` file in the repo against
   real models and string-matches the output. This is the nightly job and is skipped by the fast
   `build.yml` run, so run it deliberately:

   ```bash
   pytest --capture=tee-sys -rfE -s tests/test_examples_run.py --disable-pytest-warnings
   ```

`tests/test_examples_run.yaml` drives it:

- `check:` — restrict the run to a subset of files (leave `[]` for everything). Useful locally.
- `skip:` — files never run.
- `with_inputs:` — files needing `stdin` lines and/or an initial `scope`.
- `expected_parse_error:` / `expected_runtime_error:` — files that are supposed to fail.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IBM/prompt-declaration-language](https://github.com/IBM/prompt-declaration-language) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
