---
trigger: always_on
description: > **Status: ARCHIVED** — 66 experiments confirmed ~64% accuracy ceiling. No tradeable edge beyond the A2i production system (cron still runs). Research phase ended Feb 2026.
---

# WaveCast

> **Status: ARCHIVED** — 66 experiments confirmed ~64% accuracy ceiling. No tradeable edge beyond the A2i production system (cron still runs). Research phase ended Feb 2026.

Wavelet-shapelet financial forecasting library with SAX tokenization and transformer-based sequence prediction.

## Quick Start

```bash
cd ~/wavecast
source .venv/bin/activate
pytest tests/ -q          # 424 tests, ~9s on GPU
ruff check src/ tests/    # 0 errors
maturin develop --release # build Rust extension (optional, Python fallback available)
```

## Environment

- Python 3.12.12, venv at `.venv/`
- PyTorch 2.10.0+cu128, CUDA 13.1
- GPU: NVIDIA RTX 2060 SUPER (8GB VRAM) — auto-detected via `torch.cuda.is_available()`
- Rust 1.93.0 + maturin (build system for Rust/PyO3 extension)
- Data API: Massive.com — `MASSIVE_API_KEY` in `.env` (gitignored)
- `.env.example` has the template

## Project Layout

```
src/wavecast/
  core/         — types, config, exceptions, universe (DEFAULT=Phase3, LEGACY=Phase1-2)
  _rust.py      — Rust/PyO3 fallback wrapper (HAS_RUST flag)
  data/         — Massive.com fetcher, cache, preprocessing, storage, decomposition cache, MMapSequenceDataset
  wavelets/     — DWT decompose, CWT scalogram, reconstruction
  shapelets/    — W-TSS discovery, quality metrics, library, clustering
  dtw/          — DTW matching, ShapeDTW, similarity, subsequence search
  fractal/      — Hurst exponent, MFDFA, regime detection, self-similarity, HurstCache
  features/     — pipeline combining wavelet+shapelet+fractal+market+SAX features
  sax/          — PAA, SAX transform, Bag-of-Words + TF-IDF, price reconstruction
  tokenizer/    — SAXVocabulary, WaveletSAXTokenizer, sequence dataset builder
  models/       — WaveletLSTM, WaveletGPT (AMP), XGBoost, ensemble, registry, BatchPredictor
  evaluation/   — metrics (16 risk metrics), walk-forward backtest, token prediction eval, signal reporting
  signals/      — SignalGenerator, SignalBacktest, PositionSizer, TransactionCostModel, signal types & config
  experiments/  — ExperimentConfig, ExperimentResult, Runner, Splitter, Metrics, Storage, HPO
  forward/      — ForwardTestRunner, ForwardTestTracker, ForwardPrediction, report, config (paper trading)
  pipeline/     — stage orchestration, runner, library builder, token pipeline
  cli/          — Typer CLI: data, discover, match, analyze, forecast, library, backtest, sax, tokenize, experiment, signal, forward
  viz/          — scalogram, shapelet gallery, DTW alignment, forecast, fractal plots
rust/           — PyO3 extension crate (sax_words, bow, vocab, dataset acceleration)
tests/
  unit/         — 62 test files, 417 unit tests
  integration/  — 4 integration tests (pipeline, signal pipeline, performance, forward pipeline)
  fixtures/     — deterministic generators (seed=42)
scripts/        — experiment runners (run_C1-C7, run_D1_D4, run_F1/F4/F5/F6), fetch_phase3_universe.py, train_forward_model.py, model_audit.py
```

~115 source files, 69 test files, 424 tests passing.

## Architecture: Two Pipelines

### Pipeline 1: Wavelet-Shapelet Forecasting (Phase 1)
```
data → decompose → discover → match → fractal → features → forecast → evaluate
```
- Fetch OHLCV → DWT (db4, 5 levels) → shapelet discovery (W-TSS) → DTW matching
- Fractal analysis (Hurst, MFDFA) → ~40-50 feature vector
- Models: WaveletLSTM + XGBoost + weighted ensemble
- Evaluate: RMSE, MAE, directional accuracy, Sharpe, walk-forward backtest

### Pipeline 2: SAX Token Prediction (Phase 2) — **Primary pipeline**
```
data → decompose → SAX → tokenize → train WaveletGPT → evaluate
```
- DWT coefficients → PAA → SAX symbols → sliding window words → vocabulary
- Bag-of-Words + TF-IDF for cross-asset similarity
- WaveletGPT: causal transformer (token+position+level+sector embeddings)
- Weight tying between token embedding and output head (h=1); independent heads for h=2,4,8
- Multi-horizon prediction: horizons=[1,2,4,8], h=1-2 useful, h=4+ plateaus
- Evaluate: token accuracy, top-3 accuracy, directional accuracy (per-horizon)
- Phase 3 proved P2 dominates P1 — no ensemble benefit

### Pipeline 3: Signal Generation & Backtesting (Phase 5)
```
WaveletGPT.predict_proba() → SignalGenerator → PositionSizer → SignalBacktest → SignalBacktestResult
```
- Softmax probabilities aggregated by quartile bucket → P(up), P(down), P(flat); argmax = direction
- Temperature scaling calibration via NLL minimization on validation data
- Position sizing: fixed, linear (confidence × max), Kelly, fractional Kelly (0.5× Kelly with rolling lookback)
- Transaction costs: commission + spread (bps) + slippage (bps); direction changes double costs
- 16 risk metrics: Sharpe, Sortino, Calmar, max drawdown, profit factor, VaR, CVaR, win rate, avg win/loss ratio, expectancy, tail ratio, total/annualized return, volatility, num trades, avg trade return
- Per-trade records with entry/exit timestamps, gross/net returns, cost breakdown

### Production Trading System (A2i)
```
WaveletGPT.predict_proba() → compute_signal_b() → magnitude_filter(top_tercile) → flat_sizing → trade
```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [musicofhel/wavecast](https://github.com/musicofhel/wavecast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
