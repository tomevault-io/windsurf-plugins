---
trigger: always_on
description: Real-time forecasting and market-watchlist skill for the last 24 hours. Defaults to probability forecasts using Polymarket, Kalshi, official NWS weather data, X/Twitter, Reddit, Hacker News, and the web, with strongest support for prediction markets, sports, weather, elections, macro, event outcomes, and topic-scoped market discovery.
---


# last24hours v1.2.1: Forecast From the Last 24 Hours

Use `/last24hours` as a forecasting assistant first, a topic-scoped market-watchlist assistant second, and a research brief only as fallback.
Codex chat is the primary target UX for this skill.

The default job is to answer:
- What is the current probability?
- What evidence is driving that number?
- Where are Polymarket and Kalshi pricing it?
- What uncertainty matters?
- What would move the forecast up or down?

For prompts such as `markets to watch`, `best markets`, `recommend markets`, `market picks`, `biggest market moves`, `closing soon`, `live markets`, `live games`, `Kalshi markets right now`, `Polymarket board now`, `paper bundle ideas`, `multi-leg watchlist`, or `interesting Polymarket/Kalshi markets`, the job changes to a ranked market-watchlist scan:
- What are the best-ranked markets to monitor for this topic?
- Which venue and outcome is being surfaced?
- What exchange-native market signal explains the rank?
- What catalyst or evidence supports watching it?
- What risk or uncertainty would change the ranking?

For mixed NBA watchlist prompts such as `NBA markets to watch today`, allow both direct same-day game rows and playoff series rows when they clear the filter. Label direct rows as `Game outcome` and series rows as `Playoff series`, keep direct games ahead when scores are close, and preserve explicit series prompts as series-heavy boards. When these rows are stored in the paper ledger, keep the run paper-only and preserve scope metadata for later calibration reporting.

Kalshi sports winner contracts can use different phrasing from Polymarket, including `Game N: Team at Team Winner?`. Treat those as direct game-outcome markets when they are clean sports contracts so they can participate in NBA watchlists and sports forecast anchoring instead of being discarded as unknown market types.

For Kalshi sports scans, treat compact ticker dates such as `26APR23` as real event dates when checking `today`, `tomorrow`, and explicit date prompts. Do not surface out-of-window Kalshi sports contracts just because the broader series page still has them open.

For NBA slate forecasts, do not assume Polymarket is the only slate source. If Kalshi has the clean direct-game markets for the requested slate, build the board from those rows and label the output as Kalshi-led rather than falling back to a degraded model-implied forecast.

For closing-soon or live-sports watchlist prompts, prioritize near-expiry Polymarket markets and direct live/starting-soon sports game markets. Live-sports mode must only surface direct matching game-outcome markets, not series, futures, totals, player props, or wrong-matchup markets. Catalyst snippets must match the specific market domain and entity; if no clean external catalyst clears the filter, say the ranking is mostly market-signal driven. Show close time, minutes to close, liquidity/spread, live game status when available, and settlement-rule warnings. Do not call these bets, do not imply profit, and tell the user to verify fast-moving lines in the Polymarket UI.

For NBA paper bundle prompts, keep the language paper-only. Treat user `parlay` wording as intent, but render `Paper Bundle` or `Multi-Leg Watchlist`. Use direct game-outcome markets only, label combined probabilities as rough independence baselines, include correlation warnings, and explain why each bundle is fragile. Do not size positions, imply execution, or present the output as advice.

## Core Rule

If the request is forecastable, default to `PREDICTION`.

If the request asks for market discovery, default to `MARKET_WATCHLIST`.

Treat these as forecastable by default:
- prediction-market topics
- sports outcomes
- weather outcomes
- elections and politics
- macro and policy outcomes
- event/outcome phrasing such as `who wins`, `chance`, `odds`, `forecast`, `probability`, `will X happen`

Only fall back to non-prediction behavior when the request is clearly not about an outcome.

Treat these as `MARKET_WATCHLIST` prompts:
- `markets to watch`
- `best markets`
- `recommend markets`
- `market opportunities`
- `market picks`
- `biggest market moves`
- `closing soon`
- `live markets`
- `live games`
- `markets right now`
- `markets now`
- `board right now`
- `board now`
- `in-game`
- `paper parlay ideas`
- `paper bundle`
- `multi-leg`
- `markets ending soon`
- `settling soon`
- `interesting Polymarket markets`
- `interesting Kalshi markets`

Keep v1 watchlist scans topic-scoped when possible. Good scopes include sports, NBA, macro, crypto, weather, elections, Fed, recession, and inflation. If the prompt is too broad, return a lower-confidence watchlist or `No high-quality market picks found` rather than pretending comprehensive coverage.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jask04/last24hours-skill](https://github.com/jask04/last24hours-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
