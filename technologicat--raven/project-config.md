---
trigger: always_on
description: Local research assistant constellation. Privacy-first, 100% local.
---

# Raven - CLAUDE.md

## Project Overview
Local research assistant constellation. Privacy-first, 100% local.

**Components:**
- **Visualizer** (`raven/visualizer/`): BibTeX topic analysis, semantic clustering, keyword extraction. The original app. See `raven/visualizer/CLAUDE.md` for architecture.
- **Librarian** (`raven/librarian/`): LLM chat frontend with tree-structured branching history, hybrid RAG, tool-calling, avatar integration. See `raven/librarian/CLAUDE.md` for architecture.
- **Server** (`raven/server/`): Web API for GPU-bound ML models. Primary inference endpoint.
- **Client** (`raven/client/`): Python bindings for Server API.
- **Avatar** (`raven/avatar/`): AI-animated anime character (THA3 engine, lipsync, cel animations). Some avatar-related code (video postprocessor, colorspace) lives in Common for licensing reasons.
- **Common** (`raven/common/`): Shared utilities (video processing, audio, GUI widgets, networking). BSD-licensed; Server and Avatar pose editor are AGPL.
- **Papers** (`raven/papers/`): Academic paper tools — arXiv search/download, bibliography converters (WoS, CSV, PDF, BibTeX burst).
- **Tools** (`raven/tools/`): Miscellaneous CLI utilities (CUDA check, audio device listing, image format conversion, dehyphenation).

## Build and Development

Uses PDM with `pdm-backend`. **Python 3.11–3.12** (see `pyproject.toml`: `requires-python = "<3.13,>=3.11"`). Optional CUDA extras via `pdm install -G cuda`.

### Why the 3.12 upper cap

The cap comes from `kokoro` (Kokoro TTS) and its phonemizer `misaki`, which currently require `<3.13`. Raven's own code and every other dependency (`mcpyrate`, `unpythonic`, `torch`, `Pillow`, `numpy`, …) already support Python 3.13 and 3.14. The plan to lift the cap has two branches:

- **(a)** Kokoro/Misaki upstream expand their supported Python range — in which case we just bump `requires-python` and widen the CI matrix.
- **(b)** If those projects look dead after a reasonable wait, we vendor both. Kokoro is the TTS engine, Misaki is its English phonemizer; together they're self-contained enough to be absorbed into `raven/vendor/` alongside `tha3/`, `DearPyGui_Markdown/`, etc.

Until one of those branches lands, **don't add `3.13`/`3.14` to the CI matrix** — it would fail at dependency resolution time. The test CI currently works around this by using `pip install -e . --no-deps` and hand-picking a minimal dependency subset for the test suite, which avoids pulling in kokoro/misaki at all. That's how the test matrix can stay lightweight even though kokoro lives in the full `[project] dependencies`.

### Working-tree state: `config.py` files are edited in place

Raven is configured via in-place edits to tracked `config.py` files — paths, model choices, hardware-specific tweaks. On any dev machine, expect some subset of the following to show up as `M` in `git status` as the **normal steady state**, not as a pending change that needs committing:

- `raven/client/config.py`
- `raven/librarian/config.py`
- `raven/visualizer/config.py`

The specific files and the specific contents differ between dev machines; the pattern is the same everywhere — at least some config.py somewhere carries local overrides.

**Implication for `git add`**: add specific files by name. **Never** `git add -A`, `git add .`, or `git add raven/`. If a commit you're working on touches one of these files coincidentally (e.g. a refactor sweeps through them), check with me before staging — there may be an unrelated local override mixed in that shouldn't be part of the commit.

Version is defined in `raven/__init__.py` (`__version__`), read by PDM via `[tool.pdm.version]` in `pyproject.toml`. Tag format: `vX.Y.Z`.

```bash
pdm install              # creates .venv/ and installs deps
pdm use --venv in-project
```

Prefix commands with `pdm run` if the venv is not active.

Entry points defined in `pyproject.toml` under `[project.scripts]` — main apps are `raven-visualizer`, `raven-librarian`, `raven-server`, `raven-importer`, `raven-minichat`, `raven-xdot-viewer`, `raven-cherrypick`, `raven-conference-timer`, `raven-avatar-pose-editor`, `raven-avatar-settings-editor`.

### Running Tests

```bash
pytest                   # runs all tests (currently minimal coverage)
```

### Linting

```bash
ruff check <changed .py files>   # primary linter (config in pyproject.toml)
```

Legacy `flake8rc` also present (used by Emacs flycheck, not by CI or CC).

### Workflow Rules

1. **Lint after every code change**: `ruff check <changed .py files>`. Do this before review, testing, or committing. Catches unused imports and dead names early.

### DPG Pitfalls

See `dpg-notes.md` (project root) for the full DPG reference — threading model, callback dispatch, `split_frame` mechanics, texture upload ordering, window sizing gotchas. The notes below are the key pitfalls distilled from that reference.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Technologicat/raven](https://github.com/Technologicat/raven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
