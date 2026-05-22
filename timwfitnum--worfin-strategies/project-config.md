---
trigger: always_on
description: **The project now operates on TWO parallel tracks: Metals (primary, paper) and FX (secondary, paper → live).**
---

# CLAUDE.md — WorFIn Systematic Trading System
# Master Project Brief | Updated: April 2026
# Tim — CIO/CTO | Systematic Commodity + FX Trading

---

## 🔴 READ THIS FIRST — APRIL 2026 PIVOT

**The project now operates on TWO parallel tracks: Metals (primary, paper) and FX (secondary, paper → live).**

The metals thesis is unchanged. £5k live capital is below the metals futures threshold, so metals continues paper-trading per roadmap while FX is added as the capital-efficient live track on IBKR IdealPro. Read `WorFIn_Pivot_Memo.md` in project knowledge before starting any substantive work.

Do NOT relitigate this decision. Do NOT treat FX as a replacement for metals. If a user request is ambiguous about which book they mean, ASK.

---

## 🎯 PROJECT IDENTITY

We are building a **production-grade systematic trading system** across two asset classes:

- **Metals** (primary): Six statistically-validated strategies on LME base metals and COMEX precious metals. Daily holding periods. Fully automated signal generation. Interactive Brokers execution. Target live capital ≥ £50k.
- **FX** (secondary): Three strategies on G10 FX pairs via IBKR IdealPro. Capital-efficient at £5–50k. Permanent low-correlation diversifier thereafter.

This is a real trading business — every line of code has financial consequences.

**5-Year Roadmap:**
- **Now (Tier 0 → 1):** Backtesting infrastructure + paper trading (both books)
- **Near-term:** Live FX at £5k when paper cleared. Metals stays paper until capital ≥ £50k.
- **Year 2:** Live metals capital £50–100k personal (target H2 2027)
- **Year 4:** Fund launch £2–5m external AUM
- **Year 5:** Cape Town, £10–20m AUM, 3–4 person firm

---

## 🧠 CORE PHILOSOPHY — READ THIS FIRST

1. **Risk management IS the strategy.** Never treat risk as a constraint bolted onto alpha — it IS the central organising principle.
2. **A backtest is a hypothesis, not a result.** Only out-of-sample and live paper trading confirm edge.
3. **Overfitting is the #1 threat.** More parameters = more danger. Simpler is better.
4. **Infrastructure before capital.** Nothing goes live until paper-tested for 60+ trading days (G3 gate).
5. **One strategy done properly beats five done poorly.** S4 (Basis-Momentum) is the metals core. Build it first.
6. **Never size by conviction. Size by volatility.** Always inverse-vol targeting — applies identically to metals and FX.
7. **The kill switch must always work.** Every component must be stoppable in <60 seconds.
8. **FX is a parallel track, not a replacement.** Do not let FX work delay metals validation.

---

## 📐 CURRENT PHASE & ACTIVE WORK

**Phase:** Tier 0 → Tier 1 transition
**Active tracks:**
- Metals: complete S4 Basis-Momentum IS backtest (G0 gate)
- FX: DESIGN ONLY until metals S4 clears G0; do not start FX build yet

**Sequencing (strict order):**

1. Step 0 verification pass on files Tim added (`continuous.py`, `002_pnl_accounting.py`, `003_fx_rates.py`, `pretrade_intergation.py`)
2. S4 Basis-Momentum IS backtest (2005–2017) — must clear G0 (IS Sharpe ≥ 0.50, t-stat ≥ 3.0, max DD ≤ 20%)
3. **Asset-class-agnostic refactor** (`InstrumentSpec` hierarchy, `CostModel` polymorphism, strategy parameterisation)
4. FX data layer (FRED short rates, FX historicals, `FXSpec` instances)
5. FX backtests (FX1, FX2, FX3) — apply G0, G1 gates
6. FX paper trading (alongside metals paper, 60+ trading days)
7. Live FX at £5k when G3 cleared
8. Metals paper continues until capital ≥ £50k and metals G3 cleared, then flip to live

**When I ask you to build something, always check:**
- Which book is it for — metals, FX, or asset-agnostic?
- Does this serve the current phase?
- Is it the simplest implementation that works?
- Does it respect all risk limits defined below?
- Is it testable and auditable?
- Am I about to work on FX before metals S4 has cleared G0? (If yes: STOP.)

---

## 🏗️ TECH STACK

| Component | Choice | Notes |
|-----------|--------|-------|
| Language | Python 3.11+ | Primary for all components |
| Database | PostgreSQL 18 | Run locally for dev; migrate to VPS for live |
| ORM | SQLAlchemy 2.0 | Async-capable; use for all DB interaction |
| Migrations | Alembic | All schema changes versioned — never raw ALTER TABLE |
| Broker API | ib_insync | IBKR connectivity; IB Gateway (not TWS) in production |
| Broker venues | ICEEU (LME), NYMEX/COMEX (metals), IdealPro (FX) | All via single IBKR account |
| Backtesting | vectorbt | Fast vectorised; custom event-driven layer on top |
| Data | pandas + numpy | Standard; use polars for large datasets if needed |
| Stats | scipy + statsmodels | ADF tests, cointegration, regression, GARCH |
| Vol modelling | arch | GARCH volatility estimation |
| Metrics | Custom | empyrical | Sharpe, Sortino, Calmar, drawdown |
| Scheduling | schedule (dev) → APScheduler (prod) → Airflow (Tier 2+) |
| Monitoring | Telegram bot + structured logging | JSON logs with correlation_id; alerts to Telegram |
| Testing | pytest + pytest-asyncio | All strategy logic must have unit tests |
| Environment | pyenv + venv | Never use conda |
| Secrets | python-dotenv (.env file) | Never commit .env |
| Version control | Git + GitHub (private) | Feature branches; no direct commits to main |

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [timwfitnum/worfin_strategies](https://github.com/timwfitnum/worfin_strategies) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
