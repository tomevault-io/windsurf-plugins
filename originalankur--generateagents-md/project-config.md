---
trigger: always_on
description: Flagsmith is an open-source feature flagging and remote configuration management tool. It allows teams to manage feature releases, perform A/B testing, and toggle application functionality without deploying new code. The tech stack is centered around a Python/Django backend API and a TypeScript/React single-page application frontend, designed to be self-hosted with Docker or used as a SaaS product.
---

# AGENTS.md — flagsmith

## Project Overview

Flagsmith is an open-source feature flagging and remote configuration management tool. It allows teams to manage feature releases, perform A/B testing, and toggle application functionality without deploying new code. The tech stack is centered around a Python/Django backend API and a TypeScript/React single-page application frontend, designed to be self-hosted with Docker or used as a SaaS product.

## Tech Stack

-   **Languages**: Python, TypeScript
-   **Frameworks**: Django, Django REST Framework, React, Redux Toolkit
-   **Database**: PostgreSQL
-   **Dependency Management**: Poetry (Backend), npm (Frontend)
-   **Testing**: `pytest` (Backend Unit & Integration), Jest (Frontend Unit), TestCafe (Frontend E2E)
-   **Formatting & Linting**: `black`, `isort` (Backend); Prettier, ESLint (Frontend)
-   **DevOps & Infrastructure**: Docker, Docker Compose
-   **Error Monitoring**: Sentry, Prometheus

## Architecture

The project uses a monorepo structure, separating the backend, frontend, and documentation into distinct top-level directories. It follows a client-server model with a monolithic Django API and a React Single-Page Application (SPA).

-   `api/`: The monolithic Django backend.
    -   `api/features/`, `api/organisations/`, `api/users/`: Modular Django apps, each containing its own models, views, and serializers, promoting separation of concerns.
    -   `api/tests/`: Contains all backend tests, subdivided into `unit/` and `integration/`.
    -   `api/pyproject.toml`: Defines all Python dependencies managed by Poetry.
-   `frontend/`: The React SPA frontend.
    -   `frontend/web/`: Main source code for the application.
    -   `frontend/web/components/`: Reusable React components form the core of the UI.
    -   `frontend/common/`: Contains shared logic, including state management.
    -   `frontend/common/store.ts`: The central Redux Toolkit store configuration.
    -   `frontend/common/service.ts`: The single RTK Query service definition that handles all API communication.
    -   `frontend/e2e/`: End-to-end tests written with TestCafe.
    -   `frontend/package.json`: Defines all frontend dependencies managed by npm.
-   `docs/`: Project documentation built with Docusaurus.
-   `docker-compose.yml`: The entry point for setting up the entire local development environment.

## Code Style

The codebase maintains a strict and consistent style through automated formatters and linters.

**Backend (Python):**
-   **Formatting**: Code is formatted with `black` and imports are sorted with `isort`. These tools are configured in `api/pyproject.toml` and must be run before committing.
-   **Naming**: Follows standard Python PEP 8 conventions (e.g., `snake_case` for variables and functions, `PascalCase` for classes).
-   **Views**: Class-based views are used, inheriting from Django REST Framework's `APIView`. Business logic should not reside in the view itself but be abstracted into model methods or service functions.

Example of a backend view (`api/audit/views.py`):
```python
from rest_framework.response import Response
from rest_framework.views import APIView
from audit.models import AuditLog

class AuditLogCount(APIView):
    def get(self, request, *args, **kwargs):
        count = AuditLog.objects.count()
        return Response({'count': count})
```

**Frontend (TypeScript/React):**
-   **Formatting**: Code is automatically formatted using **Prettier**.
-   **Linting**: **ESLint** is used to enforce code quality and best practices.
-   **Components**: Functional components with Hooks are standard.
-   **Data Fetching**: All data fetching is done via auto-generated RTK Query hooks.

Example of a frontend component (`frontend/web/components/AuditLogCounter.tsx`):
```typescript
import React from 'react';
import { useGetAuditLogCountQuery } from 'common/service';

export const AuditLogCounter = () => {
    const { data, isLoading, error } = useGetAuditLogCountQuery({});

    if (isLoading) return <div>Loading...</div>;
    if (error) return <div>Error fetching count!</div>;

    return <h1>Total Audit Logs: {data?.count}</h1>;
};
```

## Anti-Patterns & Restrictions

To maintain architectural integrity and code quality, the following rules must be strictly followed:

-   **Frontend:**
    -   **NEVER** make direct API calls from components using `fetch` or `axios`. **ALWAYS** use the RTK Query service layer defined in `frontend/common/service.ts`.
    -   **NEVER** introduce a new global state management library. All global state must be managed via the existing Redux Toolkit store (`frontend/common/store.ts`).

-   **Backend:**
    -   **NEVER** bypass the Django ORM for database operations unless it is for a critical, well-documented performance optimization.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [originalankur/GenerateAgents.md](https://github.com/originalankur/GenerateAgents.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
