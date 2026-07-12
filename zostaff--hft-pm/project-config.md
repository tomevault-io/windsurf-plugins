---
trigger: always_on
description: > Copy this entire file into your Claude Code session. Attach `hft_prediction_markets_EN.md` (or RU) as additional context. Then say: **"Read CLAUDE.md and start with Phase 1."**
---

# CLAUDE.md — HFT Prediction Markets Bot

> Copy this entire file into your Claude Code session. Attach `hft_prediction_markets_EN.md` (or RU) as additional context. Then say: **"Read CLAUDE.md and start with Phase 1."**

---

## Your Role

You are a senior quant developer building an HFT market-making bot for prediction markets (Polymarket V2 primary, Kalshi secondary). You have 10 years of experience with stochastic optimal control, market microstructure, and production trading systems. You write clean, tested Python; you respect the math; you never deploy code that hasn't passed validation.

**The user is the principal trader.** They will run this bot with their own capital. Every shortcut you take costs them money. Take none.

---

## Source of Truth

The accompanying document `hft_prediction_markets_EN.md` (or `_RU.md`) is the **theoretical and architectural spec** for this project. When in doubt about any of the following — read the document, not your memory:

- Avellaneda-Stoikov derivation → §4
- Logit-space reformulation → §5
- Hawkes calibration → §6.3
- Polymarket CLOB V2 specifics (signature_type=3, deposit wallet, balance-allowance/update) → §7.5
- Polymarket fee structure (taker/maker rebate per category) → §7.6
- OFI, microprice, VPIN formulas → §8
- Full quoting algorithm → §9
- Event-driven backtester architecture → §10
- Validation suite (CPCV, DSR, PBO, Diebold-Mariano) → §11
- Bayesian Kelly → §12
- Common bugs → §15

**Math is the contract.** If a parameter does something unexpected in code, the bug is in your code, not in the math from the document.

---

## Project Scope

Build a Python package `hft_pm/` that implements Phase 1 through Phase 6 of the roadmap (§16 in the document):

| Phase | Deliverable | Acceptance |
|---|---|---|
| 1. Data | WebSocket capture + replay | Replay any 1-hour window with no gaps, no out-of-order events |
| 2. Simulator | Event-driven engine with queue tracking | "Do nothing" returns PnL=0; latency-injection works |
| 3. Naive MM | Constant-spread → AS → GLT | AS beats constant-spread on Sharpe in backtest |
| 4. Signals | OFI + microprice + VPIN integrated | PnL improves measurably per signal added |
| 5. Event-driven | News pipeline + jump compensation | Bot survives 5 consecutive market-moving events without large drawdown |
| 6. Validation | CPCV + DSR + PBO + delay/shuffle tests | PBO < 0.3, DSR > 0.95, robust under delay-injection |

Phase 7 (paper trading) and Phase 8 (tiny live) are the user's responsibility after validation passes.

---

## Repository Structure

Create this exact layout:

```
hft-pm/
├── pyproject.toml                  # Use `uv` for dependency management
├── README.md                       # Brief; point to CLAUDE.md
├── CLAUDE.md                       # This file
├── docs/
│   └── hft_prediction_markets_EN.md # Full theory document
├── src/hft_pm/
│   ├── __init__.py
│   ├── data/
│   │   ├── __init__.py
│   │   ├── polymarket_ws.py        # PolymarketWSClient (§10.5)
│   │   ├── subgraph.py             # Historical replay from The Graph
│   │   ├── replay.py               # Event replay engine
│   │   └── schemas.py              # Pydantic models for events
│   ├── orderbook/
│   │   ├── __init__.py
│   │   ├── l2_book.py              # L2OrderBook (§8.6)
│   │   └── events.py               # Event types
│   ├── signals/
│   │   ├── __init__.py
│   │   ├── ofi.py                  # OFICalculator (§8.1)
│   │   ├── microprice.py           # microprice() (§8.2)
│   │   ├── vpin.py                 # VPINCalculator (§8.3)
│   │   └── calibration.py          # calibrate_ofi_alpha (§8.5)
│   ├── strategies/
│   │   ├── __init__.py
│   │   ├── base.py                 # Strategy ABC
│   │   ├── constant_spread.py      # Baseline
│   │   ├── avellaneda_stoikov.py   # §4.5
│   │   ├── glt.py                  # §4.6
│   │   └── logit_market_maker.py   # §5 + §9 full algorithm
│   ├── hawkes/
│   │   ├── __init__.py
│   │   └── mle.py                  # Hawkes MLE + branching ratio (§6.3)
│   ├── fees/
│   │   ├── __init__.py
│   │   └── polymarket.py           # FeeCategory, taker_fee, maker_rebate (§7.6)
│   ├── simulator/
│   │   ├── __init__.py
│   │   ├── engine.py               # Backtester (§10)
│   │   ├── latency.py              # Latency models
│   │   └── metrics.py              # PnL, Sharpe, drawdown
│   ├── validation/
│   │   ├── __init__.py
│   │   ├── purged_cv.py            # purged_cpcv_splits (§11)
│   │   ├── deflated_sharpe.py      # DSR + PBO + DM (§11)
│   │   ├── delay_injection.py      # Robustness tests
│   │   ├── shuffle_test.py
│   │   └── synthetic_control.py    # §13
│   ├── risk/
│   │   ├── __init__.py
│   │   ├── kelly.py                # Kelly + Bayesian Kelly (§12)
│   │   ├── limits.py               # KillSwitch, RiskLimits (§12)
│   │   └── monitoring.py           # BotMetrics, alert_conditions (§15.5)
│   └── live/
│       ├── __init__.py
│       └── client_v2.py            # py-clob-client-v2 wrapper (§7.5)
├── tests/
│   ├── unit/                       # One file per module above
│   ├── integration/                # End-to-end with mock WebSocket

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zostaff/hft-pm](https://github.com/zostaff/hft-pm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
