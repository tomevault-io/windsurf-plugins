---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**eterniscollab** is a forecasting and LLM analysis toolkit. The project provides tools for:
- Generating probability distributions from LLM responses
- Analyzing "buzz" scores (interest × divisiveness) for topics
- Generating forecasting questions from trending news
- Comparing different LLM providers (OpenAI, Claude) via OpenRouter API
- Downloading and analyzing historical prediction market data from Polymarket

## Project Structure

```
eterniscollab/
├── Core Library Modules (use via import)
│   ├── buzz.py                      # Buzz score analysis (interest + divisiveness)
│   ├── probability_estimator.py     # Probability distribution estimation via LLMs
│   ├── generate_topic_rankings.py   # Generate and rank topics by interest/buzz
│   ├── topic_generator.py           # Generate trending topics and forecasting questions
│   ├── utils.py                     # Shared utility functions for LLM API calls
│   └── polymarket_data.py           # Polymarket historical data downloader
│
├── Configuration & Dependencies
│   ├── requirements.txt             # Python dependencies
│   ├── pytest.ini                   # Pytest configuration
│   ├── README.md                    # Main project README
│   ├── CLAUDE.md                    # This file - project guidance for Claude Code
│   └── notes.md                     # Development notes
│
├── scripts/                         # Command-line scripts and examples
│   ├── Example Scripts
│   │   ├── topic_generator_example.py
│   │   ├── example_openrouter.py
│   │   ├── example_presidential_election.py
│   │   ├── example_download_by_slug.py
│   │   ├── example_improved_workflow.py
│   │   ├── closed_markets_example.py
│   │   ├── market_history_example.py
│   │   └── probability_evolution_example.py
│   ├── Utility Scripts
│   │   ├── verify_date_filtering.py    # Verify date filtering works correctly
│   │   ├── fix_arrow_error.py          # Fix pandas/pyarrow compatibility issues
│   │   └── fix_jupyter_dependencies.py # Check and fix Jupyter dependencies
│   ├── Bulk Download Scripts
│   │   ├── download_selected_markets.py
│   │   ├── download_selected_markets_volume1wk.py
│   │   └── market_history_downloader.py
│   └── test_probability_evolution.py   # Standalone test for probability evolution
│
├── docs/                            # Documentation
│   ├── INDEX.md                     # Documentation index
│   ├── POLYMARKET_README.md         # Complete Polymarket API documentation
│   ├── QUICK_START_POLYMARKET.md    # Quick reference for Polymarket data
│   ├── API_LIMITS_EXPLANATION.md    # Explains 15-day API limit and auto-chunking
│   ├── FIDELITY_EXPLANATION.md      # Understanding fidelity parameter
│   ├── TOPIC_GENERATOR_README.md    # Topic generator guide
│   ├── UTILS_README.md              # Utilities documentation
│   ├── OPENROUTER_SETUP.md          # OpenRouter API setup
│   └── ...                          # Additional documentation files
│
├── notebooks/                       # Jupyter notebooks for analysis
│   ├── probability_distribution_analysis.ipynb
│   ├── buzz_distribution_analysis.ipynb
│   ├── probability_evolution.ipynb
│   ├── polymarket_exploration.ipynb
│   └── CLOB.ipynb
│
├── tests/                           # Unit tests (pytest)
│   ├── test_topic_generator.py
│   ├── test_closed_markets.py
│   ├── test_polymarket_download.py
│   ├── test_probability.py
│   ├── test_buzz.py
│   ├── test_reword.py
│   ├── test_utils.py
│   └── test_openrouter.py
│
└── data/                            # Data directory (git-ignored)
    └── polymarket/                  # Cached Polymarket price data (parquet files)
```

## Core Modules

### 1. **probability_estimator.py**
- **Purpose**: Estimate probability distributions by querying LLMs multiple times with reworded prompts
- **Key Functions**:
  - `get_probability_distribution()`: Main async function for collecting probability samples
  - `get_probability_distribution_over_time()`: **NEW** Track how probability estimates evolve with different knowledge cutoff dates
  - `analyze_probability_evolution()`: Helper function to compute statistics from time series results
  - `reword_prompt()`: Reword prompts with variable flexibility (temperature-controlled)
  - `query_probability()`: Get single probability estimate
- **Technology**: Uses Pydantic AI with OpenRouter API
- **Models Supported**: Any OpenRouter model (openai/gpt-4o-mini, anthropic/claude-sonnet-4, etc.)
- **New Feature**: Time-series analysis of forecast evolution (see PROBABILITY_EVOLUTION_README.md)

### 2. **buzz.py**
- **Purpose**: Calculate "buzz scores" for topics based on interest and divisiveness
- **Key Functions**:
  - `get_buzz_score()`: Calculate combined buzz score (interest × divisiveness)
  - `get_buzz_score_openrouter()`: OpenRouter version
  - `query_interest()`, `query_interest_openrouter()`: Get interest scores
  - `query_divisiveness()`, `query_divisiveness_openrouter()`: Get divisiveness scores
- **Formula**: Buzz = Interest × Divisiveness (both 0-1)
- **Use Case**: Identify topics that are both interesting AND divisive


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/williamcottrell72) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
