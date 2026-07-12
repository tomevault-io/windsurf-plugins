---
trigger: always_on
description: This is a Home Assistant custom integration for DHL eCommerce NL parcel
---

# Working in this repository

This is a Home Assistant custom integration for DHL eCommerce NL parcel
tracking. Distributed via HACS; not part of HA core.

## Always consult HA developer documentation

Home Assistant's integration patterns evolve continuously. **Do not rely
on memory of past patterns** — fetch the canonical page before changing
a topic area, and check the developer blog before introducing anything
you only "know" from training data.

| When you change | Fetch first |
|---|---|
| Entity properties, naming, lifecycle, attributes | https://developers.home-assistant.io/docs/core/entity/ |
| Sensor specifics (state/device classes, units) | https://developers.home-assistant.io/docs/core/entity/sensor |
| Config flow, options flow, reauth, reconfigure | https://developers.home-assistant.io/docs/config_entries_config_flow_handler |
| DataUpdateCoordinator pattern | https://developers.home-assistant.io/docs/integration_fetching_data |
| Quality scale rules | https://developers.home-assistant.io/docs/core/integration-quality-scale |
| Diagnostics | https://developers.home-assistant.io/docs/core/integration/diagnostics |
| Translations | https://developers.home-assistant.io/docs/internationalization/core |

Branding is handled by the local `brand/` folder (HACS reads `icon.png`
from it). The official `home-assistant/brands` repo is for HA Core
integrations and does not apply here.

### Recent developer-facing changes

Before introducing patterns you only know from training data, check:

- https://developers.home-assistant.io/blog — API deprecations, new
  patterns, breaking changes. Recent posts trump older recollection.
- https://github.com/home-assistant/architecture/discussions — design
  decisions in flight that have not made it into stable docs yet.

## What is already in place

The integration is aligned with the **silver** quality scale tier. Don't
re-propose these as improvements:

- `quality_scale: "silver"` in manifest, minimum HA version `2024.7.0`
- `ConfigEntry.runtime_data` (typed dataclass `DhlData`)
- `PARALLEL_UPDATES = 0` in `sensor.py`
- Coordinator takes `config_entry=entry` so `self.config_entry` is
  available on the base class
- Per-parcel sensors are removed by the summary sensor
  (`DhlIncomingParcelsSensor`) via `entity_registry.async_remove(entity_id)`
  when a barcode drops out of the coordinator data. The earlier
  self-remove pattern raced with coordinator-listener cleanup and left
  ghost entities behind — do not revert.
- Reauth flow uses `async_update_reload_and_abort` (one helper call
  instead of update + reload + abort). The confirm step also guards with
  `async_set_unique_id` + `_abort_if_unique_id_mismatch` so entering a
  *different* DHL account's credentials aborts instead of silently
  rebinding the entry to another account.
- **Auth-error split in `api.py`**: `async_login` raises `DhlAuthError`
  only on 401/403; any other non-200 (a 5xx outage) raises `DhlApiError`.
  Setup maps `DhlAuthError → ConfigEntryAuthFailed` (starts reauth) and
  `DhlApiError`/`ClientError → ConfigEntryNotReady` (retry with backoff).
  Do not collapse these again — a DHL outage must never push users into
  reauth.
- **First refresh runs in `__init__.py`, before `async_forward_entry_setups`.**
  Both coordinators call `async_config_entry_first_refresh()` in
  `async_setup_entry` (not in the `sensor.py` platform). Raising
  `ConfigEntryNotReady` from a *forwarded* platform is too late for HA to
  catch — it logs a warning and half-sets-up the entry (users saw only the
  button/calendar entities and no sensors). Doing the first refresh in
  `__init__` makes a transient fetch failure (e.g. a DHL **429**) fail the
  whole entry cleanly so HA retries with backoff. Do not move the first
  refresh back into a platform.
- The per-entry `ClientSession` is closed on every failed-setup path
  (login failure, a failing first refresh, and a failing platform forward)
  — without this each setup retry leaks a session.
- Diagnostics redact person/shop names: `name` (raw payloads) and the
  normalized top-level `receiver` are in `TO_REDACT`.
- `aiohttp.ClientError` is intentionally not caught in the coordinator —
  `DataUpdateCoordinator` wraps it automatically
- Diagnostics handler in `diagnostics.py` with credential and PII
  redaction
- Tests cover config flow, sensor, coordinator (incl. event firing),
  diagnostics, and setup/unload lifecycle
- `_unrecorded_attributes` on every summary sensor — parcel/shipment
  lists are kept out of the recorder long-term tables
- `_attr_attribution = "Data provided by DHL"` per entity

### Adopted in 2.0.0 (do not refactor away)

- **`ParcelStatus` enum** in `const.py` — canonical
  carrier-agnostic statuses. `normalize_parcel` maps the raw DHL
  status/category via `map_parcel_status` and reports `ParcelStatus.UNKNOWN`
  (with one-shot info log) for anything not yet in the map. The original
  DHL status string lives on the parcel's `raw_status` field; do not
  re-introduce it on `status`.
- **Events**: the coordinator fires `dhl_nl_parcel_registered`,
  `dhl_nl_parcel_status_changed` and `dhl_nl_parcel_delivery_time_changed`
  on the HA event bus. Events are suppressed on the very first refresh

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [peternijssen/ha-dhl-nl](https://github.com/peternijssen/ha-dhl-nl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
