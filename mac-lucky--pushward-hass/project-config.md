---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

PushWard for Home Assistant is a custom HACS integration that tracks HA entity state changes and surfaces them on iPhone two ways:

1. **Live Activities** (Dynamic Island + Lock Screen) — when an entity enters a configured "start" state a Live Activity is created; on an "end" state it is dismissed with a two-phase completion animation. Driven by `activity_manager` + `content_mapper`.
2. **Home Screen / Lock Screen widgets** — an entity (or several, for `stat_list`) is bound to a server-side widget that re-renders on state change or on a poll interval. Driven by `widget_manager` + `widget_mapper`.

The two surfaces are independent: each is a separate `ConfigSubentry` type (`tracked_entity` vs `tracked_widget`), has its own manager, mapper, `.storage` cache, and config-flow class. They share the API client and the icon/color helpers in `content_mapper`.

A third surface points the other way — **HA-side sensor entities** (not an iPhone surface): the integration's only polling component, `PushWardUsageCoordinator` (`coordinator.py`), fetches the account's usage/quota from `GET /auth/me` every `USAGE_UPDATE_INTERVAL` (15 min), and `sensor.py` exposes one "used" sensor per metered resource (notifications, Live Activity updates, widget updates, emails) plus a subscription-tier sensor. `PLATFORMS = [Platform.SENSOR]` is the only HA entity platform forwarded; fields stay absent (sensor → unavailable) on older servers that don't return usage to integration keys.

Requires Python 3.13.2+ and Home Assistant 2025.7.0+.

This is a **public repository** — no server internals, private URLs, API keys, or DB schemas should appear in code or commit history.

## Cross-Repository Dependencies

- **pushward-server**: This integration calls server's REST API for activity CRUD (create/update/end) and widget CRUD (create/PATCH) → server sends APNs → pushward-ios shows Live Activities / renders widgets
- API contract (endpoints, auth with integration keys `hlk_` prefix) is defined by pushward-server. Widget CRUD requires the `widgets` permission on the key; widget content field caps in `const.py` mirror `pushward-server/internal/model/widget.go`

## Commands

```bash
uv sync                                        # Install dependencies
uv run pytest tests/ -v                         # Run all tests
uv run pytest tests/test_api.py -v -k "test_x"  # Run single test
uv run ruff check . && uv run ruff format .     # Lint + format
```

## Architecture

```
config_flow.py    → ConfigEntry (integration key) + two ConfigSubentry flows:
                      PushWardEntitySubentryFlow (activities), PushWardWidgetSubentryFlow (widgets)
__init__.py       → Creates API client, starts ActivityManager + WidgetManager + usage
                      coordinator, forwards the sensor platform, registers the services
activity_manager  → Listens to HA state changes, decides activity start/update/end
content_mapper    → Translates HA State + entity config → activity content dict
                      (also exports shared helpers: resolve_icon, resolve_color, color_to_str,
                       add_tap_action, lookup_registry_icon — reused by widget_mapper)
widget_manager    → Listens to HA state changes / poll timer, diffs content, PATCHes widget
widget_mapper     → Translates HA State + widget config → widget content dict
coordinator.py    → PushWardUsageCoordinator: polls GET /auth/me (15 min) for usage/quota
sensor.py         → Usage/quota + subscription-tier sensors (the only HA entity platform)
api.py            → HTTP client with retry/backoff to PushWard server (activities + widgets + /auth/me)
```

## Key Patterns

- **Subentry two-step flow**: `config_flow.py` uses a two-step `ConfigSubentryFlow` — step 1 picks entity + template, step 2 dynamically builds schema via `_details_schema()` based on the selected template. Templates: `generic`, `countdown`, `alert`, `steps`, `gauge`, `timeline`. Each adds template-specific fields (e.g. `gauge` → min/max/unit, `timeline` → series/scale/decimals/thresholds/history_period).
- **Two-phase end**: On end state, manager sends ONGOING with completion content (green checkmark), sleeps `END_DELAY_SECONDS` (5s), then sends ENDED. The `generation` counter prevents stale ends if the activity restarts during the sleep.
- **Throttled updates with dedup**: Rate-limited per `update_interval` with content dict equality check. `flush_unsub` timer fires after cooldown.
- **Reauth**: 401/403 triggers `entry.async_start_reauth()` once via `_reauth_triggered` flag.
- **Timeline history buffer**: `TrackedEntity.history_buffer` is an in-memory ring buffer (≤300 samples) populated from live state changes and persisted to `.storage/pushward.history.<entry_id>`. Required because HA 2024.8+ strips most attributes from the recorder DB — for attribute-based entities (light brightness, climate temps), the recorder cannot be used to backfill the sparkline. For numeric-state sensors, the recorder is still used as a fallback (which is why `manifest.json` lists `after_dependencies: [recorder]` — recorder must load first).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mac-lucky/pushward-hass](https://github.com/mac-lucky/pushward-hass) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
