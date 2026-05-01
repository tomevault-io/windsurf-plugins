---
trigger: always_on
description: <!-- GSD:project-start source:PROJECT.md -->
---

<!-- GSD:project-start source:PROJECT.md -->
## Project

**EdgeTerminalUI**
You are a senior UI designer and frontend developer.
Build premium, dark-themed interfaces.
Use subtle animations, proper spacing, and visual hierarchy.
No emoji icons. No inline styles. No generic gradients.

**EdgeTerminal**

A real-time cross-market arbitrage monitoring terminal for retail prediction market traders. The system continuously ingests data from Polymarket and Kalshi, matches equivalent contracts across venues, and surfaces price discrepancies as actionable arbitrage opportunities in a live dashboard. The core engine is Polymarket-Kalshi matching and pricing; the terminal wraps this in a broader trading dashboard experience.

**Core Value:** When two prediction markets disagree on the same event, show the trader exactly where, by how much, and with what confidence — in real time, with historical proof that detected opportunities were real.

### Constraints

- **Tech stack**: Python 3.12 + FastAPI backend, Next.js + React frontend, PostgreSQL — already established
- **Free APIs only**: No paid data feeds; must work within Polymarket and Kalshi public API rate limits
- **Regulatory**: Frame as research/analysis tool, not financial advice; no automated execution without explicit user action
- **API rate limits**: Both venues rate-limit; polling strategy must be sustainable at 30-second intervals
- **Team coordination**: 15-20 people requires clear module boundaries and well-defined interfaces
- **Matching accuracy**: False arb signals from bad matching are worse than missing real arbs — precision over recall
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- Python 3.12 - Backend API, market ingestion, matching, and scheduling
- TypeScript 5 - Frontend React components, type-safe API client, utilities
- JavaScript - Build configuration (ESLint, PostCSS, Next.js config)
- SQL - PostgreSQL database schema and queries (via SQLAlchemy ORM)
## Runtime
- Python 3.12.x (Docker: `python:3.12-slim`)
- Node.js 20.x (Docker: `node:20-alpine`)
- pip - Python dependency management
- npm - JavaScript/Node.js dependency management (lockfile: `package-lock.json`)
## Frameworks
- FastAPI 0.115.0 - RESTful API, dependency injection, async handlers
- Uvicorn 0.32.0 - ASGI server (FastAPI host)
- SQLAlchemy 2.0.49 (with asyncio) - Async ORM, database models, relationships
- APScheduler 3.11.0 - Background job scheduling, periodic polling and matching tasks
- Next.js 16.2.3 - React meta-framework, SSR, routing, build tooling
- React 19.2.4 - UI component framework
- React DOM 19.2.4 - DOM rendering for React
- pytest - Python test runner and assertion library
## Key Dependencies
- sqlalchemy[asyncio] 2.0.49 - Async ORM with PostgreSQL support
- asyncpg 0.30.0 - PostgreSQL async driver
- alembic 1.14.0 - Database migration tool (schema versioning)
- httpx 0.28.0 - Async HTTP client (for Kalshi/Polymarket API calls)
- anthropic 0.43.0 - Claude API client for LLM verification of market matches
- sentence-transformers 5.3.0 - Semantic embedding model (all-MiniLM-L6-v2, ~90MB) for embedding-based matching
- torch 2.11.0 - PyTorch deep learning framework (dependency of sentence-transformers)
- numpy 2.4.4 - Numerical computing (used by embeddings)
- pydantic-settings 2.7.0 - Configuration management from environment variables
- python-dotenv 1.0.1 - Load .env files for local development
- cryptography 44.0.0 - RSA signing for Kalshi API authentication
- swr 2.4.1 - Stale-while-revalidate data fetching hook, auto-refetch with 30s intervals
- tailwindcss 4 - Utility-first CSS framework
- @tailwindcss/postcss 4 - PostCSS plugin for Tailwind
- tailwind-merge 3.5.0 - Merge and deduplicate Tailwind classes
- tw-animate-css 1.4.0 - Tailwind animation utilities
- lightweight-charts 5.1.0 - TradingView-style price charts (SpreadChart)
- clsx 2.1.1 - Conditional className utility
## Configuration
- `DATABASE_URL` - PostgreSQL connection string (default: `postgresql+asyncpg://truthlayer:truthlayer@localhost:5432/truthlayer`)
- `KALSHI_KEY_ID` - Kalshi API key identifier
- `KALSHI_PRIVATE_KEY_PATH` - Path to RSA private key for Kalshi signing
- `ANTHROPIC_API_KEY` - Claude API key for LLM verification
- `POLLING_INTERVAL_SECONDS` - How often to fetch market data (default: 30)
- `MATCHING_INTERVAL_MINUTES` - How often to run matching algorithm (default: 15)
- `LOG_LEVEL` - Logging verbosity (default: INFO)
- `NEXT_PUBLIC_API_URL` - Backend API endpoint (e.g., `http://localhost:8000`)
- `.prettierrc` - Not present; using ESLint defaults
- `eslint.config.mjs` - ESLint 9 flat config with Next.js core web vitals and TypeScript rules
- `next.config.ts` - Minimal Next.js configuration
- `tsconfig.json` - TypeScript compiler options (target ES2017, strict mode, `@/*` path alias)
- `postcss.config.mjs` - PostCSS configuration with Tailwind v4
- `pyproject.toml` - Not present (using pip + requirements.txt)
## Platform Requirements
- Docker & Docker Compose for containerized local development
- PostgreSQL 16 (provided via docker-compose)
- Python 3.12+
- Node.js 20+
- Docker images for backend and frontend
- PostgreSQL 16+ database
- Uvicorn ASGI server (or production ASGI server like Gunicorn + Uvicorn workers)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rohanthomas1202/EdgeTerminal](https://github.com/rohanthomas1202/EdgeTerminal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
