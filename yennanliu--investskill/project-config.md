---
trigger: always_on
description: Professional investment analysis prompts for Gemini CLI. 18 universal frameworks for comprehensive stock market evaluation.
---

# InvestSkill for Gemini CLI

Professional investment analysis prompts for Gemini CLI. 18 universal frameworks for comprehensive stock market evaluation.

**For multi-platform setup**, see [README.md](README.md)

---

## Quick Start

### Installation

Gemini CLI automatically loads `GEMINI.md` when you run it in the project directory.

```bash
# Navigate to InvestSkill
cd /path/to/InvestSkill

# Start Gemini CLI
gemini

# GEMINI.md auto-loaded!
```

### First Analysis

In Gemini CLI:

```
> @prompts/stock-eval.md Evaluate Apple (AAPL)
```

Or reference directly:

```
> @prompts/stock-valuation.md What's a fair valuation for Microsoft?
```

---

## All 18 Frameworks

### Core Analysis (6)
```
@prompts/stock-eval.md                  # Stock evaluation with quality scoring
@prompts/stock-valuation.md             # Multi-method valuation (DCF + comps + EV)
@prompts/fundamental-analysis.md        # Financial statement analysis
@prompts/technical-analysis.md          # Chart patterns and indicators
@prompts/dcf-valuation.md              # DCF intrinsic value model
@prompts/economics-analysis.md          # Macro indicators and recession risk
```

### Financial Reports (2)
```
@prompts/financial-report-analyst.md    # 10-K, 10-Q, annual report analysis
@prompts/earnings-call-analysis.md      # Earnings call sentiment and themes
```

### Market Monitoring (4)
```
@prompts/insider-trading.md             # SEC Form 4 insider activity
@prompts/institutional-ownership.md     # 13F institutional holdings tracking
@prompts/dividend-analysis.md           # Dividend safety and yield analysis
@prompts/short-interest.md              # Short squeeze potential
```

### Advanced Analysis (4)
```
@prompts/competitor-analysis.md         # Economic moat and Porter's Five Forces
@prompts/options-analysis.md            # Options strategy selection
@prompts/portfolio-review.md            # Portfolio allocation and optimization
@prompts/sector-analysis.md             # Sector rotation opportunities
```

### Comprehensive Research (2)
```
@prompts/research-bundle.md             # All frameworks combined
```

---

## Usage Examples

### Basic Stock Analysis

```
# Stock evaluation
> @prompts/stock-eval.md Evaluate Apple (AAPL) using Piotroski F-Score

# Valuation analysis
> @prompts/stock-valuation.md What's a fair valuation for Microsoft?

# Fundamental deep dive
> @prompts/fundamental-analysis.md Deep analysis of NVIDIA's financials

# Technical setup
> @prompts/technical-analysis.md What are Tesla's key technical levels?

# Economic context
> @prompts/economics-analysis.md Is a recession likely in the next 12 months?
```

### Financial Data Analysis

**Analyze 10-K filing:**
```
> @prompts/financial-report-analyst.md
[paste 10-K text here]

Extract key accounting red flags and management quality indicators
```

**Analyze earnings transcript:**
```
> @prompts/earnings-call-analysis.md
[paste earnings call transcript]

What's the management tone and guidance outlook?
```

**Review portfolio:**
```
> @prompts/portfolio-review.md
AAPL: 30%
MSFT: 25%
NVDA: 20%
JNJ: 15%
TSLA: 10%

Is my allocation optimal?
```

### Stock Comparison

```
# Compare valuations
> @prompts/stock-valuation.md Compare AAPL vs MSFT valuations

# Compare fundamentals
> @prompts/fundamental-analysis.md Compare AAPL and MSFT balance sheets

# Multi-stock research
> @prompts/research-bundle.md Provide comprehensive analysis and comparison:
- AAPL
- MSFT
- GOOGL
```

### Specialized Analysis

```
# Dividend safety
> @prompts/dividend-analysis.md Is the JNJ dividend safe?

# Short squeeze
> @prompts/short-interest.md What's the short squeeze potential in GME?

# Options strategy
> @prompts/options-analysis.md Find bullish option strategies for AAPL earnings

# Competitive moat
> @prompts/competitor-analysis.md Does Apple have a defensible moat?

# Insider activity
> @prompts/insider-trading.md What are insiders buying at Tesla?

# Smart money tracking
> @prompts/institutional-ownership.md Which institutions are rotating into tech?

# Sector opportunities
> @prompts/sector-analysis.md Which sectors should I rotate into?
```

### Full Research Bundle

```
# Complete analysis (all frameworks)
> @prompts/research-bundle.md Provide complete analysis on Apple (AAPL)

# Quick version
> @prompts/research-bundle.md Quick analysis of Microsoft (MSFT)

# Multi-stock comparison
> @prompts/research-bundle.md Compare AAPL, MSFT, and GOOGL comprehensively
```

---

## Features

✅ **18 Professional Analysis Frameworks**
- Universal prompts (AI-agnostic)
- No Gemini-specific syntax needed
- Works with any AI model via Gemini CLI

✅ **File Reference Support**
- Use `@prompts/<name>.md` syntax
- Auto-loads from project directory
- Supports all 18 analysis types

✅ **Financial Data Integration**
- Paste SEC filings (10-K, 10-Q)
- Paste earnings transcripts
- Paste financial statements
- Reference documents inline

✅ **Session Memory**
- Gemini CLI remembers conversation context
- Ask follow-up questions naturally
- Multi-turn analysis chains

✅ **Model Agnostic**
- Works with all Gemini models
- Works with any Claude model
- Works with any AI accessible via Gemini CLI

---

## Installation & Troubleshooting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yennanliu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
