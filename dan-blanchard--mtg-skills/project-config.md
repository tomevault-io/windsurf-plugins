---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### mtg-utils

```bash
cd mtg-utils
uv sync                              # Install dependencies
uv run pytest ../tests/mtg-utils/ -v  # Run tests
uv run ruff check src/ ../tests/mtg-utils/  # Lint
uv run ruff format src/ ../tests/mtg-utils/  # Format
uv run download-mtgjson              # Card-data source: MTGJSON AllPrintings + AllPricesToday (ADR-0033; ~609MB; first-run only)
uv run build-card-snapshot           # Regen the committed test card snapshot (gated; needs local MTGJSON bulk + phase card-data — auto-fetched via the phase release-server manifest, no cargo; NEVER CI)
```

### deck-wizard

```bash
cd deck-wizard
uv sync                              # Install dependencies (follows symlink to mtg-utils/src)
uv run pytest ../tests/deck-wizard/ -v  # Run smoke tests
```

### cube-wizard

```bash
cd cube-wizard
uv sync                              # Install dependencies (follows symlink to mtg-utils/src)
uv run pytest ../tests/cube-wizard/ -v  # Run smoke tests
```

### rules-lawyer

```bash
cd rules-lawyer
uv sync                              # Install dependencies (follows symlink to mtg-utils/src)
uv run pytest ../tests/rules-lawyer/ -v  # Run smoke tests
```

### deck-strat

```bash
cd deck-strat
uv sync                              # Install dependencies (follows symlink to mtg-utils/src)
uv run pytest ../tests/deck-strat/ -v  # Run smoke tests
```

### lgs-search

```bash
cd lgs-search
uv sync                              # Install dependencies (follows symlink to mtg-utils/src)
uv run playwright install chromium  # First-run only; downloads Chromium
uv run pytest ../tests/lgs-search/ -v  # Run smoke tests
```

### proxy-printer

```bash
cd proxy-printer
uv sync                              # Install dependencies (follows symlink to mtg-utils/src)
uv run pytest ../tests/proxy-printer/ -v  # Run smoke tests
```

### deck-forge

```bash
cd deck-forge
uv sync                              # Install deps (FastAPI/uvicorn; follows symlink to mtg-utils/src)
uv run pytest ../tests/deck-forge/ -v  # Run backend tests
uv run download-mtgjson              # First-run only; card-data source (MTGJSON AllPrintings, ADR-0033). loader auto-discovers it
uv run deck-forge                    # Launch the backend hub + open the browser UI
uv run deck-forge-phase-crosscheck <cards.json>  # Read-only audit: diff detectors vs phase-rs parse (auto-fetches phase card-data via the pinned-PHASE_TAG release-server manifest — no cargo)
# Frontend (only to develop the UI; the built bundle is committed under frontend/dist):
cd frontend && npm install && npm run build
```

### Running a single test

```bash
cd mtg-utils
uv run pytest ../tests/mtg-utils/test_parse_deck.py -v            # one file
uv run pytest ../tests/mtg-utils/test_parse_deck.py::test_name -v # one test
uv run pytest -k "moxfield and sideboard" ../tests/mtg-utils/ -v  # filter
```

### Python / tooling

- Requires Python 3.12+ (`requires-python = ">=3.12"` in `mtg-utils/pyproject.toml`).
- All eight `pyproject.toml` files use `uv` as the install/runtime driver.
- CI (`.github/workflows/ci.yml`) runs the exact commands listed above — it is the authoritative source of truth for which invocations must pass.

## Architecture

Mono-repo for MTG-related Claude Code skills. Each skill lives in its own directory matching the `name` field in its SKILL.md frontmatter.

**Source layout.** The canonical source lives in `mtg-utils/src/mtg_utils/`. `deck-wizard/src`, `cube-wizard/src`, `rules-lawyer/src`, `deck-strat/src`, `lgs-search/src`, and `proxy-printer/src` are **symlinks** to that directory. Editing a file through any skill's `src/` edits the shared source — there is exactly one copy. Each skill's `pyproject.toml` re-declares only the CLI entry points it ships; the Python package is installed once per skill `.venv` but all six point at the same files.

### mtg-utils

Shared Python package (`mtg_utils`). 39 CLI script modules (25 deck + 9 cube + 3 rules-lawyer + 2 proxy-printer) exposed as 40 entry points — `combo-search` and `combo-discover` both live in `combo_search.py`. `download-mtgjson` (the card-data source) is declared in every skill's `pyproject.toml`. The deck CLIs `deck-signals`, `slot-budgets`, and `deck-rank` are thin wrappers over the deterministic `_deck_forge` signal / budget / ranking core; `deck-tune` is the holistic **spine** over that core (`_tuner.tune`) — one call returns the scorecard + candidate swaps, the same engine deck-forge runs at `POST /api/tune`, and the deterministic basis for deck-wizard's Step-6 tuning (Commander family). deck-wizard's analysis pipeline reuses these instead of guessing. Each other skill's `pyproject.toml` re-declares only the subset of these CLIs it reuses (cube-wizard 12, rules-lawyer 5, proxy-printer 2, deck-strat 16); the remaining deck-only entry points live in `deck-wizard/pyproject.toml`.

**Deck scripts:**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dan-blanchard/mtg-skills](https://github.com/dan-blanchard/mtg-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
