---
trigger: always_on
description: These rules help AI coding agents work effectively in this repo. Keep answers concrete and project-specific.
---

# Copilot instructions for starbash

These rules help AI coding agents work effectively in this repo. Keep answers concrete and project-specific.

## Big picture
- **Starbash** automates and standardizes astrophotography workflows by organizing FITS image data, managing processing recipes via TOML "repos", and orchestrating external tools (Siril, GraXpert).
- **Entry point**: `starbash.main:app` is a Typer CLI application. Top-level commands: `select`, `info`, `process`, `repo`, `user`.
- **Core architecture**:
  - `starbash.main` — Typer CLI app with Rich markup, registers subcommand modules from `starbash.commands/`
  - `starbash.app.Starbash` — main application context manager, initializes database, repo manager, selection state, and analytics
  - `starbash.database.Database` — SQLite3-backed storage for FITS metadata (images table) and session aggregates (sessions table)
  - `starbash.selection.Selection` — persistent JSON-based state for filtering sessions by target, telescope, date range, filter, image type
  - `repo.manager.RepoManager` — separate package, loads/merges TOML repos with precedence rules, exposes `union()` (MultiDict) and `get()`
  - `starbash.tool` — tool runners for Siril, GraXpert, Python (RestrictedPython); handles safe template expansion
  - `starbash.paths` — centralized path management with test override support for config/data directories
  - `starbash.analytics` — optional Sentry.io integration for crash reports and usage analytics

## Data persistence and paths
- **User directories** (via platformdirs):
  - Config: `~/.config/starbash/` (or platform equivalent) — stores `starbash.toml` user preferences
  - Data: `~/.local/share/starbash/` (or platform equivalent) — stores `db.sqlite3` and `selection.json`
- **Test isolation**: `paths.set_test_directories(config_dir, data_dir, documents_dir_override=documents_dir)` overrides paths for test fixtures
- **Database schema**:
  - `images` table: stores FITS metadata as JSON in `metadata` column, indexed by `path`
  - `sessions` table: aggregates images by (start, end, filter, imagetyp, object, num_images, exptime_total)
  - Key constants defined in `Database` class: `EXPTIME_KEY`, `FILTER_KEY`, `START_KEY`, `OBJECT_KEY`, etc.

## CLI commands (via Typer)
- **select** — manage session filtering and display
  - `select` — show current selection summary
  - `select list` — list sessions filtered by current selection, shows totals row with bold counts
  - `select any` — clear all filters
  - `select target NAME` — filter by target name
  - `select telescope NAME` — filter by telescope/instrument name
  - `select date after|before|between DATE [DATE]` — filter by date range
  - `select export SESSION_NUM DESTDIR` — export session images via symlinks/copy
- **info** — display system and filtered data summaries
  - `info` — show user preferences location and app info
  - `info target` — list targets in current selection (with counts)
  - `info telescope` — list instruments in current selection (with counts)
  - `info filter` — list optical filters in current selection
- **process** — automated processing workflows
  - `process siril SESSION_NUM DESTDIR [--run]` — generate Siril directory tree, optionally launch GUI
- **repo add/remove/list/reindex** — manage TOML repo references
- **user name/email/analytics/setup** — manage user profile and analytics opt-in
- Console script aliases: `starbash` and `sb` (defined in `pyproject.toml`)

## Selection and filtering system
- **Selection class** (`starbash.selection.Selection`):
  - Persists state to `~/.local/share/starbash/selection.json`
  - Tracks: `targets` (list), `telescopes` (list), `date_start`, `date_end`, `filters` (list), `image_types` (list)
  - `get_query_conditions()` returns dict with keys: `OBJECT`, `TELESCOP`, `FILTER`, `date_start`, `date_end`
  - Used by `Starbash.search_session()` to filter database queries
- **Database filtering** (`Database.search_session()`):
  - Accepts conditions dict from `Selection.get_query_conditions()`
  - Extracts `date_start`/`date_end` for special `>=`/`<=` filtering on `start` column
  - Standard keys (OBJECT, TELESCOP, FILTER, IMAGETYP) use exact match

## Repos and precedence
- A "repo" is a directory with `starbash.toml` at root (examples in `doc/toml/example/recipe-repo/`)
- **Repo URLs**:
  - `file:///path/to/dir` — local directory
  - `pkg://defaults` — internal package resources (`src/starbash/defaults/`)
- Default repos listed in package defaults under `[[repo-ref]]`
- **RepoManager**:
  - Loads repos in order; later repos have higher precedence (last wins for `get()`)
  - `union()` returns MultiDict of all top-level keys
  - `get(key, default)` returns value from highest-precedence repo
  - TOML items monkey-patched with `source` (Repo instance) for relative file resolution
- **TOML Imports** (for config reuse and inheritance):
  - Repos support `[import]` tables to import nodes from other TOML files
  - Import syntax: `[target.import]` with required `node = "path.to.source"`, optional `file = "path/to/file.toml"`, optional `repo = "url"`
  - Imports are resolved during repo loading, replacing import tables with deep copies of referenced content

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [geeksville/starbash](https://github.com/geeksville/starbash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
