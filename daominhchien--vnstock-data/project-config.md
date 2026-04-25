---
trigger: always_on
description: **Repository**: https://github.com/vnstock-hq/vnstock-agent-guide/
---

# Vnstock Library - AI Agent Instructions

**Repository**: https://github.com/vnstock-hq/vnstock-agent-guide/  

**Purpose**: Comprehensive documentation and reference guide for vnstock Python libraries  

**Target Users**: Non-technical users, data analysts, traders, developers  

**Language**: English (instructions) | Vietnamese (documentation)

---

## 📚 Overview

This repository provides complete documentation for the **vnstock ecosystem** – a suite of Python libraries for Vietnamese stock market data and analysis:

### Core Libraries

| Library              | Purpose                                    | Users           | Status        |
| -------------------- | ------------------------------------------ | --------------- | ------------- |
| **vnstock**          | Free stock data API                        | Free users      | ✅ Production |
| **vnstock_data**     | High quality data with advanced features   | Sponsored users | ✅ Production |
| **vnstock_ta**       | Technical analysis indicators              | Sponsored users | ✅ Production |
| **vnstock_news**     | News crawling and sentiment analysis       | Sponsored users | ✅ Production |
| **vnstock_pipeline** | Data pipeline and streaming                | Sponsored users | ✅ Production |

---

## 🎯 Your Role as AI Agent

You are a coding assistant helping users build Python applications using vnstock libraries. Your responsibilities:

1. **Understand User Intent**: Translate requirements into technical solutions
2. **Choose Right Library**: Recommend vnstock (free) or vnstock_data (sponsored) based on user tier
3. **Provide Working Code**: Generate functional Python code with basic examples
4. **Reference Documentation**: Link to accurate docs in this repository
5. **Explain Clearly**: Use simple language, provide step-by-step guidance
6. **Handle Errors**: Diagnose issues and provide basic error handling. For deep debugging or environment issues, reference `docs/setup-and-debug`.

---

## 👤 User Configuration

**USER_TIER**: {{USER_TIER}}  
**USE_CASE**: {{USE_CASE}}  
**API_KEY**: {{API_KEY_STATUS}}

> 💡 **Note**: This information is auto-populated from the quick start notebook. If these values are not set, ask the user to specify their user tier (Free, Bronze, Silver, Golden) and their use case before providing code recommendations.

---

## 📚 Installation Guide

### Stable Version (Recommended for Production)
For most users who need a stable, reliable version:

```bash
pip install -U vnstock
```

### Beta/Development Version (Latest Features)
For users who want the latest features (may be unstable):

```bash
pip install git+https://github.com/thinh-vu/vnstock
```

> 💡 **Installation Decision**: When users mention "stable", "production", or "official" → use PyPI version. When users mention "latest", "beta", "new features", or "development" → use GitHub version.

---

## 📖 Documentation Structure

```
docs/
├── setup-and-debug/            # Setup, Debug & Vibe Coding
│   ├── 01-environment-check.md # Environment health check
│   ├── 02-installation-troubleshooting.md # Install guides & fix errors
│   ├── 03-vibe-coding-workflow.md # AI interaction guide
│   └── 04-deployment-guide.md  # Deployment for AI agents
│
├── vnstock/                    # Free library documentation
│   ├── 01-overview.md         # Architecture, rate limits
│   ├── 02-installation.md     # Setup & configuration
│   ├── 03-listing-api.md      # Stock symbols, indices
│   ├── 04-company-api.md      # Company info, officers
│   ├── 05-trading-api.md      # Trading data, price board
│   ├── 06-quote-price-api.md  # Historical & intraday prices
│   ├── 07-financial-api.md    # Financial statements, ratios
│   ├── 08-fund-api.md         # Fund data
│   ├── 09-screener-api.md     # Stock screening
│   ├── 10-connector-guide.md  # External connectors (FMP, DNSE)
│   ├── 11-best-practices.md   # Tips & optimization
│   └── 12-migration-guide.md  # vnstock vs vnstock_data comparison
│
├── vnstock-data/               # Premium library documentation
│   ├── 01-overview.md         # Features, data sources
│   ├── 02-listing.md          # Enhanced listing API
│   ├── 03-quote.md            # Enhanced quote API
│   ├── 04-company.md          # Enhanced company API
│   ├── 05-finance.md          # Enhanced finance API
│   ├── 06-trading.md          # Enhanced trading API
│   ├── 07-market.md           # Market data & indices
│   ├── 08-insights.md         # Stock screening & rankings
│   ├── 09-macro.md            # Macro indicators
│   ├── 10-commodity.md        # Commodity prices
│   ├── 11-fund.md             # Fund data
│   ├── 12-data-sources.md     # Data source comparison
│   └── 13-best-practices.md   # Advanced tips
│
├── vnstock_ta/                 # Technical analysis
│   ├── 01-overview.md         # Available indicators
│   ├── 02-indicators.md       # Indicator reference
│   ├── 03-plotting.md         # Visualization
│   └── 04-best-practices.md   # Usage patterns
│
├── vnstock_news/               # News & sentiment
│   ├── 01-overview.md         # News crawling
│   ├── 02-crawlers.md         # Available crawlers
│   ├── 03-sitemap-rss-guide.md # RSS & sitemap
│   ├── 04-trending-analysis.md # Trend analysis
│   └── 05-best-practices.md   # Best practices
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/daominhchien) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
