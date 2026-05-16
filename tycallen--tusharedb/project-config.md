---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Tushare-DuckDB is a Python library that provides a local caching layer for [Tushare Pro](https://tushare.pro/) financial data using [DuckDB](https://duckdb.org/). It reduces API calls, speeds up data access, and enables local quantitative analysis for Chinese stock market data (A-shares, indices, ETFs).

## Common Commands

### Installation
```bash
pip install -e .
```

### Environment Setup
Copy `.env.example` to `.env` and set `TUSHARE_TOKEN`:
```bash
cp .env.example .env
# Edit .env with your Tushare Pro API token
```

### Data Operations
```bash
# Initialize historical data (full sync)
python scripts/init_data.py --trade-cal --stock-basic --pro-bar

# Initialize shareholder data (top10 floatholders, holder number, executive rewards)
python scripts/init_shareholder_data.py --all

# Daily incremental update
python scripts/update_daily.py
```

### Running Tests
```bash
pytest                    # Run all tests
pytest tests/            # Run tests in tests directory
pytest tests/test_duckdb_manager.py  # Run single test file
pytest -v                # Verbose output
```

## Architecture

The codebase uses a **separation of concerns** pattern with two main components:

### DataDownloader (`src/tushare_db/downloader.py`)
- **Write-only**: Downloads data from Tushare API and stores in DuckDB
- Used for initialization scripts and daily updates
- No query functionality, simple fetch -> write flow
- Requires `TUSHARE_TOKEN` environment variable

### DataReader (`src/tushare_db/reader.py`)
- **Read-only**: Queries data from local DuckDB database
- Zero network requests, millisecond response times
- Opens database in `read_only=True` mode for concurrent access
- Used for backtesting, web APIs, data analysis

### Key Internal Components

- **DuckDBManager** (`duckdb_manager.py`): Handles database operations with UPSERT support
- **TushareFetcher** (`tushare_fetcher.py`): Wraps Tushare API with rate limiting
- **Rate Limit Config** (`rate_limit_config.py`): Pre-configured profiles for different subscription levels (trial/standard/pro)
- **ConceptDataManager** (`concept_manager.py`): Manages concept sector data from jquant_data_sync GitHub releases

### Data Flow
```
Tushare API → TushareFetcher → DataDownloader → DuckDB
                                                  ↓
                              DataReader ← DuckDB (read_only)
                              ↓
                    ConceptDataManager ← GitHub Release (jquant_data_sync)
```

## Usage Patterns

### Download Data
```python
from tushare_db import DataDownloader

downloader = DataDownloader(db_path="tushare.db")
downloader.download_trade_calendar()
downloader.download_stock_basic(list_status='L')
downloader.download_all_stocks_daily(start_date='20200101')
downloader.close()
```

### Query Data
```python
from tushare_db import DataReader

reader = DataReader(db_path="tushare.db")
df = reader.get_stock_daily('000001.SZ', '20240101', adj='qfq')
reader.close()
```

### Query Shareholder Data
```python
from tushare_db import DataReader

reader = DataReader(db_path="tushare.db")

# Top 10 floating shareholders
df = reader.get_top10_floatholders('000001.SZ')
df = reader.get_top10_floatholders('000001.SZ', period='20231231')
df = reader.get_top10_floatholders('000001.SZ', holder_name='香港中央结算')

# Shareholder count (chip concentration)
df = reader.get_stk_holdernumber('000001.SZ')
df = reader.get_stk_holdernumber(['000001.SZ', '000002.SZ'])
df = reader.get_stk_holdernumber('000001.SZ', start_date='20230101', end_date='20231231')

# Executive rewards and holdings
df = reader.get_stk_rewards('000001.SZ')
df = reader.get_stk_rewards('000001.SZ', end_date='20231231')

reader.close()
```

## Database Schema

Primary keys are defined in `TABLE_PRIMARY_KEYS` dict in `duckdb_manager.py`. Key tables:

| Table | Primary Keys | Description |
|-------|-------------|-------------|
| `daily` | ts_code, trade_date | Daily OHLCV (stocks and indices) |
| `adj_factor` | ts_code, trade_date | Price adjustment factors |
| `daily_basic` | ts_code, trade_date | Daily PE/PB/市值等 |
| `stock_basic` | ts_code | Stock metadata |
| `trade_cal` | exchange, cal_date | Trading calendar |
| `index_classify` | industry_code | 申万行业分类 |
| `index_member_all` | ts_code, l3_code, in_date | 申万行业成分（PIT支持） |
| `index_weight` | index_code, con_code, trade_date | 指数成分权重 |
| `moneyflow` | ts_code, trade_date | 个股资金流向 |
| `cyq_perf` | ts_code, trade_date | 筹码分布 |
| `dc_member` | ts_code, trade_date, con_code | 龙虎榜机构明细 |
| `fina_indicator_vip` | ts_code, end_date | 财务指标（VIP） |
| `ths_daily` | ts_code, trade_date | 同花顺板块日行情 |
| `kpl_concept` | ts_code, trade_date | 开盘啦题材库 |
| `fund_daily` | ts_code, trade_date | 场内基金日线 |
| `top10_floatholders` | ts_code, end_date, holder_name | 前十大流通股东 |
| `stk_holdernumber` | ts_code, end_date | 股东户数 |
| `stk_rewards` | ts_code, end_date, name | 高管薪酬和持股 |

## Important Notes

### Adjustment Factors
- **Forward adjustment (qfq)**: `price × (adj_factor / latest_factor)` - Most recent price equals market price
- **Backward adjustment (hfq)**: `price × adj_factor` - Historical prices stay fixed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tycallen/TushareDB](https://github.com/tycallen/TushareDB) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
