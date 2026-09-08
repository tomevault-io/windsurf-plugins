---
trigger: always_on
description: This repo is a Python library (`hyperdrive/`) for algorithmic trading, including data sourcing, exchange integration, ML prediction, and file/storage utilities.
---

# AGENTS.md

## Overview

This repo is a Python library (`hyperdrive/`) for algorithmic trading, including data sourcing, exchange integration, ML prediction, and file/storage utilities.

## Style

Preserve the strengths of the direct, low-ceremony style established on the master branch:

- Keep domain flow visible. A reader should be able to follow inputs, calculations, and side effects without crossing unnecessary wrapper layers.
- Prefer focused functions and concrete local variables over abstractions that only rename a basic operation.
- Use positive truthiness checks for present and non-empty values. Arrange the successful path first when it remains clear.
- Keep provider behavior with its provider class and script orchestration in the script entry point.
- Add helpers when they provide meaningful reuse, isolate an external boundary, or make a complex rule easier to verify.
- Preserve observable behavior during refactors and keep unrelated cleanup out of focused changes.

## Design Principles

Patterns the codebase has followed since before the tooling refactor. They are independent of which
linter runs, so formatting, import order, docstring shape, and annotations are left to ruff and `ty`
through `pyproject.toml` rather than restated here.

### Path Discipline

Every filesystem path comes from `PathFinder` in `hyperdrive/Constants.py`. No other module builds a
data path inline, and a new data type adds a `get_*_path` method there before it reads or writes
anything. Path tests assert the exact literal, as in
`finder.get_ohlc_path("aapl") == "data/ohlc/polygon/AAPL.csv"`, which is what stops a path change
from silently orphaning a bucket prefix.

### Column Vocabulary

Column names are constants in `Constants.py` (`SYMBOL`, `TIME`, `OPEN`, `CLOSE`, `EX`, `DIV`,
`RATIO`), grouped under domain comment headers. Reference the constant. A raw column string in logic
is a defect, since renaming a column should be one edit.

### Adapter Boundaries

Vendor vocabulary stops at the adapter. `MarketData.standardize` takes a field mapping and rewrites a
provider payload onto repo constants, with `standardize_dividends`, `standardize_splits`, and
`standardize_ohlc` supplying the mappings. `Kraken.standardize_order` does the same for trades.

Nothing downstream of an adapter should ever see `exDate` or `cash_amount`. When adding a provider,
write the mapping first and let the rest of the code stay unaware of who supplied the data.

### One Subclass Per Vendor

`MarketData` is subclassed by `Indices`, `AlpacaData`, `Polygon`, `LaborStats`, and `Glassnode`. `CEX`
is subclassed by `AlpacaEx`, `Kraken`, and `Binance`. A subclass sets `self.provider` and overrides
only what genuinely differs. Shared behavior belongs on the base class.

### Shared Reliability Primitives

Retry and throttling are written once and reused. Do not hand-roll either at a call site.

- `MarketData.try_again` for retries, reading `C.DEFAULT_RETRIES` and `C.DEFAULT_DELAY`
- `obey_free_limit` and `log_api_call_time` for free-tier spacing
- `Polygon.paginate` for page walking

The established idiom is a private inner closure returned through the wrapper:

```python
def get_dividends(self, **kwargs):
    def _get_dividends(symbol, timeframe="max"):
        self.obey_free_limit(C.POLY_FREE_DELAY)
        ...
    return self.try_again(func=_get_dividends, **kwargs)
```

Rate limits are named constants (`POLY_FREE_DELAY`, `POLY_MAX_LIMIT`), never inline numbers.

### Composition For Testability

Collaborators are assembled in `__init__` and held as attributes. `MarketData.__init__` builds
`writer`, `reader`, `finder`, `traveller`, and `calculator`.

This is what makes the test suite possible: `SwissArmyKnife.replace_attr` walks an object graph
recursively and `use_dev` swaps in the S3 dev bucket, with production code unaware it happened. A
collaborator constructed inside a method instead of `__init__` cannot be retargeted, so avoid it.

### Injectable Credentials

Secrets are constructor parameters with an environment fallback, resolved at construction:
`Polygon(token=os.environ.get("POLYGON"), free=True)`, and `Robinhood(usr=None, ...)` resolving
`usr or os.environ["RH_USERNAME"]`. Do not reach into `os.environ` from deep inside a method.

### Failure Budgets

Batch scripts tolerate per-item failure and still fail the run when too much breaks.
`scripts/update_ohlc.py` is the reference: each symbol is wrapped in `try/except/finally`, successes
are counted in a `multiprocessing.Value`, CI-local files are cleaned up in `finally`, and the script
ends with

```python
if counter.value / total < 0.95:
    exit(1)
```

One bad symbol never aborts the run. A bad day still turns the build red. Pick an explicit floor for
any new batch job rather than letting the first exception decide.

### Thin Scripts

`scripts/` holds procedural glue and all logic lives in `hyperdrive/`. Most scripts are under 60
lines. If a script grows a function worth testing, that function belongs in a module.

### Schedule Awareness

`Flow.get_workflow_start_time` parses the cron expression out of `.github/workflows/{name}.yml` and

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alkalescent/hyperdrive](https://github.com/alkalescent/hyperdrive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
