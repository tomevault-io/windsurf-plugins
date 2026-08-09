---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

WordFormat is a Python CLI tool that checks and auto-corrects formatting in academic Word documents (.docx). It uses an ONNX BERT model to classify paragraphs (heading, abstract, body text, keywords, references, etc.), then validates and applies formatting rules defined in a YAML config file.

**Entry points**: `wordf` and `wordformat` commands both map to `wordformat.cli:main`.

## Git conventions

- **不要添加 `Co-Authored-By` 到 commit message 中。**

## Build & test commands

```bash
# Install dev environment (creates venv, installs deps, downloads ONNX model)
make install

# Run all tests (with coverage, must reach 87%)
make tests

# Run a single test file
pytest tests/rules/test_abstract.py -v

# Run a specific test
pytest tests/rules/test_abstract.py::TestAbstractTitleContentENBase -v

# Run tests matching a keyword
pytest tests/ -k "Abstract" -v

# Lint & format
ruff check src/
ruff format src/

# Run API server locally
make server
# or: wordf startapi

# Lint only (no tests)
make lint

# Build distributable package
make build

# Build Vue UI and copy into api/static
make build-ui

# Clean build artifacts
make clean

# Run pre-commit checks on all files
pre-commit run --all-files
```

## Install variants

```bash
pip install -e "."              # core (includes FastAPI server)
pip install -e ".[test]"        # core + pytest plugins
pip install -e ".[dev]"         # everything (test, pre-commit, ruff, pyinstaller)
```

## Pre-commit hooks

Pre-commit runs on `pre-commit` and `pre-push` stages, configured in `.pre-commit-config.yaml`:

- **sync-version** — mirrors `pyproject.toml` version into `_version.py`
- **end-of-file-fixer** / **trailing-whitespace** / **debug-statements** / **check-yaml** / **pretty-format-json** — standard fixers
- **pyupgrade** — auto-modernizes Python syntax (`--py3-plus`)
- **ruff** — lint + format (auto-fix)

## Ruff config (pyproject.toml)

- **Line length**: 108 (pycodestyle), 200 (docstrings)
- **Complexity**: max 10 (mccabe)
- **Quote style**: double, space indent
- **Per-file ignores**: `__init__.py` (F401/F403/E501), `tree.py`/`cli.py` (T201 print), `body.py`/`heading.py`/`numbering.py`/`_text.py` (C901 complexity)

## Environment variables

| Variable | Purpose | Default |
|----------|---------|---------|
| `WORDFORMAT_BASE_DIR` | Override working directory | auto-detected from project root |
| `WORDFORMAT_API_KEY` | API key for model service | `""` |
| `WORDFORMAT_MODEL` | Model identifier | `""` |
| `WORDFORMAT_MODEL_URL` | Model service URL | `""` |
| `BATCH_SIZE` | ONNX inference batch size | `64` |
| `HOST` / `PORT` | API server bind | `127.0.0.1` / `8000` |

## Architecture: core data flow

The tool operates in two phases that are intentionally separated so users can inspect and manually adjust the intermediate JSON before applying formatting:

```
wordf gj (generate JSON)          wordf cf / wordf af (check/apply format)
─────────────────────          ───────────────────────────────────
.docx → ONNX classify →        JSON tree → match paragraphs by
        flat JSON per para               position order (zip) →
                                          per-node style check/apply
```

**Phase 1 — Classification** (`classify/tag.py` → `base.py`):
- `DocxBase.parse()` loads a .docx, iterates paragraphs, batches them through ONNX BERT inference (`agent/onnx_infer.py`), and returns a flat list of `{category, text, score, comment}` dicts saved as JSON.

**Phase 2 — Tree building** (`structure/`):
- `DocumentBuilder.build_from_json()` feeds the flat JSON list into `DocumentTreeBuilder.build_tree()`, which creates a hierarchical `FormatNode` tree. `LEVEL_MAP` in `structure/settings.py` maps category strings to numeric levels; `CATEGORY_TO_CLASS` is auto-built from `@register` decorators on `FormatNode` subclasses.
- `node_factory.create_node()` instantiates the right `FormatNode` subclass and calls `load_config()`, which walks the YAML dict along `NODE_TYPE` path, merges with class `DEFAULTS`, and stores a `DotDict`.

**Phase 3 — Matching & formatting** (`pipeline/stages.py`):
- Each paragraph in the document is matched to a tree node by position order: `_flatten_tree_nodes()` converts the tree to a flat DFS list, then `zip()` pairs nodes with `document.paragraphs` by index.
- Before formatting, `node.apply_replace(doc)` checks for a `replace` field in the JSON value dict; if present, it substitutes the paragraph's run text with the replacement string.
- The tree is also mutated: `promote_bodytext_in_subtrees_of_type()` replaces generic `body_text` nodes under specific parents (e.g. `AbstractTitleCN`) with typed content nodes (e.g. `AbstractContentCN`).
- `apply_format_check_to_all_nodes()` recursively traverses the tree. For each node it calls `node.check_format(doc)` or `node.apply_format(doc)`, which delegate to `node._base(doc, p, r)` — the boolean flags control whether paragraph styles (`p`) and run styles (`r`) are checked (diffed) or applied (written).

**Phase 4 — Numbering** (apply mode only, `numbering.py`):

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AfishInLake/WordFormat](https://github.com/AfishInLake/WordFormat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
