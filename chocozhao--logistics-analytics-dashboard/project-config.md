---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an AI-driven logistics analytics dashboard, a full-stack web application that enables logistics teams to explore freight data through:
1. Traditional analytics dashboard (KPIs + charts)
2. AI-powered natural language interface
3. Predictive demand forecasting

**Core Principles:**
- AI as orchestrator, not source of truth – AI interprets questions, selects tools, formats results; actual computations are performed by deterministic functions
- Read-only data – no modifications to underlying datasets
- Explainability – every answer includes filters used, metrics, query plan, and raw data access
- Simple, correct, deployable – prioritize clarity and correctness over feature completeness

## Tech Stack

**Frontend:**
- Vue 3 + Vite + Pinia for reactivity and state management
- UI component library: Element Plus or naive-ui
- Chart library: ECharts or Chart.js for dynamic chart rendering

**Backend:**
- Java 17 + Spring Boot 3 for REST APIs
- AI orchestration: LangChain4j or custom implementation with OpenAI GPT-3.5-turbo
- Database: PostgreSQL 15 with JSON and aggregation support
- Forecasting: Apache Commons Math (regression) or custom exponential smoothing implementation

**Deployment:**
- Docker + (Render.com / Fly.io / AWS ECS) + Nginx
- Publicly accessible URL required

## Architecture Overview

### High-Level Components

1. **Frontend (Vue)**: Dashboard views, natural language query interface, forecasting controls
2. **Spring Boot Controllers**: REST endpoints for `/api/dashboard/*`, `/api/query`, `/api/forecast`
3. **Orchestration Layer**: NLU service (AI + tool selector), tool registry, query planner
4. **Tool Execution Layer**: KPI aggregator, time series query executor, forecasting engine
5. **PostgreSQL Database**: Read-only access with orders table containing logistics data

### Key API Endpoints

**Dashboard APIs:**
- `GET /api/dashboard/kpis` – returns total orders, delivered, delayed, on-time rate, average delivery days
- `GET /api/dashboard/order-volume` – time series data with configurable granularity
- `GET /api/dashboard/delivery-performance` – stacked bar chart data (on-time vs delayed)
- `GET /api/dashboard/carrier-breakdown` – carrier performance metrics

**Natural Language Query:**
- `POST /api/query` – accepts natural language questions, returns structured response with answer, chart, explanation, raw data

**Forecasting:**
- `POST /api/forecast` – generates demand predictions with historical+forecast chart and inventory recommendations

### Database Schema

```sql
CREATE TABLE orders (
  id SERIAL PRIMARY KEY,
  order_date DATE NOT NULL,
  promised_delivery_date DATE NOT NULL,
  actual_delivery_date DATE,  -- NULL for not yet delivered
  status VARCHAR(20) CHECK (status IN ('pending', 'in_transit', 'delivered', 'cancelled')),
  carrier VARCHAR(50),
  destination_city VARCHAR(100),
  destination_state VARCHAR(50),
  destination_region VARCHAR(50),
  order_value DECIMAL(10,2),
  sku VARCHAR(50),
  quantity INTEGER
);
```

Key indexes on `order_date`, `carrier`, `status`, and delivery dates.

### AI Orchestration Design

**Critical Principle**: AI never generates SQL or direct data values. AI only outputs structured tool calls with validated parameters.

**Defined Tools:**
- `get_time_series` – get aggregated values over time (day/week/month)
- `get_breakdown` – get aggregated values by categorical dimension (carrier, region, status)
- `get_kpi` – get single KPI value
- `forecast_demand` – predict future order volume

**NLU Process:**
1. Receive user question
2. Call OpenAI API with system prompt + tool definitions
3. Parse response as tool call (enforce JSON schema)
4. Validate parameters (date ranges, enum values)
5. Execute corresponding Java service method
6. Format results (answer text via lightweight AI call or templates)

## Development Setup

**Note**: This is a new project based on the PRD. No code exists yet beyond the specification document.

### Expected Project Structure

```
logistics-analytics-dashboard/
├── backend/              # Spring Boot application
│   ├── src/main/java/
│   ├── pom.xml
│   └── Dockerfile
├── frontend/            # Vue 3 application
│   ├── src/
│   ├── package.json
│   └── Dockerfile
├── database/            # SQL scripts, migrations
├── docker-compose.yml
└── AI_Logistics_Analytics_Dashboard_PRD.md
```

### Development Commands (To Be Implemented)

**Backend:**
- Build: `mvn clean package` (assuming Maven)
- Run tests: `mvn test`
- Run locally: `mvn spring-boot:run`

**Frontend:**
- Install dependencies: `npm install`
- Development server: `npm run dev`
- Build for production: `npm run build`
- Lint: `npm run lint` (if configured)

**Database:**
- Initialize: `psql -f database/init.sql`
- Seed with sample data: see PRD section 12.5 for data generation requirements

### Docker Deployment

Use `docker-compose up` to start all services:
1. PostgreSQL database with preloaded data
2. Spring Boot backend
3. Nginx serving Vue frontend

Environment variables required:
- `OPENAI_API_KEY` – for NLQ functionality

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chocozhao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
