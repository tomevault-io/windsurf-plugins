---
trigger: always_on
description: Home Assistant custom integration for **GLS Netherlands** parcel tracking.
---

# Working in this repository

Home Assistant custom integration for **GLS Netherlands** parcel tracking.
Distributed via HACS; not part of HA core. Fourth carrier in the suite (with DHL,
DPD, PostNL) — same canonical shape, events and entity set; **mirror DHL when in
doubt**. Account-less (user-entered tracking codes). No DTO layer.

## Shared conventions — fetch when relevant

Suite-wide rules live in
[`.github/CONVENTIONS.md`](https://github.com/ha-parcel-integrations/.github/blob/main/CONVENTIONS.md)
and are **not** repeated here. Don't fetch it every session — fetch it **before**
you act in one of these areas:

| Before you … | Fetch `CONVENTIONS.md` § |
|---|---|
| touch entities, sensors, config/options flow, coordinator, diagnostics, translations | *Home Assistant developer docs* (its table points on to the canonical HA page — don't rely on memory) |
| add/rename a parcel field, a `ParcelStatus`, or a bus event; change first-refresh or unmapped-status logging | *Parcel contract* (this repo implements it; below is only where GLS deviates) |
| consider "fixing" a lint/pattern the skill flags (poll interval, inline client) | *Deliberate skill divergences* — likely intentional, don't re-flag |
| commit, bump, tag, release, or write release notes; add a feature without a test | *Workflow / Commits / Versioning / Testing* |

**API mechanics live in `carrier-research/gls/api/` (private research repo)** — the tracktrace
endpoint, its `text/plain` body and 204 signalling, the numeric `state` → status
map, the `scans[]` history and the two-identifier lookup. Do not duplicate them
here.

**Suite-wide tripwires, kept inline on purpose:**
- **First refresh in `__init__.py`, before `async_forward_entry_setups`** — so
  the `UpdateFailed`-on-total-failure case fails the whole entry (HA retries with
  backoff). From a forwarded platform HA can't catch `ConfigEntryNotReady`.
- **Setup stale-entity cleanup is scoped to `domain == "sensor"` and excludes
  `non_parcel_unique_ids`** — else it deletes the button / `last_update` sensor /
  live per-parcel sensors.

## The big divergence: account-less, postcode-keyed hubs

GLS has **no consumer account / feed** — the user enters tracking codes.

- **Setup asks only the postal code** (`async_step_user`), stored as the hub
  default in `entry.options[CONF_POSTAL_CODE]`; `CONF_PARCELS` starts empty. Setup
  does **not** hit the API (the endpoint needs a parcel number).
- **Multiple hubs, one per postcode.** `unique_id = <postcode>` +
  `_abort_if_unique_id_configured` (home + work both work). Device name
  `"GLS (<postcode>)"`. `single_config_entry` is deliberately **absent** (the user
  wanted multiple hubs). The shared `gls.*` services are unloaded only when **no
  other hub is still loaded**. Legacy entries with `unique_id = DOMAIN` are
  migrated to the postcode in `async_setup_entry`.
- **Tracked parcels live in `entry.options[CONF_PARCELS]`** as
  `{parcel_no, postal_code}` dicts, added three ways (options flow, the
  `gls.track_parcel` / `gls.untrack_parcel` services, a Lovelace button), all
  validated the same. Adding takes only the number — the postcode is **always** the
  hub's; the service keeps an optional `postal_code` for the rare
  different-address case.
- **Service field is `tracking_code`** (suite-wide standard); the deprecated
  `parcel_no` alias has been removed from the services (ha-gls#3). The *stored*
  dict key stays `parcel_no` (`CONF_PARCEL_NO`) — that's an internal options
  key, not the service field, and was never part of the alias; don't conflate
  them.
- **Options flow = a two-page menu** (`parcels` / `settings`), not a single
  sectioned form. The `parcels` page edits the whole tracked-code list as one
  multi-value text field (add or remove any number, then save); `settings`
  holds delivered-parcel retention, history and polling.
- **Option changes apply live, no reload.** An **update listener**
  (`_async_options_updated`) retunes `coordinator.update_interval` and calls
  `async_request_refresh()`; the coordinator re-reads options each update, so a
  refresh (not a reload) makes add/remove reflect immediately and avoids the
  config-entry-listener deprecation. **Do not** switch to `async_schedule_reload`.
- **No auth / reauth / sent-shipments coordinator.** The HA-managed session is
  used directly (no per-entry cookie jar — no cookies). Entities are
  **entry-scoped** (like DPD).

## Integration-level carrier decisions

- **Country model** (`CONF_COUNTRY` / `COUNTRIES`): each hub picks a country →
  host/culture (or `group_locale`, see below)/postcode-regex. **NL** uses a
  keyless national GET, **DE** a bearer-POST guest-account with its own
  `countries/de/session.py`, and **CZ**, **SK**, **AT**, **IE**, **FR**,
  **SI**, **HR** and **IT** use the keyless pan-EU group leaves
  (`rstt028`/`rstt029`, `countries/group/`). Adding a country = one
  `COUNTRIES` entry once a working account-less endpoint is confirmed; the setup form links
  `NEW_COUNTRY_ISSUE_URL`. `unique_id` stays the bare postcode — fine while
  a postcode is unique per hub regardless of country.
- **`culture` vs `group_locale`.** NL/DE's `culture` is a `nl-NL`-style

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ha-parcel-integrations/ha-gls](https://github.com/ha-parcel-integrations/ha-gls) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
