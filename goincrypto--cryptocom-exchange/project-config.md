---
trigger: always_on
description: Python library for Crypto.com Exchange API with full type safety, logging, and OpenAPI specification integration.
---

# Crypto.com Exchange Python Library

## Overview

Python library for Crypto.com Exchange API with full type safety, logging, and OpenAPI specification integration.

## Recent Changes (v2.0.0)

### Major Updates
- **Enhanced Pair class** - Now includes full instrument metadata (fees, tick sizes, limits)
- **Order execution flags** - Changed from single `exec_type` to `exec_flags: list[OrderExecFlag]`
- **New OrderExecFlag enum** - 7 values: POST_ONLY, REDUCE_ONLY, SMART_POST_ONLY, ISOLATED_MARGIN, MARGIN_ORDER, LIQUIDATION, NOTIONAL_ORDER
- **Logging infrastructure** - Module-based loggers with DEBUG level showing request/response times
- **OpenAPI integration** - 101 YAML specs downloaded and organized in `openapi_docs/`
- **Type safety** - 95% type error reduction (1054 → 46 false positives)

### Breaking Changes
- `Pair` constructor now requires `inst_type: InstrumentType` parameter
- `Order.exec_type` → `Order.exec_flags: list[OrderExecFlag]`
- `get_price()` returns `float | None` instead of `float`
- `listen_candles/trades/orderbook()` signatures: `*pairs: Pair` (not `list[Pair]`)

### Version
Current: **2.0.0** (major bump due to breaking changes)

## Discover the Project

### File Structure

```
cryptocom-exchange/
├── AGENTS.md                    # This file - comprehensive guide
├── README.md                    # Public documentation
├── pyproject.toml              # Dependencies and build config
├── generate.py                  # Generate pairs from API
├── openapi_docs/               # OpenAPI specification files
│   ├── README.md
│   ├── download_api_docs.py    # Download 101 YAML specs
│   ├── analyze_api.py          # Parse specs and generate summary
│   ├── api_summary.json        # Generated API documentation
│   └── specs/                  # 101 endpoint YAML files
├── src/cryptocom/exchange/
│   ├── __init__.py             # Package exports
│   ├── api.py                  # HTTP/WebSocket API provider
│   ├── market.py               # Public market data (Exchange)
│   ├── private.py              # Private trading (Account)
│   ├── structs.py              # Data structures and enums
│   └── pairs.py                # Generated trading pairs
└── tests/
    ├── test_market.py          # Market data tests (19 tests)
    ├── test_private.py         # Private trading tests (20 tests)
    └── captured/               # Captured API responses for replay
```

### Key Files to Explore

**Core Implementation:**
- `src/cryptocom/exchange/structs.py` - All data classes, enums, type definitions
- `src/cryptocom/exchange/market.py` - Exchange class, public API
- `src/cryptocom/exchange/private.py` - Account class, private API
- `src/cryptocom/exchange/api.py` - HTTP/WebSocket request handling

**Configuration & Generation:**
- `generate.py` - Regenerate pairs with latest metadata from API
- `pyproject.toml` - Dependencies, build settings, package config

**Documentation:**
- `openapi_docs/specs/*.yaml` - Official API specs (101 endpoints)
- `openapi_docs/api_summary.json` - Parsed API documentation

### Understanding the Code

**Start Here:**
1. Read `AGENTS.md` (this file) - Overview and quick start
2. Check `src/cryptocom/exchange/structs.py` - Data models and enums
3. Explore `src/cryptocom/exchange/market.py` - Public API examples
4. Look at `tests/test_market.py` - Usage examples

**Key Concepts:**
- **Pair** - Trading pair with full metadata (fees, tick sizes, limits)
- **InstrumentType** - CCY_PAIR (spot), DERIVATIVES (perps), FUTURES
- **OrderExecFlag** - Execution instructions (POST_ONLY, REDUCE_ONLY, etc.)
- **Timeframe** - Candle intervals (1m, 5m, 1h, 1D, etc.)

### Recent Chat Context

**Latest Improvements:**
1. ✅ Type safety improvements (95% error reduction)
2. ✅ Order execution flags as array (`list[OrderExecFlag]`)
3. ✅ Logging with request/response times
4. ✅ OpenAPI specs downloaded and organized
5. ✅ Enhanced Pair with instrument metadata
6. ✅ All 20 core tests passing

**Current Status:**
- Type checking: basedpyright 1.39.9
- Remaining issues: ~46 (20 false positives from dataclass init)
- Tests: 20/20 passing ✅

## Features

- ✅ Full REST API coverage (101 endpoints)
- ✅ WebSocket streaming (candles, trades, orderbook)
- ✅ Type-safe data structures with basedpyright
- ✅ Comprehensive logging infrastructure
- ✅ OpenAPI spec documentation
- ✅ Test coverage with captured responses

## Quick Start

```python
import asyncio
from cryptocom.exchange import Exchange, Account

async def main():
    # Public market data
    exchange = Exchange()
    await exchange.sync_pairs()

    # Get all spot pairs
    pairs = await exchange.get_pairs()
    print(f"Available pairs: {len(pairs)}")

    # Get ticker
    ticker = await exchange.get_ticker(pairs[0])
    print(f"Price: {ticker.trade_price}")

    # Private trading (requires API keys)
    account = Account(api_key="YOUR_KEY", api_secret="YOUR_SECRET")
    await account.sync_pairs()

    # Get balance
    balance = await account.get_balance()

    # Place order
    order_id = await account.buy_limit(
        pair=pairs[0],
        quantity=1.0,
        price=50000.0,
        exec_flags=[OrderExecFlag.POST_ONLY]  # Optional execution flags
    )

asyncio.run(main())
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goincrypto/cryptocom-exchange](https://github.com/goincrypto/cryptocom-exchange) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
