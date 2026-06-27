---
trigger: always_on
description: **Last updated:** 2026-06-25 (Session 140)
---

# CLAUDE.md — Traveller World & System Generator

**Last updated:** 2026-06-25 (Session 140)  
**Branch:** `main`  
**Virtual environment:** `.venv` (Python 3.11, includes PySide6)

This file is a router. Read `context/common.md` first on every session, then
read only the context files listed below for the specific task at hand.

---

## Always read first

- [`context/common.md`](context/common.md) — repo layout, code quality, tests, licence, CI

---

## Routing table

| Task | Read these files |
|------|-----------------|
| Any task involving data structures, type signatures, or dataclasses | [`context/data-structures.md`](context/data-structures.md) |
| Understanding the generation pipeline, entry points, or RNG/seed behaviour | [`context/generation-pipeline.md`](context/generation-pipeline.md) |
| `traveller_stellar_gen.py` or `traveller_orbit_gen.py` | [`context/stellar-orbit.md`](context/stellar-orbit.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_system_gen.py`, `traveller_world_gen.py`, `traveller_world_physical.py`, or `traveller_hydro_detail.py` | [`context/system-world.md`](context/system-world.md) + [`context/data-structures.md`](context/data-structures.md) + [`context/generation-pipeline.md`](context/generation-pipeline.md) |
| `traveller_world_atmosphere_detail.py` | [`context/atmosphere-detail.md`](context/atmosphere-detail.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_world_detail.py`, `traveller_moon_gen.py`, or `traveller_belt_physical.py` | [`context/detail-moon.md`](context/detail-moon.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_world_starport_detail.py` | [`context/starport-detail.md`](context/starport-detail.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_world_population_detail.py` | [`context/social-detail.md`](context/social-detail.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_world_government_detail.py` | [`context/government-detail.md`](context/government-detail.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_world_law_detail.py` | [`context/social-detail.md`](context/social-detail.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_world_tech_detail.py` | [`context/social-detail.md`](context/social-detail.md) + [`context/data-structures.md`](context/data-structures.md) |
| `traveller_map_fetch.py` | [`context/map-fetch.md`](context/map-fetch.md) + [`context/generation-pipeline.md`](context/generation-pipeline.md) |
| `azure-api/function_app.py` or `azure-api/shared/helpers.py` | [`context/api-layer.md`](context/api-layer.md) |
| `fastapi/app.py` or `fastapi/helpers.py` | [`context/api-layer.md`](context/api-layer.md) |
| Adding a new root-level Python module, or any azure-api sync issue | [`context/azure-sync.md`](context/azure-sync.md) |
| `system_pipeline.py` | [`context/generation-pipeline.md`](context/generation-pipeline.md) |
| `system_map.py` | [`context/system-map.md`](context/system-map.md) + [`context/data-structures.md`](context/data-structures.md) |
| `gen-ui/app.py` | [`context/gen-ui.md`](context/gen-ui.md) + [`context/data-structures.md`](context/data-structures.md) |
| Bug investigation or compliance review | Relevant module context file(s) above + [`context/session-history.md`](context/session-history.md) for prior fixes |
| Implementing a deferred feature | [`context/deferred-features.md`](context/deferred-features.md) + relevant module file(s) above |
| Any work touching temperature, HZ, or orbital position | Also read [`context/data-structures.md`](context/data-structures.md) (HZ sign convention + Orbit# vs AU table) |

---

## Always-needed facts (no file read required)

- **Pylint:** `.venv/bin/pylint <file>` — target **10.00/10 per file**
- **Tests:** `.venv/bin/pytest tests/ -q` — **2843 tests**, all must pass
- **RNG:** Injectable `random.Random` instance; each generation module has a
  module-level `_rng` sentinel (initially `random` the module). Public
  entry-point functions accept `rng: Optional[random.Random] = None`; when
  provided, they write it to `_rng`. `generate_world()` only replaces `_rng`
  when an explicit `seed` or `rng` argument is given. `generate_full_system()`
  always creates a fresh `random.Random(seed)` and propagates it.
  Adding any dice roll anywhere shifts all subsequent results for that seed.
- **`attach_detail()` is always a separate explicit step.** Never call it
  automatically inside `generate_full_system()`. Always pass `rng=rng` when
  calling it from a handler or wherever the same RNG instance must be continued.
- **New dice rolls belong at the end of the pipeline** to minimise seed disruption.

---

## Update rules

After any session in which code, APIs, tests, or documented behaviour change:

1. Add one row to [`context/session-history.md`](context/session-history.md).
2. Update **Last updated** and session number at the top of this file.
3. Update the specific context file(s) whose content changed:

| What changed | Update which file |
|---|---|
| Data structure field added/removed/renamed | `context/data-structures.md` |
| New API endpoint or error code | `context/api-layer.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elured-code/traveller-world-gen](https://github.com/Elured-code/traveller-world-gen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-27 -->
