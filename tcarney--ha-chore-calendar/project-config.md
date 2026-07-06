---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Home Assistant custom integration called **Chore Calendar** (domain: `chore_calendar`) that manages recurring household chores. Each chore list is added through Settings > Integrations (like `local_calendar` or `local_todo`), with chores managed via services. Provides native sensor/calendar entities, service-based CRUD, and built-in trigger handling. Intended for HACS distribution.

**Current state:** Core integration and card complete. See `README.md` for features and usage.

**Key files:**

- `README.md` — User-facing documentation (features, install, service examples, card config)
- `SPECS.md` — Design specification (architecture, state machines, storage schema, card design)

## Development Commands

Always use project scripts — never run `hass`, `pip`, `pytest` directly.

```bash
script/check                          # Full validation (type-check + lint + spell) — run before committing
script/lint                           # Auto-format and fix linting issues
script/type-check                     # Pyright type checking only
script/test                           # Run all tests
script/test -k test_name              # Run specific test by name
script/test tests/test_models.py      # Run specific test file
script/test --cov                     # With terminal coverage report
script/test --cov-html                # With HTML coverage report
script/test --snapshot-update         # Update Syrupy snapshots
script/develop                        # Start local HA instance (port 8123)
script/hassfest                       # Validate manifest, translations, services against HA standards
script/card/bootstrap                 # Install card npm dependencies
script/card/build                     # Build card JS (auto-bootstraps if needed)
script/card/dev                       # Watch mode — rebuild card on source changes
script/card/clean                     # Remove card node_modules/ and dist/
```

Restart HA after modifying Python files, `manifest.json`, `services.yaml`, translations, or config flow. Force restart: `pkill -f "hass --config" || true && pkill -f "debugpy.*5678" || true && ./script/develop`

Logs: live in terminal running `./script/develop`, or `config/home-assistant.log`. Set `custom_components.chore_calendar: debug` in `config/configuration.yaml`.

## Code Style

- Target the HA [Integration Quality Scale](https://developers.home-assistant.io/docs/core/integration-quality-scale/) platinum tier
- Python 3.14+, 4 spaces, 120 char lines, double quotes, full type hints, async for all I/O
- YAML: 2 spaces, modern HA syntax (no legacy `platform:` style). JSON: 2 spaces, no trailing commas
- Ruff for linting (matches HA core config), Pyright basic mode for type checking
- Google-style docstrings; comments as complete sentences with capitalization and ending period
- Import aliases: `voluptuous` as `vol`, `homeassistant.helpers.config_validation` as `cv`, `homeassistant.util.dt` as `dt_util`
- Import order: `from __future__ import annotations` → stdlib → third-party → HA core → local
- Commit messages: Conventional Commits (`feat:`, `fix:`, `chore:`, `refactor:`, `docs:`)
- Tests: `pytest` with `asyncio_mode = auto`, use `pytest-homeassistant-custom-component` fixtures
- **File size:** target 200–400 lines, max ~500 before splitting

**Never suppress checks with blanket ignores.** Use specific codes with reasons: `# noqa: F401 - reason` or `# type: ignore[attr-defined] - reason`.

## Architecture

See `SPECS.md` for full architecture details (data flow, state machines, storage schema).

### Key Conventions

- **Public API**: "item" (`create_item`, `complete_item`) — matches HA `todo` pattern
- **Internal models**: "chore" (`BaseChore`, `ScheduledChore`, `IntervalChore`, `OneshotChore`) — domain-specific
- **Domain**: `chore_calendar`, **Class prefix**: `ChoreCalendar`
- **Entities** (per config entry / list): `calendar.daily_chores` (one per list), `todo.daily_chores` (one per list), `sensor.daily_chores_<chore_name>` (one per chore). Sensor unique_id: `{entry_id}_{uid}` where uid is a standard UUID. Calendar/todo unique_id: `{entry_id}` / `{entry_id}_todo`.
- **Services over entities** for all mutations. Single-chore services accept either a sensor entity_id (chore inferred) or calendar entity_id + explicit `item` (name or UID). List-level services require the calendar entity.
- Flat modules where practical; the only sub-package is `models/` (one file per chore type). Services registered in `async_setup()`, not `async_setup_entry()`. Card source in `card/`, built JS copied to `custom_components/chore_calendar/www/`.

### Key HA Patterns

- **Services registration:** `async_setup()`, NOT `async_setup_entry()` (Quality Scale requirement)
- **Config entry data access:** `entry.runtime_data` (typed `ChoreCalendarData`)
- **Entity MRO:** `(CoordinatorEntity[ChoreCalendarCoordinator], <PlatformEntity>)` — `CoordinatorEntity` first so coordinator updates drive state; concrete platform (`CalendarEntity`, `SensorEntity`, `TodoListEntity`) second.

## Workflow Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tcarney/ha-chore-calendar](https://github.com/tcarney/ha-chore-calendar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
