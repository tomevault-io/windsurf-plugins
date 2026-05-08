---
trigger: always_on
description: Investment analysis rules and frameworks for InvestSkill project
---


# InvestSkill — Cursor Rules & Setup Guide

This is the InvestSkill repository — a collection of 18 professional investment analysis frameworks for US stocks. These rules automatically apply when you work in this project.

## Installation & Setup

### Automatic Setup

Cursor automatically loads this file when you open the InvestSkill repository. **No manual installation required.**

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yennanliu/InvestSkill.git
   ```

2. **Open in Cursor:**
   ```bash
   cd InvestSkill
   cursor .
   ```

3. **Open AI Chat** (usually `Cmd+K`)
   - Cursor automatically loads the investment analysis rules
   - All 18 frameworks are immediately available

### Verify Setup

In Cursor AI Chat, try:
```
@prompts/stock-valuation.md Analyze Apple (AAPL)
```

If the framework loads, setup is complete!

---

## Available Analysis Frameworks (20 Total)

### Core Stock Analysis (6 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Stock Evaluation | `@prompts/stock-eval.md` | Piotroski scoring, quality metrics |
| Stock Valuation | `@prompts/stock-valuation.md` | DCF + comparable companies + EV multiples |
| Fundamental Analysis | `@prompts/fundamental-analysis.md` | Financial statement deep dive |
| Technical Analysis | `@prompts/technical-analysis.md` | Chart patterns, volume, indicators |
| DCF Valuation | `@prompts/dcf-valuation.md` | Intrinsic value modeling |
| Economics Analysis | `@prompts/economics-analysis.md` | Macro indicators, recession probability |

### Financial Report Analysis (2 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Financial Report Analyst | `@prompts/financial-report-analyst.md` | 10-K, 10-Q, annual reports |
| Earnings Call Analysis | `@prompts/earnings-call-analysis.md` | Management tone, guidance, themes |

### Market Monitoring (4 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Insider Trading | `@prompts/insider-trading.md` | Form 4 insider buying/selling |
| Institutional Ownership | `@prompts/institutional-ownership.md` | 13F holdings, smart money moves |
| Dividend Analysis | `@prompts/dividend-analysis.md` | Dividend safety, yield trap detection |
| Short Interest | `@prompts/short-interest.md` | Short squeeze scoring, thesis evaluation |

### Advanced Analysis (4 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Competitor Analysis | `@prompts/competitor-analysis.md` | Moat scoring, Porter's Five Forces |
| Options Analysis | `@prompts/options-analysis.md` | Greeks, IV rank, strategy selection |
| Portfolio Review | `@prompts/portfolio-review.md` | Allocation optimization, performance |
| Sector Analysis | `@prompts/sector-analysis.md` | Sector rotation, opportunity identification |

### Full Research Bundle (2 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Research Bundle | `@prompts/research-bundle.md` | Complete multi-framework analysis |

### Meta-Analysis & Visualization (2 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Result Validator | `@prompts/result-validator.md` | Validate any analysis + confidence score |
| Chart Master | `@prompts/chart-master.md` | Generate Mermaid/ASCII/HTML charts |

---

## Usage Examples

### Natural Language Queries

Cursor understands the investment context naturally:

```
# Evaluate a stock
Perform a comprehensive stock evaluation of Microsoft

# Get a valuation
What's a fair valuation for Apple using DCF and comparable companies?

# Technical analysis
Analyze Tesla's technical chart setup and key resistance levels

# Financial statement analysis
Deep dive into NVIDIA's balance sheet and cash flow quality

# Economics outlook
Is a recession likely in the next 12 months?

# Combined analysis
Analyze Apple from both fundamental and technical angles
```

### Explicit Framework References

Use `@prompts/` references for precise methodology:

```
# Stock valuation with all methods
@prompts/stock-valuation.md Analyze AAPL using all valuation methods

# Fundamental analysis
@prompts/fundamental-analysis.md Deep dive on Microsoft's fundamentals

# Technical setup
@prompts/technical-analysis.md What are the key chart levels for NVDA?

# Financial reports
@prompts/financial-report-analyst.md Analyze this 10-K filing: [paste text]

# Earnings transcript
@prompts/earnings-call-analysis.md What's the management tone here? [paste transcript]
```

### With Financial Data

```
# Paste balance sheet
@prompts/fundamental-analysis.md
[paste balance sheet, income statement, cash flow]

# Paste SEC filing
@prompts/financial-report-analyst.md
[paste 10-Q or annual report text]

# Paste earnings transcript
@prompts/earnings-call-analysis.md
[paste earnings call transcript]

# Portfolio holdings
@prompts/portfolio-review.md Review my portfolio:
AAPL: 30%
MSFT: 25%
NVDA: 20%
JNJ: 15%
TSLA: 10%
```

### Stock Comparison

```
# Compare valuations
@prompts/stock-valuation.md Compare AAPL vs MSFT valuations

# Compare fundamentals
@prompts/fundamental-analysis.md Compare AAPL and MSFT balance sheets

# Multi-stock analysis
@prompts/research-bundle.md Analyze AAPL, MSFT, and GOOGL
```

### Specialized Analysis

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yennanliu/InvestSkill](https://github.com/yennanliu/InvestSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
