---
trigger: always_on
description: This is an AI-powered freight operations platform designed specifically for hotshot trucking businesses. The platform leverages AI agents to automate and optimize critical freight operations including dispatch, rate analysis, compliance tracking, and route optimization.
---

# Freight Operations Platform - Claude Code Instructions

## Project Overview

This is an AI-powered freight operations platform designed specifically for hotshot trucking businesses. The platform leverages AI agents to automate and optimize critical freight operations including dispatch, rate analysis, compliance tracking, and route optimization.

**Current Status**: Early-stage development. Core data models and configuration are in place; agents and tools are planned but not yet implemented.

## Current Codebase Structure

```
freight-ops/
├── src/
│   ├── __init__.py              # Package init (version 0.1.0)
│   ├── agents/
│   │   └── __init__.py          # Placeholder - agents not yet implemented
│   ├── core/
│   │   └── __init__.py          # Placeholder - infrastructure not yet implemented
│   ├── data/
│   │   ├── __init__.py
│   │   └── models/
│   │       ├── __init__.py
│   │       └── load.py          # ✅ IMPLEMENTED - Core Load model with profitability calculations
│   └── tools/
│       └── __init__.py          # Placeholder - MCP tools not yet implemented
├── config/
│   ├── config.yaml              # Business configuration (H-4 Strategic Solutions)
│   └── llms.json                # LLM provider and agent assignment configuration
├── notebooks/
│   └── getting_started.ipynb    # Introductory notebook
├── scripts/
│   └── init_project.py          # Project initialization script
├── tests/
│   └── __init__.py              # Tests not yet implemented
├── .env.example                 # Environment variable template (comprehensive)
├── pyproject.toml               # Project dependencies and tool configuration
└── README.md                    # Project documentation
```

### What's Implemented

1. **`src/data/models/load.py`** - Fully implemented Load Pydantic model including:
   - Load status/type enums
   - Location model
   - Core Load model with all freight fields
   - Computed properties: `total_miles`, `deadhead_percentage`, `gross_revenue`, `rate_per_mile`, `all_miles_rate`, `trip_duration_hours`
   - Profitability check method: `is_profitable(min_rpm, max_deadhead_pct)`

2. **Configuration** - Fully set up:
   - `config/config.yaml` - Business rules for H-4 Strategic Solutions LLC (DOT# 4486526, MC# MC-1772833-C)
   - `config/llms.json` - LLM assignments per agent with fallback models
   - `.env.example` - All required API keys and settings documented

3. **Project tooling** - Configured in `pyproject.toml`:
   - Dependencies: anthropic, openai, langchain, mcp, pydantic, httpx, pandas, sqlalchemy, etc.
   - Dev tools: pytest, ruff, mypy, black
   - Entry points defined for CLI and agent commands

### What Needs Implementation

**Priority Order:**

1. **Base Agent Class** (`src/agents/base.py`)
   - MCP protocol integration
   - Tool registry access
   - Structured logging
   - Error handling patterns

2. **Core Infrastructure** (`src/core/`)
   - `config.py` - Configuration loader using dynaconf
   - `mcp_registry.py` - Tool discovery and management
   - `sandbox.py` - Secure execution environment (optional)

3. **Data Models** (`src/data/models/`) - Extend from Load model:
   - `route.py` - Route with waypoints, distances, timing
   - `rate.py` - Rate analysis and market data
   - `driver.py` - Driver info and equipment
   - `expense.py` - Fuel, tolls, maintenance tracking
   - `settlement.py` - Driver pay calculations

4. **AI Agents** (`src/agents/`)
   - `dispatch.py` - Load matching and booking
   - `rate_analysis.py` - Market rate analysis
   - `compliance.py` - IFTA, HOS, DOT compliance
   - `route_optimizer.py` - Deadhead reduction
   - `settlement.py` - Driver pay calculations

5. **MCP Tools** (`src/tools/`)
   - Load board integrations (DAT, Truckstop.com)
   - Mapping/routing tools
   - Financial calculators

## Development Guidelines

### Code Style and Quality

```bash
# Install dependencies
uv sync

# Install dev dependencies
uv sync --extra dev

# Lint and format
ruff check src/
ruff format src/

# Type checking
mypy src/

# Run tests
pytest tests/

# Run tests with coverage
pytest --cov=src tests/
```

### Python Standards

- **Python 3.12+** required
- Use **type hints everywhere** - this domain is complex, types help
- Use **Pydantic v2** for all data models (see `load.py` for patterns)
- Follow the **computed_field** pattern for derived values
- Use **Decimal** for all monetary values (never float)
- Use **Enum** classes for fixed choices (LoadStatus, LoadType)

### Adding New Data Models

Follow the pattern in `src/data/models/load.py`:

```python
from decimal import Decimal
from datetime import datetime
from typing import Optional
from pydantic import BaseModel, Field, computed_field

class MyModel(BaseModel):
    """Docstring explaining the model."""

    # Required fields with Field() for validation/description
    required_field: str = Field(..., description="What this field represents")

    # Optional fields with defaults
    optional_field: Optional[str] = None

    # Numeric validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [grantrb95/freight-ops](https://github.com/grantrb95/freight-ops) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
