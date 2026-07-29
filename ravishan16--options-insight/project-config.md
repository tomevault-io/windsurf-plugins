---
trigger: always_on
description: **Options Insight** is a serverless quantitative finance newsletter system built on Cloudflare Workers. It automatically scans earnings calendars, performs volatility analysis using real market data, generates AI-powered trading insights, and delivers professional newsletters via email.
---

# Options Insight - AI Coding Agent Instructions

## System Overview

**Options Insight** is a serverless quantitative finance newsletter system built on Cloudflare Workers. It automatically scans earnings calendars, performs volatility analysis using real market data, generates AI-powered trading insights, and delivers professional newsletters via email.

### Core Pipeline Architecture

The system follows a **7-stage deterministic pipeline** with comprehensive error handling and graceful degradation:

```
Environment → Data Init → Earnings Scan → Market Context → AI Analysis → Validation → Newsletter Delivery
```

Each stage tracks success/failure in a `summary` object and continues gracefully with fallbacks. **Reliability over features** - the newsletter always runs, even with degraded data sources.

## Essential Architectural Patterns

### 1. Multi-Source Data Pipeline with Graceful Degradation

The system uses a **simplified resilient data strategy** prioritizing proven free APIs:

```javascript
// Primary: Yahoo Finance (100% reliable for quotes + historical data)
// Fallback: Finnhub (reliable quotes, blocked historical on free tier)  
// Final: Estimated data (never fail the pipeline)

const provider = new SimplifiedDataProvider({
    finnhubApiKey: process.env.FINNHUB_API_KEY,
    requestDelay: 1200 // Respect rate limits
});
```

**Key Principle:** Never fail the entire pipeline due to one API error. Always provide fallback data.

### 2. Rate-Limited API Integration

Financial APIs have strict limits. The system uses explicit delays and comprehensive error handling:

```javascript
// Yahoo Finance + Finnhub strategy with 1.2s delays
for (const symbol of symbols) {
    const analysis = await provider.getVolatilityAnalysis(symbol);
    if (symbols.indexOf(symbol) < symbols.length - 1) {
        await new Promise(resolve => setTimeout(resolve, 1200));
    }
}
```

**Pattern:** Respect external API limits with explicit delays and retry logic.

### 3. Emoji-Prefixed Structured Logging

```javascript
console.log("🎯 Running Options Insight Research Agent...");
console.log("📊 Scanning earnings opportunities...");  
console.log("🤖 Generating AI analysis...");
console.log("📧 Sending newsletter...");
```

**Pattern:** Each component has emoji identifiers for easy visual parsing in production logs.

## Development Workflow

### CLI-First Component Testing

The project prioritizes **component isolation** via comprehensive CLI commands:

```bash
make test-finnhub      # Test earnings data fetching
make test-alphavantage # Test volatility analysis  
make test-gemini       # Test AI analysis generation
make test-email        # Test newsletter rendering
make test-full-run     # End-to-end pipeline simulation
make preview-email     # Local newsletter preview
```

**Development Rule:** Always test components individually before integration.

### Environment & Deployment

```bash
# Local development
make push-secrets      # Sync .env to Cloudflare Workers
make verify-deployment # Production health checks

# Testing new implementations  
node test-simplified.js    # Test data provider changes
node test-data-sources.js  # Compare API reliability
```

**Security:** Never commit API keys. Use `make push-secrets` for production deployment.

### API Security & CORS

The `/subscribe` endpoint implements pattern-based CORS validation:

```javascript
// Supports wildcard domains like *.pages.dev for preview environments
function resolveAllowedOrigin(origin, allowedOrigins) {
    for (const allowed of allowedOrigins) {
        if (allowed.includes('*')) {
            const pattern = allowed.replace(/[.+?^${}()|[\]\\]/g, '\\$&').replace(/\*/g, '.*');
            const regex = new RegExp(`^${pattern}$`, 'i');
            if (regex.test(origin)) return origin;
        }
    }
}
```

## Data Models & External APIs

### Stock Universe Filtering

Analysis is restricted to **liquid S&P 500 + NASDAQ 100 stocks** to ensure data quality:

```javascript
// src/config.js - Curated universe prevents analysis of illiquid names
export const STOCK_UNIVERSE = [
  'AAPL', 'MSFT', 'GOOGL', 'AMZN', 'NVDA', 'META', 'TSLA', // ...
];
```

### Financial Data Sources

- **Yahoo Finance:** Primary source (free, 100% reliable for quotes + historical data)
- **Finnhub:** Fallback quotes + earnings calendar (60 calls/min free tier)
- **Alpha Vantage:** Legacy fallback (25 calls/day, historical data requires premium)
- **Google Gemini:** AI analysis with structured validation
- **Resend:** Newsletter delivery with audience management

### Volatility Scoring System

Consistent scoring algorithm works with both real and estimated data:

```javascript
// src/real-volatility.js - unified scoring across all data sources
export function calculateVolatilityScore(volatilityData) {
    if (!volatilityData) return 0;
    
    const ivPercentile = volatilityData.impliedVolatilityPercentile || 50;
    const ivRank = volatilityData.impliedVolatilityRank || 0;
    const liquidity = volatilityData.optionsVolume || 0;
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ravishan16/options-insight](https://github.com/ravishan16/options-insight) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
