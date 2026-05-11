---
trigger: always_on
description: **Generated:** 2026-04-17
---

# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-17
**Branch:** main

## OVERVIEW

MetaTrader 5 research/automation toolkit exposing 68 MCP tools (one of which — `market_depth_fetch` — is enabled only when `MTDATA_ENABLE_MARKET_DEPTH_FETCH=1`) for forecasting, regime detection, pattern recognition, signal processing, and trading. Python 3.14 backend (MCP server + CLI + FastAPI web API) with React/Vite frontend. ~70k Python LOC, ~3.4k TypeScript LOC.

## STRUCTURE

```
mtdata/
├── src/mtdata/
│   ├── bootstrap/      # Runtime init, settings, tool loading (4 files)
│   ├── core/           # MCP tools, CLI, server, web API, all API-facing logic
│   │   ├── cli/        # Dynamic CLI (argparse) with parsing/ and runtime/ subpackages
│   │   ├── data/       # data_fetch_candles / data_fetch_ticks / wait_event tools
│   │   ├── regime/     # Regime detection package
│   │   │   └── methods/     # HMM, BOCPD, MS-AR implementations
│   │   ├── report/     # report_generate runtime, request models, rendering
│   │   ├── report_templates/  # Per-style report templates (basic, intraday, swing, …)
│   │   ├── reports/    # Legacy/shared report helpers
│   │   └── trading/    # trade_*, account/positions/risk modules
│   ├── forecast/       # Forecasting engines, backtests, methods registry, model store
│   │   └── methods/    # Individual model implementations
│   ├── patterns/       # Chart/candlestick/Elliott wave detection
│   │   └── classic_impl/  # Classic pattern algorithm implementations
│   ├── services/       # MT5 gateway, Finviz, options/news data access
│   │   └── finviz/     # Finviz package with endpoints/ subdirectory
│   ├── shared/         # Cross-module schemas and constants
│   └── utils/          # Indicators, denoising, dimension reduction, formatting
│       └── denoise/    # Denoising package with filters/ subdirectory
├── tests/              # 158+ test files, hybrid pytest/unittest.TestCase
├── webui/              # React + Vite + Tailwind frontend
│   └── src/            # App.tsx, 4 components, hooks, API client, chart lib (16 .ts/.tsx files)
├── docs/               # User-facing documentation (26 files including forecast/ subdirectory)
├── scripts/            # MT5 time offset detection, backtest plotting
└── prompts/            # Prompt templates
```

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add/modify MCP tool | `src/mtdata/core/` | Each domain has its own module (`data/`, `forecast.py`, `trading/`, etc.). Register the file in `src/mtdata/bootstrap/tools.py::TOOL_MODULE_NAMES` if it adds a new module. |
| Add forecast method | `src/mtdata/forecast/methods/` + `forecast_registry.py` | Register in registry, implement interface |
| Background training / model store | `src/mtdata/forecast/task_manager.py`, `forecast/model_store.py`, `core/forecast_tasks.py` | Concurrency caps via `MTDATA_TRAIN_WORKERS`/`MTDATA_HEAVY_LIMIT`; cache via `MTDATA_MODEL_STORE`/`MTDATA_MODEL_TTL_DAYS` |
| Fix MT5 data access | `src/mtdata/services/data_service.py` | Main data gateway |
| Fix Finviz integration | `src/mtdata/services/finviz/` | Package with endpoints/ subdirectory |
| Modify pattern detection | `src/mtdata/patterns/` | `classic.py` delegates to `classic_impl/` |
| Change indicators | `src/mtdata/utils/indicators.py` | 100+ technical indicators |
| Edit denoising filters | `src/mtdata/utils/denoise/` | Package with filters/ subdirectory |
| Modify web UI | `webui/src/` | App.tsx is main, 4 components, features/, hooks/, lib/ |
| Server/transport config | `src/mtdata/core/server.py` | SSE, stdio, streamable-HTTP modes |
| Web API routes | `src/mtdata/core/web_api.py` (+ `web_api_runtime.py`, `web_api_handlers.py`, `web_api_models.py`) | Routes mounted under both `/api` and `/api/v1` |
| CLI changes | `src/mtdata/core/cli/` | Package with `parsing/`, `runtime/`, and `formatting/` subpackages for CLI helpers |
| Trading logic | `src/mtdata/core/trading/` | Split into `account.py`, `orders.py`, `positions.py`, `risk.py`, `validation.py`, `safety.py`, etc. |
| Report generation | `src/mtdata/core/report/` + `report_templates/` | `report/__init__.py` registers `report_generate` |
| Regime detection | `src/mtdata/core/regime/` | Package with methods/ (HMM, BOCPD, MS-AR) |
| Shared schemas | `src/mtdata/shared/schema.py` | Pydantic models |
| Runtime/env setup | `src/mtdata/bootstrap/` | `settings.py`, `runtime.py`, `tools.py` (tool bootstrap) |

## ENTRY POINTS

No root wrapper scripts. All entry points are console scripts in `pyproject.toml`:
- `mtdata-cli` → `mtdata.core.cli:main` — Dynamic CLI with argparse subparsers per tool
- `mtdata-sse` → `mtdata.core.server:main_sse` — MCP server (SSE transport)
- `mtdata-stdio` → `mtdata.core.server:main_stdio` — MCP server (stdio transport)
- `mtdata-streamable-http` → `mtdata.core.server:main_streamable_http` — MCP server (HTTP transport)
- `mtdata-webapi` → `mtdata.core.web_api:main_webapi` — FastAPI + bundled UI on :8000

Request flow: `entry point → load_environment() → bootstrap_tools() → mcp.run() / uvicorn.run()`

## CONVENTIONS

- **Python**: PEP 8, 4-space indent, type hints on public functions. `snake_case` functions/modules, `PascalCase` classes, `UPPER_SNAKE_CASE` constants.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [emerzon/mt-data-mcp](https://github.com/emerzon/mt-data-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
