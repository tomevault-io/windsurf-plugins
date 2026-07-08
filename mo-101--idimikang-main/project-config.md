---
trigger: always_on
description: Project notes for AI agents working in this repo.
---

# AGENTS.md — Idim Ikang

Project notes for AI agents working in this repo.

## Environment

- The freqtrade engine lives in `strategies/` (it IS the freqtrade source, imported as `freqtrade.*`).
- Runtime Python is **Windows Python 3.14** (`C:\Python314\python.exe`), user site-packages at `C:/Users/idona/AppData/Roaming/Python/Python314/site-packages`.
- pandas / numpy / scipy / scikit-learn are installed there.
- The full `freqtrade` stack cannot be imported on Windows Python 3.14 because the installed `ccxt` uses a metaclass incompatible with 3.14 (`Metaclasses with custom tp_new are not supported`). Syntax-check engine files with `python -c "import ast; ast.parse(open(f).read())"` rather than importing them on this host.
- **pwb-toolbox was removed**: the `Stocks-Quarterly-IncomeStatement` equities-fundamentals dataset is a category error for a crypto perpetual engine. Do not re-add it.

## Test / verify commands

```bash
python -m pytest tests/ -q                              # all tests (36)
python -m pytest tests/test_afml_toolkit.py -q          # AFML toolkit + trial registry
# AFML calibration against the 199 EMITTED sealed signals (must PASS pre-deploy):
python quant_core/afml_calibration.py
# syntax check a freqtrade engine file without importing it:
python -c "import ast; ast.parse(open('strategies/optimize/backtesting.py', encoding='utf-8').read())"
```

## Pre-deploy gates (must pass before AFML touches the live decision path)

1. **Math calibration**: `python quant_core/afml_calibration.py` must print `CALIBRATION: PASS`. This replays the 199 EMITTED signals through the corrector and checks (a) the OFF path is inert — sealed numbers unchanged when flags are disabled, and (b) the ON path's deflated Sharpe + PBO match independent hand-calculation. Last run: PASS.
2. **Trial count N is set**: the DSR correction requires the true cumulative N across the whole campaign. Before deploying, set it via one of:
   - `TrialRegistry.set_n(312)` in code (the known total across all passes),
   - `config["afml_n_trials"] = 312`,
   - or `IDIM_AFML_N_TRIALS=312` env var.
   If N is unset, the DSR refuses to deflate (raises `TrialCountUnknown` in strict mode, or warns + returns 1 in non-strict reporting mode). **Never** let N silently default to 1 — that is zero deflation, the exact failure mode this layer prevents.
3. **Engine-integration calibration (gate #3)**: `python quant_core/afml_gate3_engine_integration.py --config <config.json> --timerange <range>` must print `GATE #3: PASS`. This runs a real backtest with AFML flags off (confirms `results["afml"]` absent — inert wiring against the real engine, not the ledger stand-in) and on (confirms `afml` diagnostics surface without error — proves `_afml_postprocess` reads freqtrade's backtest output structure correctly). **Requires WSL where freqtrade/ccxt imports.** A `--smoke-test` mode is available that just confirms flags default OFF without a full backtest. Until this gate passes, the AFML layer must not touch live candidate decisions — the math calibration (gate #1) does not cover engine integration.

### Defect found and fixed during audit

`backtest_is_oos_split` originally defaulted to **0.3** (not 0.0), which meant the IS/OOS diagnostic ran on **every** backtest by default, computing a DSR with `n_trials=1` (registry empty, `strict=False` → 1). This is exactly the "side effect even when disabled" + "deflation theater" failure modes combined — every backtest got a meaningless DSR that looked rigorous. Fixed: default changed to 0.0 (disabled). The calibration script's OFF-path check masked this because it explicitly set `backtest_is_oos_split: 0.0`; the real engine default was different. This is why gate #3 (real engine, not the calibration stand-in) is load-bearing.

## AFML toolkit (`quant_core/afml/`)

Implements the backtesting-bias remedies from Lopez de Prado's *Advances in Financial Machine Learning*:

- `purged_k_fold.py` — `PurgedKFold`, `purged_train_test_split` (purge + embargo).
- `cpcv.py` — `CPCV`, `combinatorial_purged_cross_validation` (multiple backtest paths).
- `deflated_sharpe.py` — `deflated_sharpe_ratio`, `expected_max_sharpe` (selection-bias correction).
- `pbo.py` — `probability_of_backtest_overfitting` (PBO from CPCV paths).
- `walk_forward.py` — `WalkForwardSplit`, `walk_forward_windows` (walk-forward + embargo).
- `transaction_costs.py` — `TransactionCostModel`, `apply_transaction_costs` (spread + commission + slippage + impact).
- `diagnostics.py` — `backtest_report`, `is_oos_gap`, `sharpe_decay` (IS/OOS gap + verdict).
- `trial_registry.py` — `TrialRegistry`, `resolve_n_trials` (cumulative campaign-wide N for DSR; refuses to silently default).
- `quant_core/afml_guards.py` — `audit_lookahead`, `selection_bias_guard` (look-ahead + selection-bias guards for the signal engine).
- `quant_core/afml_calibration.py` — calibration harness mirroring `forward_adjudicator.backfill_calibration`.

All AFML imports in engine code are wrapped in `try/except ImportError` so a missing `quant_core` never breaks an engine.

## DSR trial count (N) — the quiet-bias-theater risk


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mo-101/IdimIkang-main](https://github.com/Mo-101/IdimIkang-main) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
