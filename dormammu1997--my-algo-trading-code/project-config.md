---
trigger: always_on
description: > Working principles for ANY coding agent in this repo: simplicity first, surgical changes,
---

# AGENTS.md — My-Algo-Trading-Code

> Working principles for ANY coding agent in this repo: simplicity first, surgical changes,
> surface tradeoffs, and verify before claiming done. This is live-money trading code — bias
> toward caution. (Claude Code additionally loads its skills per `CLAUDE.md`; everything from
> "What this project is" down is kept identical in both files — edit them together.)

## What this project is
A NIFTY index-options, multi-strategy trading system. The flow is: **fetch** 1-minute OHLC history from
the DhanHQ API → **backtest** strategies on it → **run** a multithreaded "front test" whose approximately
27-strategy core roster and independently opt-in agents execute together — on paper by default, and live
through a real broker when explicitly enabled.
Running live since May 2026; daily per-strategy results are tracked in a Google Sheet.

## Architecture (runtime)
One process, cooperating threads:
- `CentralMarketDataFetcher` (one thread) polls DhanHQ and writes into a **lock-guarded
  `SharedMarketDataStore`** (1-min OHLC + LTPs). Setting `MARKET_DATA_SOURCE=WEBSOCKET`
  (fails closed to REST on any other value; needs the paid Dhan Data API subscription)
  swaps in `WebSocketMarketDataFetcher`: Dhan marketfeed ticks build the bars/LTPs
  (pure helpers in `Dependencies/tick_bar_builder.py`), with REST kept for warmup and a
  once-per-minute true-up against official candles.
- **Approximately 27 core strategy worker threads** read that store and decide trades: the `AtmSingleLegStrategyWorker`
  family (Renko / EMA / Heikin-Ashi / Profit-Shooter / Goldmine / Money-Machine / CPR / CPR Algo 3
  (multi-instrument: spot + ITM CE + ITM PE) / Opening-Strike + 13 ported TradingBot strategies + the
  **Regime Adaptive** router), two **hedged-puts** workers, one **Delta-0.2** hedged-spread worker,
  and one **long-strangle** worker (time-based dual-leg BUY of OTM1 CE+PE, with momentum re-entry).
  **Regime Adaptive** (ported from the MIT-licensed
  `workratananmol-hub/nifty-options-paper-trading-bot`) is one worker that switches RULE on ADX:
  opening-range breakout when trending, VWAP fade when ranging, no trade when ADX is missing. Its two
  candidate rules live in `Signal Generators/Regime Adaptive Strategy/regime_candidates.py` as library
  code with NO worker of their own — deliberately, so the router and a candidate can never take the
  same signal twice. Read that folder's `REGIME_PORTING_NOTES.md` before enabling it live: the feed
  carries no volume so its VWAP is an equal-weight proxy. It is also the first user of the shared
  **bid/ask spread gate** (`<PREFIX>_MAX_SPREAD_PCT`, default 0 = off for every other strategy):
  `_spread_gate_allows_entry` reads `top_bid_price`/`top_ask_price` off the `/optionchain`
  response and refuses an entry wider than the cap in paper AND live, while an unreadable quote
  refuses LIVE only. The source's VIX and breadth vetoes remain unimplemented — absent by choice,
  not for want of data (the source runs on Dhan too).
  An **optional, opt-in CPR Codex AI Agent** is an independent five-minute SRSI/VWAP worker. It
  freezes completed-bar context behind four frozen no-argument MCP tools; Codex judges regime,
  setup, and premise exits, while the host owns deterministic entry/risk gates and execution. It is
  disabled by default, live-disabled by default, and uses the normal global-plus-strategy double gate.
  Ordinary CPR, CPR Algo 3, Regime Adaptive, and CPR AI may coexist with independent positions and P&L.
  Another **optional, opt-in** worker is LLM-driven: the **SL Hunting AI Agent** (a Claude agent via
  `claude-agent-sdk`) — off by default (`SL_HUNTING_ENABLED`), it decides once per completed 1-min bar
  (with BankNIFTY cross-confirmation, fetched per bar like CPR Algo 3, and dynamic ~₹2500 risk-based
  sizing) and acts through the same ATM `enter_position`/`exit_position`; its deps are lazily imported
  so a missing dep just disables it. Every NIFTY entry is mechanically MIRRORED with an equal-lot
  BankNIFTY ATM leg (`SL_HUNTING_BNF_MIRROR`, default true) — NOTE: the mirror roughly DOUBLES the
  basket's rupee risk beyond `SL_HUNTING_RISK_BUDGET` (operator-accepted; the daily max-loss
  kill-switch still caps the day): the legs are TIED for hard risk
  (stop/target, max-loss, 15:15 square-off close both) but the agent evaluates each leg's
  premise INDEPENDENTLY and can cut one alone via the EXIT `exit_leg` selector (NIFTY|BNF|BOTH).
  Entry stays NIFTY-only (the mirror copies it). It stops opening NEW positions after 10:30
  (`SL_HUNTING_NO_NEW_ENTRY_HOUR`/`_MINUTE`, default 10:30) — not a square-off (exits + the 15:15 square-off
  still run; when flat past the cutoff it skips the LLM call entirely). After a target, stop, or
  premise-invalidating exit, `SL_HUNTING_POST_EXIT_COOLDOWN_MINUTES` blocks re-entry from the
  moment the WHOLE NIFTY/BankNIFTY basket is confirmed flat; a lone or partly closed leg does not
  run the timer down, exits never wait for it, and corrupt guard state rejects new LIVE entries.
  It can also **learn from its own trades** (v3): a per-trade journal
  feeds an off-loop reflection coach (`sl_hunting_coach.py`) that proposes lessons; the operator promotes

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DoRmAmMu1997/My-Algo-Trading-Code](https://github.com/DoRmAmMu1997/My-Algo-Trading-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
