---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

book.ai is a production-ready AI book-writing system that transforms author briefs into complete manuscripts. The system uses real-time streaming, multi-agent collaboration via CrewAI, and comprehensive cost controls. Built as a microservices architecture with FastAPI backend, Next.js frontend, and deployed on Kubernetes.

## Architecture Overview

The system follows a three-tier architecture:

1. **Frontend**: Next.js 14 with App Router, Zustand state management, real-time SSE streaming
2. **Backend**: FastAPI with async SQLAlchemy, Redis caching, LiteLLM routing, CrewAI agents
3. **Infrastructure**: PostgreSQL, Redis, Prometheus monitoring, Kubernetes deployment

### Key Components

- **CrewAI Multi-Agent System**: 5 specialized agents (ConceptGenerator, Outliner, Writer, Editor, ContinuityChecker)
- **LiteLLM Router**: Primary/secondary/overflow model routing (gpt-5 → claude-sonnet-4-20250514 → gemini-2.5-pro)
- **SSE Streaming**: Real-time token streaming with < 300ms latency via Server-Sent Events
- **Cost Tracking**: Real-time cost monitoring with budget controls and per-agent allocation
- **Caching**: Redis-based response caching and rate limiting

## Development Commands

### Backend Development

```bash
cd backend
pip install -e .[dev]                    # Install with dev dependencies
uvicorn app.main:app --reload            # Run development server (port 8000)
pytest tests/ -v --cov=app              # Run tests with coverage
pytest tests/test_api_contract.py::test_health_endpoint  # Run single test
black app tests                          # Format code
ruff check app tests                     # Lint code  
mypy app                                 # Type check
```

### Frontend Development

```bash
cd frontend
npm install                              # Install dependencies
npm run dev                              # Run development server (port 3000)
npm run build                            # Build for production
npm run lint                             # Lint code (ESLint with Next.js rules)
npm run type-check                       # TypeScript strict mode checking
npm run test                             # Run color contrast tests
```

### Quality Assurance Commands

```bash
# Backend: Run all quality checks
cd backend && black app tests && ruff check app tests && mypy app && pytest tests/ -v

# Frontend: Run all quality checks  
cd frontend && npm run lint && npm run type-check && npm run build

# Full stack verification
cd backend && pytest tests/ -v && cd ../frontend && npm run type-check
```

### Docker Development

```bash
cd infra
docker-compose -f docker-compose.dev.yml up        # Start all services
docker-compose -f docker-compose.dev.yml up -d     # Start in background
docker-compose -f docker-compose.dev.yml logs -f   # Follow logs
docker-compose -f docker-compose.dev.yml down      # Stop all services
```

## Database Architecture

The system uses 5 main tables (defined in `backend/app/models.py`) with UUID primary keys:

- **projects**: Book projects with JSONB settings
- **sessions**: Writing sessions linked to projects  
- **events**: Action logs with JSONB payloads
- **artifacts**: Generated content (outlines, chapters) with optional blob storage
- **costs**: Token usage and cost tracking per agent/session

Database migrations are handled through SQLAlchemy with async support. All operations use proper indexing on (session_id, created_at) for performance.

## Agent System

The CrewAI agents (in `backend/app/agents/agents.py`) work in a coordinated pipeline:

1. **ConceptGenerator**: Expands brief into rich concepts
2. **Outliner**: Creates detailed chapter-by-chapter structure  
3. **Writer**: Generates chapter content following style guides
4. **Editor**: Performs structural editing and improvements
5. **ContinuityChecker**: Validates consistency across characters/timeline

Agents can work in parallel for chapter generation and use shared context through Redis caching.

## API Design Patterns

### Main Endpoints
- `/api/v1/projects/{id}/outline/stream` - Generate book outline with SSE streaming
- `/api/v1/projects/{id}/chapter/{n}/draft/stream` - Stream chapter draft generation
- `/api/v1/projects/{id}/chapter/{n}/edit/stream` - Stream editorial pass
- `/api/v1/projects/{id}/continuity/run` - Run continuity checker
- `/api/v1/projects/{id}/costs` - Get cost report
- `/ws/agents/{id}` - WebSocket for real-time agent status updates

### SSE Streaming Pattern
All streaming endpoints follow this pattern:
- Token events: `{"event": "token", "data": "content"}`
- Checkpoints: `{"event": "checkpoint", "data": "{\"progress\": 0.5}"}`
- Completion: `{"event": "complete", "data": "{\"tokens\": 1000}"}`

Authentication uses JWT with 1-hour expiry. Rate limiting is per-key (60 RPM). All costs are tracked in real-time with budget controls.

## Environment Configuration

Required environment variables:
- `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_API_KEY`: LLM provider keys
- `DATABASE_URL`: PostgreSQL connection string (default: `postgresql+asyncpg://postgres:postgres@localhost:5432/bookai`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agruai/ai-book-writer](https://github.com/agruai/ai-book-writer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
