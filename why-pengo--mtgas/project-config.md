---
trigger: always_on
description: Django web application for tracking and analyzing Magic: The Gathering Arena game statistics. Parses MTGA's Player.log file to extract match data, stores it in SQLite/PostgreSQL, and provides a dashboard with D3.js visualizations.
---

# MTG Arena Statistics Tracker - Copilot Instructions

## Project Overview

Django web application for tracking and analyzing Magic: The Gathering Arena game statistics. Parses MTGA's Player.log file to extract match data, stores it in SQLite/PostgreSQL, and provides a dashboard with D3.js visualizations.

**Tech Stack**: Django 6.0+, Python 3.10+, SQLite (default), D3.js, vanilla CSS/JS

## Build & Test Commands

### Setup
```bash
make setup              # Install dev dependencies + run migrations (installs into .venv)
make download-cards     # Download Scryfall bulk data (one-time, ~350MB)
```

### Development
```bash
make run                        # Start Django dev server on http://127.0.0.1:8000
.venv/bin/python manage.py shell  # Django shell
make import-log LOG=/path/to/Player.log  # Import MTGA log file (CLI)
# Or use web UI: http://127.0.0.1:8000/import/
# Card data management: http://127.0.0.1:8000/card-data/
```

### Testing
```bash
.venv/bin/pytest                          # Run all tests
.venv/bin/pytest tests/test_parser.py     # Run specific test file
.venv/bin/pytest tests/test_models.py::TestMatchModel  # Run specific test class
.venv/bin/pytest -k "test_parse_match"    # Run tests matching pattern
.venv/bin/pytest --cov=stats --cov=src --cov=cards  # Run with coverage
```

### Code Quality
```bash
make format      # Format with black (line-length=100) and isort
make lint        # Run flake8 (max-line-length=100)
make lint-css    # Run stylelint on CSS files
make check       # Run format-check + lint + lint-css
make ci          # Run check + test (use before commits)
# Note: make targets invoke .venv/bin/* tools directly; no need to activate venv
```

## Project Structure

### Three-Layer Architecture

1. **`src/`** - Core business logic (parser, services)
   - `parser/log_parser.py` - Parses MTGA Player.log JSON events
   - `services/scryfall.py` - Downloads/caches Scryfall bulk data
   - `services/import_service.py` - Orchestrates import workflow (used by management command and `stats/views/imports.py`)
   - `services/play_advisor.py` - Per-match strategic play analysis (per-turn suggestions)
   - `exceptions.py` - Custom exception types

2. **`stats/`** - Django app (models, views, templates)
   - `models.py` - ORM models (Match, Deck, DeckSnapshot, Card, GameAction, etc.)
   - `views/` - Django views split by domain (dashboard, decks, matches, imports)
   - `management/commands/` - CLI commands (import_log, download_cards)
   - `templates/` - Django templates
   - `static/` - CSS (`css/style.css`) and JS (`js/app.js`, `js/charts.js`)

3. **`cards/`** - Django app for physical (paper) card inventory
   - `models.py` - `PaperCard` model: stores card metadata fetched from Scryfall named-card API
   - `views.py` - `card_index`, `add_paper_card` (fuzzy name lookup → Scryfall), `paper_card_detail`
   - `templatetags/cards_extras.py` - `mana_icons` and `cmc_value` template filters
   - `urls.py` - Mounted at `/cards/` (app_name = `cards`); routes: `/`, `/add/`, `/paper/<pk>/`
   - `templates/cards/` - `index.html` (sortable/searchable table), `add_paper_card.html`, `paper_card_detail.html`

4. **`mtgas_project/`** - Django project configuration
   - `settings.py` - Django settings (INSTALLED_APPS, DATABASE, etc.)
   - `urls.py` - Root URL configuration
   - `wsgi.py` - WSGI entry point

### Key Files
- `manage.py` - Django CLI entry point
- `pyproject.toml` - Dependencies, black/isort/pytest config
- `Makefile` - Development task automation
- `data/mtga_stats.db` - SQLite database (gitignored)
- `data/cache/` - Scryfall card cache (gitignored)

## Core Concepts

### Log Parsing Flow
1. **Read**: `log_parser.py` reads Player.log line-by-line
2. **Extract**: Finds JSON events (prefixed with `[UnityCrossThreadLogger]` or timestamps)
3. **Parse**: Extracts key event types:
   - `matchGameRoomStateChangedEvent` - Match metadata (players, result, format)
   - `EventSetDeckV2` - Deck composition
   - `greToClientEvent` - In-game actions (plays, attacks, life changes)
4. **Aggregate**: Groups events by `match_id` into `MatchData` objects
5. **Import**: `import_service.py` saves to database via Django ORM

### Match Deduplication
- **Primary Key**: `match_id` (UUID from Arena logs)
- Import checks `Match.objects.filter(match_id=...).exists()` before creating
- Use `--force` flag to re-import existing matches

### Scryfall Integration
- Uses Scryfall's bulk data to map Arena's `grpId` to card names
- **Image caching**: `download_card_image()` and `get_cached_image_path()` methods
  - Downloads card images from Scryfall on demand
  - Caches locally in `data/cache/card_images/{grp_id}.jpg`
  - Batch download available in deck gallery view

### Database Schema Key Relationships
```
Match (1) ←──→ (1) DeckSnapshot ←──→ (*) DeckCard (*) ←──→ (1) Card
  ↓                    ↓
Deck (identity)    is_sideboard flag
  ↓ (1:*)
GameAction
  ↓ (1:*)
LifeChange, ZoneTransfer
```

- Each `Match` has one `DeckSnapshot` (exact cards played that match)
- `Deck` is an identity anchor; `DeckSnapshot` stores the actual card list per match

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/why-pengo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
