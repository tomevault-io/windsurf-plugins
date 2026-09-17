---
trigger: always_on
description: This file provides guidance to LLM Agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to LLM Agents when working with code in this repository.

## Project Overview

Load Juggler is a Home Assistant custom component for intelligent load management. It dynamically distributes available power across managed loads — EV chargers (via OCPP 1.6J), smart plugs, and more — based on solar production, battery state, grid capacity, and per-load operating modes.

**Key Capabilities:**

- Per-load operating modes (Standard, Solar Priority, Solar Only, Excess for EVSE; Continuous, Solar Only, Excess for plugs)
- Multi-load support with priority-based distribution and mode urgency sorting
- Circuit groups — shared breaker limits for co-located loads (post-distribution capping)
- Battery integration with SOC thresholds
- Phase-aware handling (1-phase, 2-phase, 3-phase installations)
- Symmetric and asymmetric inverter support
- Off-grid support (no grid CTs required — infers phases from inverter output)

**Backwards compatibility** — stored config entries migrate via the step chain in `async_migrate_entry` (`__init__.py`, currently minor version 5); any change to stored keys/values needs a new idempotent step there plus a bump of `MINOR_VERSION` in `config_flow/flow.py`, covered by tests in `test_config_flow_e2e.py`. Published entity ids, unique_ids, attribute names and service fields are user-facing API — keep them stable unless a break is deliberate and called out in `RELEASE_NOTES.md`.

**Bug tracking**: Open issues live in `dev/ISSUES.md`. Claude picks them up automatically at the start of each session.

**Improvement Ideas** `dev/IMPROVEMENTS.md` List of ideas for future imporovements and changes. Developer will prompt Claude to discuss and refine them.

**TODOs** Keep track of TODOs as an ordered numbered list with checkmarks in `dev/TODO.md`. Before and after making code changes, make sure that the TODO is up to date. Mark steps completed as soon as they are done. Split TODOs into 4 parts:

- **Completed**: Short one-liners (title only, no implementation details). Periodically consolidate related items and remove entries that are no longer useful context.
- **In Progress**: Clearly defined tasks to finish before reaching out to the developer. Include enough detail to implement without ambiguity.
- **Backlog**: Upcoming work. More general — make more detailed when transitioning to In Progress.
- **Other**: Non-code tasks (e.g., icon submissions, external PRs).

Each In Progress and Backlog TODO must be tagged **[BUG]** or **[FEATURE]**. Bugs are prioritized over features.

## Architecture

### Code Structure

```text
custom_components/dynamic_ocpp_evse/
├── __init__.py                    # HA setup/unload, services (re-exports the registry helpers)
├── registry.py                    # Entry-relationship lookups (get_hub_for_load, get_*_for_hub) —
│                                  #   HA-import-free, so pure tooling can load it without Home Assistant
├── manifest.json                  # Component metadata
├── config_flow/                   # HA configuration flow (initial setup + options "Configure" flow — the single
│   │                              #   edit path; no reconfigure flow. Options menu: settings / overview / summary)
│   ├── flow.py                    # LoadJugglerConfigFlow — the initial-setup step methods
│   ├── options.py                 # LoadJugglerOptionsFlow — the "Configure" edit steps
│   ├── schemas.py                 # Every voluptuous schema builder, as module functions — no handler
│   │                              #   state; `hass` is a parameter only where a form offers entity selectors
│   ├── pages.py                   # Overview / "How it decides" read-only page text builders
│   └── helpers.py                 # Everything both handlers share: unit validation, the optional-entity
│                                  #   key groups and their normalizers, entity auto-detection, priority
│                                  #   ordering, the OCPP capability probes, phase count
├── const/                         # Constants per area: common, hub, evse, plug, hot_water_tank, group,
│                                  #   inverter, modes, power_station
├── engine/                        # HA → SiteContext bridge (reads HA states, drives the calculation)
│   ├── hub_calculation.py         # Main entry point — run_hub_calculation() builds SiteContext, calls engine
│   │                              #   Keeps the core cycle: _apply_feedback_loop(), the SOC/Excess latches,
│   │                              #   household figures; everything else lives in the siblings below
│   ├── readers.py                 # HA-state edge: _read_entity() (returns _UNAVAILABLE sentinel),
│   │                              #   _smooth() (EMA) + _stale_guard() (holdover), _coerce(),
│   │                              #   grid/inverter/fleet-member reading
│   ├── load_builders.py           # _build_[evse|plug|power_station|hot_water_tank]_load(),
│   │                              #   _add_loads_to_site(), _build_circuit_groups()
│   ├── hub_result.py              # _compute_forecast_advice(), _build_hub_result() (the published dict)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeoAlioth/Load-Juggler](https://github.com/LeoAlioth/Load-Juggler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-16 -->
