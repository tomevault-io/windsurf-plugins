---
trigger: always_on
description: Fast orientation for AI agents working on this repo. Humans should start with [`CONTRIBUTING.md`](CONTRIBUTING.md), which documents the architecture, conventions, test layout, and release process in full. This file complements that with quick-navigation pointers for agents starting cold; defer to CONTRIBUTING.md when in doubt.
---

# AGENTS.md

Fast orientation for AI agents working on this repo. Humans should start with [`CONTRIBUTING.md`](CONTRIBUTING.md), which documents the architecture, conventions, test layout, and release process in full. This file complements that with quick-navigation pointers for agents starting cold; defer to CONTRIBUTING.md when in doubt.

Sections 2, 3, and 5 mirror the canonical text in [`pymyhondaplus/AGENTS.md`](https://github.com/enricobattocchi/pymyhondaplus/blob/main/AGENTS.md) — update there first, then propagate.

## 1. What this repo is

The Home Assistant custom integration for My Honda+ / Honda Connect Europe vehicles, distributed via HACS. It consumes the [`pymyhondaplus`](https://github.com/enricobattocchi/pymyhondaplus) library for all upstream API work and surfaces vehicle data and remote controls as Home Assistant entities and services.

## 2. Naming

*Mirrored from `pymyhondaplus/AGENTS.md` — update there first.*

Refer to the upstream service as "the My Honda+ API" or "the Honda Connect Europe API" in code, comments, commit messages, PR descriptions, log strings, and test names — matching the framing used in the public READMEs. See [`CONTRIBUTING.md`](CONTRIBUTING.md) for the full style guide.

## 3. The three-repo ecosystem

*Mirrored from `pymyhondaplus/AGENTS.md` — update there first.*

[`pymyhondaplus`](https://github.com/enricobattocchi/pymyhondaplus) (Python library + CLI) is consumed by:

- [`myhondaplus-homeassistant`](https://github.com/enricobattocchi/myhondaplus-homeassistant) — Home Assistant integration, pinned `==X.Y.Z` (HA convention).
- [`myhondaplus-desktop`](https://github.com/enricobattocchi/myhondaplus-desktop) — PyQt6 desktop app, pinned `>=X.Y.Z`.

**Ownership boundaries** — each concern lives in exactly one repo:

- **Library owns**: API request/response shapes, auth flow, `EVStatus` parsing, enum normalization (`charge_status`, `home_away`, `climate_temp`, geofence states), `VehicleCapabilities` resolution, capability raw-API-key labels, geofence state labels, geofence error messages, library-side translations (CLI strings + the `t_lib()` keys consumers bridge to).
- **HA integration owns**: entity descriptors, coordinators, config flow, services, `strings.json` + `translations/*.json`, error-handling conventions for HA.
- **Desktop owns**: view layer (MainWindow / widgets), controller, workers, dashboard / trip / geofence / vehicle UI, desktop `translations/*.json`, PyInstaller bundling.

If a task feels like it crosses boundaries, default to "the library owns the API/parsing/canonical enums; consumers are presentation" and confirm with the maintainer before editing across repos.

**Triage rule.** When investigating an issue or fix in a consumer repo (HA or desktop), use the ownership boundaries above. If the symptom is in library-owned territory (API request/response shape, parsing, enum normalization, capability resolution, library-owned translation strings), the issue or PR should be opened in `pymyhondaplus` — even if it was first surfaced through a consumer. When in doubt, a short Python repro against the library is the fastest way to confirm.

**Editing this file.** Sections `## 2. Naming`, `## 3. The three-repo ecosystem`, and `## 5. Cross-repo workflows` are mirrored verbatim from `pymyhondaplus/AGENTS.md` into the two consumer repos. When changing any of those sections, use **identical wording** across the three repos (no per-repo customization — keep it generic enough to apply everywhere) and run `python scripts/check_agents_mirror.py` from the HA or desktop repo before pushing. The same check enforces parity in CI and will block consumer PRs if the wording drifts.

## 4. Where to touch code

| Task | Files |
|---|---|
| Add a new entity / sensor / button | platform module under `custom_components/myhondaplus/` (`binary_sensor.py` / `button.py` / `device_tracker.py` / `lock.py` / `number.py` / `select.py` / `sensor.py` / `switch.py`); shared helpers in `entity.py`; user-visible labels in `strings.json`; translations in `custom_components/myhondaplus/translations/<lang>.json` for all 13 locales; tests in `tests/` (95% coverage gate) |
| Add a new service | register in `custom_components/myhondaplus/__init__.py`; schema in `services.yaml`; descriptions under `services` in `strings.json`; translations; tests |
| Config-flow change | `custom_components/myhondaplus/config_flow.py`; labels under `config.step` in `strings.json`; translations |
| New translated string paired with a library string | `strings.json` + `translations/<lang>.json`; if it pairs with a `pymyhondaplus.translations.TRANSLATIONS` key, run `tests/test_translation_drift.py` and move the pair into `ENFORCED_OVERLAPS` (or out of `_KNOWN_DRIFT`) when wording converges |
| Bump library dep | `custom_components/myhondaplus/manifest.json` `requirements` — exact pin `pymyhondaplus==X.Y.Z`; manifest `version` for the integration release |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enricobattocchi/myhondaplus-homeassistant](https://github.com/enricobattocchi/myhondaplus-homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
