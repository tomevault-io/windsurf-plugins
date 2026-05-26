---
trigger: always_on
description: This repo wraps the Fubon Securities Python SDK (`fubon_neo`) into an MCP server. These rules make AI agents productive quickly in this codebase.
---

## Fubon API MCP Server — Copilot Rules (Concise)

This repo wraps the Fubon Securities Python SDK (`fubon_neo`) into an MCP server. These rules make AI agents productive quickly in this codebase.

### Architecture & Key Files
- **Monolith server**: `fubon_api_mcp_server/server.py` (all MCP tools, resources, global state)
- **Service classes**: `trading_service.py`, `market_data_service.py`, `account_service.py`, `reports_service.py`, `indicators_service.py` - each registers MCP tools with the main FastMCP instance
- **Service responsibilities**: Services follow a pattern — `MarketDataService` handles REST/futopt data parsing and normalization; `TradingService` encapsulates all order operations including condition and time-slice orders; `AccountService` provides accounting endpoints and PnL queries; `ReportsService` surfaces SDK callbacks into MCP tools.
- **Config & Utils**: `config.py` (env vars, data dirs), `utils.py` (account validation, error handling), `enums.py` (safe enum conversions)
- **Globals in server.py**: `sdk`, `accounts`, `reststock` (stock REST), `restfutopt` (futures/options REST), report lists (`latest_order_reports`, etc.)
- **VS Code extension**: `vscode-extension/` (spawns `python -m fubon_api_mcp_server.server`)
- **Examples**: `examples/` (demos), **Tests**: `tests/` (pytest with fixtures mocking SDK)

### MCP Tool Patterns (must follow)
- Decorate with `@mcp.tool()`. Validate inputs via Pydantic args classes defined near usage.
- Always call `validate_and_get_account(account)` (from `utils.py`) before trading APIs - reinitializes SDK per call.
- Unified response shape: `{"status": "success|error", "data": ..., "message": ...}`; add counts when useful.
- Error guard for services: if `reststock`/`restfutopt` is None, return `"期貨/選擇權行情服務未初始化"` (for futopt) or stock equivalent.
- API result handling:
  - Stock intraday/snapshot/historical: returns plain dict/list from REST client.
  - Fut/Opt intraday: returns object with `is_success` + `.data` (e.g., `ticker/quote/candles/volumes/trades`).
  - Fut/Opt `tickers`/`products`: dict with top-level keys (type, exchange, data[]). Parse `result["data"]` and normalize keys.
- Pass SDK parameters as keyword args (not dict param). Tests assert `assert_called_once_with(symbol="TX00", session="afterhours")` style.

### Important Gotchas & Conventions ⚠️
- MCP tools always validate and reinitialize accounts per-call — call `validate_and_get_account(account)` early in a tool function. This both fetches an account object and re-configures `sdk` for the active call.
- SDK results are not uniform — prefer `if result and hasattr(result, "is_success") and result.is_success:` then use `result.data`. If not `is_success`, check `result.message`.
- `@mcp.resource` endpoints are read-only and often return local `data/` cache only (eg: `twstock://{symbol}/historical`). This pattern avoids unnecessary remote API calls.
- Do not mutate globals in `server.py` such as `sdk`, `reststock`, `restfutopt`, `accounts`; tests rely on these being patched and reinitialized via `validate_and_get_account`.

### Concurrency & Long-running Ops 🔁
- `batch_place_order` and other batch operations use `ThreadPoolExecutor` for concurrency — follow its pattern for parallelizable tasks.
- Time-slice orders (`place_time_slice_order`) and other split strategies accept `split_count` and `single_quantity`. Ensure quantity units are *shares* (1000 shares = 1張).

### Tests & Mocking Patterns 🧪
- Unit tests in `tests/` mock global `sdk` and `accounts` via fixtures (`tests/conftest.py`). Patch and assert call styles: `sdk.stock.place_order.assert_called_once_with(**kwargs)` (keyword args).
- For fut/opt clients, tests assert normalization of `result["data"]` and top-level keys in `products/tickers` calls.

### Quick Examples (copy-paste) ✂️
- Place a stock order (follow `trading_service.py::place_order`):
```py
account_obj, err = validate_and_get_account(account)  # required
params = {"account": account_obj, "symbol": "2330", "price": "100", "quantity": 1000, "buy_sell": "Buy"}
result = sdk.stock.place_order(**params)
if result and hasattr(result, "is_success") and result.is_success:
  return result.data
```
- Parse a fut/opt ticker (follow `market_data_service.py`): call `restfutopt.tickers(product=...)` and check `data` + `is_success`.

### Testing Workflow (pytest)
- Fixtures: see `tests/conftest.py` (mocks `sdk`, `accounts`, and server globals via patching).
- Typical commands (PowerShell on Windows):
  ```pwsh
  python -m pytest -q
  python -m pytest --cov=fubon_api_mcp_server --cov-report=html
  python -m pytest tests/test_market_data_service.py::TestGetIntradayFutOptTickers -v
  ```
- Common fut/opt expectations used by tests:
  - `tickers/products`: input filters echoed in `filters_applied`; aggregate `total_count`, `type_counts`.
  - Service not initialized => specific error message above.
  - Normalize option fields: `contract_type`, `expiration_date`, `strike_price`, `option_type`, `underlying_symbol`.

### Dev Routines & Debugging
- Start MCP server: `python -m fubon_api_mcp_server.server` (logs under `log/`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mofesto/fubon-api-mcp-server](https://github.com/Mofesto/fubon-api-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
