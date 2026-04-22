---
trigger: always_on
description: This is a **Home Assistant custom component** for monitoring Hydro-Québec electricity accounts. It provides real-time consumption data, billing info, peak period alerts, winter credits tracking, and hourly consumption history import. The integration supports two connection modes:
---

# Hydro-Québec Home Assistant Integration - AI Coding Agent Guide

## Project Overview

This is a **Home Assistant custom component** for monitoring Hydro-Québec electricity accounts. It provides real-time consumption data, billing info, peak period alerts, winter credits tracking, and hourly consumption history import. The integration supports two connection modes:

- **Portal Mode** (`AUTH_MODE_PORTAL`): Full authentication with Hydro-Québec account - provides complete billing, consumption, winter credit data, and hourly consumption history
- **OpenData Mode** (`AUTH_MODE_OPENDATA`): No credentials required - fetches publicly available peak event data from HQ's open data API

**Tech Stack**: Python 3.13, Home Assistant 2024.1+, async/await, hydroqc API wrapper library, **uv** package manager

## Architecture

### Core Components

**IMPORTANT**: The codebase uses a modular package structure for maintainability. See "Modular Code Organization" section below for details.

1. **`coordinator/`** package - `HydroQcDataCoordinator`: Central data fetcher using HA's `DataUpdateCoordinator`
   - **`coordinator/base.py`**: Core coordinator class and data fetching logic
   - **`coordinator/calendar_sync.py`**: CalendarSyncMixin for DPC/DCPC calendar event management
   - **`coordinator/consumption_sync.py`**: ConsumptionSyncMixin for consumption history sync
   - **`coordinator/sensor_data.py`**: SensorDataMixin for sensor value retrieval
   - **`coordinator.py`**: Root-level re-export for backward compatibility
   - Manages both Portal mode (`WebUser`/`Customer`/`Account`/`Contract` hierarchy) and OpenData mode (`PublicClient`)
   - **Manual scheduling only** - `update_interval=None` disables DataUpdateCoordinator's automatic polling
   - Instead of automatic polling every X seconds, uses explicit time-based triggers with `async_track_time_change()`
   - **Three independent schedulers**:
     - **OpenData**: Every 15 minutes with random offset (checks time windows before fetching)
     - **Portal**: Every hour with random offset (checks time windows before fetching)
     - **Calendar**: Every 15 minutes at fixed intervals (refreshes calendar-based sensor data)
   - **Smart time windows**:
     - OpenData: 10:30-15:00 EST (15 min interval) / other times (60 min) / off-season (disabled)
     - Portal: 00:00-08:00 EST (60 min interval) / other times (180 min)
   - **Anti-thundering herd**: OpenData scheduler uses random minute (0-14) and second offset based on integration start time to distribute API calls across users
   - **Calendar as source of truth**: Peak sensors read from `CalendarPeakHandler` which loads events from HA calendar entity, making data persistent across restarts
   - **Sensors only update when data actually fetched** - raises `UpdateFailed` when no new data to prevent unnecessary sensor updates
   - Uses dot-notation paths (`get_sensor_value()`) to extract nested data: e.g., `"contract.cp_current_bill"` → `coordinator.data["contract"].cp_current_bill`
   - `is_sensor_seasonal()`: Portal mode sensors with `peak_handler` are seasonal (Dec 1 - Mar 31), OpenData sensors are always available
   - **Data preservation**: Previous portal data preserved when updates skipped (prevents "unknown" values)
   - **Portal status tracking**: `_portal_available` tracks portal online status, checked before operations
   - **Task management**: Separate background tasks per coordinator instance (per contract):
     - `_csv_import_task`: Bulk historical consumption import
     - `_regular_sync_task`: Incremental consumption sync
     - `_calendar_sync_task`: Calendar event synchronization from OpenData to HA calendar
   - `is_consumption_history_syncing`: Only checks `_csv_import_task` status (not regular sync)
   - **Logging prefixes**: Portal mode logs use `[Portal]` prefix, public API logs use `[OpenData]` prefix

2. **`config_flow/`** package - Multi-step config flow:
   - **`config_flow/base.py`**: HydroQcConfigFlow class for integration setup
   - **`config_flow/options.py`**: HydroQcOptionsFlow for options management
   - **`config_flow/helpers.py`**: API functions and mappings (fetch_available_sectors, fetch_offers_for_sector, SECTOR_MAPPING, RATE_CODE_MAPPING)
   - **`config_flow.py`**: Root-level re-export for backward compatibility
   - Step 1: Choose `AUTH_MODE_PORTAL` or `AUTH_MODE_OPENDATA`
   - Step 2a (Portal mode): Login → fetch customers → select account → select contract → import history (0-800 days) → create entry
   - Step 2b (OpenData mode): Select sector (Residentiel/Affaires) → select rate and option (with preheat field) → create entry
   - **History import** (Portal mode only): `NumberSelector` for 0-800 days of consumption history
     - 0-30 days: Regular sync only (efficient for recent data)
     - >30 days: CSV import triggered automatically on integration setup
   - **Preheat configuration**: Only shown for rates with peak events (DPC, DCPC in Portal mode; all peak rates in OpenData mode)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hydroqc/hydroqc-ha](https://github.com/hydroqc/hydroqc-ha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
