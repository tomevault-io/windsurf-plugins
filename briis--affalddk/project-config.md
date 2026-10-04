---
trigger: always_on
description: Home Assistant custom integration (HACS) that fetches waste pickup data for
---

# AGENTS.md — affalddk

Home Assistant custom integration (HACS) that fetches waste pickup data for
Danish municipalities. The core is the standalone library `pyaffalddk`, which
talks to a different backend API per municipality. Codeowners: @briis,
@TermeHansen.

## Commands

```bash
# Tests (no Home Assistant install needed — conftest.py stubs it)
CI=true TZ=Europe/Copenhagen pytest tests --disable-warnings

# Lint (CI runs exactly this)
ruff check --config pyproject.toml

# Live probe of a single municipality (network required)
python scripts/async_test_module.py Aarhus --zipcode 8000 --street Rådhuspladsen --number 2 --pickup
python scripts/async_test_module.py --municipalities   # list municipalities

# Weekly live API check (run from a server inside Denmark — see "CI")
python scripts/weekly_api_check.py [--notify <webhook-url>]
```

Environment: uv (`uv sync`, deps in `pyproject.toml`, locked by `uv.lock`) —
CI uses it too. A standalone conda env with the same packages is kept at
`scripts/conda_env.yaml` for local mamba users (python 3.12, aiohttp,
pytest, pytest-asyncio, freezegun, requests, beautifulsoup4, ical, ruff).
Ruff config lives in `pyproject.toml` (`E, F, T, B, S`; `T201`, `S101`,
`E501`, `B006` ignored — prints and asserts are fine here).

## Layout

```
custom_components/affalddk/        The HA integration (HACS payload)
├── __init__.py                    Coordinator (AffaldDKDataUpdateCoordinator)
├── sensor.py / calendar.py        Entities driven by the coordinator
├── config_flow.py                 Address search + municipality setup
├── const.py                       Integration constants + TRANSLATIONS (i18n)
├── i18n/ translations/ images/    UI strings and entity pictures
└── pyaffalddk/                    The standalone library
    ├── api.py                     GarbageCollection orchestrator + APIS map
    ├── interface.py               One class per provider backend
    ├── municipalities.py          Municipality code/name → provider mapping
    ├── const.py                   SUPPORTED_ITEMS, ICON_LIST, NAME_LIST, regexes
    ├── data.py                    PickupEvents, PickupType, AffaldDKAddressInfo
    └── supported_items.json       Serialized SUPPORTED_ITEMS (kept in sync)
tests/
├── conftest.py                    Stubs the homeassistant package (no HA needed)
├── test_api.py                    Unit tests + smoketest (mocked get_garbage_data)
├── test_interface.py              Per-provider tests, LIVE API calls
├── test_calendar.py / test_sensor.py  Entity tests with fixture data
└── data/                          Fixtures: *.data (json/ics), *.p (pickle),
                                   compare_data.p, smoketest_garbage_data.p,
                                   smoketest_fractions.json, const_tests.py
scripts/                           Dev helpers (async_test_module.py — live probe,
                                   weekly_api_check.py — live API check, see "CI",
                                   random_regression.py — random-address sweep,
                                   conda_env.yaml — standalone conda env,
                                   README.md — setup + usage)
.github/workflows/                 CI (see "CI" below)
```

## How data flows

`GarbageCollection(municipality)` → looks up the provider in
`MUNICIPALITIES_LIST` (`municipalities.py`), instantiates the matching class
from the `APIS` map in `api.py` → `get_address_list()` / `get_address()` give
an address ID → `get_pickup_data(address_id)` calls the provider's
`get_garbage_data(address_id)`, normalizes rows into `PickupType` events via
`update_pickup_event()` → `set_next_event()` adds the synthetic
`next_pickup` entry.

Fraction names from the APIs are Danish. They are mapped to internal keys by
`get_garbage_types()` in `api.py` using `SUPPORTED_ITEMS` (exact match after
`clean_fraction_string()`), `SPECIAL_MATERIALS`, and `NON_SUPPORTED_ITEMS`.
Unknown fractions print `missing: ...` and warn — or raise when the
`GarbageCollection` was created with `fail=True`. When a provider renames a
fraction, add the new spelling to `SUPPORTED_ITEMS` in `const.py` (and to
`tests/data/const_tests.py` if it fits a category).

## Testing model — read before touching tests

- `tests/test_interface.py` hits **live APIs** (adressevaelger.dk plus each
  municipality's backend). It needs network; failures can be real API changes,
  not code bugs. **Several municipality endpoints are geo-blocked outside
  Denmark** and hang (until timeout) from foreign runners — that is why the
  weekly API check is a local script (`scripts/weekly_api_check.py`), not a
  GitHub Actions job.
- Results are compared against pickled baselines with
  `update_and_compare(name, data, UPDATE)`. To refresh a baseline, set
  `UPDATE = True` at the top of the file, run once, set it back to `False`,
  and commit the regenerated `tests/data/compare_data.p`.
- `get_garbage_data` is monkeypatched with fixture data in every test; the
  live calls are the address-list/address lookups. Live garbage-data pulls
  are not part of the test suite — they live in
  `scripts/weekly_api_check.py` (see below). Only the VestFor address block
  is additionally gated on `CI=true` (`if not CI:` in `test_interface.py`;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [briis/affalddk](https://github.com/briis/affalddk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-04 -->
