---
trigger: always_on
description: Fast orientation for AI agents working on this repo. Humans should start with [`CONTRIBUTING.md`](CONTRIBUTING.md), which documents the architecture, conventions, test layout, and release process in full. This file complements that with quick-navigation pointers for agents starting cold; defer to CONTRIBUTING.md when in doubt.
---

# AGENTS.md

Fast orientation for AI agents working on this repo. Humans should start with [`CONTRIBUTING.md`](CONTRIBUTING.md), which documents the architecture, conventions, test layout, and release process in full. This file complements that with quick-navigation pointers for agents starting cold; defer to CONTRIBUTING.md when in doubt.

Sections 2, 3, and 5 mirror the canonical text in [`pymyhondaplus/AGENTS.md`](https://github.com/enricobattocchi/pymyhondaplus/blob/main/AGENTS.md) — update there first, then propagate.

## 1. What this repo is

The unofficial PyQt6 desktop app for My Honda+ / Honda Connect Europe vehicles, distributed as PyInstaller bundles for Linux, macOS, and Windows. It consumes the [`pymyhondaplus`](https://github.com/enricobattocchi/pymyhondaplus) library for all upstream API work and presents vehicle status, trips, geofence, and remote controls in a native GUI.

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
| New dashboard tile | `src/myhondaplus_desktop/widgets/dashboard.py`; translation key in `src/myhondaplus_desktop/translations/en.json` (then propagate to the other 12 locales); access `EVStatus` as a dict (`.get()`, `[]`, `in`) |
| New modal / dialog | new widget under `src/myhondaplus_desktop/widgets/`; wire it in `main_screen_controller.py`; translation keys; tests in `tests/` |
| New worker for an API call | `src/myhondaplus_desktop/workers.py` (`QThread` subclass with `finished` / `error` signals); the controller connects them; UI mutation can be optimistic |
| New translated string | `src/myhondaplus_desktop/translations/en.json` (canonical) + 12 other locales; `tests/test_i18n.py` enforces coverage. **Library-owned strings** (geofence states, capability labels, geofence errors) come via `i18n.t_lib(key)` — do not duplicate them in desktop JSONs. |
| Bump library dep | `pyproject.toml` (`pymyhondaplus>=X.Y.Z`) and `README.md` must mirror the same pin; `scripts/check_release_version.py` enforces tag = `__version__` |
| Capabilities modal change | `src/myhondaplus_desktop/widgets/vehicle.py`; relies on the library's `caps.active_api_keys()` / `not_supported_api_keys()` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enricobattocchi/myhondaplus-desktop](https://github.com/enricobattocchi/myhondaplus-desktop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
