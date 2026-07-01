---
trigger: always_on
description: > Instead of brutforce the input values of pine indicator like dummy monkey do I AM FORCING YOU to research the internet, the candles dump for searching of profitable concept applicable to current market state. If attempt failed and you don't known exactly why try another one. Only minimal adjust when you already see the result is allowed in this context and this improve it. Deep research is required for candles dump and internet sources both. Do not read the latest 100 candles instead of the fu
---

## Guide

> Instead of brutforce the input values of pine indicator like dummy monkey do I AM FORCING YOU to research the internet, the candles dump for searching of profitable concept applicable to current market state. If attempt failed and you don't known exactly why try another one. Only minimal adjust when you already see the result is allowed in this context and this improve it. Deep research is required for candles dump and internet sources both. Do not read the latest 100 candles instead of the full timeframe and fake you did the job

### How to Write a Strategy

**What NOT to do**

- Don't read all project files and bloat the context.

   Strategies are written as simple `.pine` files; the command to run them is below.

- Don't brute-force iterate.

   The worst thing you can do is start incrementally writing into an existing project file. That's not how this works — you need market analysis, not work for the sake of work.

- Don't sacrifice efficiency for universality.

   Markets change. By building a universal solution you lose the optimization that is the competitive edge actually generating profit at any given moment.

- Don't write `.pine` files with side effects.

   You don't need `var` and `na` in PineScript — compute all values on every iteration. This makes errors and unpredictable behavior more likely to surface before going to production. Keep the code easy to understand; avoid premature optimization.

- Don't use hacks in trading strategy code.

   You cannot disguise the absence of an SL by using ATR when the exit keeps shifting relative to the close price on every iteration. Trailing criteria must be finite — you cannot keep shifting the stop loss forever hoping for a bounce or a drop. Avoid HOLD in any form.

- Don't build strategies that produce one signal every few days.

   Three profitable signals is not a successful trading strategy — it's luck. To evaluate a strategy statistically you need at least one signal per day.

**What TO do**

- Every strategy is written for a single calendar month.

   Follow the naming pattern or refuse to work. The money is in optimizing for current market conditions; a backtest spanning two or more months is mathematically meaningless because the final balance will wipe out profit through commission whipsaw.

   * `./math/jan_2026.pine`, `./content/jan_2026.strategy.ts`
   * `./math/feb_2026.pine`, `./content/feb_2026.strategy.ts`
   * `./math/march_2026.pine`, `./content/march_2026.strategy.ts`
   * `./math/apr_2026.pine`, `./content/apr_2026.strategy.ts`
   * `./math/may_2026.pine`, `./content/may_2026.strategy.ts`

- Read the news background for the chosen time period.

   The focus should ALWAYS be on negative news. Searching for the Bitcoin price gives you marketing trash. Searching for analytics gives you SEO garbage. Use queries like:

   * Bitcoin negative news March 2026 price drop regulatory problems…
   * bitcoin price February 5 2024 current level forecast analytics BTC
   * bitcoin negative news February 2024 problems regulator crackdown bitcoin
   * bitcoin negative news March 2026 regulatory problems bans
   * bitcoin security hackers fraud regulation negative news problems

- Create a `--dump` to output candles.

   You need to see where the money actually is in the market. Identify the general trend: if it's bearish, protect against LONGs; if it's bullish, protect against SHORTs. There may be a short-term bounce or panic driven by geopolitical news.

- The market may be ranging (sideways).

   There are cases when no position should be opened at all — your analysis must account for this.

- TP/SL should be dynamic, but not scalping.

   The exchange charges 0.2% to enter and 0.2% to exit. You may think the strategy is profitable, but it's whipsaw. Minimum TP: 1%.

- Don't try to build an all-weather strategy.

   I need to understand where the money is in the market only within the specified time period. If the strategy stops being profitable I'll simply ask you to run the analysis again.

- Don't build HOLD strategies.

   I need to find where the money actually is in the market, not sit in a position hoping for luck. The criterion for "where the money is" must be expressed as a formula that finds effective entry points that lead to profit directly.

- Don't brut force strategies.

    Use fresh strategies with different concepts. Do not edit existing strategy one cause this will give you a loop even if you coded it. I need concept engineering

### Market Candle Dump

File `BTCUSDT_500_15m_1772236800000.jsonl` will be created at `./dump/BTCUSDT_500_15m_1772236800000.jsonl`

```
npm start -- --dump --timeframe 15m --limit 500 --when "2026-02-28T00:00:00.000Z" --jsonl
```

### Running `.pine` Files

File `impulse_trend_15m.jsonl` will be created at `./math/dump/impulse_trend_15m.jsonl`

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tripolskypetr/backtest-kit](https://github.com/tripolskypetr/backtest-kit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
