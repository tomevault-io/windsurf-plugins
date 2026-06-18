---
trigger: always_on
description: CLI client for the massive.com financial data API. Outputs JSON to stdout, errors to stderr.
---

# massive-cli

CLI client for the massive.com financial data API. Outputs JSON to stdout, errors to stderr.

## Prerequisites

- `MASSIVE_API_KEY` environment variable set with your API key
- Get a key at https://massive.com/dashboard/signup

## Command Reference

### stocks

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `aggregates` | `-ticker -multiplier -timespan -from -to -adjusted -limit -sort` | `-ticker` | `-multiplier 1 -timespan day -from 30d_ago -to today -adjusted true` |
| `trades` | `-ticker -from -to -limit -sort` | `-ticker` | |
| `quotes` | `-ticker -from -to -limit -sort` | `-ticker` | |
| `snapshot` | `-ticker` | `-ticker` | |
| `snapshots` | `-tickers -include-otc` | | |
| `gainers` | `-include-otc` | | |
| `losers` | `-include-otc` | | |
| `sma` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |
| `ema` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |
| `macd` | `-ticker -from -to -timespan -series-type -short-window -long-window -signal-window -limit -sort` | `-ticker` | `-timespan day -series-type close -short-window 12 -long-window 26 -signal-window 9` |
| `rsi` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 14 -timespan day -series-type close` |

### options

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `contracts` | `-underlying -type -expiration-date -strike-price -limit -sort` | `-underlying` | |
| `contract` | `-ticker` | `-ticker` | |
| `aggregates` | `-ticker -multiplier -timespan -from -to -adjusted -limit -sort` | `-ticker` | `-multiplier 1 -timespan day -from 30d_ago -to today -adjusted true` |
| `snapshot` | `-underlying -ticker` | `-underlying -ticker` | |
| `snapshots` | `-underlying -limit` | `-underlying` | |
| `chain` | `-underlying -type -strike-price -expiration-date -limit -sort` | `-underlying` | |
| `trades` | `-ticker -from -to -limit -sort` | `-ticker` | |
| `quotes` | `-ticker -from -to -limit -sort` | `-ticker` | |

### forex

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `aggregates` | `-ticker -multiplier -timespan -from -to -adjusted -limit -sort` | `-ticker` | `-multiplier 1 -timespan day -from 30d_ago -to today -adjusted true` |
| `quotes` | `-ticker -from -to -limit -sort` | `-ticker` | |
| `snapshot` | `-ticker` | `-ticker` | |
| `snapshots` | `-tickers` | | |
| `conversion` | `-from -to` | `-from -to` | |
| `sma` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |
| `ema` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |

### crypto

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `aggregates` | `-ticker -multiplier -timespan -from -to -adjusted -limit -sort` | `-ticker` | `-multiplier 1 -timespan day -from 30d_ago -to today -adjusted true` |
| `trades` | `-ticker -from -to -limit -sort` | `-ticker` | |
| `snapshot` | `-ticker` | `-ticker` | |
| `snapshots` | `-tickers` | | |
| `sma` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |
| `ema` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |

### indices

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `aggregates` | `-ticker -multiplier -timespan -from -to -limit -sort` | `-ticker` | `-multiplier 1 -timespan day -from 30d_ago -to today` |
| `snapshots` | `-ticker -tickers -limit -sort` | | |
| `sma` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |
| `ema` | `-ticker -from -to -window -timespan -series-type -limit -sort` | `-ticker` | `-window 50 -timespan day -series-type close` |

### futures

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `contracts` | `-ticker -product-code -active -limit -sort` | | |
| `aggregates` | `-ticker -resolution -from -to -limit -sort` | `-ticker` | `-resolution 1day -from 30d_ago -to today` |
| `quotes` | `-ticker -from -to -limit -sort` | `-ticker` | |
| `trades` | `-ticker -from -to -limit -sort` | `-ticker` | |
| `snapshots` | `-ticker -tickers -product-code -limit -sort` | | |

### economy

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `treasury` | `-from -to -limit -sort` | | |
| `inflation` | `-from -to -limit -sort` | | |
| `labor` | `-from -to -limit -sort` | | |

### reference

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `tickers` | `-search -type -market -active -limit -sort -order` | | `-active true` |
| `details` | `-ticker -date` | `-ticker` | |
| `types` | | | |
| `dividends` | `-ticker -limit -sort` | `-ticker` | |
| `splits` | `-ticker -limit -sort` | `-ticker` | |
| `financials` | `-ticker -timeframe -limit -sort` | `-ticker` | |
| `filings` | `-ticker -type -limit` | `-ticker` | |

### news

| Subcommand | Flags | Required | Defaults |
|---|---|---|---|
| `market` | `-limit -sort` | | |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mtzanidakis/massive-cli](https://github.com/mtzanidakis/massive-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
