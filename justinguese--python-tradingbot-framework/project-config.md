---
trigger: always_on
description: Trading Bot Framework — Research Briefing
---

# Trading Bot System - LLM Guide

Trading Bot Framework — Research Briefing  
This is a Python-based automated trading framework running on Kubernetes. Bots run as CronJobs, fetch market data, make decisions, and execute
paper/live trades stored in PostgreSQL.

---

Bot Interface — Two Patterns

Pattern A: decisionFunction(row) -> int (simple, backtestable)

- Override this method; the base class handles data fetching, looping, and execution
- Called once per OHLCV row. Return 1 (buy), -1 (sell), 0 (hold)
- Supports local_backtest(), local_optimize(), hyperparameter tuning

Pattern A1 — Single-ticker (symbol=):

- The framework buys/sells self.symbol automatically

class MyBot(Bot):
def **init**(self):
super().**init**("MyBot", symbol="QQQ", interval="1d", period="1y")

      def decisionFunction(self, row) -> int:
          if row["momentum_rsi"] < 30:
              return 1
          elif row["momentum_rsi"] > 70:
              return -1
          return 0

Pattern A2 — Multi-ticker (tickers=[...]):

- Pass tickers= instead of symbol=; decisionFunction is called per ticker per bar
- Position sizing: equal-weight — each ticker targets total_portfolio_value / N
- Fully backtestable via local_backtest() and local_optimize()

class MyMultiBot(Bot):
def **init**(self):
super().**init**("MyMultiBot", tickers=["SPY", "QQQ", "GLD"],
interval="1d", period="1y")

      def decisionFunction(self, row) -> int:
          if row["momentum_rsi"] < 30:
              return 1   # buy this ticker toward equal-weight target
          elif row["momentum_rsi"] > 70:
              return -1  # sell all holdings of this ticker
          return 0

Pattern B: makeOneIteration() -> int (complex, not backtestable)

- Override this method directly for multi-asset bots or external data sources
- Manually call self.buy(symbol), self.sell(symbol), self.rebalancePortfolio(weights)
- Use when: portfolio rebalancing across N symbols, signals come from a DB table, external API (Fear & Greed), AI agent flows

---

Data Available

1. Yahoo Finance OHLCV — via self.getYFDataWithTA(interval, period)

- Returns a DataFrame with timestamp, open, high, low, close, volume + ~150 TA indicators
- Intervals: 1m, 5m, 15m, 30m, 1h, 4h, 1d, 1wk, 1mo
- Periods: 1d, 5d, 7d, 1mo, 3mo, 6mo, 1y, 2y, max (minute data capped at 60 days by Yahoo)
- Multi-symbol: self.getYFDataMultiple(symbols, interval, period) — returns long-format DataFrame

2. Technical Indicators — via ta library (add_all_ta_features)
   All ~150 indicators are pre-computed and available as columns. Key ones:

- Momentum: momentum_rsi, momentum_stoch, momentum_stoch_signal, momentum_macd, momentum_macd_signal, momentum_cci, momentum_williams_r
- Trend: trend*macd, trend_macd_signal, trend_macd_diff, trend_sma_fast, trend_sma_slow, trend_ema_fast, trend_ema_slow, trend_adx,
  trend_adx_pos, trend_adx_neg, trend_ichimoku*\*, trend_aroon_up/down
- Volatility: volatility_bbm, volatility_bbh, volatility_bbl, volatility_bbw, volatility_atr, volatility_kcp, volatility_dcp
- Volume: volume_obv, volume_adi, volume_cmf, volume_fi, volume_mfi, volume_em, volume_vpt

3. PostgreSQL Tables

┌──────────────────────┬───────────────────────────────────────────────────┬────────────────────────┐
│ Table │ Contents │ Used by │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ bots │ Portfolio state JSON per bot │ All bots │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ trades │ Trade history (symbol, price, qty, isBuy) │ All bots │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ run_logs │ Execution history, success/error │ All bots │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ portfolio_worth │ Daily portfolio value snapshots │ Dashboard │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ historic_data │ Cached OHLCV (avoids re-fetching) │ All bots │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ stock_news │ Recent news headlines per symbol from yfinance │ StockNewsSentimentBot │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ stock_earnings │ Earnings dates, EPS estimate vs actual, surprise% │ EarningsInsiderTiltBot │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ stock_insider_trades │ Insider buy/sell transactions │ EarningsInsiderTiltBot │
├──────────────────────┼───────────────────────────────────────────────────┼────────────────────────┤
│ telegram_messages │ Telegram channel messages + AI summaries + symbol │ TelegramSignalsBankBot │
└──────────────────────┴───────────────────────────────────────────────────┴────────────────────────┘

4. AI — via OpenRouter

- Cheap model (default openrouter/free): self.run_ai_simple(system, user) — classification, extraction, single-turn
- Main model (default deepseek/deepseek-v3.2): self.run_ai(system, user) — multi-turn with tools (portfolio lookup, market data, trade  
  history, news lookup)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JustinGuese/python_tradingbot_framework](https://github.com/JustinGuese/python_tradingbot_framework) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
