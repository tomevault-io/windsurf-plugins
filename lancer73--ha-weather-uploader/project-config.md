---
trigger: always_on
description: handles credentials and publishes location data; that section is not
---

# CLAUDE.md

Guidance for Claude Code when working in this repository.

## What this is

A Home Assistant custom integration that reads mapped sensor entities on
an interval and pushes observations to several public weather networks.
No polling of external services, no incoming data. Purely outbound.

## Working style for this repo

- **Minimal diffs.** Change only what the task requires. Do not
  reformat, reorder imports, or "tidy" adjacent code.
- **Say when unsure.** If provider API behaviour cannot be verified,
  say so in the response and mark it in the code or docs. Do not present
  a guess as fact. There are already unverified items in this repo
  (see below) and they are labelled as such deliberately.
- **No flattery.** Skip preamble. Lead with the answer or the diff.
- **SemVer strictly.** See Versioning below.
- **Keep a Changelog format.** See Changelog below.

## Layout

```
custom_components/weather_uploader/
├── __init__.py           setup/unload, uploader construction
├── manifest.json         version lives here too
├── const.py              DOMAIN, SENSOR_KEYS, service ids, hosts
├── config_flow.py        3-step config flow + options flow
├── coordinator.py        entity read, unit normalization, fan-out
├── binary_sensor.py      one connectivity entity per network + a data-health entity
├── sensor.py             per-network sensors: last-error status (short code) + last-success timestamp
├── diagnostics.py        redacted Download-diagnostics snapshot (credentials + coordinates redacted)
├── translations/en.json  all user-facing strings
├── brand/                icon.png + icon@2x.png (generated)
└── uploaders/
    ├── __init__.py       build_uploader() factory
    ├── base.py           BaseUploader ABC + unit helper functions
    ├── wunderground.py
    ├── wowbe.py          WOW-BE via its WeatherUnderground endpoint
    ├── cwop.py           CWOP over NATIVE APRS/TCP - not HTTP
    ├── pwsweather.py
    ├── windy.py          Windy v2, GET query, metric, WU-compatible
    └── openweathermap.py JSON POST, SI
```

## Architecture

**Data flow, once per interval:**

1. `UploadCoordinator._async_update_data` calls `read_sensors()`.
2. `read_sensors()` walks `self._map` (key → entity_id), reads each
   state, rejects non-numeric and unavailable states, then converts to
   the internal unit via `_convert()` using the entity's declared
   `unit_of_measurement`.
3. If the resulting dict is empty, the cycle is skipped. No empty
   uploads.
4. Otherwise every uploader's `send()` is awaited in parallel via
   `asyncio.gather(..., return_exceptions=True)`.
5. Results become `{"data": ..., "results": ..., "errors": ...}`, which
   the binary sensors read.

**Staleness is not optional, and it MUST use `last_reported`.**
`read_sensors()` drops any reading whose `_reported_at(state)` is older
than `max_sensor_age`. Two rules, both learned the hard way:

1. **Never `last_updated`.** HA's state machine discards a write when
   state and attributes are unchanged: it refreshes `last_reported` and
   leaves `last_updated` alone. For weather, constant values are normal
   — rain sits at 0.0 for days, solar and UV sit at 0.0 every night.
   Keying on `last_updated` drops rain from nearly every payload and
   drops solar/UV nightly. Worse, it *cannot detect the failure it was
   written for*: a healthy dry rain sensor and a dead station have
   identical `last_updated`. Only `last_reported` separates them. This
   was a real bug, caught in review — do not reintroduce it.
2. **Never remove the check.** It is the only thing between a dead
   station and publishing its last reading to NOAA forever. A stale
   value passes every other check: not unknown, not unavailable, parses
   as a float.

The default (3600s) only needs to exceed the station's *reporting*
interval, not its rate of change. It is a heuristic, not a contract.

`_reported_at()` keeps a `getattr` fallback to `last_updated` for cores
older than 2024.4. `hacs.json` requires 2024.6.0, so it should never
fire; leave it.

**Sensor mapping validation is advisory, never blocking.** Three
layers: device_class mismatch and missing-unit warnings at config time
(a confirm step shown only when there is something to flag), and a
runtime plausibility bounds-check (`PLAUSIBLE_RANGE`, `_is_plausible`)
that drops out-of-range values and lists them in `implausible_sensors`.
The DIY/template audience routinely runs sensors with no device_class
or units, so none of this may hard-block a mapping. If asked to enforce
device_class strictly, push back: it would hide valid sensors. Ranges
are wide on purpose — they catch mis-mappings and unit errors (Pascals
vs hPa), not real weather extremes. Do not tighten them to "realistic"
values.

**Two entities, two questions.** `UploadStatusEntity` answers "is the
network accepting our data". `SourceDataEntity` answers "is our data
real". They are independent: a dead station produces green uploads. Do
not merge them or derive one from the other.

**Throttling.** The coordinator polls on one global cadence, but each
uploader gates itself via `is_due()` / `mark_sent()` against
`MIN_SERVICE_INTERVAL[service]`. Networks that are not due are skipped
for that tick and keep their previous status (`_carry_forward`). Two

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lancer73/ha-weather-uploader](https://github.com/lancer73/ha-weather-uploader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
