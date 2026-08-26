---
trigger: always_on
description: ﻿# Horse Racing Predictions — Copilot instructions
---

﻿# Horse Racing Predictions — Copilot instructions

Purpose: concise, repo-specific guidance for AI coding agents working on the horse-racing-predictions project.

Key idea (big picture)
- Data pipeline: raw API racecards -> data/raw/ -> processing & feature engineering (scripts/) -> data/processed/ (Parquet/CSV) -> model artifacts in models/ -> predictions outputs in data/processed/predictions_YYYY-MM-DD.csv
- ML model: XGBoost horse-win classifier trained via `scripts/phase3_build_horse_model.py`; features computed with pure functions and expanding-window career stats to avoid lookahead bias.
- UI: Multipage Streamlit app with lightweight main page (predictions.py - 733 lines) for Today/Tomorrow predictions and heavy data explorer page (pages/data_explorer.py - 767 lines) for historical analysis. Common utilities in shared/utils.py (257 lines).

Critical workflows (commands)
- Setup: create/activate `.venv/` and `pip install -r requirements.txt`
- Run UI: `streamlit run predictions.py` (multipage app: main predictions page + data explorer)
- Generate predictions (single day): `python scripts/predict_todays_races.py --date YYYY-MM-DD`
- Build engineered dataset: `python scripts/build_engineered_dataset.py`
- Train stacked ensemble: `python scripts/ensemble_model.py`
- Run backtesting: `python scripts/backtest_walk_forward.py --folds 6`
- Run bankroll management: `python scripts/rl_bankroll_manager.py --date YYYY-MM-DD --report --update-history`
- Add weather features: `python scripts/add_weather_features.py --offline`
- Batch generate: `python scripts/batch_generate_predictions.py` (scans `data/raw/`)
- Fetch racecards (example): `python scripts/fetch_racecards.py --date YYYY-MM-DD` (saves `data/raw/racecards_YYYY-MM-DD.json`)
- Tests: `pytest tests/` (tests avoid live network calls; use fixtures)
- GitHub Actions: Daily predictions (07:00 UTC) and weekly model training (Monday 07:00 UTC)

Repository conventions & patterns
- Environment & secrets: use `.venv/`, store secrets in `.env` (gitignored), mirror any new vars in `.env.example`.
- Data caching: Always prefer offline cached API responses in `data/raw/` for development and tests. Tests must not do live API calls.
- Models: artifacts live in `models/` and are gitignored; training logic lives in `scripts/`.
- Feature engineering: functions should be pure where possible and use expanding/windowed aggregations to prevent leakage. Weight-related features exist for handicap races: `weight_lbs`, `weight_vs_avg`, `is_top_weight`, `weight_change`.
- **DATA LEAKAGE PREVENTION (CRITICAL)**: When adding new temporal features, ALWAYS:
  1. Sort by grouping variable (horse/sire/jockey) AND date before calculations
  2. Use `shift(1)` to exclude the current race from expanding/rolling windows
  3. If filtering data (e.g., by going type), re-sort by [group_var, date] after filtering
  4. Never use cumulative functions without shift(1) first
  5. Verify race-level features only compare within the same race (no cross-race contamination)
  6. Document temporal integrity in function docstrings
  7. **MANDATORY**: After adding ANY new features, run `python scripts/verify_no_leakage.py` to verify temporal integrity
  8. **MANDATORY**: When evaluating model performance, ALWAYS use temporal split (not random split) to prevent leakage
  9. **VERIFICATION**: Use `scripts/verify_no_leakage.py` to check for common leakage patterns before committing feature changes

Integration points and constraints
- The Racing API: HTTP Basic Auth via `RACING_API_USERNAME` / `RACING_API_PASSWORD` (see `examples/api_example.py` and `scripts/fetch_racecards_api.py`).
- The Odds API: API key in `ODDS_API_KEY` passed as `?apiKey=...` (see `examples/odds_api_example.py` and `scripts/fetch_odds.py`).
- Rate limits: both APIs are limited (500 calls/month). Cache aggressively and document call counts in long-running scripts.

Testing notes
- Use fixtures in `tests/fixtures/` (e.g., saved racecards) and monkeypatch network calls in `tests/conftest.py`.
- Offline-first tests: prefer `data/raw/` test fixtures over live requests.

Streamlit and UI gotchas
- Newer Streamlit: the `use_container_width` parameter has been removed in newer Streamlit versions.
  - Replacement mapping: `use_container_width=True` → `width='stretch'`, `use_container_width=False` → `width='content'`.
  - IMPORTANT: **Do NOT re-add `use_container_width` anywhere** — it is deprecated/removed and may raise runtime errors; always use `width='stretch'` or `width='content'` instead.
- Timezone: set `APP_TIMEZONE` to force server-side day boundaries; `predictions.py` respects `APP_TIMEZONE`.

Quick file map (where to start)
- `predictions.py` — Streamlit main page (lightweight, 733 lines): Today/Tomorrow predictions, Model Insights
- `pages/data_explorer.py` — Historical data explorer (767 lines): Horses, Courses, Jockeys, Overall, Raw Data, Predicted Fixtures, Betting Watchlist
- `shared/utils.py` — Common utilities (257 lines): load_model, load_data, get_dataframe_height, safe_st_call, memory profiling
- `scripts/predict_todays_races.py` — single-day prediction runner
- `scripts/batch_generate_predictions.py` — batch runner that scans `data/raw/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gmalbert/horse-racing-predictions](https://github.com/gmalbert/horse-racing-predictions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
