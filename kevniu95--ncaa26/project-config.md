---
trigger: always_on
description: Read this file at the start of every session. Follow these rules exactly.
---

# CLAUDE.md — NCAA26 Project Rules

Read this file at the start of every session. Follow these rules exactly.

---

## Project Mission

Build a March Madness prediction system that beats KenPom in forecast accuracy:
1. **Kaggle** — March Machine Learning Mania 2026 (Brier score, $50K prize, deadline **March 19 2026**)
2. **ESPN** — 10-person bracket pool (round multiplier: R64=1, R32=2, S16=4, E8=8, F4=16, Champ=32)
3. **DraftKings/FanDuel** — only if evidence of edge after ESPN/Kaggle locked in

Selection Sunday: **~March 15, 2026** | First games: **March 20, 2026**

### Data Freshness Checklist (re-check before final submission)

Kaggle raw data was last refreshed on **2026-03-16**. The following files may update after Selection Sunday:

- [ ] **`MNCAATourneySeeds.csv` / `WNCAATourneySeeds.csv`** — 2026 seeds (required for submission). Not yet available as of 2026-03-16.
- [ ] **`SampleSubmissionStage2.csv`** — Stage 2 matchup IDs for 2026. May update.
- [ ] **Regular season results** — refreshed 2026-03-15; re-pull if conference tournament games are still ongoing.
- [ ] **`MGameCities.csv` / `WGameCities.csv`** — refreshed 2026-03-15; may add conference tourney city data.

**To refresh**: `kaggle competitions download -c march-machine-learning-mania-2026`, then re-run `scripts/02_compute_features.py` and `scripts/03_train_models.py`.

---

## IMPORTANT: No External Ratings as Model Inputs

KenPom and Barttorvik are VALIDATION TOOLS only — never model features.
We build everything from raw box scores. KenPom is used ONLY to validate our efficiency engine (r > 0.95).

**Exception**: Massey Ordinals (`MMasseyOrdinals.csv`) are approved as model features.
These are Kaggle-provided competition data aggregating ~196 public computer ranking systems.
We use the consensus (median rank) rather than individual systems.

---

## Tech Stack (locked)

- **Python**: 3.13, **Package manager**: Poetry
- **Linting**: ruff (PostToolUse hook auto-runs on every file write)
- **Storage**: Parquet via pyarrow (not CSV, except Kaggle submission)
- **Primary model**: XGBoost Cauchy loss on point spread + UnivariateSpline calibration
- **Tuning**: Optuna (100 trials), **CV**: Season-grouped GroupKFold — NEVER shuffle seasons

---

## Git Branch Strategy

Branch naming: `agent/{phase}{letter}-{task}` (e.g., `agent/03a-feature-store-glm`)
Always tell agents: "Work on branch agent/{name}. Do not merge to main."
Merge (linear history): rebase feature branch onto main, then fast-forward:
```bash
git checkout agent/{branch} && git rebase main
git checkout main && git merge --ff-only agent/{branch}
```
**NEVER merge to main without explicit user approval.**

---

## How to Run

```bash
export VENV="VIRTUAL_ENV=/Users/kniu91/Documents/projects/ncaa26/.venv"
$VENV poetry run python scenarios/<test_file> [--fixtures]
$VENV poetry run python scripts/<script_file>
```

### Ad-Hoc Analysis Rule

Any ad-hoc analysis or exploration kicked off directly by the agent in-chat (e.g., inline Python scripts, one-off ablation runs) **must complete within ~5 minutes**. Before running:
- Use early stopping, reduce rounds, subsample, or limit folds to stay within budget.
- If the task cannot reasonably finish in 5 minutes, **tell the user** with an estimated runtime and get approval before executing.
- When replicating pipeline logic (e.g., CV loops), use the existing library functions (`run_season_cv`, etc.) rather than hand-rolling — they already have early stopping and other safeguards.

---

## Data Paths

```
PROJECT_ROOT = /Users/kniu91/Documents/projects/ncaa26
RAW_KAGGLE = data/raw/kaggle/march-machine-learning-mania-2026/
RAW_KENPOM = data/raw/kenpom/
RAW_BARTTORVIK = data/raw/barttorvik/
PROCESSED = data/processed/
SUBMISSIONS = data/submissions/

# Prior year reference
NCAA25_ROOT = /Users/kniu91/Documents/projects/ncaa25
```

All hyperparameters and paths live in `src/ncaa26/utils/constants.py` — do not change without CV evidence.

---

## Source Code Layout

```
src/ncaa26/
├── ingestion/         # kaggle_downloader, kenpom_scraper, espn_scraper
├── features/          # game_preparation, four_factors, momentum, garbage_time,
│                      #   tourney_distance, massey_ordinals, auto_qualifier
├── rating/            # elo_ratings, srs_ratings, rating_pipeline
├── models/            # feature_store, xgb_spread_model, logistic_model,
│                      #   calibration, cross_validator
├── competition/       # kaggle_submission, bracket_simulator, bracket_optimizer
└── utils/             # city_utils, slot_matchups, constants
```

---

## Known Gotchas

1. **Gender encoding**: TeamID < 3000 = Men's (1), >= 3000 = Women's (0).
2. **Submission ID format**: `{Season}_{lower_TeamID}_{higher_TeamID}` — prob from lower TeamID's perspective.
3. **Season labels**: KenPom ending year = Kaggle Season (2024-25 = Season 2025).
4. **`DayNum > 118` cutoff**: Last 14 days of regular season for momentum stats.
5. **Cauchy loss `c=5000`**: Calibrated for ncaa25 scale. May need re-tuning.
6. **`num_parallel_tree=10`**: Each XGBoost "round" = 10 trees. Effective LR much lower than `eta`.
7. **Poetry venv workaround**: Prefix `poetry run` with `VIRTUAL_ENV=.venv`. See gotcha #9 in git history for details.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kevniu95) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-15 -->
