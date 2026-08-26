---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an enhanced ETF Sharpe ratio optimization system (增强版ETF投资组合优化系统) that provides comprehensive quantitative investment decision support. It uses Tushare API to fetch ETF data and implements multiple optimization strategies including Sharpe ratio maximization, risk parity, and multi-objective optimization. The system features ETF Chinese name support, complex growth prediction, and professional HTML report generation. Written in Python and targets Chinese-speaking quantitative finance community.

## Latest Enhancements (v2.3.0 - Code Refactoring)

### Major Refactoring
- **Unified Optimization Engine**: Merged CVXPY and SciPy portfolio optimizers into a single module with automatic backend selection
- **Unified Quantitative Signals**: Integrated simple and advanced quantitative signal modules with mode switching support
- **Code Optimization**: Reduced from 21 files to 19 files (-9.5%) and 10,047 lines to 9,564 lines (-4.8%)
- **Eliminated Redundancy**: Removed duplicate modules, backup files, and streamlined imports
- **Architecture Enhancement**: Improved error handling, backward compatibility, and maintainability

### Technical Improvements
- **Smart Backend Selection**: Automatic detection and fallback between CVXPY and SciPy optimizers
- **Mode-based Signal Processing**: Support for simple/advanced/auto modes in quantitative signal generation
- **Enhanced Error Handling**: Robust fallback mechanisms and comprehensive exception handling
- **Streamlined Dependencies**: Simplified import structure and reduced module interdependencies
- **Code Quality**: Enhanced readability, maintainability, and architectural consistency

### Verified Functionality
- **Full Testing**: Successfully tested in conda environment with all core features working
- **Backward Compatibility**: All original functionality preserved with consistent APIs
- **Performance**: Intelligent backend selection improves computational efficiency
- **Robustness**: Enhanced error handling ensures system stability

## Previous Enhancements (v2.2.0)

### New Features
- **Quantitative Signal Analysis System**: Multi-dimensional quantitative indicators calculation and composite signal synthesis
- **Enhanced Portfolio Optimization**: Intelligent optimization engine based on quantitative signals
- **Chinese Font Configuration Optimization**: Centralized font management and better Chinese display support
- **Simplified Module Support**: Added simplified versions of quantitative signals and optimizer modules as fallback
- **Professional HTML Reports Enhancement**: New quantitative signal analysis and enhanced optimization comparison modules
- **Methodology Detailed Introduction**: Complete methodology documentation for quantitative investment and enhanced optimization

### Technical Improvements
- **Font Management**: Centralized font configuration module (`font_config.py`) for consistent Chinese character display
- **Signal Processing**: Advanced quantitative indicators including momentum, volatility, trend, and quality signals
- **Enhanced Optimization**: Traditional optimization enhanced with quantitative signal integration
- **Robust Architecture**: Simplified modules as fallback for enhanced functionality
- **Visualization Improvements**: Better Chinese font support in all charts and visualizations

## Previous Enhancements (v2.1.0)

### New Features
- **ETF Chinese Name Support**: Automatically fetches and displays ETF Chinese names in all outputs
- **Complex Growth Prediction**: Monte Carlo-based growth forecasting with probability distributions, scenario analysis, and multi-year projections
- **Professional HTML Reports**: Comprehensive HTML reports with interactive charts and detailed analysis
- **Correlation Analysis**: Portfolio correlation risk assessment and diversification scoring
- **Enhanced Visualization**: All charts and reports support ETF Chinese name display

## Key Architecture

### Main Components

- **main.py**: Entry point containing the `EnhancedETFSharpeOptimizer` class that orchestrates the entire workflow
- **src/**: Modular source code directory with clear separation of concerns:

#### Core Modules
- `config.py`: Configuration management (handles config.json and environment variables)
- `data_fetcher.py`: Tushare API integration for fetching ETF daily price data with Chinese name support
- `data_processor.py`: Data processing and returns calculation
- `portfolio_optimizer.py`: **🔧 Unified portfolio optimization engine** (supports both CVXPY and SciPy with automatic backend selection)
- `evaluator.py`: Portfolio performance evaluation with 8+ key financial metrics
- `visualizer.py`: Chart generation (4+ visualization types) with Chinese name support
- `utils.py`: Utility functions for logging, timing, and result saving

#### Enhanced Modules (v2.0+)
- `multi_objective_optimizer.py`: Multi-objective optimization engine with 4 strategies
- `risk_manager.py`: Advanced risk management (VaR/CVaR, stress testing, concentration analysis)
- `rebalancing_engine.py`: Dynamic rebalancing strategies with transaction cost optimization

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StanleyChanH/SharpETF](https://github.com/StanleyChanH/SharpETF) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
