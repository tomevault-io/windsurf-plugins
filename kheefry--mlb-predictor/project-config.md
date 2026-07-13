---
trigger: always_on
description: This is a personal-use MLB prediction system that pulls live data from free
---

# MLB Predictor — Project Brief

This is a personal-use MLB prediction system that pulls live data from free
APIs, projects team scores and per-player stats for each game, and ranks
sportsbook bets by edge. It was built across several Claude Code sessions.

**You are reading this because the user asked for a fresh-eyes review.** Don't
rubber-stamp the design. Push back where it's weak. The "Soft spots" section
below points to specific things to scrutinize.

---

## Goal

Given a date, output:
1. Per-game team-score predictions (home & away runs) using park + weather + matchup features.
2. Per-player projections for batters (H, HR, TB, RBI, R, K, BB) and starters (IP, K, BB, H, ER, HR).
3. A ranked leaderboard of +EV bets vs Bovada's lines (or The Odds API consensus if a key is set), including game lines (ML / total / RL) and player props.

Outputs are a CLI (`scripts/predict.py`) and a Streamlit web app (`app.py`).

---

## Data sources (all free, no auth)

- **MLB Stats API** (`statsapi.mlb.com`) — schedule, boxscores, season + by-date-range stats, rosters, handedness. Disk-cached at `data/cache/`.
- **Open-Meteo** — historical (archive) and forecast weather; auto-routed by date. Hourly resolution; we pull the hour containing first pitch. Cached per-park-per-hour.
- **Bovada** (`bovada.lv` JSON coupon endpoint) — game lines + pitcher props (with line) + batter props (yes/no thresholds). No auth, but TOS technically prohibits scraping; treat as personal-use only.
- **The Odds API** (optional) — set `ODDS_API_KEY` env var; gives consensus US-book lines. Falls back to Bovada gracefully.

---

## Pipeline

```
scripts/build_dataset.py       Pull 2026 schedule + stats. Builds weekly Monday snapshots under
                                data/games/snapshots_2026/ for leak-free historical features.
                                Current-day stats also saved to snapshot_2026.json for live use.
scripts/build_dataset_2025.py  One-time: pull 2025 with weekly stat snapshots (~10 min).
scripts/train.py               Fit team-runs model (Poisson GLM + GBT ensemble); 2026-only holdout.
scripts/train_combined.py      Fit team-runs model on 2025+2026 combined (current production model).
scripts/train_props.py         Fit per-stat HistGBT models. Uses per-game weekly snapshots from
                                data/games/snapshots_2026/ — requires build_dataset to have run first.
scripts/fit_dispersion.py      Empirical NegBin dispersion per stat from boxscores.
scripts/backtest.py            Player projection backtest on last-7-day holdout.
scripts/backtest_2025.py       Walk-forward backtest on 2025 full season (5 monthly folds).
scripts/build_props_2025.py    Build props_bat_2025.csv / props_pit_2025.csv — analytical projections
                                vs actuals for 2025. Feeds calibrate_reliability.py for stable r estimates.
                                All API calls are cached so runs ~30 s after build_dataset_2025.py.
scripts/predict.py             CLI: today's slate + value bets.
app.py                          Streamlit: same data + per-game cards + Backtest page.
```

Run order from a clean checkout:
```
python -m scripts.build_dataset         # also generates snapshots_2026/
python -m scripts.build_dataset_2025    # one-time
python -m scripts.train_combined        # produces team_runs.joblib
python -m scripts.train_props           # produces prop_*.joblib (needs snapshots_2026/)
python -m scripts.fit_dispersion        # produces dispersion.json
python -m scripts.build_props_2025      # one-time; generates props_*_2025.csv (~30 s, all cached)
python -m scripts.calibrate_reliability # updates stat_reliability.json from 2025+2026 holdout correlations
python -m streamlit run app.py
```

> **Note:** use `python -m streamlit run app.py` — bare `streamlit` is not on PATH on this machine.
> All models in `data/models/` were last regenerated Apr 29 2026 and are current.

---

## Architecture

### Team-runs model (`src/model.py`)

- One row per (game, batting team) — long form gives 2 rows per game.
- **PoissonRegressor** GLM (log link) for calibrated mean runs.
- **HistGradientBoostingRegressor** with Poisson loss for non-linearities. Blend weight learned on holdout (currently ends up at 1.0 GLM-only — GBT not adding signal at this sample size).
- Features: offense (RPG, OPS, wOBA, BABIP, BB%), recent form (14d), opposing starter (xFIP, FIP-, BB/9, HR/9), bullpen ERA, park factors, weather (runs/HR multipliers, wind to CF, temp), engineered interactions (off×park, opp_sp×off), is_home.
- Wired for **isotonic recalibration** but turned off (didn't help on small sample).
- Trained on **2025 + 2026 combined (~2,800 games)** — gives stable coefficients.

### Player projections (`src/projections.py`)

Two layers stacked:

1. **Analytical** — empirical-Bayes shrunk rate stats × expected PA × matchup multipliers (opp pitcher quality, park, weather, L/R platoon split).
2. **ML stack** (`src/prop_models.py`) — per-stat HistGBT trained on 2026 boxscore data. Each model has a holdout-tuned blend weight stored on its `StatModel.blend_weight`. Some stats prefer pure ML (HR, runs, RBI), some prefer pure analytical (hits), some blend.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kheefry/Mlb-Predictor](https://github.com/kheefry/Mlb-Predictor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-13 -->
