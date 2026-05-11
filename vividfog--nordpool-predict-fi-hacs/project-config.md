---
trigger: always_on
description: - Custom Home Assistant integration; core code under `custom_components/nordpool_predict_fi/`.
---

# Nordpool Predict FI – Agent Handbook

## Snapshot
- Custom Home Assistant integration; core code under `custom_components/nordpool_predict_fi/`.
- Python 3.12 target; asynchronous I/O via Home Assistant helpers.
- External data: realized prices from [Sähkötin](https://sahkotin.fi/hours) plus forecasts from `prediction.json` and `windpower.json`.
- Integration platforms: `sensor`.
- Sample Lovelace cards in repo root (ApexCharts) for price + wind.

## Primary Modules
- `__init__.py`: sets up `DataUpdateCoordinator`, registers platforms, normalises config entry options.
- `const.py`: domain constants, default config, attribute keys.
- `coordinator.py`: fetches Sähkötin CSV + JSON artifacts, merges realized/forecast price timelines, applies Helsinki release rules, exposes `SeriesPoint`, `PriceWindow`, and `DailyAverage` dataclasses plus price/wind payloads.
- `sensor.py`: sensor entities (upcoming + now price, daily averages, upcoming + now wind) keyed off coordinator data.
- `manifest.json`: minimal metadata (version, requirements, HA integration info).
- `tests/conftest.py`: injects project root into `sys.path` so tests can import `custom_components`.

## Coordinator Facts
- `_async_update_data`:
  - Resolves timezone via `ZoneInfo("Europe/Helsinki")`; raises `UpdateFailed` if tzdata missing.
  - Calculates `data_cutoff`: today midnight Helsinki time for showing all available data from beginning of today onwards.
  - Filters price and wind data ≥ `data_cutoff` to show aligned timelines.
  - Pulls Sähkötin CSV for the current Helsinki day and merges realized rows with forecast data from today onwards.
  - Current point found from merged series (latest point ≤ now). If no point exists at or before `now`, current is left unknown (no fallback to future).
  - Cheapest windows (3h/6h/12h) calculated across the merged series beginning at today’s Helsinki midnight (realized data followed by forecast points), preferring windows whose end is still ahead of `now`.
- Wind series filtered the same way as price (from today midnight).
- Cheapest rolling windows (3h/6h/12h) are derived from contiguous hourly points across full merged data and cached for sensor use. All fixed windows respect a shared lookahead horizon (hours ahead from the current hour anchor); candidate windows must end before the horizon expires.
- Full Helsinki days (00:00-23:00) are grouped into `DailyAverage` payloads for downstream sensors and UI.
- Custom cheapest window searches honour a user-defined lookahead horizon (hours ahead from the current hour anchor); candidate windows must end before the horizon expires.
- Window attributes and lookahead limits are emitted in Helsinki local time year-round (handles DST transitions).
- Networking via `aiohttp` session + `async_timeout`.

### Time semantics (important)
- "Now" means the latest sample at or before the current time, never a future value.
- "Next X" windows start strictly at the next full hour (T+1) and span X contiguous hours (e.g., next 3h = T+1..T+3). The current hour is excluded.
- When source data lacks a past/current sample, sensors must not present a future value as "now"; they should surface `unknown`/no state and include `raw_source` for transparency.

## Entity Contracts
- Price sensors:
  - `sensor.nordpool_predict_fi_price` → attributes `forecast`, `raw_source`.
  - `sensor.nordpool_predict_fi_price_now` → attributes `timestamp`, `raw_source`.
- `sensor.nordpool_predict_fi_price_daily_average` → attributes `daily_averages`, `daily_average_span_start`, `daily_average_span_end`, `raw_source`, `extra_fees`; state is the averaged price across every hour covered by the available full Helsinki days.
- `sensor.nordpool_predict_fi_price_next_1h` → attributes `timestamp`, `raw_source` (average over next starting hour: T+1).
- `sensor.nordpool_predict_fi_price_next_3h` → attributes `timestamp`, `raw_source` (average over next 3 hours: T+1 to T+3).
- `sensor.nordpool_predict_fi_price_next_6h` → attributes `timestamp`, `raw_source` (average over next 6 hours: T+1 to T+6).
- `sensor.nordpool_predict_fi_price_next_12h` → attributes `timestamp`, `raw_source` (average over next 12 hours: T+1 to T+12).
- Wind sensors:
  - `sensor.nordpool_predict_fi_windpower` → attributes `windpower_forecast`, `raw_source`.
  - `sensor.nordpool_predict_fi_windpower_now` → attributes `timestamp`, `raw_source`.
  - Naming is unified as `windpower` everywhere (not `wind_power`).
- Cheapest price window sensors (`sensor.nordpool_predict_fi_cheapest_{3|6|12}h_price_window`) expose lowest rolling averages across the Helsinki-today merged timeline (skipping windows that ended before `now`) along with `window_start`, `window_end`, `window_points`, `window_lookahead_hours`, `window_lookahead_limit`, and `raw_source` attributes. The matching `*_window_active` sensors flip to `True` while that window includes the present hour.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vividfog/nordpool-predict-fi-hacs](https://github.com/vividfog/nordpool-predict-fi-hacs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
