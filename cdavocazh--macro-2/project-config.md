---
trigger: always_on
description: Macroeconomic indicators dashboard with large-cap equity financials. Fetches 88+ indicators from financial APIs (yfinance, FRED, SEC EDGAR, OpenBB/Finviz, web scrapers, MOF Japan, AAII, Hyperliquid), displays them across **4 dashboard frontends** (Streamlit, Plotly Dash, Grafana, React), caches locally for fast startup, and exports to CSV. Includes 5-year OHLCV history for commodities and futures (back to 2021), sector ETF tracking, high-frequency macro proxies, and Hyperliquid DeFi perpetual fu
---

# CLAUDE.md

## What is this project?

Macroeconomic indicators dashboard with large-cap equity financials. Fetches 88+ indicators from financial APIs (yfinance, FRED, SEC EDGAR, OpenBB/Finviz, web scrapers, MOF Japan, AAII, Hyperliquid), displays them across **4 dashboard frontends** (Streamlit, Plotly Dash, Grafana, React), caches locally for fast startup, and exports to CSV. Includes 5-year OHLCV history for commodities and futures (back to 2021), sector ETF tracking, high-frequency macro proxies, and Hyperliquid DeFi perpetual futures with 1-minute refresh and multi-interval OHLCV candlestick charts.

**Repo:** https://github.com/cdavocazh/macro_2
**Branch:** main

## Quick commands

```bash
# Run the Streamlit dashboard (loads from cache if available, otherwise fetches live)
streamlit run app.py

# Run the Plotly Dash dashboard (standalone, reads from same cache)
python dash_dashboard/app.py              # http://localhost:8050

# Run the Grafana dashboard (local mode with API bridge)
bash grafana_dashboard/start.sh local     # http://localhost:3000

# Run the React dashboard (Vite + FastAPI)
bash react_dashboard/start.sh             # http://localhost:5173

# Refresh data manually (updates cache + CSVs, skips if <15min old)
python scheduled_extract.py
python scheduled_extract.py --force    # ignore freshness guard

# Extract all historical data to CSV (dual-source equity financials)
python extract_historical_data.py

# Batch extract S&P 500 financials (~30-40 min for full run)
python extract_sp500_financials.py                              # full S&P 500, Yahoo
python extract_sp500_financials.py --source both                # Yahoo + SEC
python extract_sp500_financials.py --resume --exclude-top20     # skip existing + Top 20
python extract_sp500_financials.py --tickers CRM,AMD,NFLX      # specific tickers

# Monitor earnings dates and flag stale data
python monitor_earnings.py                          # scan all companies in database
python monitor_earnings.py --auto-update            # scan + re-extract stale tickers
python monitor_earnings.py --tickers AAPL,MSFT      # check specific tickers only

# Weekly data freshness review (compares local data vs SEC EDGAR)
python review_data_freshness.py                     # full S&P 500 review
python review_data_freshness.py --auto-update       # review + re-extract stale
python review_data_freshness.py --top20-only        # only Top 20
python review_data_freshness.py --report            # save CSV to data_export/

# Extract 13F institutional fund holdings (SEC EDGAR)
python extract_13f_holdings.py                              # all 5 funds, last 8 quarters
python extract_13f_holdings.py --funds berkshire_hathaway,citadel
python extract_13f_holdings.py --max-filings 4              # only last 4 quarters
python extract_13f_holdings.py --list-funds                 # show available funds

# Test individual Fidenza Macro gap-fill extractors
python -c "from data_extractors.fidenza_extractors import get_brent_crude; print(get_brent_crude())"
python -c "from data_extractors.fidenza_extractors import get_sofr_futures_term_structure; print(get_sofr_futures_term_structure())"
python -c "from data_extractors.fidenza_extractors import get_aaii_sentiment; print(get_aaii_sentiment())"
python -c "from data_extractors.fred_extractors import get_adp_employment; print(get_adp_employment())"

# Test OpenBB-based extractors (all use fallbacks if OpenBB not installed)
python -c "from data_extractors.openbb_extractors import get_vix_futures_curve; print(get_vix_futures_curve())"
python -c "from data_extractors.openbb_extractors import get_ecb_policy_rates; print(get_ecb_policy_rates())"
python -c "from data_extractors.openbb_extractors import get_fama_french_factors; print(get_fama_french_factors())"
python -c "from data_extractors.openbb_extractors import get_equity_risk_premium; print(get_equity_risk_premium())"

# Fast extraction — real-time yfinance only (~5s, safe for 5-min polling)
python fast_extract.py                # run once
python fast_extract.py --dry-run      # list what would be extracted
python fast_extract.py --force        # ignore freshness guard

# Hyperliquid extraction — 1-minute perp + spot data (~0.5s)
python hl_extract.py                  # run once
python hl_extract.py --dry-run        # show what would be extracted
python hl_extract.py --force          # ignore freshness guard

# Install macOS launchd auto-scheduler (all 3 jobs)
bash setup_launchd.sh                 # install scheduled-extract + fast-extract + hl-extract
bash setup_launchd.sh --status        # check all jobs
bash setup_launchd.sh --uninstall     # remove all jobs

# Run data review agent (requires MINIMAX_API_KEY)
python -m agent.openai_agents.agent "Scan all companies for missing data"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cdavocazh/macro_2](https://github.com/cdavocazh/macro_2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
