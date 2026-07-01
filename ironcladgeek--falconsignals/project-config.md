---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🎯 CRITICAL: DRY Principle (Don't Repeat Yourself)

**MANDATORY FOR ALL DEVELOPMENT**: This project MUST follow the DRY principle rigorously.

**Definition**: Every piece of knowledge should have a single, unambiguous representation in the system.

**Current Status** ⚠️:
- The system currently has **TWO SEPARATE EXECUTION PATHS** for LLM and rule-based modes (documented in `docs/ARCHITECTURE_ANALYSIS.md`)
- This is a **KNOWN VIOLATION** that must be eliminated

**Requirements for New Code**:
1. ✅ **Single execution path** - All analysis modes must converge to one pipeline
2. ✅ **No duplicate logic** - If code exists elsewhere, reuse it
3. ✅ **Single source of truth** - One location for each piece of functionality
4. ✅ **Before writing new code**, check if similar functionality already exists

**Before Committing, Ask**:
- Does this logic already exist elsewhere?
- Can I reuse an existing component?
- Will future changes require modifying code in multiple places?

If YES to questions 1 or 3 → **REFACTOR FIRST, then add feature**

See `docs/roadmap.md` and `docs/ARCHITECTURE_ANALYSIS.md` for detailed DRY violations and refactoring plan.

---

## Project Overview

**FalconSignals** is an AI-powered financial analysis and investment recommendation system that generates daily investment signals and portfolio allocation suggestions. It uses a multi-agent CrewAI architecture to analyze global financial markets across fundamental, technical, and sentiment dimensions.

**Key characteristics:**
- Cost-conscious design: Target €50-90/month operational cost
- Multi-agent AI orchestration using CrewAI
- Dual analysis modes: LLM-powered and Rule-based
- Runs once daily, completing analysis in <15 minutes
- Outputs: Markdown reports, JSON signals, portfolio recommendations
- All 6 phases complete: Full-featured production-ready system

## Essential Commands

### Development Workflow

```bash
# Install dependencies
uv sync

# Format and lint code (run before committing)
uv run poe lint

# Run all pre-commit hooks manually
uv run poe pre-commit

# Run tests
pytest

# Run single test file
pytest tests/path/to/test_file.py

# Run tests with specific pattern
pytest -k "test_pattern_name"
```

### Analysis Commands

```bash
# Quick test (rule-based)
uv run python -m src.main analyze --test

# Quick test (LLM mode)
uv run python -m src.main analyze --test --llm

# Analyze specific tickers
uv run python -m src.main analyze --ticker AAPL,MSFT,GOOGL

# Analyze with LLM
uv run python -m src.main analyze --ticker INTU --llm

# Analyze market/category
uv run python -m src.main analyze --market us --limit 20
uv run python -m src.main analyze --group us_tech_software
```

**Note**: When database is enabled (default), all analysis runs automatically:
- Create a run session to track the analysis
- Store investment signals immediately after creation (non-blocking)
- Enable partial report generation even if some tickers fail
- Track analysis metadata (mode, tickers, timestamps, status)

### Report Generation Commands

```bash
# Generate report from a specific analysis session (using session ID)
uv run python -m src.main report --session-id 1

# Generate report for all signals from a specific date
uv run python -m src.main report --date 2025-12-04

# Generate JSON report instead of markdown
uv run python -m src.main report --session-id 1 --format json

# Preview report without saving to file
uv run python -m src.main report --session-id 1 --no-save
```

**Database Benefits**:
- ✅ Reports generated even when some tickers fail (partial results saved)
- ✅ Historical report regeneration from stored data
- ✅ Audit trail of all analysis runs
- ✅ Foundation for performance tracking

### Performance Tracking Commands

```bash
# Track prices for active recommendations (run daily)
uv run python -m src.main track-performance

# Track with custom parameters
uv run python -m src.main track-performance --max-age 90  # Track up to 90 days old
uv run python -m src.main track-performance --signals buy,strong_buy,hold,hold_bullish  # Specify signal types
uv run python -m src.main track-performance --benchmark QQQ  # Use QQQ as benchmark

# Generate performance reports
uv run python -m src.main performance-report  # 30-day report (default)
uv run python -m src.main performance-report --period 90  # 90-day report
uv run python -m src.main performance-report --ticker AAPL  # Specific ticker
uv run python -m src.main performance-report --signal buy --mode llm  # Filtered
uv run python -m src.main performance-report --format json  # JSON output
```

**Performance Tracking Features**:
- ✅ Daily price tracking for active recommendations
- ✅ Benchmark comparison (default: SPY)
- ✅ Returns, win rate, alpha, Sharpe ratio calculation
- ✅ Confidence calibration analysis
- ✅ Multiple time periods (7, 30, 90, 180 days)
- ✅ **Bug Fixed**: Now correctly stores and tracks prices (see Bug Fixes section below)

**Important Notes**:
- Default signal filter is `buy,strong_buy` - add `hold,hold_bullish` if tracking those
- Requires valid `current_price` in recommendations (fixed in December 2025)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ironcladgeek/FalconSignals](https://github.com/ironcladgeek/FalconSignals) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
