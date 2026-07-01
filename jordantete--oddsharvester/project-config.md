---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OddsHarvester is a Python web scraper that extracts sports betting odds from oddsportal.com. It uses Playwright for browser automation and BeautifulSoup/lxml for HTML parsing. Supports multiple sports (football, tennis, basketball, rugby, ice hockey, baseball, American football, handball, volleyball), various betting markets, and stores output locally (JSON/CSV) or remotely (AWS S3).

## Before You Code — Read This

**`docs/agentic-gotchas.md`** documents recurring OddsPortal-specific traps that are not deducible from the code alone — stale/phantom SSR data, silent truncation by client-side rendering (pagination ellipsis, lazy-load, URL conventions), per-bookmaker data format variation, league sponsor renames, CLI normalization layering, and anti-bot detection symptoms. Read it before:

- Adding or modifying any DOM/JSON parsing in `base_scraper.py` or `market_extraction/`
- Iterating over rendered DOM collections (pagination, listings, scroll, market dropdowns)
- Parsing or extracting bookmaker odds, names, or any per-row attribute
- Adding a new league or modifying `sport_league_constants.py` / `league_aliases.py`
- Adding a CLI option or modifying option-validation logic in `cli/commands/`
- Changing Playwright browser args, stealth scripts, or anti-detection config in `playwright_manager.py`
- Triaging a "0 results returned" symptom before assuming it's a parsing bug

When a fix exposes a new OddsPortal behaviour worth remembering, append it to `docs/agentic-gotchas.md` (criteria are listed at the bottom of that file).

## Behavioral Guidelines (Karpathy / Multica)

Bias toward **caution over speed**. For trivial tasks, use judgment. **Project rules below override these** when they conflict.

### 1. Think Before Coding

- **State assumptions explicitly.** If multiple interpretations exist, present them — don't pick silently.
- **If unclear, stop and ask.** Don't hide confusion behind plausible-looking code.
- **If a simpler approach exists, say so.** Push back when warranted.

### 2. Simplicity First

- Minimum code that solves the problem. No speculative features, no abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested. No error handling for impossible scenarios.
- If you wrote 200 lines and 50 would do, rewrite.

### 3. Surgical Changes

- Touch only what the task requires. Don't "improve" adjacent code, comments, or formatting.
- Match existing style even if you'd do it differently.
- If you spot unrelated dead code, **mention it — don't delete it**.
- Only clean up imports/symbols _your own_ changes orphaned.
- Every changed line must trace directly to the user's request.

### 4. Goal-Driven Execution

Transform tasks into verifiable goals **before** coding:

- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step work, state a brief plan with per-step verification.

## Commands

**Package manager**: uv

```bash
# Install
uv sync

# Run scraper
uv run oddsharvester scrape-upcoming --sport football --date 20250101 --markets 1x2
uv run oddsharvester scrape-historic --sport football --leagues england-premier-league --season 2022-2023 --markets 1x2

# Tests
uv run pytest tests/ -q --ignore=tests/integration/                                # unit
uv run pytest tests/integration/ -q -m integration                                 # integration (HAR replay, default)
uv run pytest tests/integration/ -q -m integration --live                          # integration (live network)
uv run pytest tests/core/test_url_builder.py::TestUrlBuilder::test_method_name -q  # single test
uv run pytest --cov=src/oddsharvester --cov-report=term --ignore=tests/integration/

# Lint / format
uv run ruff format .
uv run ruff check --fix src/

# Validate league URLs (diagnostic, requires internet)
uv run python scripts/validate_league.py -s football -l brazil-serie-a --season 2024
uv run python scripts/validate_league.py -s football --all
```

## Architecture

Four-layer flow: `CLI (cli/) → Core (core/) → Data (utils/) → Storage (storage/)`. Entry point: `oddsharvester` (or `python -m oddsharvester`).

**Core orchestration** (`src/oddsharvester/core/`):

- `scraper_app.py` orchestrates browser + scraper + storage
- `odds_portal_scraper.py` navigates pages and coordinates per-match scraping
- `playwright_manager.py` owns browser lifecycle (reads `ODDSHARVESTER_HAR_REPLAY` / `ODDSHARVESTER_HAR_RECORD`)
- `browser/` — focused helpers (`CookieDismisser`, `PageScroller`, `MarketTabNavigator`, `SelectionManager`)
- `odds_portal_market_extractor.py` + `market_extraction/` — odds extraction, submarket grouping, odds history, navigation
- `url_builder.py`, `sport_market_registry.py`, `sport_period_registry.py`, `odds_portal_selectors.py`
- `retry.py` — **canonical location for `TRANSIENT_ERROR_KEYWORDS`** and retry/backoff utilities
- `scrape_result.py`, `exceptions.py` — `ScrapeResult` / `FailedUrl` / `ScrapeStats` and exception hierarchy


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jordantete/OddsHarvester](https://github.com/jordantete/OddsHarvester) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
