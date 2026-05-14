---
trigger: always_on
description: > This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

## twsemcpserver

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TWStockMCPServer is a Model Context Protocol (MCP) server for Taiwan stock market data analysis. Built with FastMCP (Python) and `requests`. Data sources:
- **TWSE OpenAPI** (`openapi.twse.com.tw`) — 143 tools: 公司治理、ESG、財報、交易、指數、券商
- **TWSE exchangeReport** (`twse.com.tw/exchangeReport`) — 4 tools: 歷史日K、月均價、估值、融資融券（legacy JSON，非 Swagger）
- **MIS 即時報價** (`mis.twse.com.tw`) — 1 tool: 盤中多股即時報價
- **TPEx OpenAPI** (`tpex.org.tw/openapi`) — 3 tools: 上櫃日收盤、三大法人、本益比
- **TAIFEX OpenAPI** (`openapi.taifex.com.tw`) — 16 tools: 三大法人系列、大額交易人部位、每日行情、選擇權分析（Delta/OI增減）、保證金、年月統計

## Development Commands

| Task | Command |
|------|---------|
| Install dependencies | `uv sync` |
| Install with test deps | `uv sync --extra dev` |
| Run server (dev) | `uv run fastmcp dev server.py` |
| Run server (prod) | `uv run fastmcp run server.py` |
| Run all tests | `uv run pytest` |
| Run specific test file | `uv run pytest tests/e2e/test_esg_api.py -v` |
| Run tests by category | `python run_tests.py esg` (also: `company`, `financials`, `trading`, `warrants`, `other`, `history`, `realtime`, `otc`, `taifex`, `api`, `e2e`) |
| Quick test (fail fast) | `python run_tests.py quick` |
| Tests with coverage | `python run_tests.py cov` (opens HTML report) |
| Run server directly | `python server.py` (HTTP on port 8000) |

## Code Architecture

### High-Level Structure

```
server.py                     # Thin entrypoint: FastMCP init, prompt registration, tool registration
models/                       # Pydantic-style data models (MarketInfo, BrokerInfo, RealTimeStats)
utils/
├── api_client.py             # TWSEAPIClient - all TWSE HTTP calls
├── config.py                 # APIConfig, DisplayConfig, TestConfig (env var overrides)
├── constants.py              # Localized message templates (Chinese)
├── decorators.py             # @handle_api_errors, @handle_empty_response
├── formatters.py             # Data → string formatting functions
├── tool_factory.py           # create_company_tool() for dynamically named tools
└── types.py                  # TWSEDataItem TypedDict, DataFormatter Protocol
tools/
├── __init__.py               # register_all_tools() - auto-discovers and registers all tool modules
├── broker.py                 # Broker data tools (top-level module)
├── other.py                  # Misc tools: funds, bonds, holidays (top-level module)
├── company/                  # Company tools: basic_info, financials, esg, listing, news
├── trading/                  # Trading tools: daily, periodic, valuation, dividend_schedule, etf, market, warrants
├── market/                   # Market tools: indices, statistics, foreign
├── history/                  # TWSE legacy exchangeReport: stock_day, stock_day_avg, bwibbu_all, margin_balance
├── realtime/                 # MIS real-time quotes: stock_info
├── otc/                      # TPEx OTC market: daily_close, institutional, peratio
└── taifex/                   # TAIFEX derivatives: futures_position, put_call_ratio, institutional_general,
                              #   institutional_details, daily_market_report, large_traders_oi,
                              #   options_analytics, margin, trading_statistics
prompts/                      # 5 prompt templates registered in server.py
```

### Key Architectural Patterns

**Dependency Injection**: `server.py` creates one `TWSEAPIClient` instance and passes it to `register_all_tools(mcp, api_client)`. The auto-discovery engine in `tools/__init__.py` uses `pkgutil.iter_modules` to find all tool modules, then calls `module.register_tools(mcp, client)` on each.

**Tool Module Contract**: Every tool module must expose:
```python
def register_tools(mcp: FastMCP, client: Optional[TWSEAPIClient] = None) -> None:
```
The `client` is captured via closure. Tools are registered with `@mcp.tool` — the function docstring becomes the MCP tool description.

**Auto-Discovery**: `tools/__init__.py` scans direct modules (`tools/broker.py`, `tools/other.py`) and subpackage modules (`tools/company/*.py`, etc.) automatically. No manual registration needed in `server.py` when adding new tool modules.

**API Client**: `TWSEAPIClient` has instance methods (`fetch_data`, `fetch_company_data`, `fetch_latest_market_data`) and class-method wrappers (`get_data`, `get_company_data`, `get_latest_market_data`) for backward compatibility. Instance methods are preferred. Includes built-in rate limiting (0.5s between requests). For non-OpenAPI sources (legacy TWSE, MIS, TPEx, TAIFEX), use `fetch_json(url, params)` / `get_json(url, params)` which accepts full URLs with query parameters and returns raw JSON.

**Decorators**: Tool functions use decorators from `utils/decorators.py`:
- `@handle_api_errors(data_type="...", use_code_param=True)` — wraps in try/except, returns localized error message
- `@handle_empty_response(data_type="...")` — returns localized "no data" message for None/empty results

**Formatters**: `utils/formatters.py` provides:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tomevault-io/copilot-plugins](https://github.com/tomevault-io/copilot-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
