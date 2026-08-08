---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

JoinQuant miniQMT Live Follower — a two-sided system where a JoinQuant cloud strategy sends trade signals via Redis Stream, and a Windows-side service consumes them to execute real orders through miniQMT/xtquant.

## Commands

```bash
# Run all tests (tests/ is a local-only suite, not tracked in this repo)
python -m unittest discover -v

# Compile-check runtime and tests (catches ImportErrors before runtime; tests/ is local-only)
python -m compileall miniqmt_follower bigqmt_follower tests

# Run the Windows execution service
python main.py
python main.py --config config.yaml --workers 8

# Run a single test module
python -m unittest tests.test_executor -v
```

## Architecture

### Two-side split

This project contains both sides of the system. Local strategy deployment copies are intentionally kept outside version control because they may embed environment-specific Redis credentials.

1. **JoinQuant side** (cloud strategy): the sender functions are documented in the standalone `joinquant_signal_sender.py` (placeholder Redis config, no imports from `miniqmt_follower/`). Copy that file into the JoinQuant strategy and fill in the real Redis config before deployment; private strategy deployment copies embed these functions with real credentials and stay outside version control. They serialize a signal as JSON, write it to Redis Stream via `XADD`, and return immediately (fire-and-forget). The five-argument `publish_trade_signal_to_redis(context, action, code, amount, price)` is frozen — exact buy/sell call sites stay untouched. The file also provides `publish_watchlist_to_redis(context, codes)`, `publish_daily_plan_to_redis(context, codes_to_sell, codes_to_buy)`, `publish_sell_half_to_redis(context, code, price)`, and `publish_sell_all_to_redis(context, code, price)`.

2. **Windows side** (this repo's runtime): `miniqmt_follower/` — a long-running service (`python main.py`) that consumes Redis Stream via consumer group with `XREADGROUP`, executes orders through miniQMT, and ACKs messages only after the order reaches a terminal state.

### Execution pipeline (Windows side)

```
Redis Stream → RedisStreamClient.read_forever()
    → app.py filters by redis.allowed_strategy_ids (non-empty ⇒ unknown strategies are logged + ACKed, never executed)
    → plan messages route to PlanExecutor: derived sell_all/auto_buy signals enter the same pools
    → app.py routes each signal to a per-direction ThreadPoolExecutor (separate buy/sell pools, --workers threads each, default 8)
    → 09:25–09:30 pre-open SELL signals register in OpeningSellBarrier; BUY workers wait for its release
    → OrderExecutionEngine.execute()
        → SQLiteExecutionStore.try_accept_signal()  [idempotency gate: INSERT OR IGNORE on signal_id PK]
        → resolve intent quantities via sizing.py (sell_all / sell_half / auto_buy) or use the exact amount
        → MarketDataAdapter.latest_quote()  [last price + ask1/bid1]
        → pricing.calculate_order_price()  [auction-queue / order-book / slippage pricing, plus the A-share price cage]
        → query fresh resources: BUY → available cash, SELL → available position
        → BrokerAdapter.submit_order()
        → poll BrokerAdapter.get_order_snapshot() until terminal or timeout
        → if non-terminal: request cancel → wait for confirmed terminal state → reconcile final fill
        → if quantity remains: fetch fresh quote/resources → re-submit remaining qty
        → SQLiteExecutionStore.update_signal_status()  [terminal state]
    → RedisStreamClient.ack()  [only after the worker's future completes]
```

Key design decisions in this pipeline:
- **Idempotency**: SQLite `signals` table primary key is `signal_id`. Duplicate signals return `DUPLICATE_IGNORED` with the existing filled quantity — no second order ever reaches the broker.
- **Resource capping fails closed**: if the fresh cash/position query fails, no order is submitted. BUY cash query, 100-share-lot capping, and broker submission are serialized as one compound operation; order polling remains parallel. SELL quantities cap to the latest closeable position, and zero live position ends as `SKIPPED_NO_POSITION` without an order.
- **Each attempt refreshes state**: every broker submission uses a fresh quote and fresh MiniQMT cash/position query.
- **Cancel confirmation**: a cancel request is not treated as completion. The engine waits for `FILLED`, `CANCELED`, or `REJECTED`, then reconciles fills that arrive during cancellation before re-submitting. If the cancel state remains uncertain, the process latches a trading halt and later queued signals cannot submit.
- **Adaptive polling**: `_wait_for_terminal_or_timeout` polls at ≤50ms for the first second, then falls back to `poll_interval_sec`.
- **Opening SELL barrier**: the strategy publishes SELLs at 09:27 and publishes no BUY at 09:29. MiniQMT submits those SELLs in parallel. After 09:30, the first ordinary SELL attempt receives only a 0.5-second report grace, then uses the existing cancel-confirm/reconcile/reprice flow. BUY execution waits until every ordinary pre-open SELL has completed or explicitly terminated.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Elio-Liu/JoinQuant-QMT-Trader](https://github.com/Elio-Liu/JoinQuant-QMT-Trader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
