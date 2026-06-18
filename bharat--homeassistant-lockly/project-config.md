---
trigger: always_on
description: This is the canonical agent guide for `bharat/homeassistant-lockly`. New Claude/Codex/Cursor sessions should read this before making changes.
---

# AGENTS.md — Lockly HA Integration

This is the canonical agent guide for `bharat/homeassistant-lockly`. New Claude/Codex/Cursor sessions should read this before making changes.

## What this is

A Home Assistant custom integration + Lovelace card suite for managing **PIN-code access slots on Zigbee2MQTT smart locks**. Despite the name, it doesn't speak to Lockly hardware directly — it's a generic Z2M-via-MQTT slot manager that happens to be tuned to Lockly's quirks (firmware echoes, double events). Anything Z2M exposes through the standard PIN cluster works.

Architecturally: an MQTT-driven coordinator owns slot state (`LocklySlot` dataclass), a manager queues add/remove/apply jobs and dedups noisy lock events through `ActivityBuffer`, and two custom Lovelace cards (`lockly-card`, `lockly-activity-card`) render the UI. There's no polling, no cloud API, no Bluetooth — pure MQTT push (`iot_class: "local_push"`).

## Layout

```
.
├── README.md                   # Feature overview, install, configuration, services
├── CONTRIBUTING.md             # Fork/PR flow
│
├── custom_components/lockly/
│   ├── __init__.py             # async_setup_entry, services, MQTT subscribe, WebSocket, logbook
│   ├── manifest.json           # version "0.0.0" sentinel — see Releases section
│   ├── const.py                # DOMAIN, storage keys, _resolve_version() (mtime fallback)
│   ├── config_flow.py          # User flow: name + first/last slot + MQTT topic + endpoint; v2 migration
│   ├── data.py                 # TypedDict LocklyData wrapping coordinator/manager/subscriptions
│   ├── coordinator.py          # LocklySlotCoordinator (push-based; update_interval=None)
│   ├── manager.py              # Job queue, MQTT publish, response handling, dedup pipeline
│   ├── activity.py             # ActivityBuffer + 4 dedup rules (firmware echo, repeats, physical+auto, fallback)
│   ├── entity.py               # CoordinatorEntity base
│   ├── event.py                # LocklyLockEvent (16+ action types)
│   ├── sensor.py               # LocklySlotSensor (per-slot status)
│   ├── logbook.py              # Maps lock activity to HA logbook with labels
│   ├── services.yaml           # add_slot, remove_slot, apply_slot, push_slot, apply_all,
│   │                           #   update_slot, get_slot, export_slots, import_slots
│   ├── frontend/
│   │   ├── __init__.py         # JSModuleRegistration; cleans up legacy /local/lockly-card resources
│   │   ├── lockly-card.js      # Slot management UI (~957 LOC)
│   │   └── lockly-activity-card.js  # Recent events / per-lock summary (~704 LOC)
│   ├── strings.json
│   └── translations/en.json
│
├── assets/                     # Hand-authored brand assets (logo SVGs + README screenshots)
│                               #   — source files; not served by HA
├── brands/custom_integrations/lockly/
│                               # HA's brand registry artwork (icon.png, dark_icon.png + @2x)
├── www/lockly-card/lockly-card.js
│                               # Legacy fallback; frontend/__init__.py actively REMOVES the
│                               #   Lovelace resource pointing here (see lines 55–61)
│
├── tests/
│   ├── conftest.py             # `pytest_plugins = "pytest_homeassistant_custom_component"`
│   └── test_*.py               # 8 test modules (config flow, activity, services, frontend, ...)
│
├── config/                     # Runtime dev HA config (mosquitto.conf is checked in here)
│   ├── configuration.yaml      # Minimal — no default_config; pre-seeded MQTT entry
│   ├── mosquitto.conf          # Anon broker on 0.0.0.0:1883, no persistence
│   └── mosquitto-verbose.conf  # Same with verbose logging
│
├── scripts/
│   ├── setup                   # apt + pip + npm i -g concurrently + pre-commit + act
│   ├── develop                 # Mosquitto + (optional simulator) + HA via concurrently
│   ├── seed_dev_mqtt.py        # Idempotently writes MQTT config entry into .storage/
│   ├── lint                    # ruff check --fix && ruff format --check
│   └── simulate_devices.py     # Fake Z2M lock devices for ./scripts/develop
│
├── pytest.ini                  # asyncio_mode = auto
├── .ruff.toml                  # select = ALL minus ~5; max-complexity 25
├── .pre-commit-config.yaml     # ruff + EOF/whitespace + check-yaml + local pytest with coverage
└── requirements.txt            # HA, frontend, paho-mqtt, ruff, pre-commit, pytest helpers
```

## Dev workflow

```bash
# First time inside the devcontainer (PostCreateCommand=scripts/setup):
pre-commit install                                  # If not already

# One-command dev stack (Mosquitto + HA + fake locks)
./scripts/develop                                   # All three
./scripts/develop --no-sim                          # Skip the simulator
./scripts/develop --mqtt-verbose                    # Verbose broker logs

# scripts/seed_dev_mqtt.py runs first — pre-seeds the MQTT integration in .storage/
# so HA boots with MQTT already configured. No manual "Add Integration" step needed.

# HA dashboard: http://localhost:8123
# Lockly integration is added via UI: Settings → Devices & Services → Add → Lockly

# Tests

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bharat/homeassistant-lockly](https://github.com/bharat/homeassistant-lockly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
