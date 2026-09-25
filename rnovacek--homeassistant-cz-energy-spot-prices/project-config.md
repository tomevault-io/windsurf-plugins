---
trigger: always_on
description: This Home Assistant custom integration exposes Czech OTE electricity and gas spot prices, currency/unit conversions, buy/sell templates, and cheapest-price windows. Electricity supports hourly and quarter-hour intervals; gas uses daily prices.
---

# AGENTS.md - AI Coding Agent Guide

## Project and sources of truth

This Home Assistant custom integration exposes Czech OTE electricity and gas spot prices, currency/unit conversions, buy/sell templates, and cheapest-price windows. Electricity supports hourly and quarter-hour intervals; gas uses daily prices.

- Use the Python range and development dependencies in [pyproject.toml](pyproject.toml). Do not duplicate version pins here or install additional checkers implicitly.
- Runtime requirements and the integration release version belong in [manifest.json](custom_components/cz_energy_spot_prices/manifest.json). Preserve semantic versioning and the `X.Y.ZbN` beta convention; change the release version only when the task calls for it.
- Check [the test workflow](.github/workflows/tests.yaml) for CI environment preparation and commands.
- Read the affected implementation and nearby tests before editing. Preserve unrelated work and keep changes scoped to the requested behavior.

## Architecture and implementation references

All integration modules below are under `custom_components/cz_energy_spot_prices/`.

| File | Responsibility and useful examples |
| --- | --- |
| [__init__.py](custom_components/cz_energy_spot_prices/__init__.py) | Setup/unload, shared coordinator consumers, config-entry and entity-ID migration, subentry-to-runtime conversion |
| [coordinator.py](custom_components/cz_energy_spot_prices/coordinator.py) | Shared `SpotRateCoordinator` and `FxCoordinator`, per-entry `EntryCoordinator`, `EntryConfig`, interval/trade data, persistence and retries |
| [spot_rate.py](custom_components/cz_energy_spot_prices/spot_rate.py) | OTE SOAP/XML client, UTC interval parsing, gas placeholder handling |
| [cnb_rate.py](custom_components/cz_energy_spot_prices/cnb_rate.py) | CNB FX client and previous-day rate fallback |
| [cheapest_blocks.py](custom_components/cz_energy_spot_prices/cheapest_blocks.py) | `PriceBlockSearch`, validation, legacy block conversion, window calculations |
| [config_flow.py](custom_components/cz_energy_spot_prices/config_flow.py) | Parent configuration/options and native price-block subentry flows |
| [const.py](custom_components/cz_energy_spot_prices/const.py) | Shared constants and enums |
| [spot_rate_mixin.py](custom_components/cz_energy_spot_prices/spot_rate_mixin.py) | Shared entity updates; the constructor requires `hass`, `coordinator`, `device_id`, and `trade` |
| [sensor.py](custom_components/cz_energy_spot_prices/sensor.py) | Follow `SpotRateElectricitySensor` or `TodayGasSensor` and their existing base classes for price entities |
| [binary_sensor.py](custom_components/cz_energy_spot_prices/binary_sensor.py) | Follow `SearchBasedCheapestElectricitySensor` for search entities; also owns shared tomorrow-data entities |

Data flows from the OTE/CNB clients through shared source coordinators to `EntryCoordinator`, which selects intervals, converts prices, and applies templates. Entities consume coordinator data.

### Async work, schedules, and lifecycle

- Keep HTTP access in API clients invoked by source coordinators. Entity properties and per-entry calculations must not fetch data or block the event loop.
- Reuse Home Assistant's HTTP session via `async_get_clientsession(hass)`; clients must not close an injected session. Preserve standalone client session ownership behavior.
- Preserve existing publication schedules, jitter, capped exponential retries, and CNB fallback behavior. Read scheduling constants and methods before changing them; do not add independent polling loops.
- OTE publication checks use Prague time. FX refreshes use Home Assistant local midnight; this is the integration's refresh schedule, not a claim about CNB publication time.
- Preserve valid cached data on transient failures and the existing persistence format. Missing required FX rates must not produce fabricated converted prices.
- Unloading or reloading one entry must not stop shared coordinators still used by another. Release timers, retry callbacks, and listeners when their owner stops or the final shared consumer unloads. Preserve shared entity ownership during reloads.

## Code conventions and behavior contracts

- Add type hints to new or changed interfaces and follow nearby Home Assistant patterns. Use `typing.override` for overridden methods, `typing.final` for classes intended to be final, and `attr.dataclass` for data classes.
- Reuse the concrete implementations linked above instead of introducing parallel abstractions. Keep type suppressions narrow and explain why they are needed; do not silence a checker globally to accommodate a change.

### Readability and reuse

- Keep entity classes thin. Put parsing, conversion, and window selection in clients, coordinators, or calculation helpers; entities should primarily expose state, attributes, and availability.
- Separate calculations from orchestration. Prefer pure functions for calculations that need no Home Assistant state, passing time and configuration explicitly so behavior is easy to understand and test.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rnovacek/homeassistant_cz_energy_spot_prices](https://github.com/rnovacek/homeassistant_cz_energy_spot_prices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
