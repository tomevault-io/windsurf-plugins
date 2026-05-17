---
trigger: always_on
description: Data access layer for Bloomberg Terminal via blpapi + MCP (Model Context Protocol).
---

# Bloomberg MCP — Project Guide

Data access layer for Bloomberg Terminal via blpapi + MCP (Model Context Protocol).
This server exposes Bloomberg data tools to AI assistants via natural language.

**Current state:** 12 tools, modular structure (PHASE 0 complete).
**Target state:** ~22 tools, full Bloomberg API surface coverage.

---

## Architecture Overview

```
src/bloomberg_mcp/
├── server.py                # FastMCP init + entry point (~89 lines)
├── models/
│   ├── __init__.py          # Re-exports all models
│   ├── enums.py             # ResponseFormat, EconomicCalendarModeInput, EarningsModeInput
│   └── inputs.py            # ALL Pydantic BaseModel input classes
├── handlers/
│   ├── __init__.py          # Imports all handler modules to trigger registration
│   ├── reference.py         # bloomberg_get_reference_data handler
│   ├── historical.py        # bloomberg_get_historical_data handler
│   ├── intraday.py          # bloomberg_get_intraday_bars + ticks handlers
│   ├── search.py            # search_securities + search_fields + field_info
│   ├── screening.py         # run_screen + get_universe + dynamic_screen
│   └── calendars.py         # economic_calendar + earnings_calendar
├── formatters.py            # _format_security_data, _format_historical_data, etc.
├── utils.py                 # _expand_fields, _normalize_date, _truncate_response, _get_session
├── core/
│   ├── session.py           # BloombergSession singleton (376 lines, solid)
│   ├── requests.py          # blpapi request builders using fromPy() (507 lines, solid)
│   └── responses.py         # Response parsers using toPy() (416 lines, needs BDS/TA/BQL)
└── tools/
    ├── reference.py         # get_reference_data() — thin wrapper
    ├── historical.py        # get_historical_data() — thin wrapper
    ├── intraday.py          # get_intraday_bars/ticks() — thin wrapper
    ├── search.py            # search_securities/fields/field_info (204 lines)
    ├── screening.py         # run_screen() — BEQS wrapper
    ├── dynamic_screening/   # DSL system: FieldSet + Filter + DynamicScreen (2,567 lines, mature)
    ├── morning_note/        # Market data collection system (3,648 lines, Japan-focused)
    ├── economic_calendar/   # Macro event calendar (874 lines)
    └── earnings_calendar/   # Earnings announcements (640 lines)
```

Connection: localhost:8194 (Bloomberg Desktop API). No authentication needed.

---

## Quick Reference — Function Signatures

```python
# Primary imports
from bloomberg_mcp.tools import (
    get_reference_data,      # BDP — current field values
    get_historical_data,     # BDH — time series
    get_intraday_bars,       # Intraday OHLCV bars
    get_intraday_ticks,      # Raw ticks
    search_securities,       # Find securities (//blp/instruments)
    search_fields,           # Discover fields (//blp/apiflds)
    get_field_info,          # Field metadata (//blp/apiflds)
)

# Data types
from bloomberg_mcp import (
    SecurityData,            # Reference data result
    HistoricalData,          # Historical data result
    IntradayBar,             # Single bar
    IntradayBarData,         # Bars collection
    ScreenResult,            # BEQS screen result
)
```

### Reference Data (BDP)
```python
def get_reference_data(
    securities: List[str],           # ["AAPL US Equity", "700 HK Equity"]
    fields: List[str],               # ["PX_LAST", "PE_RATIO"]
    overrides: Optional[Dict[str, Any]] = None,
) -> List[SecurityData]:
```

### Historical Data (BDH)
```python
def get_historical_data(
    securities: List[str],
    fields: List[str],
    start_date: str,                 # "YYYYMMDD" format
    end_date: str,
    periodicity: str = "DAILY",      # DAILY|WEEKLY|MONTHLY|QUARTERLY|YEARLY
) -> List[HistoricalData]:
```

### Data Types
```python
@dataclass
class SecurityData:
    security: str
    fields: Dict[str, Any]           # {"PX_LAST": 150.0, "PE_RATIO": 25.0}
    errors: List[str]

@dataclass
class HistoricalData:
    security: str
    data: List[Dict[str, Any]]       # [{"date": datetime, "PX_LAST": 150.0}, ...]
    errors: List[str]
```

---

## PHASE 0: Refactor server.py — COMPLETE

server.py split from 1,798 lines to 89 lines. All 12 tools preserved.
Duplicated fieldset_map consolidated to `utils.py` as single source of truth.

- [x] server.py < 200 lines (89 lines)
- [x] All 12 tools still work (same names, same params, same output)
- [x] `pytest tests/` passes (16 passed, 51 skipped, 0 failed)
- [x] No import errors

---

## PHASE 1: Add BDS Tool + Caching + New FieldSets

### Tool: bloomberg_get_bulk_data (BDS)

BDS is NOT a separate request type — it uses the same `ReferenceDataRequest`.
The difference is in the RESPONSE: bulk fields return arrays instead of scalars.

Detection: `element.isArray()` returns True for bulk fields.
The existing `parse_reference_data_response()` already handles this via `field.toPy()`,
but the response is not optimized for AI consumption.

```python
# New tool in handlers/bulk.py
@mcp.tool(name="bloomberg_get_bulk_data")
async def bloomberg_get_bulk_data(params: BulkDataInput) -> str:
    """
    Get Bloomberg bulk data (BDS) — returns tabular/array data.
    
    Unlike reference data (BDP) which returns single values,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [QmQsun/Bloomberg-MCP](https://github.com/QmQsun/Bloomberg-MCP) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
