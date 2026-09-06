---
trigger: always_on
description: Scout is an agentic sales intelligence platform for VAR/SI (Value Added Reseller / Systems Integrator) sales teams. Given a company name, industry, and a vague project/initiative, Scout autonomously researches the open internet using a multi-agent system to build a living, interactive intelligence profile — everything a seller needs for effective conversations.
---

# CLAUDE.md — Project Scout

## Project Overview

Scout is an agentic sales intelligence platform for VAR/SI (Value Added Reseller / Systems Integrator) sales teams. Given a company name, industry, and a vague project/initiative, Scout autonomously researches the open internet using a multi-agent system to build a living, interactive intelligence profile — everything a seller needs for effective conversations.

## Architecture

### Multi-Agent System

Scout uses 4 specialized AI agents running in a loop:

1. **Prime Agent** — Strategist. Plans research, delegates to sub-agents, reviews results, decides next cycle or stop. Never calls tools directly. Uses Sonnet 4 with extended thinking.
2. **Research Sub-Agents** (up to 5 in parallel) — Focused researchers. Each gets a single assignment, executes tool calls (search, scrape, etc.), returns structured findings. Stateless and independent. Uses Sonnet 4 without extended thinking.
3. **Synthesis Agent** — Analyst. Merges raw findings from all sub-agents, deduplicates, resolves contradictions, categorizes, assesses confidence. Uses Sonnet 4 with extended thinking.
4. **Presentation Agent** — Storyteller. Formats structured intelligence into dashboard-ready content with summaries, insights, and portfolio mapping. Uses Sonnet 4 without extended thinking.

The cycle: Prime plans → Sub-agents research in parallel → Synthesis merges → Presentation formats → Dashboard updates → Prime reviews and plans next cycle OR stops.

### Tech Stack

- **Backend**: Python 3.12+ / FastAPI (async)
- **Frontend**: Next.js 14+ / React / TypeScript / Tailwind CSS
- **Database**: PostgreSQL with JSONB columns for flexible findings storage
- **AI**: Anthropic Claude API (claude-sonnet-4-5-20250514) via `anthropic` Python SDK
- **Web Search**: Brave Search API
- **Web Scraping**: httpx + BeautifulSoup4 (Playwright for JS-heavy pages as fallback)
- **SEC Data**: SEC EDGAR API (free, no key)
- **Real-time**: Server-Sent Events (SSE) for dashboard updates
- **Auth**: Email/password + JWT (v0.1)
- **Deployment**: Docker Compose (dev), GCP Cloud Run + Cloud SQL for PostgreSQL + Vercel (prod)
- **Cloud Storage**: Google Cloud Storage (GCS) for raw scraped content and documents
- **Secrets**: GCP Secret Manager for API keys and credentials
- **Logging**: Google Cloud Logging (via Cloud Run integration)
- **Container Registry**: Google Artifact Registry

### Project Structure

```
scout/
├── backend/
│   ├── app/
│   │   ├── main.py                    # FastAPI app entry point and route registration
│   │   ├── config.py                  # Pydantic Settings — env vars, API keys
│   │   ├── models/                    # Pydantic models (request/response schemas)
│   │   │   ├── company.py
│   │   │   ├── initiative.py
│   │   │   ├── research.py
│   │   │   └── user.py
│   │   ├── db/                        # Database layer
│   │   │   ├── database.py            # Async SQLAlchemy engine + session
│   │   │   ├── tables.py             # SQLAlchemy ORM table definitions
│   │   │   └── queries.py            # Reusable query functions
│   │   ├── agents/                    # Multi-agent research engine
│   │   │   ├── orchestrator.py        # Top-level cycle loop manager
│   │   │   ├── prime_agent.py         # Strategic planning agent
│   │   │   ├── research_agent.py      # Focused research worker
│   │   │   ├── synthesis_agent.py     # Findings merger and analyzer
│   │   │   ├── presentation_agent.py  # Dashboard content formatter
│   │   │   ├── prompts/              # System prompt strings per agent
│   │   │   │   ├── prime.py
│   │   │   │   ├── research.py
│   │   │   │   ├── synthesis.py
│   │   │   │   └── presentation.py
│   │   │   └── tools/                # Tool functions for research sub-agents
│   │   │       ├── base.py           # Base tool interface + Anthropic tool schema helpers
│   │   │       ├── web_search.py     # Brave Search API wrapper
│   │   │       ├── web_scrape.py     # httpx + BeautifulSoup scraper
│   │   │       ├── sec_filings.py    # SEC EDGAR API client
│   │   │       ├── job_postings.py   # Career page scraper
│   │   │       └── news_search.py    # Brave News search wrapper
│   │   ├── routers/                  # FastAPI route handlers
│   │   │   ├── research.py           # Research session endpoints
│   │   │   ├── companies.py          # Company profile endpoints
│   │   │   ├── portfolio.py          # Portfolio management endpoints
│   │   │   └── auth.py              # Authentication endpoints
│   │   ├── services/                 # Business logic layer
│   │   │   ├── research_service.py
│   │   │   ├── company_service.py
│   │   │   └── portfolio_service.py
│   │   └── streams/                  # SSE streaming
│   │       └── events.py            # Event types and SSE helpers
│   ├── tests/
│   │   ├── agents/
│   │   ├── tools/
│   │   ├── routers/
│   │   └── conftest.py
│   ├── alembic/                      # Database migrations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aquaregiaswarm-blip/scout](https://github.com/aquaregiaswarm-blip/scout) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
