---
trigger: always_on
description: This file contains development guidance for AI agents and contributors working on Pivotal.
---

# Agent Instructions

This file contains development guidance for AI agents and contributors working on Pivotal.

For user-facing Pivotal language syntax and examples, see `PIVOTAL.md`. Keep this file focused on development process, project structure, testing expectations, and release hygiene.

## Project Context

Pivotal is a pipeline-oriented data transformation DSL for Python and Jupyter. The package published to PyPI is `pivotal-lang`.

The implementation currently lives mostly in `pivotal/dsl_parser.py`, with supporting validation, magic commands, CLI helpers, and tests elsewhere in the repo.

## Important Files

- `PIVOTAL.md`: canonical user-facing syntax reference.
- `README.md`: PyPI/GitHub landing page and high-level examples.
- `CHANGELOG.md`: notable user-visible changes, especially anything needed for the next release notes.
- `pyproject.toml`: package metadata, dependencies, extras, and version.
- `pivotal/dsl_parser.py`: Lark grammar, transformer/AST handling, backend code generation, SQL CTE generation, parser execution/export logic.
- `pivotal/validator.py`: semantic validation and normalization after parsing, before code generation.
- `pivotal/magic.py`: Jupyter/IPython magic integration and widget-facing behavior.
- `pivotal/__main__.py`: command-line compile/export behavior.
- `pivotal/errors.py`: user-facing error translation and formatting.
- `editors/vscode/package.json`: VS Code extension metadata, commands, version, and build scripts.
- `editors/vscode/syntaxes/`: VS Code TextMate grammars for `.pivotal` files and `%%pivotal` Python injections.
- `editors/jupyterlab/package.json`: JupyterLab extension JavaScript package metadata and build scripts.
- `editors/jupyterlab/pyproject.toml`: Python package metadata for the `pivotal-lab` JupyterLab extension.
- `editors/jupyterlab/build.ps1`: Windows-friendly JupyterLab extension build script.
- `tests/test_commands.py`: core parser and pandas behavior tests.
- `tests/test_commands_polars.py`: Polars backend tests.
- `tests/test_commands_duckdb.py`: DuckDB backend tests.
- `tests/test_phase5_sql_cte.py`: SQL CTE backend tests.
- `tests/jupyter_demo_test.py`: Playwright-based Jupyter demo regression test.
- `docs/syntax/`: detailed syntax documentation by topic.
- `docs/jupyter.md`: Jupyter-specific documentation.

## When Changing The Grammar

If Pivotal syntax changes, check whether each surface below needs updating.

- Lark grammar: `grammar_indented` in `pivotal/dsl_parser.py`.
- Transformer / AST handling: `DSLTransformer` in `pivotal/dsl_parser.py`.
- Semantic validation and normalization: `pivotal/validator.py`.
- Backend code generation: `CodeGenerator` methods in `pivotal/dsl_parser.py`.
- Pandas backend behavior: `generate_*_pandas` methods in `pivotal/dsl_parser.py`.
- Polars backend behavior: `generate_*_polars` methods in `pivotal/dsl_parser.py`.
- DuckDB backend behavior: `generate_*_duckdb` methods in `pivotal/dsl_parser.py`.
- SQL CTE backend behavior: `generate_*_sql` methods and `_generate_code_sql` in `pivotal/dsl_parser.py`.
- Jupyter magic behavior, if syntax is exposed through notebooks: `pivotal/magic.py`.
- CLI compile/export behavior, if syntax affects file/notebook export: `pivotal/__main__.py`.
- Parser and pandas tests: `tests/test_commands.py`.
- Backend tests: `tests/test_commands_polars.py`, `tests/test_commands_duckdb.py`, and `tests/test_phase5_sql_cte.py`.
- Error tests: `tests/test_errors.py`.
- Syntax reference: `PIVOTAL.md`.
- User docs: relevant files under `docs/syntax/`, plus `docs/jupyter.md` when notebook behavior changes.
- README examples: `README.md`.
- Release notes: `CHANGELOG.md` under `Unreleased`.
- Shared editor/Pygments syntax tokens: `pivotal/syntax_tokens.json`; regenerate VS Code and JupyterLab syntax assets with `python scripts/generate_syntax_assets.py`.
- Editor extension builds: rebuild VS Code and JupyterLab extension artifacts after updating syntax highlighting, autocomplete, or grammar-adjacent editor behavior, then test the rebuilt extensions rather than only the source files.
- Binder/Jupyter demo repo: sibling checkout such as `C:\Code_win\pivotal-demo`.

## Testing Expectations

Run the smallest useful test set while developing, then broaden testing when the change touches shared syntax or backend behavior.

Useful commands:

```powershell
python -m pytest tests/test_commands.py
python -m pytest tests/test_commands_polars.py
python -m pytest tests/test_commands_duckdb.py
python -m pytest tests/test_phase5_sql_cte.py
python -m pytest
```

For grammar changes, prefer running the full test suite before finishing. If optional backend dependencies are missing, report which tests could not be run and why.

## Binder Demo Repo

The Binder demo for Pivotal in JupyterLab lives in a separate repository. The local development convention is to keep it beside this repo, for example:

- Main package repo: `C:\Code_win\pivotal-py`
- Demo repo: `C:\Code_win\pivotal-demo`

Grammar or user-facing behavior changes may require updating the demo repo so the Binder notebook still runs. When changing syntax, examples, Jupyter behavior, package startup, or dependencies, check whether the demo notebook and Binder config need corresponding changes.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nealbob/pivotal-py](https://github.com/nealbob/pivotal-py) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
