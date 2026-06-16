---
trigger: always_on
description: Tactical guide for AI agents working on Thread. For conceptual overview (what fidelity score means, what Thread is for), read `README.md` — do not duplicate that prose here.
---

# Agent Instructions

Tactical guide for AI agents working on Thread. For conceptual overview (what fidelity score means, what Thread is for), read `README.md` — do not duplicate that prose here.

**Canonical file:** this is it. `CLAUDE.md` points here. Keep all agent guidance in this file.

---

## Build & Test

```bash
uv sync                               # install dependencies
uv run pytest -m "not integration"    # 134 unit tests — no dolt binary needed
uv run pytest -m integration          # 13 integration tests — require dolt on PATH
uv run pytest                         # all 147 tests
```

Integration tests spin up a real `dolt sql-server` against a user-provided `sample-data/.beads/` directory at the repo root. `sample-data/` is gitignored — drop any real Beads project in there (or symlink one) to exercise the full pipeline. When `sample-data/.beads/` is absent the integration suite auto-skips. If tests fail with connection errors, check `dolt version` is on PATH.

Run the CLI during development:

```bash
uv run python -m thread.cli refresh
uv run python -m thread.cli prime
uv run python -m thread.cli prime --json
uv run python -m thread.cli report --output /tmp/thread-report.html
uv run python -m thread.cli query "SELECT COUNT(*) FROM dim_bead"
uv run python -m thread.cli query "SELECT * FROM dim_bead" --csv --limit 10
uv run python -m thread.cli sessions --json --limit 5
uv run python -m thread.cli interactions --tools
```

## Architecture at a glance

```
thread/
  dolt.py              # dolt sql-server lifecycle, connection context managers
  extractor.py         # reads Dolt → populates 10 DuckDB tables (6 v1 + 4 v2)
  actor_classifier.py  # 4-tier cascade: hop_uri → role_type → session → heuristic
  schema.sql           # 10 tables + 22 views, all LEFT JOINs, COALESCE on nullables
  prime.py             # v2 project health: 15 metric signals with verdicts, sessions,
                       #   compliance, interactions, agent knowledge, relative cost
  report.py            # v2 HTML report: 6 headline cards, session timeline, compliance
                       #   scorecard, audit trail, interactive histogram, insights
  cli.py               # click entrypoint: refresh, prime, report, query, sessions,
                       #   interactions
```

**Pipeline:** `refresh` starts dolt sql-server → extracts into `thread.duckdb` → `prime` / `report` / `query` / `sessions` / `interactions` read from that file. Read-only against Beads. Output lands in `.beads/thread.duckdb`.

**Tables:** `dim_bead`, `dim_hierarchy`, `dim_actor`, `fact_bead_lifecycle`, `fact_bead_events`, `fact_dep_activity`, `dim_session`, `bridge_session_bead`, `fact_interactions`, `dim_agent_memory`.

**Views:** `v_bead_scores`, `v_bead_dep_activity`, `v_weekly_trends`, `mart_epic_summary`, `mart_project_summary`, `mart_session_summary`, `v_daily_trends`, `v_interaction_summary`, `v_model_usage`, `v_tool_usage`, `v_interaction_hourly`, `v_status_transitions`, `v_close_reasons`, `v_daily_activity`, `v_close_velocity`, `v_dep_order_violations`, `v_title_reason_pairs`, `v_queue_wait`, `v_spec_quality_correlation`, `v_priority_performance`, `v_type_performance`, `v_session_compliance`.

## Critical invariants

These are load-bearing. Break one and the numbers lie.

- **`closer_actor_key` must never be `'root'`.** Dolt embedded mode commits every change as user `root`. Closer attribution must come from the events table, not Dolt commit author. Integration test `test_closer_is_not_dolt_root` enforces this.
- **LEFT JOIN everywhere.** Orphan rows (hierarchy entries with no matching `dim_bead`, lifecycle rows without actors, etc.) must participate in scoring. Never `INNER JOIN` in `schema.sql`.
- **`COALESCE` on every nullable before arithmetic.** Effort, fidelity, and penalty formulas all coalesce to 0 before multiplying. Missing the coalesce silently drops rows from aggregates.
- **Views must work on empty tables.** `test_views_work_on_empty_tables` runs every view against a fresh empty schema and asserts it returns 0 rows (not an error).
- **Cost is relative, never dollars.** All cost is expressed as multiples of the project's median cycle time. `effort_score` in v_bead_scores is the per-bead cost multiple (active_time / project median). `fidelity_score` is compliance-based (penalises skip-claim, reopens, rejections).
- **Plain outcome language in all user-facing strings.** Never surface column names, formulas, or internal terms (`fidelity_score`, `compaction_level`, `v_bead_scores`) in `prime` output. `test_signals_are_plain_language` scans for these.
- **Workflow-awareness is mandatory in `prime` and `report`.** Detect `epic` / `flat` / `mixed` / `empty` via `mart_project_summary` and adapt headline metrics accordingly. Do not assume epics exist.

## Non-obvious gotchas

- **`quality_score` is ambiguous across joins.** `v_bead_scores` and `fact_bead_lifecycle` both carry it. When joining, always qualify as `s.quality_score` (or whichever alias). Same for `fidelity_score` and `effort_score`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jklenk/thread](https://github.com/jklenk/thread) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
