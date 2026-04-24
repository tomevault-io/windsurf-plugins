---
trigger: always_on
description: > Free, real-time cryptocurrency news API aggregating 200+ sources. No API key required.
---

# free-crypto-news — AI Agent Integration Guide

> Free, real-time cryptocurrency news API aggregating 200+ sources. No API key required.
> REST JSON, RSS/Atom, WebSocket, GraphQL, AI analysis, MCP server, ChatGPT plugin.
> Open source (MIT). https://cryptocurrency.cv

---

## Quick Start for Agents

```bash
# Latest news (no auth needed)
curl https://cryptocurrency.cv/api/news

# Search
curl "https://cryptocurrency.cv/api/search?q=bitcoin+ETF"

# AI briefing
curl https://cryptocurrency.cv/api/ai/brief

# Sentiment
curl "https://cryptocurrency.cv/api/sentiment?asset=BTC"
```

---

## Discovery Endpoints

| Endpoint | URL | Purpose |
|----------|-----|---------|
| **LLM docs (compact)** | https://cryptocurrency.cv/llms.txt | Concise API reference for LLMs |
| **LLM docs (full)** | https://cryptocurrency.cv/llms-full.txt | Complete inline reference with schemas |
| **OpenAPI 3.1 spec** | https://cryptocurrency.cv/api/openapi.json | Machine-readable API specification |
| **ChatGPT plugin** | https://cryptocurrency.cv/.well-known/ai-plugin.json | OpenAI plugin manifest |
| **Agent card** | https://cryptocurrency.cv/.well-known/agent.json | Standard agent discovery metadata |
| **AI permissions** | https://cryptocurrency.cv/ai.txt | AI crawling/training permissions |
| **x402 discovery** | https://cryptocurrency.cv/.well-known/x402 | Autonomous payment endpoints |
| **Health check** | https://cryptocurrency.cv/api/health | API status and uptime |

---

## MCP Server (Model Context Protocol)

Install for Claude Desktop, Claude Code, or any MCP-compatible client:

```bash
npx -y @smithery/cli install cryptocurrency-news
```

Or configure manually:

```json
{
  "mcpServers": {
    "crypto-news": {
      "command": "npx",
      "args": ["-y", "@anthropic-ai/mcp-server-crypto-news"]
    }
  }
}
```

### MCP Tools (20)

| Tool | Description |
|------|-------------|
| `get_crypto_news` | Latest news from 200+ sources (category, limit, ticker) |
| `search_crypto_news` | Full-text search across archive |
| `get_bitcoin_news` | Bitcoin-specific news |
| `get_defi_news` | DeFi protocol news |
| `get_breaking_news` | Articles from last 2 hours |
| `get_market_data` | Prices, market caps, 24h change |
| `get_fear_greed_index` | Crypto Fear & Greed Index |
| `get_gas_prices` | Ethereum gas prices |
| `get_regulatory_news` | Regulatory and legal updates |
| `get_whale_alerts` | Large on-chain transactions |
| `get_funding_rates` | Perpetual futures funding rates |
| `get_liquidations` | Recent liquidation events |
| `get_defi_yields` | DeFi protocol yield rates |
| `get_ai_market_brief` | AI-generated market summary |
| `compare_coins` | Compare multiple coins |
| `get_exchange_flows` | Exchange inflow/outflow data |
| `get_token_unlocks` | Upcoming token unlock schedules |
| `get_social_sentiment` | Social media sentiment for a coin |
| `get_news_sources` | List all 200+ news sources |
| `get_trending_topics` | Trending keywords and topics |

All tools are annotated with `readOnlyHint: true` — no confirmation prompts needed.

---

## ChatGPT Plugin

Manifest: `https://cryptocurrency.cv/.well-known/ai-plugin.json`
OpenAPI: `https://cryptocurrency.cv/api/openapi.json`

### Available Actions

| Action | Endpoint | Description |
|--------|----------|-------------|
| `getLatestNews` | `GET /api/news` | Latest crypto news |
| `searchNews` | `GET /api/search` | Search by keywords |
| `getDefiNews` | `GET /api/defi` | DeFi-specific news |
| `getBitcoinNews` | `GET /api/bitcoin` | Bitcoin-specific news |
| `getBreakingNews` | `GET /api/breaking` | Last 2 hours |
| `getSources` | `GET /api/sources` | All 200+ sources |

---

## Core API Endpoints

### News

| Endpoint | Description |
|----------|-------------|
| `GET /api/news` | Latest articles (limit, source, page, category, lang) |
| `GET /api/search?q={query}` | Full-text search |
| `GET /api/search/semantic?q={query}` | Semantic vector search |
| `GET /api/bitcoin` | Bitcoin news |
| `GET /api/defi` | DeFi news |
| `GET /api/breaking` | Last 2 hours |
| `GET /api/trending` | Trending keywords |
| `GET /api/sources` | All sources with status |
| `GET /api/rss` | RSS 2.0 feed |
| `GET /api/atom` | Atom 1.0 feed |

### AI & Intelligence

| Endpoint | Description |
|----------|-------------|
| `POST /api/ai` | Summarize, sentiment, facts, factcheck, translate |
| `GET /api/ask?q={question}` | Natural language Q&A |
| `GET /api/ai/brief` | AI market briefing |
| `GET /api/ai/research?q={topic}` | Deep research mode |
| `GET /api/ai/narratives` | Emerging market narratives |
| `GET /api/ai/synthesize` | Multi-article synthesis |
| `GET /api/ai/explain` | Plain-English explainer |
| `GET /api/ai/correlation` | News-price correlation |
| `GET /api/rag/ask?q={question}` | RAG-powered Q&A |
| `GET /api/rag/search?q={query}` | Semantic archive search |
| `GET /api/rag/stream?q={query}` | Streaming RAG (SSE) |

### Market Data

| Endpoint | Description |
|----------|-------------|
| `GET /api/market/coins` | Full coin list with prices |
| `GET /api/sentiment?asset={ticker}` | AI sentiment analysis |
| `GET /api/fear-greed` | Fear & Greed Index |
| `GET /api/prices?coins={list}` | Current prices |
| `GET /api/gas` | Ethereum gas prices |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nirholas/cryptocurrency.cv](https://github.com/nirholas/cryptocurrency.cv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
