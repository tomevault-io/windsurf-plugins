---
trigger: always_on
description: This is a **Home Assistant custom integration** called **Aquarium AI** (domain: `aquarium_ai`). It uses Home Assistant's built-in `ai_task` service to perform AI-powered analysis of aquarium sensor data and optional camera feeds, providing natural-language health assessments, water change recommendations, and persistent notifications.
---

# Copilot Instructions for Aquarium AI Home Assistant Integration

## Project Overview

This is a **Home Assistant custom integration** called **Aquarium AI** (domain: `aquarium_ai`). It uses Home Assistant's built-in `ai_task` service to perform AI-powered analysis of aquarium sensor data and optional camera feeds, providing natural-language health assessments, water change recommendations, and persistent notifications.

The integration is distributed via **HACS** (Home Assistant Community Store) and has no Python package dependencies beyond Home Assistant itself.

---

## Repository Structure

```
custom_components/aquarium_ai/   # Main integration code
├── __init__.py                  # Integration setup, core AI analysis logic, helper functions
├── config_flow.py               # Multi-step UI configuration flow (ConfigFlow + OptionsFlow)
├── const.py                     # All constants, defaults, and default AI prompts
├── manifest.json                # Integration metadata (domain, version, dependencies)
├── sensor.py                    # Sensor entities (AI analysis text sensors, per-parameter + overall)
├── binary_sensor.py             # Binary sensors (water change needed, parameter problem)
├── button.py                    # Button entity (Run Analysis)
├── select.py                    # Select entities (Update Frequency, Notification Format)
├── switch.py                    # Switch entities (auto-notifications, per-parameter analysis toggles)
├── services.yaml                # Service definitions (run_analysis, run_analysis_for_aquarium)
├── strings.json                 # Default English UI strings
└── translations/
    ├── en.json                  # English translations
    ├── de.json                  # German translations
    └── template.json            # Template for new translations

.github/
├── workflows/
│   ├── hassfest.yaml            # Validates integration against Home Assistant standards
│   └── validate.yml             # HACS validation + hassfest
└── ISSUE_TEMPLATE/
    ├── bug_report.md
    └── feature_request.md

hacs.json                        # HACS metadata
README.md                        # User-facing documentation
CONTRIBUTING.md                  # Contribution guidelines
TRANSLATION_GUIDE.md             # Instructions for adding translations
```

---

## Key Architecture Concepts

### Integration Setup Flow
1. `async_setup_entry` in `__init__.py` is the main entry point
2. It forwards setup to all platforms: `sensor`, `binary_sensor`, `switch`, `select`, `button`
3. Schedules periodic AI analysis using `async_track_time_interval`
4. Optionally runs analysis on startup (60-second delay via `async_call_later`)
5. Registers `run_analysis` and `run_analysis_for_aquarium` services

### AI Analysis Pipeline (`__init__.py: send_ai_aquarium_analysis`)
1. Collects sensor data from configured HA sensor entities using `get_sensor_info()`
2. Checks per-parameter analysis toggle switches (stored in `entry.data`)
3. Builds a structured prompt using configurable AI prompt templates from `const.py`
4. Calls `ai_task.generate_data` service with a structured response schema
5. Stores the AI response in `hass.data[DOMAIN][entry_id]["sensor_analysis"]`
6. Sends a `persistent_notification` (if auto-notifications enabled)
7. Triggers state updates on sensor entities via `async_write_ha_state()`

### Shared Data Pattern
Entities read AI analysis results from `hass.data[DOMAIN][entry_id]["sensor_analysis"]` — a dict populated after each AI call. Sensor entities poll this dict on `async_update()`.

### Config Entry Data Storage
All configuration (sensors, toggles, AI prompts, tank info) is stored directly in `config_entry.data`. The options flow updates `entry.data` directly via `hass.config_entries.async_update_entry()` — it does **not** use `entry.options`.

---

## Entity Naming Conventions

All entities are named with the tank name as a prefix:
- `{tank_name} Temperature Analysis` (sensor)
- `{tank_name} Water Change Needed` (binary sensor)
- `{tank_name} Run Analysis` (button)
- `{tank_name} Update Frequency` (select)
- `{tank_name} Analyze Temperature` (switch)

Unique IDs follow the pattern: `{entry_id}_{descriptor}` (e.g., `{entry_id}_temperature_analysis`).

All entities share the same device under:
```python
{"identifiers": {(DOMAIN, config_entry.entry_id)}, "name": f"Aquarium AI - {tank_name}"}
```

---

## AI Response Structure

The AI call uses `ai_task.generate_data` with a `structure` dict. Response keys follow these patterns:
- `{param_name_snake_case}_analysis` — brief (≤200 chars), used for sensor state
- `{param_name_snake_case}_notification_analysis` — detailed, used in notifications
- `overall_analysis` / `overall_notification_analysis` — overall health
- `water_change_recommended` — `"Yes/No + brief reason"` (drives binary sensor state)
- `water_change_recommendation` — detailed recommendation for notifications

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheRealFalseReality/Aquarium-AI-Homeassistant](https://github.com/TheRealFalseReality/Aquarium-AI-Homeassistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
