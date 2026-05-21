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

**Version 2.0** — disregard backwards compatibility. No migration processes needed.

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
├── __init__.py                    # HA component initialization
├── manifest.json                  # Component metadata
├── const.py                       # Constants and defaults
├── config_flow.py                 # HA configuration flow
├── dynamic_ocpp_evse.py          # Main entry point — reads HA states, builds SiteContext, calls engine
│                                  #   Key helpers: _derive_solar_production(), _smooth(), _coerce(),
│                                  #   _read_entity() (returns _UNAVAILABLE sentinel), _apply_feedback_loop()
├── entity_mixins.py              # HubEntityMixin, ChargerEntityMixin (device_info, data write helpers)
├── auto_detect.py                # Grid CT inversion + phase mapping auto-detection
├── [button|number|select|sensor|switch].py  # HA entities
├── calculations/                  # Core calculation logic (PURE PYTHON - no HA dependencies)
│   ├── models.py                  # Data models (SiteContext, LoadContext, CircuitGroup, PhaseConstraints)
│   ├── context.py                 # Context builder (HA → models)
│   ├── target_calculator.py       # Main calculation engine
│   └── utils.py                   # Utility functions (is_number, compute_household_per_phase)
└── translations/                  # Localization files
```

### Core Design Principle: Generality Over Special Cases

**CRITICAL**: Always strive for the most general solution possible. Minimize unnecessary distinctions.

- **Don't create separate code paths** for 1-phase vs 3-phase unless absolutely necessary
- **Use per-phase calculations universally** instead of creating special logic for each site type
- **The same algorithm should handle all cases**: 1-phase, 2-phase, 3-phase, symmetric, asymmetric
- **Make use of helper functions** for readability and error reduction

**Example**: Instead of `if site.num_phases == 3:` and branching, use per-phase arrays `[A, B, C]` where unused phases are 0.

### Multi-Phase Constraint Principle

**CRITICAL**: ALL calculation functions must return a constraint dict with keys:

- `'A'`, `'B'`, `'C'` - Single-phase limits
- `'AB'`, `'AC'`, `'BC'` - Two-phase limits (for 2-phase chargers)
- `'ABC'` - Three-phase limit (total)

This properly enforces constraints for every charger configuration:

- 1-phase charger on phase A: Uses `constraints['A']`
- 2-phase charger on AB: Uses `min(constraints['A'], constraints['B'], constraints['AB'])`
- 3-phase charger: Uses `min(constraints['A'], constraints['B'], constraints['C'], constraints['ABC'])`

**Why**: Physical reality — inverters and breakers have limits for EACH phase combination, not just individual phases.

### Calculation Flow

The calculation engine follows a 5-step process (see `target_calculator.py`):

```text
0. Refresh SiteContext (done externally in HA integration)
   → Subtract charger draws from consumption (feedback loop correction)
   ↓
1. Calculate absolute site limits (per-phase physical constraints)
   → _calculate_site_limit()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeoAlioth/Dynamic_OCPP_EVSE](https://github.com/LeoAlioth/Dynamic_OCPP_EVSE) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
