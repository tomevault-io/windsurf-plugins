---
trigger: always_on
description: Polars-native quantitative research platform. This memo is for AI assistants working on the codebase.
---

# Elvers -- AI Development Memo

Polars-native quantitative research platform. This memo is for AI assistants working on the codebase.

For contribution workflow, numerical invariants, and coding standards, see [CONTRIBUTING.md](CONTRIBUTING.md).
For operator specifications, see [OPERATORS.md](OPERATORS.md).

---

## Conduct

**When uncertain, ask. Do not guess.** Flag assumptions explicitly: "I am assuming X. Is that correct?"

- **Design before implementing.** Think through module placement, API surface, and long-term implications before writing code.
- **Write cold, factual prose.** No emotional language. No marketing. State facts and trade-offs.
- **Describe upstream behavior, do not complain.** Say "Polars applies ddof asymmetrically; Elvers isolates this by using ddof=1" -- not "Polars has a bug".
- **`__init__.py` is for imports/exports only.** New functionality goes in dedicated modules.
- **Each module has one responsibility.** Do not let a module exceed its scope. If unsure where code belongs, ask.

---

## Architecture

```
elvers/
  __init__.py                Imports/exports only. __version__ is the single version source.
  _meta.py                   Diagnostics (show_versions)

  core/
    factor.py                Factor (column name + Panel ref, zero data storage)
    panel.py                 Panel (single DataFrame, _add_col + memoization)

  ops/                       Step 4: Factor computation (72 operators)
    base.py                  Arithmetic
    timeseries.py            Time-series (per-symbol rolling window)
    cross_sectional.py       Cross-sectional (across symbols per timestamp)
    math.py                  Math (element-wise)
    neutralization.py        Neutralization and group operators
    _dev.py                  Experimental (not exported)
    _validation.py           Input validation helpers

  data/                      Step 2+3: Data acquisition and storage
    providers/               Exchange adapters (binance, okx, local)
    store.py                 Parquet on disk (incremental update)
    loader.py                DataFrame -> Panel (schema validation + balance)
    sample/                  Built-in sample data

  universe/                  Step 1: Instrument selection and filtering
  analysis/                  Step 5: IC, decay, turnover, coverage, correlation
  synthesis/                 Step 6: Orthogonalization, combination, selection
  portfolio/                 Step 7: Optimization, constraints
  backtest/                  Step 8: Unified signal -> PnL engine
  risk/                      Step 9: Exposure, limits, VaR
  execution/                 Step 10+11: Trading + post-trade analysis
  monitor/                   Step 12: Dashboard, alerts, logging

tests/
  conftest.py                Fixtures: make_ts, make_factor, make_panel_ts, make_panel_cs
  test_*.py                  One file per module
```

## Core Patterns

- Time-series operators: `expr.over("symbol")`
- Cross-sectional operators: `expr.over("timestamp")`
- All operators: `Factor -> Factor`, stateless, functional
- Binary/multi-factor operators: all factors must share the same Panel (`_check_panel`)
- Factor.name = column name in Panel._df = human-readable expression
- `_add_col` skips computation if column already exists (memoization)

## Data Flow

```
data/providers -> data/store -> data/loader -> Panel
                                                 |
Panel["close"] -> Factor -> ops -> computed Factor
                                                 |
            analysis (IC, decay, turnover) -> report
                                                 |
      synthesis (orthogonalize -> combine) -> alpha Factor
                                                 |
               backtest (signal -> PnL) -> metrics
                                                 |
            execution (rebalance -> orders) -> trades
```

Backtest accepts any `signal()` output: single factor, multi-factor composite, or portfolio-optimized weights. Same interface, same output format.

---

## Operator Template

```python
def ts_example(f: Factor, window: int) -> Factor:
    name = f"ts_example({f.name},{window})"
    expr = pl.col(f._col).rolling_mean(window, min_samples=window).over("symbol")
    f.panel._add_col(expr, name)
    return Factor(name, f.panel)
```

After implementing:
1. Add to `ops/__init__.py` imports and `__all__`
2. Add tests in matching `test_*.py`
3. Add entry to `OPERATORS.md`

---

## Known Limitations

- `trade_when`: sentinel value (-1.79e308) should become struct-based
- `_validation.py`: not wired into all operator entry points
- `_dev.py`: Python callbacks, not production-grade
- No property-based testing or performance benchmarks
- Column name collisions possible if same operator is called with identical name but different semantics
- Steps 1-3, 5-12 are scaffolded but not yet implemented

---

## Next Steps (0.4.0 -> 0.5.0)

Current status: Step 4 (Factor Computation) is complete with 72 operators. Column-based architecture, memoization, pyright CI all in place. Tests need rewrite for new architecture.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [quantbai/elvers](https://github.com/quantbai/elvers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
