---
trigger: always_on
description: BavarianData — a Home Assistant (HA) custom integration (HACS) that connects HA
---

# CLAUDE.md

BavarianData — a Home Assistant (HA) custom integration (HACS) that connects HA
directly to **BMW CarData**: a live MQTT stream plus a REST API, using the
user's personal BMW client ID. Domain: `bavariandata`. Repo:
`JustChr/BavarianData`. License: MIT. Read-only — CarData cannot command the car.

## Layout

- `custom_components/bavariandata/` — the integration (everything shipped to users).
  - `api.py` — REST client (auth headers, quota accounting).
  - `device_flow.py` — OAuth 2.0 device-authorization flow (no HA imports; unit-testable).
  - `stream.py` — MQTT streaming client (paho-mqtt).
  - `coordinator.py` — central state: token refresh, stream lifecycle, quota, charging-session tracking + `bavariandata_charging_*` events, derived charged-energy sensors.
  - `config_flow.py` — setup wizard: client ID → device auth → cluster picker → in-browser bookmarklet activator (shared by guided, manual and Configure via `_StreamActivatorFlow`; BMW has **no API** for Data Selection, it's portal-only).
  - `sensor.py` / `binary_sensor.py` / `image.py` / `device_tracker.py` / `entity.py` — entity platforms. One device per VIN.
  - `descriptors.py`, `keys.py`, `units.py` — descriptor → entity mapping; `keys.py` derives the HA `translation_key` and is shared by runtime **and** generators so they can't drift.
  - `www/bavariandata-card.js` — bundled Lovelace card (vanilla JS, registered automatically by `__init__.py`; no build step). Groups entities via their `cluster`/`category` attributes, not names.
- `tools/` — catalogue generation pipeline (see below).
- `tests/` — pytest, **no Home Assistant required** (see below).
- `docs/reference/` — BMW API notes + generated field reference.
- `blueprints/automation/bavariandata/` — shipped automation blueprints.

## Generated files — never hand-edit

These are outputs of the `tools/` pipeline (inputs: BMW's catalogue exports +
the project-authored `tools/curated_titles.json` and
`tools/derived_entities.json`):

- `custom_components/bavariandata/catalogue.json`
- `custom_components/bavariandata/descriptor_metadata.py`
- `custom_components/bavariandata/translations/en.json` and `de.json` —
  **only the `entity` block**; `config`/`options` sections are hand-maintained
  and preserved by the generator
- `custom_components/bavariandata/translations/en-GB.json` — fully generated,
  a *delta* over `en.json` (HA overlays a language on top of `en` key by key, so
  only the ~75 differing strings exist). `en.json` is **US English**; the US/UK
  word list is `tools/spelling_en_gb.json`. Re-run `tools/generate_en_gb.py`
  after **any** `en.json` edit, hand-written flow strings included
- `docs/reference/telematics-fields.md`

Entities without a BMW descriptor (derived/diagnostic sensors, device tracker,
vehicle image) are named from `tools/derived_entities.json` — never a hardcoded
`_attr_name`, or German installs silently fall back to English.

To change an entity name, edit `title_en` in `tools/curated_titles.json`, then
re-run steps 1–5 from `tools/README.md` (`build_catalogue.py`,
`generate_metadata.py`, `generate_translations.py`, `generate_reference_doc.py`,
`generate_en_gb.py`) and run `python -m pytest tests/test_catalogue.py`
(checks consistency and generator idempotence).

## Tests

```
python -m pytest tests/
```

Deps: `requirements_test.txt` — light on purpose (aiohttp, pytest, PyYAML,
plus a pinned `ruff`); nothing there pulls in Home Assistant. `tests/conftest.py`
loads integration modules in isolation via a synthetic package so nothing
imports Home Assistant — keep new test targets HA-import-free, or they won't be
testable here. There is no HA test harness in this repo; config-flow/entity
behavior is verified against a live HA instance manually.

Some tests run the shipped card under Node and skip themselves when it is
absent; CI pins Node 24 so that coverage cannot silently disappear.

`tests/test_card_snapshots.py` pins a golden render of **every card view for
every drivetrain**, so a layout change shows up as a readable diff instead of
having to be spotted by eye in Home Assistant. When a change to the card is
intended, approve it with `python -m pytest tests/test_card_snapshots.py
--snapshot-update` and **read the resulting diff** — that review is the point.
The harness freezes the clock and forces English; without that the snapshots
would rot daily.

## User documentation

User-facing docs live in three tiers — keep them **in lockstep with the code**:

- `README.md` — Tier-1 shop window only (overview, screenshots, requirements,
  the 4-step quick start, links into the Wiki). Keep it lean; detail belongs in
  the Wiki, not here.
- `docs/wiki/*.md` — the full manual and the **source of truth** for the GitHub
  Wiki (a separate git repo). Publish with `bash scripts/publish-wiki.sh` after
  changing any page. Pages reference screenshots via
  `raw.githubusercontent.com/JustChr/BavarianData/main/screenshots/…`, so
  screenshot files must be committed to `main` for the images to load.
- `docs/reference/*` — generated deep reference; never hand-edit generated files.

**A feature isn't done until its docs are updated in the same change.** Any new
config-flow step, Configure/options screen, card view, service (`services.yaml`),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustChr/BavarianData](https://github.com/JustChr/BavarianData) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
