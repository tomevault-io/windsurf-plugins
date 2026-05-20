---
trigger: always_on
description: Provides interactive test runner with:
---

# MAX Agentic Cookbook - Project Context

## Overview

MAX Agentic Cookbook is a fullstack cookbook application showcasing the agentic AI capabilities of Modular MAX as a complete LLM serving solution. Built with FastAPI (Python) backend + React (TypeScript) SPA frontend for maximum flexibility and performance.

**Key benefits:**

-   **Python-first backend** - Direct access to AI ecosystem (MAX, transformers, etc.)
-   **Type safety** - End-to-end TypeScript frontend, Python type hints backend
-   **Clean separation** - Independent frontend/backend projects (not a monorepo)
-   **Modern tooling** - React Router v7, SWR, Mantine v7, FastAPI, uv

## Architecture

```
max-recipes/
├── backend/              # FastAPI + uv (Python 3.11+)
├── frontend/             # Vite + React + TypeScript SPA
├── docs/                 # Architecture, contributing, Docker guides
├── Dockerfile            # Demo server (MAX + backend + frontend)
├── ecosystem.config.js   # PM2 config for running all services
└── .dockerignore         # Docker build exclusions
```

### Backend (FastAPI + uv)

**Tech:** FastAPI, uvicorn, uv for dependency management, python-dotenv, openai

**Ports:**

-   Local dev: 8010
-   Docker: 8010

**Configuration:**

-   `.env.local` with `COOKBOOK_ENDPOINTS` JSON array
-   CORS configured for localhost:5173 (dev only)
-   Serves frontend static files from `backend/static/` directory

**Structure:**

```
backend/
├── src/
│   ├── main.py                 # Entry point, loads .env.local, includes recipe routers
│   ├── core/                   # Config and utilities
│   │   ├── endpoints.py        # Endpoint management with caching
│   │   ├── models.py           # Models listing (proxies /v1/models)
│   │   └── code_reader.py      # Source code reading utility for /code endpoints
│   └── recipes/                # Recipe routers
│       ├── multiturn_chat.py   # Multi-turn chat recipe (SSE streaming)
│       └── image_captioning.py # Image captioning (NDJSON streaming)
└── pyproject.toml              # Python dependencies (uv)
```

**API Endpoints:**

-   `GET /api/health` - Health check
-   `GET /api/recipes` - List available recipe slugs (programmatically discovers registered routes)
-   `GET /api/endpoints` - List configured LLM endpoints (from .env.local)
-   `GET /api/models?endpointId=xxx` - List models for endpoint (proxies OpenAI-compatible /v1/models)
-   `POST /api/recipes/multiturn-chat` - Multi-turn chat endpoint (SSE streaming)
-   `GET /api/recipes/multiturn-chat/code` - Get multiturn-chat backend source as plain text
-   `POST /api/recipes/image-captioning` - Image captioning with NDJSON streaming
-   `GET /api/recipes/image-captioning/code` - Get image-captioning backend source as plain text
-   Frontend source: Static files at `/code/{recipe-name}/ui.tsx` (copied by build script)

**Core Modules:**

The backend provides reusable utilities in `src/core/` for recipe development:

-   **endpoints.py** - Endpoint configuration management with caching:

    ```python
    from ..core.endpoints import get_cached_endpoint

    endpoint = get_cached_endpoint(endpoint_id)
    if not endpoint:
        raise HTTPException(status_code=404, detail="Endpoint not found")

    client = AsyncOpenAI(
        base_url=endpoint.base_url,
        api_key=endpoint.api_key
    )
    ```

    -   Loads from `COOKBOOK_ENDPOINTS` environment variable
    -   In-memory caching for fast lookups
    -   Never exposes API keys to client

-   **models.py** - Proxies OpenAI-compatible `/v1/models` endpoint:

    ```python
    GET /api/models?endpointId={id}
    ```

    -   Returns available models for the specified endpoint

-   **code_reader.py** - Utility for reading recipe source code:

    ```python
    from ..core.code_reader import read_source_file

    source_code = read_source_file(__file__)
    ```

    -   Returns Python source code as a string
    -   Enables the code viewer feature in the frontend

See [API Reference](../docs/api.md) for complete endpoint documentation.

### Frontend (Vite + React)

**Tech:** Vite, React 18, TypeScript, React Router v7, Mantine v7, SWR, highlight.js, Prettier

**Ports:**

-   Local dev: 5173 (Vite dev server with proxy to backend)
-   Docker: 8010 (served as static files by FastAPI backend)

**Key Features:**

-   Auto-generated routes from registry using utility functions in `routing/`
-   Build output: `backend/static/` directory (served by FastAPI in production)
-   Vite proxy to backend port 8010 (no CORS issues in dev)
-   Mantine v7 with custom theme (nebula/twilight colors), 70px header height
-   AppShell with collapsible sidebar, responsive Header

**State Management:**

-   **Server State:** SWR (API data fetching, caching, automatic revalidation)
-   **Client State:** URL query params (`?e=endpoint-id&m=model-name`) via custom hooks

**Structure:**

```
frontend/
├── src/
│   ├── recipes/                # Recipe components + registry.ts
│   │   ├── registry.ts         # Pure data - recipe metadata only
│   │   ├── components.ts       # React component mapping (UI + README)
│   │   ├── multiturn-chat/     # Multi-turn chat recipe
│   │   │   ├── README.mdx      # Recipe documentation
│   │   │   └── ui.tsx          # Demo component (exports Component function)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [modular/max-agentic-cookbook](https://github.com/modular/max-agentic-cookbook) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
