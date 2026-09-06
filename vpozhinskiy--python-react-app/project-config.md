---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Full-stack web application with Django backend, React frontend, PostgreSQL database, and Nginx reverse proxy. All services run in Docker containers with hot-reload enabled for development.

**Tech Stack:**
- Backend: Django 5.2 + Django REST Framework
- Frontend: React with TypeScript
- Database: PostgreSQL 16
- Web Server: Nginx (reverse proxy)
- Build: CRACO (Create React App Configuration Override)

## Common Commands

### Essential Docker Commands
```bash
# Build all images
make build

# Start all services (foreground)
make up

# Stop all services
make down

# View logs from all services
make logs

# View specific service logs
make logs-backend
make logs-nginx
make logs-postgres

# Check service status
make ps
make health

# Clean everything (removes containers and volumes)
make clean

# Rebuild from scratch (no cache)
make rebuild
```

### Backend Development
```bash
# Run migrations (after model changes)
make makemigrations
make migrate

# Create superuser for admin panel
make createsuperuser

# Run tests
make test

# Access Django shell
make shell

# Access backend container bash
make exec-backend

# Access PostgreSQL CLI
make exec-postgres
```

### Frontend Development
```bash
# Frontend runs automatically with hot-reload on port 3000
# To restart frontend after changes:
docker-compose restart frontend

# Frontend npm scripts (run inside container):
# npm start   - Start dev server
# npm build   - Build production bundle
# npm test    - Run tests
```

### Database Operations
```bash
# Direct PostgreSQL access
docker-compose exec postgres psql -U app_user -d app_db

# Or use the make command
make exec-postgres
```

## Architecture

### Service Architecture
```
Client → Nginx (Port 80) → Backend (Port 8000)
                         → Frontend (Port 3000 - dev only)

Backend ↔ PostgreSQL (Port 5432, isolated network)
```

### Network Isolation
- **backend_network**: PostgreSQL ↔ Backend (database isolated from public)
- **frontend_network**: Nginx ↔ Backend (public-facing)

### Django Backend Structure (Clean Architecture)

The backend follows a layered architecture pattern:

```
backend/apps/submissions/
├── models/              # Domain models (pure data structures)
│   └── submission.py    # Submission model with DB indexes
├── dto/                 # Data Transfer Objects
│   └── dtos.py          # SubmissionDTO for service layer
├── serializers/         # DRF serializers (request/response validation)
│   └── serializers.py   # SubmissionRequestSerializer, HistoryResponseSerializer
├── repositories/        # Data access layer
│   └── submission_repository.py  # Database operations only
├── services/            # Business logic layer
│   ├── submission_service.py     # Validation, processing delay, orchestration
│   └── history_service.py        # History retrieval logic
├── views/               # Presentation layer (HTTP handling)
│   ├── submission_view.py        # POST /api/submit
│   └── history_view.py           # GET /api/history
└── urls.py              # URL routing
```

**Layering Principles:**
- **Models**: Pure Django models, no business logic
- **DTOs**: Simple data structures for passing data between layers
- **Repositories**: Only database operations (CRUD), no business logic
- **Services**: All business logic (validation, delays, orchestration)
- **Views**: HTTP handling only (request/response), dependency injection
- **Serializers**: Input validation and response serialization

**Dependency Flow**: Views → Services → Repositories → Models

When adding new features:
1. Define model in `models/`
2. Create DTO in `dto/`
3. Create repository methods in `repositories/`
4. Implement business logic in `services/`
5. Create serializers for request/response in `serializers/`
6. Create view in `views/` that wires everything together
7. Add route in `urls.py`

### React Frontend Structure

```
frontend/src/
├── api/                 # API client configuration
│   └── client.ts        # Axios instance with interceptors
├── components/          # Reusable UI components
│   ├── common/          # Generic components (Button, Link, Spinner)
│   ├── form/            # Form components (Input, Select, FieldError)
│   └── table/           # Table components
├── pages/               # Page-level components
│   ├── HomePage/        # Landing page with navigation
│   ├── SubmissionPage/  # Form submission page
│   └── HistoryPage/     # View submission history
├── features/            # Feature-specific logic
├── hooks/               # Custom React hooks
│   └── useApi.ts        # API call hook with loading/error states
├── types/               # TypeScript type definitions
├── utils/               # Utility functions
└── styles/              # Global styles
```

**Component Structure**: Each component has its own directory with `ComponentName.tsx` and `index.ts`

### API Endpoints

**Backend (Django REST Framework):**
- `POST /api/submit` - Submit form data
  - Request: `{date: string, first_name: string, last_name: string}`
  - Validation: No whitespace allowed in names
  - Processing: Random delay 0-3 seconds

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vpozhinskiy/python_react_app](https://github.com/vpozhinskiy/python_react_app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
