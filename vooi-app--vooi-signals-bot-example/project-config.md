---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

The bot uses Typer; every command is `python -m bot <cmd>` (or `bot <cmd>` after `pip install -e .`).

| Need | Command |
|---|---|
| Start all 7 async tasks (foreground) | `python -m bot run` |
| Connectivity / DB / VOOI health | `python -m bot status` |
| Interactive Telethon login (one-time) | `python -m bot tg-login` |
| Add a channel | `python -m bot channels add @username` |
| Re-parse a stored message without trading | `python -m bot signal-dryrun <signal_id>` |
| Force a breakeven check on one position | `python -m bot simulate-breakeven --position-id <N>` |
| Export VOOI errors for triage | `python -m bot vooi-errors export out.json` |
| Apply DB migrations | `alembic upgrade head` (also runs automatically on `bot run`) |
| Unit tests (no external deps) | `pytest tests/unit/` |
| One test file or one test | `pytest tests/unit/test_tp_calculator.py::test_floor_applied` |
| Integration tests (needs local Postgres) | `pytest tests/integration/` |

`pyproject.toml` sets `asyncio_mode = "auto"`, so async tests do **not** need `@pytest.mark.asyncio`.

### One-off operator scripts

These exist under `scripts/` and are not part of `bot run`. Run with the bot stopped:

- `python -m scripts.reset_tp [--apply]` — recompute TP on every open position and replace the live trigger. Dry-run by default.
- `python -m scripts.reset_sl [--apply]` — same for SL (use after changing `DEFAULT_SL_PCT` or flipping `USE_SIGNAL_SL`).
- `python -m scripts.mae_winners` — pull 1m Binance klines for every `closed_tp` position and report Maximum Adverse Excursion. Useful when tuning SL distance from empirical data.

## Architecture

A single Python process runs **seven concurrent asyncio tasks** plus a one-shot `startup_cleanup`. They share a Postgres DB and an `httpx`-based VOOI client.

```
ingester → parser → router → place_entry  ──(SSE order frame, status=filled)──>  post_fill_placer
                                                                                 │
                                                                                 ▼
                                                                       places SL + TP triggers
                                                                                 │
                                       sse_listener.on_market_price_frame
                                            └─▶ evaluate_breakeven_trigger  (in-memory index)
                                                     └─▶ asyncio.create_task → move_sl_to_breakeven
                                       breakeven_supervisor (10s) — heartbeat + reconcile + stale-SSE rescue
                                       sl_safety_check      (30s)
                                       tp_safety_watchdog   (30s)   re-places missing TPs
                                       lighter_sl_watchdog  (30s)   re-places dropped SLs
                                       reconciler           (60s)   DB ↔ exchange diff
```

Tasks are registered in `bot/cli.py:_run_all_tasks`. Breakeven detection is **SSE-driven, not polled**: every `marketPrice` frame triggers `evaluate_breakeven_trigger`, which looks up the in-memory `trigger_index` (keyed by `(exchange, symbol)`) and schedules a BE move via `asyncio.create_task` if the threshold is crossed. Per-trigger `asyncio.Lock` + `fired` flag guarantee idempotency under torrents of ticks. The supervisor's only jobs are (a) update `tp_breakeven_watcher_last_tick` so `sl_safety_check` doesn't flag `ERROR_WATCHER_HUNG`, (b) reconcile `trigger_index` against the DB every 10s (self-heals missed register/unregister calls), and (c) batch-quote REST for positions whose SSE feed has gone stale — the only place breakeven logic ever hits REST.

### The signal funnel — what filters out what

Reading `signals.is_signal` and `signals.skip_reason` is the fastest way to debug "why didn't my signal trade":

```
message  →  parsed signal (is_signal=true)  →  router gates  →  entry order  →  fill  →  post_fill (SL+TP)
              else: stored as noise           else: skip_reason   else: rejected  else: open_pending_tp_sl
```

Common `skip_reason`s: `symbol_not_found`, `rate_limit_global`, `rate_limit_per_channel`, `already_in_position`, `symbol_active_on_all_available_exchanges`, `duplicate_signal`, `no_entry_price`, `leverage_set_failed`. Market-orders are **forbidden** in v1 — empty `entry_prices` → `no_entry_price`.

### TP and SL math — read this before touching `tp_calculator.py`

**All `*_PCT` settings in the trading section are denominated as % of MARGIN (collateral), not price.** The bot converts to price by dividing by leverage. This applies to:

- `DEFAULT_SL_PCT`           (e.g. 5 → at lev=5, SL is 1% from entry → loses 5% of margin if hit)
- `MIN_PROFIT_PCT_OF_COLLATERAL` (required net profit)
- `TP_OVERHEAD_FLOOR_PCT`    (floor on cost-overhead allowance)
- `BREAKEVEN_TRIGGER_PCT`    (when to move SL to breakeven)

So the TP formula in `compute_tp_price` is:

```
required_gain (price) = MIN_PROFIT_PCT_OF_COLLATERAL / 100 / L
floor         (price) = TP_OVERHEAD_FLOOR_PCT       / 100 / L

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vooi-app/vooi-signals-bot-example](https://github.com/vooi-app/vooi-signals-bot-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
