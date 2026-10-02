---
trigger: always_on
description: Act as a concise, senior Python and Home Assistant collaborator. Confirm
---

# AI Coding Agents Guide

## Purpose

Act as a concise, senior Python and Home Assistant collaborator. Confirm
uncertainties before changing behavior, prefer the smallest correct diff, and
ground decisions in current repository code, provider data, and Home Assistant
documentation.

## Important directives

- Keep replies and commit messages concise and concrete.
- Ask before making a significant product or architecture decision when the
  requirement is ambiguous.
- Add narrow debug logging and request the resulting output when runtime behavior
  cannot be established from code and tests. Do not guess.
- Commit only when directly asked. Use conventional commit messages.
- When updating `AGENTS.md`, preserve its structure and style. Add or correct
  relevant facts without rewriting unrelated sections.

<instruction>Keep this guide synchronized with implemented architecture and tooling.</instruction>

## Design Log

- Before repository work, read `.agents/log/index.md`.
- Search `.agents/log/` by touched paths and 2-3 task keywords, then read matching
  entries in full.
- Treat `done` entries as binding decisions and `wip` entries as current direction.
  Newer decisions win; surface conflicts before proceeding.
- Never edit a `done` entry. Keep a matching `wip` entry and the index current for
  significant feature work; skip routine chores.
- Do not mark the initial integration entry `done` until the user explicitly
  confirms the feature is complete.

## Project Overview

This repository implements the Home Assistant custom integration **Ukrainian
Hydrometeorological Center** (`ukr_hmc`). It exposes weather observations,
forecasts, radiation measurements, and daily hydrology observations from
[meteo.gov.ua](https://www.meteo.gov.ua/). Integration code lives in
`custom_components/ukr_hmc`.

One integration config entry owns multiple typed subentries. Weather stations,
exact forecast locations, radiation monitoring stations, and hydrology posts use
separate subentry and device types. Keep future provider products in sibling
types.

### Code structure

- `__init__.py` - creates the shared API client and coordinator, stores typed
  `entry.runtime_data`, and forwards sensor and weather platforms.
- `api/` - Home Assistant-independent async client, constants, errors, immutable
  data models, and parsers. Keep it ready for extraction to a standalone package.
- `condition.py` - maps Ukrainian provider descriptions to canonical Home
  Assistant weather conditions.
- `config_flow.py` - creates the single service entry and typed weather,
  radiation, and hydrology subentries.
- `const.py` - integration constants, subentry types, and the 15-minute update
  interval.
- `coordinator.py` - fetches shared weather, radiation, and hydrology snapshots
  plus direct forecasts for configured map locations.
- `data.py` - `UkrHMCRuntimeData` and the typed `UkrHMCConfigEntry` alias.
- `entity.py` - shared weather data access, availability, and device metadata.
- `icons.json` - frontend icons for generic sensor types and canonical weather
  condition states.
- `sensor.py` - weather, radiation, and hydrology sensor descriptions and
  entities.
- `weather.py` - current weather plus forecast modes supported by each location
  type.
- `translations/` - English and Ukrainian UI strings.
- `tests/` - focused API, condition, config-flow, coordinator, entity, and setup
  coverage.
- `meteo.md` - reverse-engineering notes for provider schemas and additional
  researched endpoints. The JSON lookup files preserve researched icon and wind
  data.

## Architecture contracts

### Provider API isolation

- Keep `custom_components/ukr_hmc/api/` free of Home Assistant imports.
- Inject an `aiohttp.ClientSession`; keep all HTTP and provider parsing inside the
  API package.
- Convert provider payloads to typed Python models before returning them to the
  coordinator. The coordinator and entities must not depend on raw field names.
- Keep provider snapshots immutable. Preserve provider fields in API models even
  when Home Assistant cannot expose them natively.
- Parse JSON-compatible JavaScript assignments as data. Never evaluate or execute
  provider JavaScript.

### Runtime data and polling

- Use one shared `UkrHMCClient` and one `UkrHMCCoordinator` per config entry.
- Store both in `entry.runtime_data`; do not introduce globals or singleton state.
- The coordinator downloads the global station, observation, forecast, lookup,
  and day/night data once every 15 minutes when weather-station subentries need
  it, the radiation catalog and snapshot when radiation-station subentries need
  them, the hydrology catalog and daily snapshot when hydrology-post subentries
  need them, plus one direct forecast for each weather-location subentry. Do not
  add per-location coordinators or duplicate global requests.
- Entities read cached coordinator data only. Never perform I/O in entity
  properties or forecast callbacks.
- Convert provider failures to the appropriate Home Assistant coordinator or
  config-flow errors while preserving useful exception context.

### Typed subentries

- Catalog records are physical meteorological stations, not cities.
- Use `weather_station` for physical stations and `weather_location` for exact

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denysdovhan/ha-ukr-hmc](https://github.com/denysdovhan/ha-ukr-hmc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
