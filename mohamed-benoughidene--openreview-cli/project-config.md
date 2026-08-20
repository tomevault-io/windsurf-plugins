---
trigger: always_on
description: OpenCode instructions for `openreview`. Read this before touching anything in the repo.
---

# AGENTS.md

OpenCode instructions for `openreview`. Read this before touching anything in the repo.

## Status

Python 3.12 project, pre-alpha but far along: document parsing (PDF/DOCX/clause detection), PII stripping (Presidio, encrypted mapping), AI Gateway (routing, cost tracking, registry, wizard, redaction), review pipeline (extraction → QA → comparison, memo generation, 24 bundled playbooks), chunking/retrieval/grounding/negotiation/bilateral/recovery/graph/benchmark modules, a Textual TUI, and a Typer CLI with many subcommands (`parse`, `chunk`, `ingest`, `retrieve`, `negotiate`, `export`, `precheck`, `gateway`, `playbook`, `pii`, `client`, `config`, `graph`, `benchmark`, `prompt`).

Product design lives in `products/openreview/` (gitignored) and is **preliminary, not final**. Spec-driven development via spec-kit: specs in `specs/` (001–034), deferred work tracked in `specs/DEFERRED.md` — check it before touching a module with open deferrals.

## Tracked vs. local

- **Gitignored:**  (this file — local only), `products/`, `Papers/`, `.venv/`.
- **Tracked:** `specs/`, `.specify/` (constitution at `.specify/memory/constitution.md`), `.opencode/`, everything in "Layout" below.
- **Submodule:** `.tools/ponytail` — after clone, `git submodule update --init` or `opencode.json` points at a missing path and the ponytail plugin won't load.

## Audience

The product's user audience is **intentionally not mentioned** in repo metadata (`pyproject.toml` description, `README.md`, AGENTS.md) or in spec excerpts quoted into user-facing artifacts. Strip audience references from text you draft into the repo. `products/` may mention it — internal material, not metadata.

## Layout

```
src/openreview_cli/          # Package (src layout). Entry: app.py (Typer), __main__.py
  ├─ slots.py                # VALID_SLOTS — outside gateway/ on purpose (see Gotchas)
  ├─ llm_json.py             # Shared LLM JSON/fence-stripping helper
  ├─ config/ storage/        # Auth, paths, YAML/TOML loader; SQLite layer
  ├─ parsing/                # PdfParser (PyMuPDF streaming), DocxParser, clause detector
  ├─ pii/                    # Presidio engine, recognizers, encryption, mapping, audit
  ├─ gateway/                # Model routing, registry (models.json), cost, wizard
  ├─ review/                 # Pipeline + agents + memo; playbooks/ (24 YAML)
  ├─ pipeline/ chunking/ retrieval/ grounding/ negotiation/ bilateral/ recovery/ graph/ benchmark/ prompts/
  └─ tui/                    # Textual app (app.py, screens/, domain/, widgets/)
tests/{unit,integration,fixtures,helpers}   # ~110 unit + ~93 integration files
scripts/                     # 4 standalone benchmark scripts
specs/                       # spec-kit specs 001–034 + DEFERRED.md
.github/workflows/{ci,release}.yml
```

Version is hardcoded in **two** places: `pyproject.toml` and `src/openreview_cli/__init__.py` (`__version__`). Bump both. A third site, `uv.lock` (editable package entry), is rewritten automatically by the mypy pre-commit hook's reinstall — commit it alongside (the hook will fail/rollback once if you don't stage it).

## Setup

```bash
git submodule update --init   # required — ponytail plugin
uv sync                       # runtime + dev deps into .venv
```

## Commands

```bash
uv run openreview --help
uv run pytest -m "fast" -q              # fast feedback (<1s tests, core dev loop)
uv run pytest -m "fast or slow" -q      # all offline tests except memory (safe for pre-commit)
uv run pytest -m slow                   # TUI tests only (~105, Textual run_test startup cost)
uv run pytest -m memory -q              # memory tests ALWAYS run solo (see Gotchas)
uv run pytest tests/unit/test_x.py::test_y  # single test
uv run ruff check . && uv run ruff format .
uv run mypy src/ tests/                 # strict
uv run pre-commit run --all-files       # before every commit
```

Pytest markers (pyproject.toml): `fast`, `slow`, `integration`, `e2e`, `memory`, `no_memory`, `benchmark`, `accuracy`, `network`, `live`.

**CI** (`ci.yml`, push to main + PRs): 6 parallel jobs — `lint`, `types`, `test` (unit only), `memory` (installs spaCy model), `tui`, `benchmark` (main only, `|| true`). Uses `actions/checkout@v7` + `astral-sh/setup-uv@v8.2.0`.

**Pre-commit**: hygiene hooks, `ruff --fix`, `ruff-format`, `mypy` (`uv run mypy src/ tests/`), `pytest-fast` (collect-only). `uv run pre-commit install` once per clone; sub-agents in fresh shells must verify `.git/hooks/pre-commit` exists or run `uv run pre-commit run --all-files` before `git add` and stage reformats.

**Release** (`release.yml`): tag `v*.*.*` → build → GitHub release → PyPI via **OIDC trusted publishing** (no API token; `environment: release` + required-reviewer approval gate). Publishing waits in "Review" until a human approves. Do not push release tags casually — tag protection restricts them to the owner anyway. Release chain: bump both version files (+ commit `uv.lock`) → commit → `git tag vX.Y.Z` → push tag → approve in Actions → verify on PyPI.

## Benchmarks & measurements

Quick sanity checks and accuracy benchmarks. All are runnable locally (some need configured gateway or downloaded corpus).

### Quick profiling

```bash
# CLI startup latency + RSS

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mohamed-benoughidene/openreview-cli](https://github.com/mohamed-benoughidene/openreview-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
