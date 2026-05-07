---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**TradingAgents-CN** is a Chinese stock analysis platform using a multi-agent AI system built with LangGraph. It consists of:

- **Backend API**: FastAPI (`app/`) - Web server, database management, real-time notifications
- **Frontend (Legacy)**: Vue 3 + Element Plus (`frontend/`) - Original SPA UI
- **Frontend (New)**: React 19 + Ant Design (`frontend-react/`) - Experimental new UI
- **Core Engine**: LangGraph multi-agent system (`app/engine/`) - Analysis workflow
- **Data Layer**: Multi-source stock data with MongoDB + Redis caching

## Common Commands

### Development Setup

```bash
# Install Python dependencies (requires Python 3.10+)
# pyproject.toml is primary; requirements.txt is deprecated
pip install -e .
# or using uv
uv sync

# Install legacy frontend dependencies
cd frontend && npm install

# Install React frontend dependencies
cd frontend-react && npm install

# Configure environment
copy .env.example .env  # Windows
cp .env.example .env    # Linux/Mac, then edit with your API keys
```

### Running the Application

```bash
# Start backend (from project root)
python -m app
# or with uvicorn directly
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Start legacy frontend (in frontend/ directory)
cd frontend
npm run dev          # Development server
npm run build        # Production build
npm run type-check   # TypeScript check
npm run lint         # ESLint

# Start React frontend (in frontend-react/ directory)
cd frontend-react
npm run dev
npm run build
npm run lint
```

### Docker Deployment

```bash
# Full stack with Docker Compose (includes MongoDB, Redis, Nginx)
docker compose -f docker-compose.build.yml up -d

# Access: http://localhost:3000 (Nginx proxy)
# API: http://localhost:8000/api
```

### Running Tests

```bash
# Run all pytest tests
python -m pytest tests/

# Run specific test file
python -m pytest tests/integration/test_subgraph_architecture.py
python -m pytest tests/mcp/test_basic.py
python -m pytest tests/routers/test_tools.py
```

## Architecture Overview

### Major Refactor Notice

The `tradingagents/` directory was merged into `app/engine/` (commit `951efcb`). All core agent logic, graph workflows, and tools now live under `app/engine/`. Any remaining imports referencing `tradingagents` are compatibility shims.

### Multi-Agent Workflow (`app/engine/`)

The core analysis engine uses **LangGraph** with a 4-stage pipeline:

1. **Stage 1 (Analysts)**: Vertical analysts (market, news, fundamentals, etc.) run in parallel
   - Configured in `config/agents/phase1_agents_config.yaml`
   - Dynamic loading via `app/engine/agents/analysts/dynamic_analyst.py`

2. **Stage 2 (Research Debate)**: Bull/Bear researchers debate based on stage 1 outputs
   - Configured in `config/agents/phase2_agents_config.yaml`

3. **Stage 3 (Risk Management)**: Risk teams evaluate trading plans
   - Configured in `config/agents/phase3_agents_config.yaml`

4. **Stage 4 (Trader)**: Trader agent generates final trading decision

**Key Files**:
- `app/engine/graph/trading_graph.py` - Main workflow orchestration and LLM provider routing
- `app/engine/graph/setup.py` - Graph construction
- `app/engine/graph/propagation.py` - State propagation logic
- `app/engine/default_config.py` - Default configuration
- `app/engine/agents/analysts/dynamic_analyst.py` - Dynamic analyst factory with MCP circuit breaker integration

### MCP Tool Architecture (`app/engine/tools/mcp/`)

MCP (Model Context Protocol) tools are managed as application-level infrastructure:

- **Loader**: `loader.py` - Based on `langchain-mcp-adapters`, loads local and external MCP tools
- **Task Manager**: `task_manager.py` - Task-level MCP manager with circuit breakers, retry mechanisms, and concurrency control
- **Health Monitor**: `health_monitor.py` - 30-second health check loop for all MCP connections
- **Tool Node**: `tool_node.py` - Custom tool node creation with error handlers
- **Config Utils**: `config_utils.py` - `MCPServerConfig`, config validation, and persistence

MCP connections are initialized at application startup (`app/main.py` lifespan) and kept alive throughout the application lifecycle.

### Data Layer (`app/data/`)

Unified data access with automatic fallback:

- **Interface**: `interface.py` - Public API for all data access (A-share, HK, US)
- **Data Source Manager**: `data_source_manager.py` - Multi-source management with auto-degradation
- **Providers**:
  - China: Tushare, AKShare, Baostock (in `providers/china/`)
  - US: Yahoo Finance, Finnhub (in `providers/us/`)
  - HK: HK stock providers (in `providers/hk/`)
- **Cache**: Multi-level caching (file, MongoDB, Redis) controlled by `TA_CACHE_STRATEGY`

**Usage**:
```python
from app.data.interface import get_china_stock_data_unified

data = get_china_stock_data_unified("000001", "2024-01-01", "2024-12-31")
```

### Backend API (`app/`)

FastAPI application structure:

- `main.py` - Application entry, lifespan management, APScheduler setup
- `routers/` - API route handlers (auth, analysis, config, MCP, tools, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BG8CFB/TradingAgents-CN](https://github.com/BG8CFB/TradingAgents-CN) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
