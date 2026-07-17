---
trigger: always_on
description: <!-- AUTO-GENERATED FILE. DO NOT EDIT MANUALLY. -->
---

<!-- AUTO-GENERATED FILE. DO NOT EDIT MANUALLY. -->
<!-- Source: tools/ai/llms-base.txt + tools/ai/ai_context_header.md -->
<!-- Regenerate with: python tools/ai/generate_ai_context_files.py -->

# Skforecast — Development Context

## For Contributors Working Inside This Repository

### Testing

```bash
pytest skforecast/recursive/tests/ -vv           # Run a specific module's tests
pytest --cov=skforecast --cov-report=html         # Coverage report
pytest -n auto                                    # Parallel execution (pytest-xdist)
```

Markers: `@pytest.mark.slow` for long-running tests (skip with `-m "not slow"`).

### Code Style

- NumPy-style docstrings
- Type hints for function signatures
- PEP 8 compliant (max line length 88, enforced by ruff)
- Double quotes for strings (ruff `quote-style = "double"`)
- Relative imports within package
- When generating code comments, docstrings, and documentation, do not use en dashes (–), or em dashes (—). Use commas, colons, semicolons, or parentheses for punctuation instead.

### Dependencies

Core: numpy>=1.26, pandas>=2.1,<3.0, scikit-learn>=1.4, scipy>=1.12, optuna>=4.0, joblib>=1.3, numba>=0.59, tqdm>=4.66, rich>=13.9
Optional: statsmodels>=0.13,<0.15 (stats), matplotlib>=3.7,<3.11 + seaborn>=0.12,<0.14 (plotting), keras>=3.0,<4.0 (deep learning)

### Python environment

Before running any Python command (tests, scripts, notebooks, `pip install`, etc.)
for the first time in a session, run `conda env list` and ask which environment to
use. Do not assume the active environment. Once the user confirms an environment,
reuse it for the rest of the session without asking again.

---

# Skforecast — Complete API & Workflow Reference

(The content below is the full `llms-base.txt` and applies to any user of skforecast)

# Skforecast

> Python library for time series forecasting using scikit-learn compatible models, statistical methods, and foundation models

This document is for skforecast v0.23.0+. If you are using an older version, check the documentation at skforecast.org.

Skforecast is a Python library for time series forecasting using scikit-learn compatible models, statistical methods, and foundation models. It works with any estimator compatible with the scikit-learn API (LightGBM, XGBoost, CatBoost, Keras, etc.).

## Quick Info

- Version: 0.23.0
- License: BSD-3-Clause
- Python: 3.10, 3.11, 3.12, 3.13, 3.14
- Repository: https://github.com/skforecast/skforecast
- Documentation: https://skforecast.org
- PyPI: https://pypi.org/project/skforecast/

## Installation

```bash
pip install skforecast
```

Optional dependencies:
```bash
pip install skforecast[stats]        # For ARIMA, SARIMAX, ETS models
pip install skforecast[plotting]     # For visualization
pip install skforecast[deeplearning] # For RNN/LSTM models
```

## Project Structure

```
skforecast/
├── base/                    # ForecasterBase - abstract parent class for all forecasters
├── recursive/               # ForecasterRecursive, ForecasterRecursiveMultiSeries,
│                            # ForecasterRecursiveClassifier, ForecasterStats, ForecasterEquivalentDate
├── direct/                  # ForecasterDirect, ForecasterDirectMultiVariate
├── deep_learning/           # ForecasterRnn, create_and_compile_model
├── foundation/              # FoundationModel, ForecasterFoundation
│                            # (zero-shot: Chronos-2, TimesFM 2.5, Moirai-2, TabICL, TabPFN-TS, TFC-T0)
├── stats/                   # Arima, Sarimax, Ets, Arar, acf, pacf, calculate_lag_autocorrelation
├── preprocessing/           # TimeSeriesDifferentiator, RollingFeatures, CalendarFeatures,
│                            # QuantileBinner, ConformalIntervalCalibrator, reshape_* functions
├── model_selection/         # backtesting_forecaster, grid/random/bayesian search, TimeSeriesFold
├── feature_selection/       # select_features, select_features_multiseries
├── metrics/                 # MASE, RMSSE, sMAPE, CRPS, coverage, pinball loss
├── datasets/                # 30+ built-in datasets (fetch_dataset, load_demo_dataset)
├── drift_detection/         # RangeDriftDetector, PopulationDriftDetector
├── utils/                   # Shared validation and transformation functions
├── exceptions/              # Custom warnings and exceptions
├── plot/                    # plot_residuals, plot_prediction_intervals, plot_prediction_distribution,
│                            # plot_multivariate_time_series_corr, set_dark_theme, backtesting_gif_creator
└── experimental/            # Experimental features (API may change)
```

### Module Relationships

- **Forecasters inheriting from `ForecasterBase`**: ForecasterRecursive, ForecasterRecursiveMultiSeries, ForecasterRecursiveClassifier, ForecasterDirect, ForecasterDirectMultiVariate, ForecasterRnn
- **Standalone forecasters (no inheritance)**: ForecasterStats, ForecasterEquivalentDate, ForecasterFoundation
- Statistical models in `stats/` are wrapped by `ForecasterStats` (in `recursive/`)
- `ForecasterFoundation` (in `foundation/`) wraps a `FoundationModel`, which delegates to an adapter class (`ChronosAdapter`, `TimesFMAdapter`, `MoiraiAdapter`, `TabICLAdapter`, `TabPFNAdapter`, `T0Adapter`) resolved from the HuggingFace `model_id`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skforecast/skforecast-ai](https://github.com/skforecast/skforecast-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-17 -->
