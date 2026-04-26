---
trigger: always_on
description: A multi-agent system where specialized AI agents collaborate and debate on market analysis in real-time, simulating a hedge fund research team decision-making process.
---

# The Trading Floor — Multi-Agent Market Analysis Council

A multi-agent system where specialized AI agents collaborate and debate on market analysis in real-time, simulating a hedge fund research team decision-making process.

## Project Overview

This project creates a visual council of AI agents, each with distinct expertise, that analyze financial instruments and produce consensus-based trading recommendations. Agents engage in structured debate, challenge each other's assumptions, and synthesize diverse perspectives into actionable insights.

## Agent Framework

This project uses **CrewAI** as the primary agent orchestration framework.

### Why CrewAI

- Native support for role-based agent specialization with distinct backstories and goals
- Built-in task delegation and agent collaboration patterns
- Hierarchical process mode enables a "manager" agent to coordinate specialist agents
- Sequential and consensual process modes for structured debate workflows
- Strong integration with LangChain tools for extensibility
- Active community with extensive documentation available via Context7

### Alternative Consideration

LangGraph could be used if more granular control over agent state transitions and cyclic workflows is needed. However, CrewAI's higher-level abstractions better match the council/debate paradigm of this project.

## Technology Stack

### Backend (Python)

- **CrewAI** — Agent orchestration and multi-agent workflows
- **yfinance** — Market data retrieval (prices, fundamentals, historical data)
- **FastAPI** — REST API and WebSocket server for real-time streaming
- **Pydantic** — Data validation and structured output schemas
- **LangChain** — Tool creation and LLM integrations
- **Anthropic SDK** — Claude as the underlying LLM for all agents

### Frontend (Next.js)

- **Next.js 14+** — App router with server components
- **TypeScript** — Strict mode enabled throughout
- **Tailwind CSS** — Styling and responsive design
- **Framer Motion** — Agent conversation animations and transitions
- **Server-Sent Events (SSE)** — Real-time streaming of agent deliberations

### Infrastructure

- **Python 3.11+** — Backend runtime
- **Bun** — Frontend package management and runtime

## Project Structure

```
trading-floor/
├── backend/
│   ├── agents/           # Individual agent definitions
│   ├── crews/            # Crew configurations and workflows
│   ├── tools/            # Custom tools for market data and analysis
│   ├── schemas/          # Pydantic models for inputs/outputs
│   ├── api/              # FastAPI routes and WebSocket handlers
│   └── main.py           # Application entry point
├── frontend/
│   ├── app/              # Next.js app router pages
│   ├── components/       # React components for agent UI
│   ├── hooks/            # Custom hooks for SSE and state
│   ├── lib/              # Utility functions and API clients
│   └── types/            # TypeScript type definitions
├── shared/
│   └── types/            # Shared type definitions between frontend/backend
├── pyproject.toml
├── package.json
└── README.md
```

## Agent Definitions

### The Council Members

Each agent must be defined with a clear role, goal, and backstory in CrewAI. The backstory shapes the agent's perspective and analytical bias.

#### 1. Quant Analyst

- **Focus**: Technical analysis, price patterns, statistical indicators
- **Data Sources**: Historical prices, volume, moving averages, RSI, MACD, Bollinger Bands
- **Perspective**: Data-driven, skeptical of narratives, focused on quantifiable signals
- **Output**: Technical score with key indicator readings and pattern identification

#### 2. Sentiment Scout

- **Focus**: Market sentiment, news flow, social metrics
- **Data Sources**: News headlines (via web search tools), fear/greed proxy indicators, volume anomalies
- **Perspective**: Contrarian-aware, understands crowd psychology, tracks narrative shifts
- **Output**: Sentiment score with supporting evidence and narrative summary

#### 3. Macro Strategist

- **Focus**: Macroeconomic context, sector dynamics, intermarket relationships
- **Data Sources**: Index performance, sector ETFs, yield data, currency movements
- **Perspective**: Top-down thinker, connects micro to macro, identifies regime changes
- **Output**: Macro alignment score with sector and economic context

#### 4. Risk Manager

- **Focus**: Position sizing, correlation risks, downside scenarios
- **Data Sources**: Volatility metrics, historical drawdowns, correlation matrices
- **Perspective**: Conservative, focused on capital preservation, stress-tests assumptions
- **Output**: Risk assessment with position size recommendation and key risks

#### 5. Portfolio Chief

- **Focus**: Synthesis and final decision-making
- **Data Sources**: All other agent outputs
- **Perspective**: Balanced, resolves conflicts, weighs evidence quality
- **Output**: Final recommendation with confidence level and dissenting opinions noted

## Workflow Design

### Primary Workflow: Consensus Analysis

1. **Input Reception**: User submits a ticker symbol and optional analysis context

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CloudAI-X/the-trading-floor-v2](https://github.com/CloudAI-X/the-trading-floor-v2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
