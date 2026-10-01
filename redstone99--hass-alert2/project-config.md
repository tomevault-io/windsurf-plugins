---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Alert2 is a Home Assistant custom integration (`custom_components/alert2`) that replaces the
built-in `alert` integration with richer templating, event-based firing, dynamic generators,
supersede/priority logic, and a companion web UI card (separate repo: `hass-alert2-ui`).

Domain constant: `alert2`. Alert entities live in `alert2.*`; generator entities live in `sensor.*`.
Current version is tracked in `custom_components/alert2/manifest.json`.

## Commands

```bash
# One-time env setup (creates venv, installs deps, ensures a config/ dir for HA)
scripts/setup.sh

# Manual dependency install
pip install -r requirements.txt -r requirements_test.txt

# Run the full test suite
pytest

# Run a single test file
pytest tests/test_t1.py

# Run a single test
pytest tests/test_t1.py::test_name

# Run with debug logging
pytest --log-cli-level=DEBUG

# Load the integration source from a different HA config checkout
JTESTDIR=/path/to/ha/config pytest --show-capture=no tests/test_t1.py

# Lint / type-check (no repo-specific config committed; run with defaults)
ruff check custom_components/alert2
mypy custom_components/alert2
```

There is no build step — this is a Python HA custom component, deployed by copying
`custom_components/alert2/` into a Home Assistant `custom_components` directory (or via HACS).
CI (`.github/workflows/validate.yml`) only runs HACS and hassfest manifest validation, not tests.

To exercise the companion UI card's backend from this repo, run the dummy notify server:
`JTEST_JS_DIR=/path/to/hass-alert2-ui pytest tests/dummy_server.py` (listens on port 50005).

## Architecture

### Central coordinator

`Alert2Data` (`__init__.py`), stored in `hass.data[DOMAIN]`, owns everything:
- `alerts` — `{domain: {name: ConditionAlert}}`
- `tracked` — `{domain: {name: EventAlert}}`
- `generators` — `{name: AlertGenerator}`
- `component` / `sensorComponent` — `EntityComponent`s for the `alert2` and `sensor` domains
- `supersedeMgr`, `supersedeNotifyMgr`, `delayedNotifierMgr`, `uiMgr`

Startup: `async_setup()` (YAML) and `async_setup_entry()` (config entry) both call `init2()`,
which is idempotent (guarded against double-init) and is also called again on `alert2.reload`.
`init2()` applies built-in defaults, then YAML defaults, then merges UI-stored settings, declares
the three built-in internal alerts, starts `DelayedNotifierMgr` (30s startup grace period so
notify platforms finish loading), installs an asyncio exception handler, and registers services.
`processConfig()` then loads YAML alerts/tracked and UI-created alerts, and schedules entity
registry GC after a 10s delay. Alerts with `early_start=false` only start watching after
`EVENT_HOMEASSISTANT_STARTED`.

### Entity model (`entities.py`)

Three alert entity types, all deriving from `AlertCommon(Entity)` → `AlertBase(AlertCommon,
RestoreEntity)`:

- **`EventAlert`** — fires on a HA trigger spec plus optional condition template. State is the
  ISO timestamp of last fire, or `"has never fired"`. An `EventAlert` with no trigger is a
  "tracked" alert, fired only via the `report()` Python API (used for `alert2.error` etc.).
- **`ConditionAlert`** — on/off state driven by `condition`, `threshold` (numeric hysteresis),
  or split `condition_on`/`condition_off`/`trigger_on`/`trigger_off`. Supports `supersedes`,
  `delay_on_secs`, `done_notifier`, `done_message`, `reminder_message`.
- **`AlertGenerator(AlertCommon, SensorEntity)`** — wraps a Jinja2 `generator:` template that
  produces a list; for each element it incrementally creates/destroys child `EventAlert` or
  `ConditionAlert` entities (not recreated wholesale on every list change). Lives in `sensor.*`,
  not `alert2.*`. Injects `genElem`/`genEntityId`/`genIdx`/`genGroups`/`genRaw` into child
  templates, and must use `rawConfig` (not the already-processed config) when building children,
  to avoid double-interpreting template fields.

Supporting classes in `entities.py`: `TriggerCond` (wraps `async_initialize_triggers` plus an
optional condition template), `Tracker` (wraps `async_track_template_result`, handles Bool/Str/
Float/List coercion, blocks template self-reference to avoid feedback loops), `MovingSum`
(10-bucket sliding-window counter backing throttling).

Supersede classes live in `__init__.py`: `SupersedeMgr` is a directed graph (`supersedesMap`/
`supersededByMap`) with transitive traversal and cycle detection — when touching supersede logic,
keep both maps consistent. `SupersedeNotifyMgr` resolves near-simultaneous firings between related
alerts using an `asyncio.Event` + `asyncio.wait_for(timeout=debounce_secs)` so a higher-priority
alert can preempt a lower one's notification.

### Notification pipeline

```
report() / trigger fires
  → AlertBase._notify_pre_debounce()      builds message, updates last_fired_time
  → SupersedeNotifyMgr.processNotify()    waits debounce_secs if a superseding alert may fire
  → AlertBase._notify_post_debounce()     can_notify_now() checks snooze/throttle/reminder timing,
                                           resolves notifier list via notifierTemplateToList()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [redstone99/hass-alert2](https://github.com/redstone99/hass-alert2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
