---
trigger: always_on
description: This is a Japanese horse racing (Keiba) AI prediction system for JRA (Japan Racing Association) races. The goal is to predict Exacta (1st and 2nd place in exact order) outcomes and maximize ROI using expected value-based betting strategy.
---

# Claude Code Instructions

## Project Overview

This is a Japanese horse racing (Keiba) AI prediction system for JRA (Japan Racing Association) races. The goal is to predict Exacta (1st and 2nd place in exact order) outcomes and maximize ROI using expected value-based betting strategy.

## Tech Stack

- **Analysis/ML Training**: Python (use `uv` for package management)
- **Inference & Live Prediction**: Rust (single binary, no Python dependency)
- **Data Source**: Kaggle JRA Dataset (2019-2021) + JRA official (future: JRA-VAN paid data)

## Project Structure

```
keiba-ai/
├── config/settings.py        # Configuration
├── data/
│   ├── raw/                  # Raw data (Kaggle CSV files)
│   ├── processed/            # Processed/feature-engineered data
│   └── cache/                # Scraper cache (auto-created)
├── src/
│   ├── data_collection/      # Data download scripts
│   ├── preprocessing/        # Feature engineering
│   ├── models/               # ML models, backtesting, calibration
│   └── api/                  # Rust inference API & CLI
│       ├── src/
│       │   ├── main.rs       # Entry point (CLI + server)
│       │   ├── cli.rs        # CLI commands (serve, predict, backtest, live)
│       │   ├── routes.rs     # API handlers
│       │   ├── model.rs      # ONNX inference
│       │   ├── backtest.rs   # Walk-forward backtesting
│       │   ├── exacta.rs     # Exacta probability (Harville formula)
│       │   ├── trifecta.rs   # Trifecta probability
│       │   ├── quinella.rs   # Quinella probability
│       │   ├── trio.rs       # Trio probability
│       │   ├── wide.rs       # Wide probability
│       │   ├── betting.rs    # EV, Kelly criterion
│       │   ├── calibration.rs # Probability calibration
│       │   ├── config.rs     # Configuration
│       │   ├── types.rs      # Request/response types
│       │   ├── scraper/      # Live race scraper (Rust)
│       │   │   ├── mod.rs           # Module definition
│       │   │   ├── browser.rs       # chromiumoxide browser automation
│       │   │   ├── cache.rs         # File-based cache with TTL
│       │   │   ├── rate_limiter.rs  # Token bucket rate limiter
│       │   │   ├── feature_builder.rs # 39 ML features
│       │   │   ├── parsers/         # HTML/JSON parsers
│       │   │   │   ├── race_card.rs # Race card parser
│       │   │   │   ├── horse.rs     # Horse profile parser
│       │   │   │   ├── jockey.rs    # Jockey profile parser
│       │   │   │   ├── trainer.rs   # Trainer profile parser
│       │   │   │   └── odds.rs      # Odds API parser
│       │   │   └── historical/      # Historical data scraper
│       │   │       ├── mod.rs       # URL builders
│       │   │       ├── race_list.rs # Race schedule parser
│       │   │       ├── race_result.rs # Race result parser
│       │   │       └── odds_history.rs # Historical odds parser
│       │   └── storage/         # SQLite storage layer
│       │       ├── mod.rs
│       │       ├── schema.rs    # Table definitions
│       │       └── repository.rs # CRUD operations
│       └── scripts/
│           └── prepare_backtest_data.py
├── data/
│   └── historical/
│       └── keiba.db             # SQLite database (scraped data)
├── scripts/                  # Python scripts
│   ├── retrain.py            # Model retraining pipeline
│   ├── run_validation.py     # Validation backtest
│   └── export_onnx.py        # ONNX model export
├── tests/                    # Python unit tests (213 tests)
│   └── test_*.py             # Model/backtesting tests
└── notebooks/                # Jupyter exploration
```

## Development Phases

All phases completed:

- [x] **Phase 1**: Data Collection & Exploration - Kaggle dataset (2019-2021)
- [x] **Phase 2**: Model Building - LightGBM position probability model
- [x] **Phase 3**: Backtesting - Walk-forward validation (+19.3% ROI with calibration, on optimistic post-race/estimated odds — see Known Issues; not verified on real combination odds)
- [x] **Phase 4**: Rust Inference API - REST API with all 5 bet types
- [x] **Phase 5**: Live Race Scraper - netkeiba.com integration
- [x] **Phase 6**: Full Rust Migration - Single binary with `live` command (no Python dependency)

## Data Source

### Primary: Kaggle JRA Horse Racing Dataset
- URL: https://www.kaggle.com/datasets/takamotoki/jra-horse-racing-dataset
- Period: 1986-2021 (use 2019-2021 for this project)
- Format: CSV (pre-processed, easy to use)
- Contents: Race results, betting odds, lap times, corner passing orders

### Historical: netkeiba.com Scraper
- URL: https://db.netkeiba.com/
- Period: 2022-2025 (scraped via `scrape-historical` command)
- Format: SQLite database (`data/historical/keiba.db`)
- Contents: Race results, **final win odds (単勝) per horse**, horse/jockey/trainer data
- **NOT available**: pre-race odds for all combinations. netkeiba does not retain
  full-combination odds after a race finishes — only winning-combination payouts
  remain. Real all-combination odds require JRA-VAN (paid) or forward-capture of
  live odds before post time.

### Future: JRA-VAN DataLab (Paid)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tsukasaI/keiba-ai](https://github.com/tsukasaI/keiba-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
