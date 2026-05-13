---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Finance-Bro is an AI-powered financial trading platform using a multi-agent architecture with FastAPI backend and Next.js frontend. The platform combines machine learning, real-time market data, and automated trading strategies with a modern, responsive web interface built using Untitled UI components.

## Key Commands

### Development
```bash
# Start both frontend and backend (from root)
make dev

# Frontend development
cd frontend && npm run dev

# Backend only development
make dev-backend
# or directly:
cd backend && langgraph dev --port 8000

# Docker development
docker-compose -f docker-compose.dev.yml up
```

### Testing
```bash
# Backend Tests
cd backend && make test

# Run specific test categories
cd backend && python run_tests.py --marker unit
cd backend && python run_tests.py --marker integration
cd backend && python run_tests.py --marker api

# Test with coverage
cd backend && make test_coverage

# Test specific components
make test_formula    # Formula engine tests
make test_api       # API tests

# Frontend Tests
cd frontend && npm test           # Run all tests
cd frontend && npm run test:watch    # Watch mode
cd frontend && npm run test:coverage # Coverage report
cd frontend && npm run test:ci      # CI mode
```

### Code Quality
```bash
# Lint and type check
cd backend && make lint

# Format code
cd backend && make format
```

## Architecture

The system uses a **multi-agent microservices architecture** with four specialized AI agents:

1. **EventAgent** (`backend/src/EventAgent/`): Market event analysis and trading signals using LangGraph
2. **Research_Agent** (`backend/src/Research_Agent/`): Fundamental analysis and market research
3. **ts_agent** (`backend/src/ts_agent/`): Time series forecasting with 8+ ML models (TimeGPT, DeepAR, NBEATS, etc.)
4. **reward_agent** (`backend/src/reward_agent/`): Reinforcement learning for strategy optimization

### Key Components

- **Main API**: `backend/comprehensive_api.py` - FastAPI server with WebSocket support
- **Formula Engine**: `backend/src/formula_engine/` - Custom DSL for financial modeling with 50+ functions
- **Portfolio Management**: `backend/src/ts_agent/portfolio_manager.py` - Position tracking and risk management
- **Time Series Models**: `backend/src/ts_agent/models/` - Multiple forecasting implementations

### Agent Communication Flow
```
User Request → comprehensive_api.py → Agent Router → Specific Agent → Response
                                           ↓
                                    LangGraph State Management
```

## Technology Stack

- **Backend**: FastAPI, LangGraph, LangChain, Python 3.11+
- **AI/ML**: Google Generative AI, Nixtla TimeGPT, GluonTS, PyTorch
- **Financial Data**: Yahoo Finance, Alpha Vantage, Interactive Brokers API
- **Testing**: pytest with custom markers (unit, integration, api, formula, ts, portfolio)
- **Frontend**: Next.js 15, React 19, TypeScript, Untitled UI components, Tailwind CSS

## Important Configuration

### Environment Variables
Create `.env` file in backend directory:
```bash
# OpenAI API Configuration (REQUIRED)

# Other API Keys (optional)
NIXTLA_API_KEY=your_key        # For TimeGPT forecasting
ALPHA_VANTAGE_API_KEY=your_key # For market data
NEWS_API_KEY=your_key           # For news sentiment
USE_IBKR=false                  # Set true for Interactive Brokers integration
```

### Test Markers
When writing tests, use appropriate markers:
- `@pytest.mark.unit` - Unit tests
- `@pytest.mark.integration` - Integration tests
- `@pytest.mark.api` - API endpoint tests
- `@pytest.mark.formula` - Formula engine tests
- `@pytest.mark.ts` - Time series tests
- `@pytest.mark.network` - Tests requiring network access

## Development Notes

1. **Paper Trading Default**: All trading operations are simulated by default for safety
2. **Agent State Management**: Each agent uses LangGraph for state management - check `graph.py` in agent directories
3. **Formula Engine**: Custom DSL parser in `formula_engine/parser.py` - extend with new functions in `formula_functions.py`
4. **Time Series Models**: Add new models by extending `BaseTimeSeriesModel` in `ts_agent/models/base_model.py`
5. **API Endpoints**: All endpoints defined in `comprehensive_api.py` with Pydantic models for validation

## Common Tasks

### Adding a New Trading Strategy
1. Implement in `backend/src/formula_engine/formula_functions.py`
2. Add tests in `backend/tests/test_formula_engine.py`
3. Update DSL documentation if adding new functions

### Implementing New Time Series Model
1. Create model class in `backend/src/ts_agent/models/`
2. Inherit from `BaseTimeSeriesModel`
3. Register in `TimeSeriesPredictor` class
4. Add unit tests with `@pytest.mark.ts` marker

### Extending Agent Capabilities
1. Modify agent's `tools_and_schemas.py` for new tools
2. Update `graph.py` for workflow changes
3. Test with integration tests using `@pytest.mark.integration`

## Frontend Architecture

### Next.js 15 Setup
The frontend uses Next.js 15 with the App Router and React 19, built with Untitled UI components and Tailwind CSS v4.

### Key Frontend Components


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tanglumy/Finance-Bro](https://github.com/Tanglumy/Finance-Bro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
