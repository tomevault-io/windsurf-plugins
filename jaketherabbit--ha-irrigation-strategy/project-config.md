---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## System Overview

Advanced Crop Steering System for Home Assistant. Currently shipping
`v2.3.1` of the rule-based 4-phase controller, with `[Unreleased] - 3.0.0-dev
"RootSense"` work landed on `main`. RootSense layers a four-pillar adaptive
intelligence platform on top of the existing controller — every pillar is
opt-in via a switch, default OFF, so existing v2.x installs are unaffected on
upgrade.

**Read this first:**
- `docs/SYSTEM_OVERVIEW.md` — the unified mental model of the whole stack.
- `docs/upgrade/SEQUENCE_OF_OPERATIONS.md` — operator-facing contract for what every actuator does.
- `docs/upgrade/CONTROL_THEORY.md` — design rationale for the control layer.
- `docs/upgrade/ROOTSENSE_v3_PLAN.md` — substrate intelligence design.
- `docs/upgrade/CLIMATESENSE_PLAN.md` — environmental control design.
- `docs/upgrade/RECONCILIATION.md` — maps gap-analysis items onto plan phases.
- `docs/upgrade/LLM_ADVISOR_NOTES.md` — salvage notes from the archived
  `llm-integration` branch (now reachable via tag `archive/llm-integration-v0.1`).
- `MIGRATION.md` — operator-facing v2.3.x → v3.0 upgrade guide.

## Development Commands

### Linting & Validation
```bash
ruff check .
black --check .
yamllint -s .
# Full CI validation:
ruff check . && black --check . && yamllint -s .
```

### Testing
```bash
# Integration calculations (28 tests):
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests/test_calculations.py -v

# RootSense intelligence pillars (39 tests):
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests/intelligence/ -v

# Full suite:
PYTEST_DISABLE_PLUGIN_AUTOLOAD=1 python -m pytest tests/ -v
```

The `PYTEST_DISABLE_PLUGIN_AUTOLOAD=1` env var dodges a broken
hydra/omegaconf plugin in some local Python installs. CI is unaffected.

### Home Assistant runtime
```bash
# Reload integration without restart (after code changes):
# Developer Tools → YAML → Reload Custom Components

# Monitor events (Developer Tools → Events → Listen):
# Existing v2.x events:
#   crop_steering_phase_transition
#   crop_steering_irrigation_shot
#   crop_steering_transition_check
#   crop_steering_manual_override
# RootSense v3 events:
#   crop_steering_custom_shot
#   crop_steering_dryback_complete
#   crop_steering_field_capacity_observed
#   crop_steering_anomaly
#   crop_steering_run_report

# View AppDaemon logs:
docker logs addon_a0d7b954_appdaemon -f
```

## Architecture

### Three-Layer System Design

1. **Home Assistant Integration** (`custom_components/crop_steering/`)
   - Provides 100+ entities (sensors, numbers, switches, selects).
   - Config flow UI for setup; no YAML editing required.
   - Services: `transition_phase`, `execute_irrigation_shot`,
     `check_transition_conditions`, `set_manual_override`,
     `custom_shot` (RootSense v3).
   - Pure helpers split out into `calculations.py` for testability.

2. **AppDaemon — legacy controller** (`appdaemon/apps/crop_steering/`)
   - `master_crop_steering_app.py` — the original autonomous coordinator.
     Phase transitions, hardware sequencing, dryback detection, sensor fusion.
     Untouched in the v3 work.
   - `phase_state_machine.py`, `advanced_dryback_detection.py`,
     `intelligent_sensor_fusion.py`, `intelligent_crop_profiles.py`,
     `ml_irrigation_predictor.py` — supporting libraries the master app uses.

3. **AppDaemon — RootSense v3 intelligence pillars**
   (`appdaemon/apps/crop_steering/intelligence/`)
   - `base.py` — `IntelligenceApp` mixin with module-enable gating + helpers.
   - `bus.py` — in-process pub/sub (`RootSenseBus`).
   - `store.py` — SQLite analytics store at
     `appdaemon/apps/crop_steering/state/rootsense.db`.
   - `root_zone.py` — Pillar 1: substrate analytics, FC detection, dryback
     episode tracker.
   - `adaptive_irrigation.py` — Pillar 2: intent slider, profile interpolation,
     bandit shot-size optimisation.
   - `agronomic.py` — Pillar 3: Penman-Monteith transpiration, VPD ceiling,
     nightly run reports.
   - `orchestration.py` — Pillar 4: `crop_steering.custom_shot` event handler,
     emergency rescue, EC flush.
   - `anomaly.py` — cross-cutting scanner (emitter blockage, EC drift, sensor
     flat-line, peer-group VWC deviation).

Each pillar gates itself behind its `switch.crop_steering_intelligence_*_enabled`
entity. Default OFF.

### Critical Files
- `custom_components/crop_steering/config_flow.py` — integration setup wizard.
- `custom_components/crop_steering/sensor.py` — calculated sensor entities.
- `custom_components/crop_steering/calculations.py` — pure helpers (testable).
- `custom_components/crop_steering/services.py` — service handlers + event firing.
- `custom_components/crop_steering/const.py` — constants, single source of truth.
- `custom_components/crop_steering/number.py` — number entities including
  RootSense intent slider and dryback drop sliders.
- `custom_components/crop_steering/switch.py` — switches including the 5
  RootSense module-enable switches.
- `appdaemon/apps/crop_steering/master_crop_steering_app.py` — legacy coordinator.
- `appdaemon/apps/crop_steering/intelligence/*.py` — RootSense pillars.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JakeTheRabbit/HA-Irrigation-Strategy](https://github.com/JakeTheRabbit/HA-Irrigation-Strategy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
