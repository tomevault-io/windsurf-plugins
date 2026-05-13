---
trigger: always_on
description: This repository contains 18 professional investment analysis frameworks for US stock markets. When working in this workspace, Copilot automatically loads these analysis methodologies to provide institutional-quality investment analysis.
---

# InvestSkill — GitHub Copilot Setup & Usage Guide

This repository contains 18 professional investment analysis frameworks for US stock markets. When working in this workspace, Copilot automatically loads these analysis methodologies to provide institutional-quality investment analysis.

## Installation & Setup

### Automatic Setup

GitHub Copilot automatically loads this file when you work in the InvestSkill repository. **No manual installation required.**

1. **Clone the repository:**
   ```bash
   git clone https://github.com/yennanliu/InvestSkill.git
   cd InvestSkill
   ```

2. **Open in your editor:**
   - **VS Code**: `code .`
   - **JetBrains IDE** (IntelliJ, PyCharm, etc.): Open the folder
   - **GitHub.com Web Editor**: Click `.` key in any GitHub repo view

3. **Open Copilot Chat** (usually `Ctrl+K` or `Cmd+K`)
   - Copilot automatically loads `copilot-instructions.md`
   - You now have access to all 18 analysis frameworks

### Verify Setup

In Copilot Chat, try:
```
Use the stock-valuation framework to analyze Apple (AAPL)
```

If Copilot references the frameworks, setup is complete!

---

## Available Analysis Frameworks (20 Total)

### Core Stock Analysis (6 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Stock Evaluation | `prompts/stock-eval.md` | Comprehensive stock quality scoring |
| Stock Valuation | `prompts/stock-valuation.md` | Multi-method valuation (DCF + comps + EV) |
| Fundamental Analysis | `prompts/fundamental-analysis.md` | Deep financial statement dive |
| Technical Analysis | `prompts/technical-analysis.md` | Chart patterns and indicators |
| DCF Valuation | `prompts/dcf-valuation.md` | Intrinsic value modeling |
| Economics Analysis | `prompts/economics-analysis.md` | Macro outlook and recession risk |

### Financial Report Analysis (2 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Financial Report Analyst | `prompts/financial-report-analyst.md` | 10-K/10-Q analysis |
| Earnings Call Analysis | `prompts/earnings-call-analysis.md` | Management tone and guidance |

### Market Monitoring (4 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Insider Trading | `prompts/insider-trading.md` | Form 4 insider activity tracking |
| Institutional Ownership | `prompts/institutional-ownership.md` | 13F smart money moves |
| Dividend Analysis | `prompts/dividend-analysis.md` | Dividend safety and sustainability |
| Short Interest | `prompts/short-interest.md` | Squeeze potential and bearish thesis |

### Advanced Analysis (4 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Competitor Analysis | `prompts/competitor-analysis.md` | Moat and competitive positioning |
| Options Analysis | `prompts/options-analysis.md` | Greeks, strategy selection, earnings plays |
| Portfolio Review | `prompts/portfolio-review.md` | Allocation optimization |
| Sector Analysis | `prompts/sector-analysis.md` | Sector rotation opportunities |

### Full Research Bundle (2 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Research Bundle | `prompts/research-bundle.md` | Comprehensive multi-framework analysis |
| (Combined) | (all frameworks) | Complete investment thesis |

### Meta-Analysis & Visualization (2 frameworks)

| Framework | File | Best For |
|-----------|------|----------|
| Result Validator | `prompts/result-validator.md` | Validate any analysis + confidence score (0–100) |
| Chart Master | `prompts/chart-master.md` | Generate Mermaid/ASCII/HTML charts from financial data |

---

## Usage Examples

### Natural Language Queries

Copilot understands the frameworks context naturally:

```
# Evaluate a stock
Perform a stock evaluation of Microsoft using the Piotroski F-Score methodology

# Valuation analysis
What's a fair valuation for Apple using DCF, comparable companies, and EV multiples?

# Financial statement analysis
Deep dive into Tesla's balance sheet and cash flow quality

# Market context
What's the current economic outlook? Is recession likely?

# Technical setup
What are the key technical levels for NVDA?

# Combined analysis
Analyze Nvidia from both fundamental and technical perspectives
```

### Explicit Framework References

Reference specific prompt files for precise methodology:

```
# Stock valuation with all methods
Use the framework in prompts/stock-valuation.md to analyze AAPL

# Fundamental deep dive
Apply prompts/fundamental-analysis.md to Microsoft's financials

# Technical analysis
Reference prompts/technical-analysis.md for Tesla chart patterns

# Financial report analysis
Use prompts/financial-report-analyst.md to analyze this 10-K filing: [paste text]

# Earnings transcript analysis
Apply prompts/earnings-call-analysis.md to this earnings call: [paste transcript]
```

### With Financial Data

```
# Paste financial statements
Use the fundamental-analysis framework to analyze these financial statements:
[paste balance sheet, income statement, cash flow]

# Paste SEC filings
Use the financial-report-analyst framework on this 10-Q:
[paste 10-Q text]


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yennanliu/InvestSkill](https://github.com/yennanliu/InvestSkill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
