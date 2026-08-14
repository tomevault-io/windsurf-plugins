---
trigger: always_on
description: This repository contains the **Energy Performance Simulation Manager (EPSM)**, a containerized web application for managing building energy simulations using EnergyPlus.
---

# EPSM - Copilot Instructions

This repository contains the **Energy Performance Simulation Manager (EPSM)**, a containerized web application for managing building energy simulations using EnergyPlus.

## Project Overview

EPSM is a full-stack application designed to streamline the process of running EnergyPlus simulations for building performance optimization. It enables building owners, researchers, and engineers to explore and evaluate energy renovation strategies across large building stocks.

**Key Purpose:** Manage materials, constructions, and building templates; run baseline simulations; create renovation scenarios; execute batch simulations; and analyze results with interactive visualizations.

## Technology Stack

### Frontend
- **Framework:** React 18 with TypeScript 5.9
- **Build Tool:** Vite 5 with Hot Module Replacement (HMR)
- **UI Components:** Material-UI (MUI) version 5.15
- **Styling:** Tailwind CSS 3.4
- **State Management:** React Context API
- **HTTP Client:** Axios
- **Charts:** Chart.js, Recharts
- **Icons:** Lucide React
- **Location:** `frontend/` directory

### Backend
- **Framework:** Django 3.2 with Django REST Framework
- **Language:** Python 3.11+
- **WebSockets:** Django Channels with Daphne
- **File Processing:** eppy (EnergyPlus IDF parsing), lxml
- **Location:** `backend/` directory

### Infrastructure
- **Database:** PostgreSQL 15 (port 5432)
- **Caching:** Redis 7 Alpine (port 6379)
- **Containerization:** Docker Compose for development and production
- **Simulation Engine:** EnergyPlus via NREL Docker image
- **Reverse Proxy (Production):** Nginx

## Repository Structure

```
epsm/
├── frontend/              # React TypeScript frontend
│   ├── src/
│   │   ├── components/   # React components (auth, baseline, simulation, results)
│   │   ├── context/      # React Context providers
│   │   ├── lib/          # API clients and utilities
│   │   ├── types/        # TypeScript type definitions
│   │   └── utils/        # Helper functions
│   ├── Dockerfile        # Development container
│   └── Dockerfile.prod   # Production container
├── backend/              # Django backend
│   ├── config/           # Django settings (settings.py, urls.py, asgi.py)
│   ├── simulation/       # Core simulation logic
│   ├── database/         # Database models for materials
│   ├── media/            # File uploads (IDF/EPW files, simulation results)
│   └── requirements.txt  # Python dependencies
├── scripts/              # Shell scripts (start.sh, stop.sh, status.sh, deploy.sh)
├── docs/                 # Documentation (markdown files)
├── tests/                # Test files
├── configs/              # Configuration files (symlinked to root for tool compatibility)
├── database/             # Database migrations and exports
├── .docker/              # Docker configurations
├── docker-compose.yml    # Development services
└── docker-compose.prod.yml  # Production services
```

### Important Note on Configuration Files
Configuration files like `tsconfig.json`, `vite.config.ts`, `tailwind.config.js`, `postcss.config.js`, and `eslint.config.js` are stored in the `configs/` directory but **symlinked to the root** for build tool compatibility. Don't modify the root symlinks—edit the actual files in `configs/`.

## Development Workflow

### Starting Development
1. Use `./scripts/start.sh` to start all Docker services
2. Frontend runs on http://localhost:5173 (Vite dev server with HMR)
3. Backend API runs on http://localhost:8000
4. Django admin available at http://localhost:8000/admin/ (admin/admin123)

### Making Code Changes

#### Frontend Changes
- Files in `frontend/src/` auto-reload via HMR
- TypeScript compilation happens automatically
- Follow existing component patterns (functional components with TypeScript)
- Use Material-UI components for consistency
- State management through React Context (see `context/` directory)

#### Backend Changes
- Django auto-reloads on Python file changes
- For model changes, run:
  ```bash
  docker-compose exec backend python manage.py makemigrations
  docker-compose exec backend python manage.py migrate
  ```
- Follow Django REST Framework conventions
- Use serializers for API responses

### Testing
- Run all tests: `./scripts/test.sh`
- Backend tests: `docker-compose exec backend python manage.py test`
- Frontend tests: `docker-compose exec frontend npm test`

## Code Conventions & Patterns

### Frontend
- **Components:** Functional components with TypeScript interfaces
- **File naming:** PascalCase for components (e.g., `BaselinePage.tsx`)
- **API calls:** Use `authenticatedFetch` from `lib/auth-api.ts`
- **Styling:** Combine Material-UI `sx` prop with Tailwind utility classes
- **Type safety:** Always define TypeScript interfaces for props and API responses

### Backend
- **API endpoints:** RESTful design with Django REST Framework
- **Models:** Use Django ORM, define in `models.py`
- **Views:** Class-based views or function-based with `@api_view` decorator
- **Serializers:** DRF serializers for data validation and transformation
- **File handling:** Store uploads in `media/` directory

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SB-Chalmers/epsm](https://github.com/SB-Chalmers/epsm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
