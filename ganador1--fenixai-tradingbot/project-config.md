---
trigger: always_on
description: FenixAI employs a **multi-agent architecture** where specialized AI agents collaborate to analyze cryptocurrency markets. Each agent is an expert in a specific domain, and their outputs are combined through a weighted consensus mechanism.
---

# 🤖 FenixAI v2.0 - Agent System

## Overview

FenixAI employs a **multi-agent architecture** where specialized AI agents collaborate to analyze cryptocurrency markets. Each agent is an expert in a specific domain, and their outputs are combined through a weighted consensus mechanism.

## Agent Hierarchy

```
                    ┌─────────────────────────────┐
                    │     LangGraph Orchestrator   │
                    │      (State Machine)         │
                    └─────────────┬───────────────┘
                                  │
        ┌─────────────────────────┼─────────────────────────┐
        │                         │                         │
        ▼                         ▼                         ▼
┌───────────────┐       ┌───────────────┐       ┌───────────────┐
│   Technical   │       │    Visual     │       │   Sentiment   │
│    Agent      │       │    Agent      │       │    Agent      │
│  (30% weight) │       │  (25% weight) │       │  (15% weight) │
└───────┬───────┘       └───────┬───────┘       └───────┬───────┘
        │                       │                       │
        └───────────────────────┼───────────────────────┘
                                │
                    ┌───────────▼───────────┐
                    │     QABBA Agent       │
                    │    (30% weight)       │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │    Decision Agent     │
                    │  (Weighted Consensus) │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │    Risk Manager       │
                    │   (Final Approval)    │
                    └───────────────────────┘
```

---

## 1. Technical Analyst Agent

**File:** `src/agents/enhanced_technical_analyst.py`

### Purpose

Analyzes market structure, technical indicators, and price patterns to generate trading signals.

### Capabilities

- **Indicator Analysis**: RSI, MACD, ADX, ATR, Bollinger Bands
- **Market Structure**: Trend detection, support/resistance levels
- **Multi-timeframe**: Analyzes multiple timeframes for confluence
- **Validation**: Checks indicator consistency before signaling

### Input

```python
{
    "kline_data": {"open": [...], "high": [...], "low": [...], "close": [...], "volume": [...]},
    "indicators": {"rsi": 45.2, "macd": {...}, "adx": 28.5, ...},
    "mtf_context": {"15m": {...}, "1h": {...}, "4h": {...}}
}
```

### Output (EnhancedTechnicalOutput)

```python
{
    "signal": "BUY" | "SELL" | "HOLD",
    "confidence": 0.75,  # 0.0 - 1.0
    "reasoning": "RSI showing oversold conditions with bullish divergence...",
    "confluence_score": 0.68,
    "entry_price": 42150.0,
    "stop_loss": 41800.0,
    "take_profit": 43200.0,
    "risk_reward_ratio": 2.5
}
```

### Configuration

```yaml
# config/fenix.yaml
agents:
  technical_weight: 0.30
  enable_technical: true
```

---

## 2. Visual Analyst Agent

**File:** `src/agents/visual_analyst_enhanced.py`

### Purpose

Analyzes TradingView chart screenshots to identify visual patterns that may not be captured by numerical indicators.

### Capabilities

- **Pattern Recognition**: Head & Shoulders, triangles, flags, wedges
- **Candlestick Analysis**: Doji, engulfing, hammer, shooting star
- **Trend Analysis**: Visual trend lines, channels
- **Indicator Reading**: Interprets SuperTrend, SAR points from chart
- **Security**: Validates chart paths to prevent stale data analysis

### Input

```python
{
    "chart_path": "/path/to/BTCUSDT_15m_chart.png",
    "symbol": "BTCUSDT",
    "timeframe": "15m"
}
```

### Output (EnhancedVisualChartAnalysisOutput)

```python
{
    "action": "BUY" | "SELL" | "HOLD",
    "confidence": 0.72,
    "reason": "Bullish engulfing pattern at key support level...",
    "key_candlestick_patterns": ["bullish_engulfing", "morning_star"],
    "chart_patterns": ["ascending_triangle"],
    "trend_analysis": {"direction": "BULLISH", "strength": 0.7},
    "support_resistance_levels": {"support": [41500, 41000], "resistance": [43000, 44500]},
    "next_candle_prediction": {"direction": "UP", "probability": 0.65}
}
```

### Requirements

- Vision-capable LLM (qwen3-vl:8b or similar)
- TradingView chart screenshots via Playwright

---

## 3. Sentiment Analyst Agent

**File:** `src/agents/sentiment_enhanced.py`

### Purpose

Analyzes news articles, social media, and market sentiment to gauge market psychology.

### Capabilities

- **News Analysis**: Crypto news from multiple sources
- **Social Sentiment**: Twitter/X, Reddit analysis
- **Fear & Greed**: Interprets market sentiment indices
- **Event Detection**: Identifies market-moving events

### Input

```python
{
    "news_data": [
        {"title": "Bitcoin ETF sees record inflows", "source": "coindesk", "sentiment": "positive"},
        ...
    ],
    "social_data": {"twitter": {...}, "reddit": {...}},
    "fear_greed_index": 65
}
```

### Output

```python
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ganador1/FenixAI_tradingBot](https://github.com/Ganador1/FenixAI_tradingBot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
