---
trigger: always_on
description: Home Assistant custom integration providing 15-minute electricity spot prices across global markets. **Production code affecting real homes and electricity costs.**
---

# GitHub Copilot Instructions for GE-Spot

Home Assistant custom integration providing 15-minute electricity spot prices across global markets. **Production code affecting real homes and electricity costs.**

## Core Principles

1. **Read entire files before editing** - Never edit based on grep/partial reads
2. **Configuration-driven, never hardcode** - All values from `const/` modules
3. **Generic naming only** - `interval_prices`, not `hourly_prices` or `15min_data`
4. **No backward compatibility** - Rename directly, update all callers, no aliases
5. **Test after every change** - Clear `__pycache__`, run tests, verify logs
6. **No verbose summaries** - User sees changes in editor, only respond if asked
7. **Keep it simple** - Clean code over complexity, cleanup as you go
8. **Ask before acting** - When uncertain, clarify first
9. **No analysis/summary files in git** - All `*ANALYSIS*.md`, `*SUMMARY*.md` files stay untracked
10. **Format with black** - Run `black` on all Python code before committing. Consistent formatting is mandatory.

## Architecture

### Configuration-Driven Design
Single constant controls entire system: `TimeInterval.DEFAULT = "PT15M"` (15 minutes, 96 intervals/day)

```python
# ✅ Correct
from ..const.time import TimeInterval
intervals = TimeInterval.get_intervals_per_day()  # 96

# ❌ Wrong
intervals = 96  # NEVER hardcode
```

### Three-Layer Structure

1. **API Layer** (`api/`): Source-specific clients (Nordpool, ENTSO-E, AEMO, etc.)
   - Clients in `api/*.py`, parsers in `api/parsers/`
   - Base classes in `api/base/` (session, errors, data structures)
   - All return `StandardizedPriceData`

2. **Coordinator Layer** (`coordinator/`): `UnifiedPriceCoordinator` manages lifecycle
   - Source fallback (tries priority order, auto-switches on failure)
   - Caching via `CacheManager` (TTL, persistence, deep copies to prevent mutation)
   - Rate limiting (15min minimum between fetches per area)

3. **Sensor Layer** (`sensor/`): Home Assistant entities
   - Current price, next interval, averages, peaks, tomorrow forecasts
   - Attributes contain 96 interval entries for automations
   - Source-agnostic (entity IDs stable regardless of API source)

**Data Flow:**
```
API Client → Parser → StandardizedPriceData → UnifiedPriceManager → 
CacheManager/DataProcessor → Sensors → Home Assistant
```

**Cross-Cutting:**
- **Timezone** (`timezone/`): Normalizes all timestamps to HA timezone, handles DST (92-100 intervals on transitions)
- **Price** (`price/`): Currency conversion (ECB rates), VAT, unit formatting (EUR/kWh vs cents/kWh)
- **Constants** (`const/`): `Config`, `TimeInterval`, `Source`, `Defaults`

## Critical Rules

### Always Read Full Files
Before editing, use `read_file` for entire file. Understand context, identify all change locations.

### Use Configuration
```python
# ✅ Do this
interval_minutes = TimeInterval.get_interval_minutes()
items_per_cycle = TimeInterval.get_intervals_per_day()

# ❌ Never this
interval_minutes = 15
items_per_cycle = 96
```

### Generic Naming
```python
# ✅ Generic
normalize_interval_prices(), interval_key, IntervalCalculator

# ❌ Specific
normalize_hourly_prices(), hour_key, QuarterHourCalculator
```

### No Backward Compatibility
```python
# ❌ Don't create aliases
def old_name(*args, **kwargs):
    return new_name(*args, **kwargs)

# ✅ Just rename and update ALL callers
def new_name(data, ...):
    # implementation
```

### Verify Your Work
```bash
# Search for old terms
grep -r "old_function_name" custom_components/

# Find hardcoded values
grep -r "= 96" custom_components/

# Test imports
python3 -c "from custom_components.ge_spot.module import NewClass"

# Format with black
black custom_components/ tests/
```

## Code Standards

**Format with Black:** All Python code must be formatted with `black` (line length 88) before committing. Run `black .` at project root or use pre-commit hooks.

**Imports:** Group as stdlib, third-party, local. Relative imports within `custom_components/ge_spot/`.

**Error Handling:** Specific exceptions, log with context, graceful fallback.
```python
try:
    result = api.fetch_data()
except APIError as e:
    _LOGGER.error(f"API fetch failed: {e}")
    return cached_data
```

**Type Hints:** Required on all functions. Use `Optional`, `Union`, `Dict[str, Any]`.

**Async:** Almost everything is async. Use `await`, sessions via `async_get_clientsession(hass)`.

**Docstrings:** Every public function/class. Include types, example for complex functions.

**Comments:** Explain WHY, not WHAT. Update or remove when changing code.

## Testing

**Priority:** Test production code, not test helpers. Mock external APIs/expensive ops, use real calls for integration tests.

```python
# ✅ Test production
def test_api_fetch_real_data():
    api = ProductionAPI(api_key="test_key")
    result = api.fetch_data(region="TEST")
    assert result is not None
    assert len(result["data"]) > 0

# ❌ Test mocks
def test_mock_returns_structure():
    mock_data = create_mock_data()
    assert mock_data["field"] == "expected"
```

**When to Mock:** External APIs with rate limits/costs, authentication, slow operations, non-deterministic behavior


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [enoch85/ge-spot](https://github.com/enoch85/ge-spot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
