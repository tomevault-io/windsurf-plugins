---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Dexter Protocol is an advanced AI-powered liquidity management platform for decentralized exchanges (DEXs). The project has evolved into a comprehensive DeFi infrastructure with multiple integrated components:

### Core Components:
1. **Position Management System**: Advanced Uniswap V3 position management with auto-compounding
2. **AI Optimization Engine**: ML-driven strategies for yield optimization and risk management  
3. **Backend/DexBrain**: Centralized intelligence system processing data across all agents
4. **Website Integration**: Separate repository for user interface (https://github.com/MeltedMindz/dexter-website)
5. **Smart Contracts**: On-chain infrastructure for position management and fee distribution
6. **Uniswap V4 Hook System**: Next-generation concentrated liquidity optimization with AI-powered hooks

### Current Development Status (June 2025):
- ✅ **Complete Position Management System**: Professional-grade auto-compounding with AI integration
- ✅ **Advanced Smart Contracts**: DexterCompoundor.sol with enhanced TWAP protection and AI optimization
- ✅ **ERC4626 Vault Infrastructure**: Complete vault system with Gamma-inspired dual-position strategies
- ✅ **Production-Ready Frontend**: React/Next.js interface with real-time analytics and vault pages
- ✅ **Complete AI Service Deployment**: All 7 core AI services deployed and operational on production
- ✅ **Enhanced ML Pipeline**: Advanced feature engineering and performance tracking
- ✅ **Uniswap V4 Hook Infrastructure**: Complete V4 development environment with AI-powered hooks
- 🔄 **Dynamic Fee Management System**: Advanced volatility-based fee optimization (0.01%-100% range)

## Key Commands

### Development Environment

```bash
# Setup Python environment (Backend)
python -m venv env
source env/bin/activate  # Linux/Mac
pip install -r dexter-liquidity/requirements.txt

# Setup Frontend
cd frontend
npm install
npm run dev  # Development server

# Setup Full Stack
npm run dev  # Frontend (port 3000)
python -m main  # Backend (from dexter-liquidity/)
```

### Running the Application

```bash
# Production with Docker
sudo docker compose build
sudo docker compose up -d

# Development - Backend
cd dexter-liquidity
python -m main

# Development - Frontend  
cd frontend
npm run dev

# Run DexBrain separately
cd backend
python -m dexbrain.core
```

### Testing

```bash
# Run all tests
pytest

# Run unit tests only
pytest tests/unit/

# Run integration tests
pytest tests/integration/

# Run tests with coverage
pytest --cov
```

### Code Quality

```bash
# Format code with Black
black .

# Sort imports
isort .

# Type checking
mypy .
```

## Architecture Overview

### Core Components

1. **Position Management System** (`contracts/`, `frontend/components/`):
   - `DexterCompoundor.sol`: Core auto-compounding contract
   - `DexterVault.sol`: ERC4626 vault implementation with Gamma-inspired strategies
   - `VaultFactory.sol`: Template-based factory for vault deployment
   - `PositionManager.tsx`: Frontend position management interface
   - **Features**: 200 positions per address, 50 positions per batch, up to 10 ranges per vault

2. **AI Agents** (`dexter-liquidity/agents/`): Risk-based trading strategies
   - `conservative.py`: Low-risk strategy ($100k min liquidity, 15% max volatility)
   - `aggressive.py`: Medium-risk strategy ($50k min liquidity, 30% max volatility)
   - `hyper_aggressive.py`: High-risk strategy ($25k min liquidity, no volatility limit)

3. **Data Infrastructure** (`dexter-liquidity/data/`): Multi-layered data collection
   - 4-dimensional data quality monitoring (completeness, accuracy, consistency, timeliness)
   - Historical backfill service (45-180 records/minute capacity)
   - Automated gap detection and healing workflows

4. **ML Pipeline** (`backend/dexbrain/`, `/opt/dexter-ai/`): **COMPLETE** integrated intelligence system
   - **Production Deployment**: Running as `dexter-ml-pipeline.service` with 4 trained models
   - **Continuous Learning**: Auto-retraining every 30 minutes with performance tracking
   - LSTM price prediction with PyTorch
   - Real-time strategy optimization with market regime detection
   - MLflow experiment tracking and model versioning

5. **Frontend Dashboard** (`frontend/`): **PRODUCTION-READY** web interface
   - Complete vault routing: `/vaults`, `/vaults/create`, `/vaults/[address]`
   - Real-time analytics with APR, fees, impermanent loss tracking
   - AI-managed vs manual position filtering
   - Template-based vault creation workflow

6. **Uniswap V4 Hook System** (`dexter-liquidity/uniswap-v4/`): Next-generation liquidity optimization
   - `SimpleDexterHook.sol`: Production-ready V4 hook with dynamic fee management
   - Dynamic fee adjustment (0.01%-100%) based on volatility
   - ML prediction integration for real-time market regime detection
   - Gas-optimized algorithms with modular design

### Key Design Patterns

- **AI-First Architecture**: Every component designed with ML optimization in mind
- **ERC4626 Vault Standard**: Standard vault interfaces for institutional DeFi integration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MeltedMindz/Dexter](https://github.com/MeltedMindz/Dexter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
