---
trigger: always_on
description: This file provides guidance to agents (i.e., ADAL) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to agents (i.e., ADAL) when working with code in this repository.

## Essential Commands

### Package Management

This project uses **uv** (not Poetry). The lock file is `uv.lock`.

```bash
uv sync                                    # Core deps only
uv sync --extra dev --extra notebooks      # With dev/notebook extras
uv sync --all-extras                       # All optional features
source .venv/bin/activate
```

Optional dependency groups: `dev`, `notebooks`, `ml` (torch/transformers/litellm), `tuning`, `viz`, `storage`, `full`.

### Testing

```bash
uv run pytest -m "not integration"         # Unit tests only (CI/CD - no API keys needed)
uv run pytest -m integration               # Integration tests (requires .env with API keys)
uv run pytest tests/data/test_indicators.py  # Specific test file
uv run pytest -k "test_portfolio"          # Tests matching pattern
uv run pytest --cov=quantrl_lab            # With coverage
```

### Code Quality (Pre-commit Hooks)

Pre-commit hooks run automatically on `git commit`. Run manually with:

```bash
pre-commit install                         # One-time setup
pre-commit run --all-files
pre-commit run --files path/to/file.py     # Before committing changes
```

Hook configuration (`.pre-commit-config.yaml`):
- **Black**: `--line-length=120 --skip-string-normalization`
- **isort**: `--profile black`
- **flake8**: `--max-line-length=120`
- **docformatter**: Google-style, `--wrap-summaries=72`

**CRITICAL: Always verify pre-commit hooks pass after making changes.**
Common failures: unused imports (flake8), line length violations, missing/malformed docstrings.

### Documentation

```bash
uv run mkdocs serve    # Local preview
uv run mkdocs build    # Build (always run before committing API changes)
```

When renaming modules or changing public APIs, update `docs/api-reference/` to match actual module paths and run `mkdocs build` to verify.

**Standalone guide tabs** (top-level nav, each a self-contained reference):
- `docs/DATA_SOURCES.md` — data loaders, capability matrix, usage per source
- `docs/data-processing.md` — `DataProcessor` and `DataPipeline` with all 7 pipeline steps
- `docs/environments.md` — action/observation/reward spaces, all reward strategies, full env example
- `docs/experiments.md` — `BacktestRunner`, `ExperimentJob`, `JobGenerator`, `AgentExplainer`, `OptunaRunner`

### Notebooks

```bash
# Install Jupyter kernel (one-time)
python -m ipykernel install --user --name quantrl-lab --display-name "QuantRL-Lab"

# Start Jupyter
jupyter notebook
# Then select "QuantRL-Lab" kernel
```

**Key notebooks:**
- `notebooks/backtesting_example.ipynb` - Main workflow demo
- `notebooks/feature_selection.ipynb` - Feature engineering
- `notebooks/hyperparameter_tuning.ipynb` - Optuna tuning

### Development Gotchas

- **Never commit `.env`** - contains API keys (Alpaca, Alpha Vantage, etc.)
- **Use `.env.example` as template** for required environment variables
- **Python 3.10+ required** (see pyproject.toml)
- **Module imports**: `quantrl_lab.*` (package installed in editable mode)

## Architecture

### Core Design Pattern: Strategy Injection

The environment accepts 3 pluggable strategy objects at instantiation — the central pattern of the entire codebase:

```python
env = SingleStockTradingEnv(
    data=df,
    config=config,
    action_strategy=action_strategy,          # How actions are processed
    reward_strategy=reward_strategy,          # How rewards are calculated
    observation_strategy=observation_strategy  # What the agent observes
)
```

This decouples environment logic from algorithmic choices. Change reward functions without touching environment code. Swap observation features without rewriting state logic.

**Step execution order** (inside `env.step(action)`):
1. Store `prev_portfolio_value`
2. `portfolio.process_open_orders()` — execute pending limit/stop orders
3. `action_strategy.handle_action(env, action)` — decode & execute new order
4. Advance `current_step`, check `terminated`/`truncated`
5. `reward_strategy.calculate_reward(env)` — compute reward
6. Clip reward to `reward_clip_range`
7. `reward_strategy.on_step_end(env)` — stateful hook
8. `observation_strategy.build_observation(env)` — compute state

### Data Flow

```
DataLoader (Alpaca/YFinance/AlphaVantage/FMP)
  → get_historical_ohlcv_data() → DataFrame with OHLCV
  → DataProcessor.data_processing_pipeline() → DataFrame with indicators
  → SingleStockTradingEnv → step() delegates to strategies
  → RL Agent (PPO/SAC/A2C via stable-baselines3)
```

### Key Architectural Patterns

**1. Protocol-Based Data Sources** (`src/quantrl_lab/data/interface.py`)
Data sources implement capability protocols for runtime feature detection:
- `HistoricalDataCapable`, `LiveDataCapable`, `NewsDataCapable`, `StreamingCapable`, `FundamentalDataCapable`, `MacroDataCapable`, `AnalystDataCapable`
- Check capabilities: `loader.supported_features()` → `{'historical': True, 'live': False, ...}`

**2. Indicator Registry** (`src/quantrl_lab/data/indicators/registry.py`)
Technical indicators are auto-registered via decorator:
```python
@IndicatorRegistry.register('RSI')
def rsi(df: pd.DataFrame, window=14) -> pd.DataFrame: ...


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [whanyu1212/QuantRL-Lab](https://github.com/whanyu1212/QuantRL-Lab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
