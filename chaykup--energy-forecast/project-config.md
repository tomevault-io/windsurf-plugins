---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Pipeline Commands

The pipeline is linear. Each step must complete before the next.

```bash
# Step 1 — Ingest raw data from APIs (~10–15 min for CAISO)
python -m src.data.ingest_all --market CAISO
python -m src.data.ingest_all --market ERCOT
python -m src.data.ingest_all --market both

# Step 2 — Build feature matrix
python -m src.data.feature_engineering --market CAISO

# Step 3 — Train all 6 models + evaluate
# CRITICAL: Set these env vars first on Intel Mac to avoid LSTM deadlock:
#   export MKL_THREADING_LAYER=GNU && export OMP_NUM_THREADS=1
python -m src.training.train_all_models --market CAISO
python -m src.training.train_all_models --market CAISO --skip-timegpt   # reuse saved TimeGPT preds

# Step 4a — Compute extended metrics (wMAPE, skill scores, arbitrage capture, breakdowns)
# Reads existing parquets — does NOT retrain. Outputs model_comparison.json.
export MKL_THREADING_LAYER=GNU && export OMP_NUM_THREADS=1
python -m src.evaluation.evaluate --market both

# Step 4b — Upload metrics + predictions to Supabase
# Run migrations/model_metrics_v2.sql in Supabase SQL editor first (one-time).
python -m src.deployment.upload_results --market both
python -m src.deployment.upload_results --market both --sample-rate 6   # every 6th row = ~4h resolution
python -m src.deployment.upload_results --market both --skip-predictions  # metrics only
python -m src.deployment.upload_results --market both --skip-metrics      # predictions only

# Frontend dev server
cd frontend && npm run dev
cd frontend && npm run build
```

## Architecture

**6-model benchmark pipeline** for hourly electricity LMP (Locational Marginal Price) forecasting in CAISO and ERCOT. Tests whether HMM regime detection improves a hybrid ML architecture vs. Nixtla's TimeGPT foundation model.

**Data flow:**
```
External APIs → data/raw/*.parquet → data/processed/*_features.parquet
→ data/models/{CAISO,ERCOT}/ (artifacts) + data/results/{CAISO,ERCOT}/ (predictions + JSON)
→ Supabase Postgres (model_metrics, predictions, model_registry tables)
→ React frontend on Vercel (project: lmp-model-leaderboard)
```

**The 6 models** (numbered as trained in `train_all_models.py`):
1. Naive baseline — time-of-day heuristic, no ML
2. TimeGPT zero-shot — Nixtla API
3. TimeGPT fine-tuned — Nixtla API, 30 finetune steps
4. XGBoost only — single model, no regime detection
5. HMM + XGBoost — 3 per-regime XGBoost models
6. HMM + XGBoost + LSTM — full hybrid (`HybridPipeline` class)

**Model architecture detail:**
- `RegimeDetector` (hmmlearn `GaussianHMM`) classifies each hour into one of 3 regimes based on LMP return, 24h rolling volatility, and LMP level
- `RegimeXGBoost` trains one XGBoost per regime; `get_residuals()` feeds the LSTM
- `RegimeLSTM` (2-layer PyTorch LSTM) predicts the next XGBoost residual from a 24h sliding window of past residuals — final forecast = XGBoost prediction + LSTM correction
- `HybridPipeline` orchestrates all three and also generates battery charge/discharge recommendations

**Temporal splits** (same for both markets):
- Train: up to 2024-03-31
- Val: 2024-04-01 → 2024-07-31
- Test: 2024-08-01 → end (held-out, never touched during training)

## Critical Constraints

**Timestamp handling:** All timestamps are normalized to **naive UTC** throughout the Python pipeline. The `to_naive_utc()` helper in `feature_engineering.py` handles both Series and DatetimeIndex. Never compare tz-aware with tz-naive; use `df["hour"]` (naive UTC) not `df["Time"]` (tz-aware) for splits and comparisons.

**HMM row alignment:** `prepare_observations()` calls `pct_change()` + `dropna()`, dropping the first ~24 rows. Downstream code must align via `df.iloc[-len(states):]`, not by integer position.

**CAISO LMP locations use hyphens:** `TH_NP15_GEN-APND`, not `TH_NP15_GEN_APND`.

**EIA API quirks:** Use respondent `"CISO"` for CAISO and `"ERCO"` for ERCOT. Append `T00` to start/end date strings for hourly frequency.

**ERCOT LMP source:** Uses `ErcotAPI.get_spp_day_ahead_hourly()` (not gridstatus's ERCOT class). Requires `ERCOT_API_USERNAME`, `ERCOT_API_PASSWORD`, `ERCOT_PUBLIC_API_SUBSCRIPTION_KEY` env vars. The `SPP` column is renamed to `LMP`.

**TimeGPT input format:** `_prepare_input()` uses `Location` as `unique_id` (one series per pricing node — 3 for CAISO, 4 for ERCOT). Collapsing to one `unique_id` causes duplicate timestamp errors.

**Per-regime XGBoost:** Strip `early_stopping_rounds` from params (the `REGIME_XGB_PARAMS` dict in `train_all_models.py`). Per-regime models have no `eval_set`, so early stopping is unavailable.

**`src/data/__init__.py`:** Imports only client classes (`LMPClient`, `EIAClient`, `WeatherClient`, `FREDClient`). Do NOT import `FeatureEngineer` or `ingest_all` there — causes circular import when running as `-m` module.

**Supabase:** The service key (`SUPABASE_SERVICE_KEY`) is used only in Python backend scripts. The React frontend uses the anon key (`VITE_SUPABASE_ANON_KEY`) exclusively. Never put the service key in frontend code.

## Key Files

| File | Role |
|------|------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chaykup/energy-forecast](https://github.com/chaykup/energy-forecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
