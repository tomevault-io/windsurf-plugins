---
trigger: always_on
description: **Version**: 2.1 (Intelligent Edition) | **Updated**: 2025-12-16
---

# AI Agent Guidelines for Litestar Fullstack Inertia

**Version**: 2.1 (Intelligent Edition) | **Updated**: 2025-12-16

A modern Litestar fullstack application built with Inertia.js, React 19, and shadcn/ui providing a seamless SPA experience.

---

## Intelligence Layer

This project uses an **intelligent agent system** that:

1. **Learns from codebase** before making changes
2. **Adapts workflow depth** based on feature complexity
3. **Accumulates knowledge** in pattern library
4. **Selects tools** based on task requirements

### Pattern Library

Reusable patterns in `specs/guides/patterns/`:
- Consult before implementing similar features
- Add new patterns during review phase

### Complexity-Based Checkpoints

| Complexity | Checkpoints | Triggers |
|------------|-------------|----------|
| Simple | 6 | CRUD, config change, single file |
| Medium | 8 | New service, API endpoint, 2-3 files |
| Complex | 10+ | Architecture change, new domain module |

---

## Quick Reference

### Technology Stack

| Backend | Frontend |
|---------|----------|
| Litestar 2.8+ | React 19 |
| SQLAlchemy 2.0 | Inertia.js |
| advanced-alchemy | shadcn/ui |
| litestar-granian | Tailwind CSS |
| pytest | Vite |
| | Biome |

### Essential Commands

```bash
make install        # Install all dependencies
make test           # Run pytest test suite
make lint           # Run all linting (pre-commit + type-check + slotscheck)
make pre-commit     # Run pre-commit hooks (ruff, codespell)
make type-check     # Run mypy and pyright
make coverage       # Run tests with coverage
make start-infra    # Start Docker containers (PostgreSQL, Redis)
make stop-infra     # Stop Docker containers

# Frontend (use bun, not npm)
bun run dev         # Start Vite dev server
bun run build       # Build production assets
npx biome check resources/  # Lint frontend

# Type Generation
uv run app assets generate-types  # Generate TypeScript types from OpenAPI schema

# Database
uv run app database upgrade         # Apply migrations
uv run app database make-migrations # Create migration

# Run application
uv run app run      # Start server with Granian
```

---

## Code Standards

### Python

| Rule | Standard |
|------|----------|
| Type hints | Use `T \| None` (PEP 604), not `Optional[T]` |
| Future annotations | `from __future__ import annotations` in all files |
| Docstrings | Google style |
| Tests | Function-based pytest (not class-based) |
| Line length | 120 characters |
| Datetime | Always timezone-aware: `datetime.now(timezone.utc)` |

### TypeScript/React

| Rule | Standard |
|------|----------|
| Linting | Biome |
| Components | Functional with TypeScript interfaces |
| Styling | Tailwind CSS with shadcn/ui |
| State | Inertia.js `useForm`, `usePage` |

---

## Slash Commands

| Command | Description |
|---------|-------------|
| `/prd [feature]` | Create PRD with pattern learning |
| `/implement [slug]` | Pattern-guided implementation |
| `/test [slug]` | Testing with 90%+ coverage |
| `/review [slug]` | Quality gate and pattern extraction |
| `/explore [topic]` | Explore codebase |
| `/fix-issue [#]` | Fix GitHub issue |

---

## Subagents

| Agent | Mission |
|-------|---------|
| `prd` | PRD creation with pattern recognition |
| `expert` | Implementation with pattern compliance |
| `testing` | Test creation (90%+ coverage) |
| `docs-vision` | Quality gates and pattern extraction |

---

## Architecture

### Backend Structure

```
app/
├── domain/           # Domain modules
│   ├── accounts/     # User auth & profiles
│   │   ├── controllers.py
│   │   ├── services.py
│   │   ├── repositories.py
│   │   └── schemas.py
│   ├── teams/        # Multi-tenant teams
│   ├── tags/         # Tagging system
│   └── web/          # Inertia page controllers
├── db/
│   └── models/       # SQLAlchemy models
├── lib/              # Shared utilities
├── server/
│   ├── core.py       # ApplicationCore plugin
│   └── plugins.py    # Plugin instances
└── config.py         # Configuration
```

### Frontend Structure

```
resources/
├── pages/            # Inertia pages
│   ├── dashboard.tsx
│   ├── auth/
│   └── profile/
├── components/
│   └── ui/           # shadcn/ui components
├── layouts/
│   ├── app-layout.tsx
│   └── guest-layout.tsx
└── main.tsx          # Entry point
```

---

## Key Patterns

### Inertia Page Controller (Preferred Style)

Use `component` kwarg in route decorator:

```python
from litestar import Controller, get, post
from litestar_vite.inertia import InertiaRedirect

class FeatureController(Controller):
    path = "/feature"

    @get(component="feature/list", path="/", name="feature.list")
    async def list(self, service: FeatureService) -> dict:
        items = await service.list()
        return {"items": items}

    @post(component="feature/create", path="/", name="feature.create")
    async def create(self, request: Request, data: CreateSchema, service: FeatureService) -> InertiaRedirect:
        await service.create(data.to_dict())
        return InertiaRedirect(request, request.url_for("feature.list"))
```

### Service Pattern

```python
from advanced_alchemy.service import SQLAlchemyAsyncRepositoryService
from app.db.models import FeatureModel
from app.domain.feature.repositories import FeatureRepository


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [litestar-org/litestar-fullstack-inertia](https://github.com/litestar-org/litestar-fullstack-inertia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
