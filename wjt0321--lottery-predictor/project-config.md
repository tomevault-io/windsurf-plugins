---
trigger: always_on
description: This file gives non-Claude agents the minimum context needed to work in this repository. Use `README.md` as the full user-facing reference.
---

# AGENTS.md

This file gives non-Claude agents the minimum context needed to work in this repository. Use `README.md` as the full user-facing reference.

## Project Snapshot

- Repository purpose: entertainment-only Double Color Ball prediction and backtesting tool
- Main entry: `predict.py`
- Default flow: `team` mode
- Experimental flow: `team-cover` mode
- Config hub: `project_config.py` (`GLOBAL_CONFIG` singleton → `to_runtime_config()`)
- Expert registry: `agent_registry.py` (8 experts: hot/cold/missing/balanced/random/cycle/sum/zone)
- Analysis loop: `analyze_archive.py` + `prediction_archive/` + `config/*.latest.json`

## Architecture Overview

```
lottery_data.json ──→ predict.py (team mode pipeline)
                        │
                        ├── 1. _load_patches(): auto-load weight/matrix/param patches
                        ├── 2. differential_learning(): expert weight backtest learning
                        ├── 3. 8 expert proposals
                        ├── 4. build_core_pool_snapshot(): aggregate red pool (22) + blue pool (10)
                        │       └── position weights applied at scoring stage
                        ├── 5. _build_debate_pool(): anti-consensus debate (experts re-evaluate excluded 11 balls → re-rank)
                        ├── 6. BlueBallEngine.predict(): multi-dim blue scoring
                        ├── 7. _build_blue_debate(): blue anti-consensus debate (promote low-score blues with standout dimensions)
                        ├── 8. generate_team_matrix_tickets(): matrix + ticket-5 dynamic 0/1/2 offset
                        │       └── weakest row becomes ticket 5: preserve row or apply 1/2 evidence-backed offsets
                        └── 9. archive → prediction_archive/YYYYXXX.txt

prediction_archive/ ──→ analyze_archive.py
                           ├── read archives + backfill actual results from lottery_data.json
                           └── export 3 patch types → config/*.latest.json (re-injected)
```

**Self-learning loop**: `predict → archive → analyze → patches → predict`

## Core Module Roles

| File | Role | Key Notes |
|------|------|-----------|
| `predict.py` | Main entry / orchestration | team/single/team-cover modes; ticket generation; backtest workflows; patch injection; anti-consensus debate |
| `project_config.py` | **Central config** | `ProjectConfig` dataclass → `to_runtime_config()` produces pool/fusion/matrix/blue/cover param groups |
| `agent_registry.py` | Expert registry | Fixed 8-expert set, excludes lstm |
| `blue_ball_engine.py` | Blue ball engine | 7-dim analysis (missing/parity/zone/amplitude/heat/moving-avg/Bayesian); params via config dict |
| `analyze_archive.py` | Offline analyzer | Reads archive KV, backfills results, groups by reproducible version identity, exports 3 patch types + JSON/CSV/versions CSV |
| `archive_provenance.py` | Archive provenance | Canonical runtime/patch hashes plus schema/seed/commit metadata |
| `backtest_reporting.py` | Backtest reporting | Stability statistics, bootstrap CI, rolling-fold/candidate helpers, JSON/CSV export |
| `backtest_cache.py` | Backtest cache | Bounded LRU cache for invariant sample contexts, deterministic keys, telemetry |
| `parameter_promotion.py` | Promotion guard | Evidence gates; writes audit decision and candidate patch only; never activates latest patch |
| `enhanced_analysis.py` | Enhanced analysis | Pool influence, extended data fusion weights |
| `feature_importance.py` | Feature importance | Pearson/Spearman correlation, zero extra deps |
| `visual_analyzer.py` | Visualization | matplotlib charts (optional dep) |
| `update_data.py` | Data update | Playwright + BeautifulSoup scraping from 500.com |
| `manual_data_import.py` | Manual import | JSON/CSV/TXT external data import |

## Key Internal Functions (predict.py)

team mode call chain:

```
run_team_mode()
  ├── _load_patches()              # auto-discover weight/matrix/param patches
  ├── differential_learning()      # expert weight backtest learning
  ├── [8 × agent_*_propose()]      # per-expert proposals
  ├── build_core_pool_snapshot()   # weighted aggregation + position weight scoring
  ├── _build_debate_pool()         # anti-consensus debate: experts re-evaluate excluded 11 balls
  ├── BlueBallEngine(records, blue_params).predict()  # blue ball scoring
  ├── _build_blue_debate()         # blue debate: promote low-score blues with standout dimensions
  ├── generate_team_matrix_tickets()      # matrix + ticket-5 dynamic 0/1/2 offset
  │     ├── _build_offset_candidate_profiles() # independent counter-evidence over all 33 reds
  │     ├── _select_scientific_offset_reds()   # constrained 1/2-ball search for ticket 5
  │     ├── _choose_dynamic_offset_plan()      # threshold policy chooses 0/1/2
  │     └── _select_blue_ball_for_row()        # blue dedup selection
  └── _archive_prediction()        # write archive
```

Backtest entry points: `team_matrix_backtest_report()` / `team_cover_backtest_report()` / `team_stability_backtest_report()` / `team_threshold_calibration_report()`

## Configuration System

`project_config.py::ProjectConfig` — single source of truth:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wjt0321/lottery-predictor](https://github.com/wjt0321/lottery-predictor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
