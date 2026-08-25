---
trigger: always_on
description: This file provides guidance to AI agents working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents working with code in this repository.

## Project Overview

A Home Assistant custom integration (`cap_alerts`) that creates **one entity per active weather alert**, solving the 16KB attribute limit in `nws_alerts`. Alert data is modeled using CAP (Common Alerting Protocol) field names via a `CAPAlert` frozen dataclass. Ships with NWS and ECCC (Environment Canada) providers; designed for future providers (BoM, MeteoAlarm, DWD, WMO CAP, etc.).

Companion frontend: [weather_alerts_card](../weather_alerts_card) — the card's `cap.ts` adapter is a thin passthrough since this integration handles all normalization.

## Architecture

See `docs/architecture.md` for design rationale (alert identity, field mappings, provider layer) and `docs/roadmap.md` for planned work. `plans/` is gitignored scratch for `/plan` output — not reference material.

### Entity Model

- **Device**: groups all entities for a configured location
- **Count sensor** (`sensor.cap_alerts_<provider>_alert_count`): `state` = number of
  active alerts, attributes `active`/`upcoming` split it on `onset`,
  `EntityCategory.DIAGNOSTIC`
- **Last updated sensor** (`sensor.cap_alerts_<provider>_last_updated`): `state` = ISO timestamp, `EntityCategory.DIAGNOSTIC`
- **Alert entities** (`sensor.cap_alerts_<provider>_cap_alert_<slug>_<hash>`): one per active alert, dynamically created/removed each poll cycle. The device-name prefix is HA's, applied because these set `has_entity_name`; the integration only suggests `cap_alert_<slug>_<hash>`
- **Refresh button** (`button.cap_alerts_<provider>_refresh`): forces an off-cycle fetch, `EntityCategory.DIAGNOSTIC`
- **Stream connectivity** (`binary_sensor.cap_alerts_eccc_real_time_stream`): NAAD socket state, `EntityCategory.DIAGNOSTIC`, ECCC-with-streaming only

### Data Flow

```
Weather API → Provider.async_fetch() → list[CAPAlert]
                ↑ (NWS: GeoJSON, ECCC: Atom XML, future: varies)
  Coordinator._async_update_data() calls provider
    normalize_alerts() → sets severity_normalized, phase
    store.process() → diffs vs previous, sets phase_changed, fires HA events
    ├─ CountSensor (state = len)
    └─ coordinator listener → diffs alert IDs vs tracked entities
         → async_add_entities / registry remove
           └─ AlertEntity (finds own CAPAlert by ID in coordinator.data)
```

### File Structure

```
custom_components/cap_alerts/
  __init__.py       # entry setup, coordinator wiring, platform forwarding; owns the shared GeometryStore and registers the REST view + WS command once per HA instance; syncs the ECCC sunset repairs on setup/update/remove
  const.py          # domain, defaults, user-agent format
  config_flow.py    # the composed flow handler (provider menus) + options flow; hassfest requires this exact filename
  flows/            # per-provider config flow steps, mixed into the handler above
    common.py       # validators, schema helpers, the entry-title rule, and the canonical scope key (entry unique_id) shared by providers
    nws.py          # NWS steps + zone validation
    eccc.py         # ECCC steps + province validation, language/streaming/feed options
    meteoalarm.py   # MeteoAlarm steps: country, region picker, fully-mobile mode
    wmo.py          # WMO steps: source picker, geocode narrowing, language option
    gdacs.py        # GDACS steps + event-type/alert-level options
  coordinator.py    # orchestrates provider, feeds list[CAPAlert] to entities; owns device_info + NAAD stream lifecycle; provider-neutral post-fetch filters (marine, geocode-prefix); writes/purges geometry refs
  diagnostics.py    # config-entry diagnostics download: scope, endpoints, update health, filters, convention rows in effect; redacts location + credentials
  sensor.py         # CountSensor, LastUpdatedSensor, AlertEntity, dynamic lifecycle
  button.py         # RefreshButton: on-demand provider fetch (all providers)
  binary_sensor.py  # StreamConnectivitySensor: NAAD socket state (ECCC streaming only)
  model.py          # CAPAlert dataclass + to_attributes()
  conventions.py    # per-source convention table: marine prefixes, terminal lifecycle tokens, severity derivations, per-sender dialects (identity/keep hooks + explode/merge pipeline stages); an episode dialect declares its own run rule — MeteoFrance merges consecutive forecast days, FMI contiguous windows — over one shared pipeline
  normalize.py      # shared normalization: severity, phase, Buddhist-Era year fix, state truncation
  payload.py        # attribute-payload budget: measures what the recorder measures, trims long-form text then redundant keys in priority order (#150)
  store.py          # alert store: inter-poll diffing, transition detection, HA event firing (incl. removal_reason)
  icons.py          # event-type → mdi dispatch; MeteoAlarm classifies on awareness_type, others on event tables
  geometry_store.py # in-memory LRU cache of full GeoJSON polygons, keyed by geometry_ref (RFC §2.4); never persisted
  issues.py         # repairs issues owed by an entry's config (#163): ECCC streaming off / feed source pinned to the retiring NAAD host; issue-registry only, no repairs import

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seevee/cap_alerts](https://github.com/seevee/cap_alerts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
