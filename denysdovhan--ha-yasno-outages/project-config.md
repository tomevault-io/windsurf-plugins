---
trigger: always_on
description: Agents act as senior Python collaborators. Keep responses concise,
---

# AI Coding Agents Guide

## Purpose

Agents act as senior Python collaborators. Keep responses concise,
clarify uncertainty before coding, and align suggestions with the rules linked below.

## Important directives

- In all interactions and commit messages, be extremely concise and sacrifice grammar for the sake of concision.
- If anything here is unclear, tell me what you want to do and I'll expand these instructions.
- If you struggle to find a solution, suggest to add logger statements and ask for output to get more context and understand the flow better. When logger output is provided, analyze it to understand what is going on.
- When updating this file (`agents.md`), DON'T CHANGE the structure, formatting or style of the document. Just add relevant information, without restructuring: add list items, new sections, etc. NEVER REMOVE tags, like <important> or <instruction>.
- At the end of each plan, give me a list of unresolved questions to answer, if any. Make the questions extremely concise. Sacrifice grammar for the sake of concision.
- Always use conventional commits format for commit messages.
- ALWAYS run `scripts/lint` after any changes to the code.
- ALWAYS run `scripts/test` before committing changes.

## Project Overview

This repository is a Home Assistant custom integration providing electricity outage schedules for Ukraine using the [Yasno API](https://yasno.ua). Main codebase lives under `custom_components/yasno_outages`.

### Code structure

- `translations/` - folder containing translations (en.json, uk.json).
- `__init__.py` - init file of the integration, creates entries, sets up platforms, handles entry reload/unload. Stores runtime data (API, coordinator, integration) in `entry.runtime_data` as a `YasnoOutagesData` dataclass.
- `api/` - package containing API classes for fetching data. Should be Home Assistant agnostic. Uses `aiohttp` for async HTTP requests.
  - `__init__.py` - exports `YasnoApi` facade providing unified access to planned and probable APIs.
  - `models.py` - data models: `OutageEvent`, `OutageSlot`, `OutageEventType` enum (DEFINITE, NOT_PLANNED), `OutageSource` enum (PLANNED, PROBABLE).
  - `base.py` - `BaseYasnoApi` with shared functionality (regions, providers, slot parsing).
  - `planned.py` - `PlannedOutagesApi` for fetching planned outages (today/tomorrow).
  - `probable.py` - `ProbableOutagesApi` for fetching probable outages (weekly recurring).
  - `const.py` - API-specific constants (endpoints, status values).
- `config_flow.py` - a file describing a flow to create new entries and options flow for reconfiguration. Multi-step flow: region → service (DSO) → group.
- `const.py` - a file containing constants used throughout the project. Use `homeassistant.const` for commonly used constants.
- `coordinator.py` - a data fetching coordinator (`DataUpdateCoordinator`). Fetches data from API facade, filters NOT_PLANNED events, transforms to CalendarEvents. Polls API every 15 minutes. Takes API instance as a parameter.
- `data.py` - defines runtime data types: `YasnoOutagesData` dataclass holding API, coordinator, and integration instances, and `YasnoOutagesConfigEntry` type alias for typed config entries.
- `entity.py` - a base entity class (`YasnoOutagesEntity`) that is used as a template when creating sensors and calendar. Contains important `DeviceInfo` joining different entities into a single device.
- `repairs.py` - repair flow for detecting and notifying users about deprecated configuration (API v1 → v2 migration).
- `manifest.json` - a file declaring an integration manifest.
- `sensor.py` - declares sensors using entity descriptors. Implements sensors: electricity state (enum), schedule updated timestamp, next planned outage, next probable outage, next connectivity. Retrieves coordinator from `entry.runtime_data.coordinator`.
- `calendar.py` - implements calendar entity showing outage events in a timeline format. Retrieves coordinator from `entry.runtime_data.coordinator`.
- `helpers.py` - shared helpers, e.g., `merge_consecutive_outages` used to merge adjacent outage slots before creating calendar events (always applied before status all-day events are added).
- `diagnostics.py` - exposes `async_get_config_entry_diagnostics` with coordinator/api snapshots (states, ids, raw data) for HA diagnostics download.

<instruction>Fill in by LLM assistant memory</instruction>

### Using Coordinator to Fetch Data

We use a single `DataUpdateCoordinator` per config entry that polls the Yasno API every 15 minutes. The coordinator is created in `__init__.py` during setup and stored in `entry.runtime_data` as part of the `YasnoOutagesData` dataclass. Platforms (sensors, calendar) retrieve the coordinator from `config_entry.runtime_data.coordinator`.

The coordinator:

- Receives the API instance as a parameter (dependency injection)
- Resolves region/service names to IDs on first refresh
- Fetches outage schedules for the configured region, service (DSO), and group
- Transforms API data into `CalendarEvent` objects
- Computes derived values (current state, next outage times, etc.)
- `current_event` wraps `planned.get_current_event` in try/except; use it instead of direct calls.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denysdovhan/ha-yasno-outages](https://github.com/denysdovhan/ha-yasno-outages) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
