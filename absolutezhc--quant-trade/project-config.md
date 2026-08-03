---
trigger: always_on
description: Hybrid architecture crypto quantitative trading system for personal use + research.
---

# Crypto Quant Trading System

Hybrid architecture crypto quantitative trading system for personal use + research.
Medium-low frequency, multi-coin (BTC + altcoins), spot + perpetuals.

**项目状态 & 待办：** `docs/superpowers/PROJECT_STATUS.md` — 每次 session 开始前读这个文件了解当前进度和下一步任务。

## Architecture (4-layer)

```
Layer 1: Data Hub → Layer 2: Factor Studio → Layer 2.5: Risk Gate → Layer 3: Trading Engine (freqtrade)
```

- **Data Hub:** Multi-source ingestion (CEX via ccxt, on-chain, sentiment, valuation). Thin adapters → unified schema.
- **Factor Studio:** Factor library (YAML), auto mining (gplearn/LightGBM), evaluation (IC/IR), signal generation. Human-in-the-loop — system suggests, human decides.
- **Risk Gate:** Independent process. Layer 0: structural safety (hardcoded, 2 rules). Layer 1: all configurable (YAML). Layer 2: anomaly detection (independently toggleable). Fail-closed on crash.
- **Trading Engine:** Freqtrade for backtesting + live trading. File-based signal bus (MVP — JSONL + .ready flag).

**Design doc:** `docs/superpowers/specs/2026-05-12-crypto-quant-trading-system-design.md`

## Key Principles

1. **Adapter pattern:** New data source = thin adapter (~50-400 LOC). Zero downstream changes.
2. **Signal bus decoupling:** Factor Studio → JSONL signal file → Risk Gate → freqtrade. Clean interface.
3. **Human-in-the-loop:** System suggests, human decides. No autonomous parameter changes.
4. **MVP-first:** Parquet+DuckDB only. File polling only. Date-partitioned files. Add complexity when proven necessary.
5. **All parameters configurable:** Only 2 structural safety rules are hardcoded. Everything else is YAML.
6. **Interface contracts:** All schemas in `contracts.py`. Tests as executable documentation.

## Directory Structure

```
quant/
├── data_hub/            # Layer 1 - collectors, pipeline (cleaner + validator + store), features
├── factor_studio/        # Layer 2 - library (YAML), mining, evaluation, signal
├── risk_gate/            # Layer 2.5 - independent process, fail-closed
├── trading/              # Layer 3 - freqtrade strategies, signal receiver (file poller)
├── adapters/             # Thin adapters for new data sources
├── contracts.py          # Global interface definitions
├── config/               # Per-mode + per-strategy YAML configs
├── data/                 # Date-partitioned Parquet files
├── tests/                # One test file per module
└── notebooks/            # Jupyter exploration
```

## Development Modes

| Mode | Storage | Command |
|------|---------|---------|
| Dev | Parquet + DuckDB | `make dev` |
| Research | Parquet + DuckDB | `make research` |
| Live | Parquet + DuckDB + file signal bus | `make live` |

## Signal Format (Interface Contract)

```json
{"signal_id":"uuid","timestamp":"...","symbol":"TOKEN/USDT","signal":"BUY/SELL",
 "strength":0.82,"position":1.0,"stop_loss":null,"take_profit":null,
 "strategy":"...","factors":{...},"metadata":{"timeframe":"15m","expires_at":"..."}}
```

- signal_id: required, UUID for full audit trail
- stop_loss/take_profit: optional, strategy-custom prices. Absent → Risk Gate default formula
- entry_price: intentionally NOT included. Execution price determined at trade time.

## Data Collection Priority (by real-trading ROI)

| Priority | Data | Source |
|----------|------|--------|
| P0 | OHLCV, funding rate, OI, exchange netflow | ccxt |
| P1 | Stablecoin mint/burn, TVL, listing announcements | DeFiLlama, CryptoPanic |
| P2 | Twitter mentions, partnership events, vesting unlock | Twitter API, public tokenomics |
| P3 | Whale tracking, multisig, Telegram/Discord, NLP | Various (low ROI) |

## Data Cleaning Rules

- Dedup: keep last for duplicate timestamps
- Missing: forward fill, max 5 consecutive candles
- Outlier: FLAG ONLY, NEVER DROP. Crypto extreme values are real signal.
- Validation: price>0, volume>=0, timestamp monotonic, OHLC consistent

## Session Context

- Each module built and tested independently. One module per session.
- Every module has a test file.
- After completing each module, update "Completed Modules" below.
- New session: Claude reads this file → knows architecture, progress, next step.
- Interface contracts in `contracts.py` define all data shapes.

## Completed Modules
**Phase 1 COMPLETE** — all 11 tasks done, 30 tests passing, E2E pipeline verified.

- Task 1: `quant/contracts.py` — all interface definitions
- Task 2: `quant/data_hub/collectors/market.py` — MarketCollector (ccxt OHLCV + derivatives)
- Task 3: `quant/data_hub/pipeline/cleaner.py` + `validator.py` — dedup, gap fill, outlier flag
- Task 4: `quant/data_hub/pipeline/store.py` — DuckDBStore (Parquet date-partitioned)
- Task 5: `quant/data_hub/features/preprocessing.py` — SMA, returns, volatility, price position
- Task 6: `quant/factor_studio/library/loader.py` + `pump_coin_factors.yaml` — 5 initial factors
- Task 7: `quant/factor_studio/signal/generator.py` + `evaluation/evaluator.py` — IC-weighted signal gen
- Task 8: `quant/risk_gate/gate.py` — Layer 2.5, fail-closed, YAML-configurable
- Task 9: `quant/trading/strategies/signal_receiver.py` — freqtrade strategy + JSONL signal bus
- Task 10: `quant/adapters/alpha_klines.py` — BN Alpha thin adapter

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AbsoluteZhc/quant-trade](https://github.com/AbsoluteZhc/quant-trade) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
