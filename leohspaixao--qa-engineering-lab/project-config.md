---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

QA Solar is a monorepo for studying and practicing test automation with multiple frameworks. It includes full-stack applications (frontend, backend, dashboard) and comprehensive test suites using Cypress, Playwright, Robot Framework, and Selenium.

**Key Technologies:**
- **Monorepo Management:** Yarn Workspaces + Turborepo
- **Frontend:** Vue 3 + TypeScript + Vite
- **Dashboard:** Vue 3 + Pinia + Chart.js
- **Backend:** NestJS + Prisma + PostgreSQL
- **Testing:** Cypress, Playwright, Robot Framework, Selenium
- **Documentation:** Docusaurus (https://leohspaixao.github.io/qa-engineering-lab/)
- **CI/CD:** GitHub Actions

## Repository Structure

```
qa-engineering-lab/
├── apps/
│   ├── frontend/       # Vue 3 main application (user management)
│   ├── backend/        # NestJS API with Prisma ORM
│   └── dashboard/      # Vue 3 quality dashboard with Chart.js
├── tests/
│   ├── cypress/        # E2E and component tests
│   ├── playwright/     # E2E tests
│   ├── robot/          # Robot Framework tests (Python-based)
│   ├── selenium/       # Selenium tests
│   └── performance/    # Performance tests
├── packages/
│   ├── preprocessor/   # Test results preprocessor (converts XML/JSON formats)
│   └── scripts/        # Shared scripts (e.g., timestamp generation)
└── docs/               # Docusaurus documentation site
```

## Common Commands

### Root Level
```bash
# Install dependencies
yarn install

# Run all apps in dev mode (parallel execution via Turborepo)
yarn dev

# Build all apps
yarn build

# Run all tests across workspaces
yarn test

# Lint all projects
yarn lint

# Docker commands
yarn docker:build      # Build and start all services
yarn n8n               # Start n8n automation server
yarn stop-n8n
yarn restart-n8n

# Version management (Changesets)
yarn changeset
yarn version-packages
```

### Frontend App (`apps/frontend`)
```bash
cd apps/frontend

yarn dev              # Start dev server on http://127.0.0.1:5173
yarn build            # Build for production
yarn test             # Run Vitest unit tests
yarn test:coverage    # Run tests with coverage
yarn type-check       # TypeScript type checking
yarn lint             # ESLint + auto-fix
```

### Backend App (`apps/backend`)
```bash
cd apps/backend

yarn dev              # Start NestJS dev server
yarn build            # Build for production
yarn test             # Run Jest tests
yarn test:coverage    # Run tests with coverage
yarn prisma:seed      # Seed database

# Prisma commands (use npx)
npx prisma migrate dev
npx prisma studio
npx prisma generate
```

### Dashboard App (`apps/dashboard`)
```bash
cd apps/dashboard

yarn dev              # Start dev server on http://127.0.0.1:5173
yarn build            # Build for production
yarn lint             # ESLint + auto-fix
```

### Cypress Tests (`tests/cypress`)
```bash
cd tests/cypress

yarn test                  # Run all tests (E2E + component)
yarn cy:open               # Open Cypress UI
yarn cy:e2e:run           # Run E2E tests headless
yarn cy:component:run     # Run component tests headless
yarn cy:clean             # Clean coverage and misc files
yarn cy:coverage          # Generate coverage report
```

### Playwright Tests (`tests/playwright`)
```bash
cd tests/playwright

yarn play:install     # Install browser binaries
yarn test             # Run all tests
yarn play:run         # Run tests headless
yarn play:open        # Open Playwright UI
yarn play:report      # Show test report
```

### Robot Framework Tests (`tests/robot`)
```bash
cd tests/robot

# Setup Python virtual environment first
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt

make test             # Run all tests
make test-scenario    # Run specific scenario
make test-file        # Run specific file
make lint             # Run Robocop linter
```

### Selenium Tests (`tests/selenium`)
```bash
cd tests/selenium

make test             # Run all tests
make building         # Build project
make coverage         # Generate coverage report
make clean            # Clean project
make help             # Show Makefile help
```

## Architecture

### Backend Architecture (NestJS)

The backend follows a modular NestJS architecture:

- **Modules:** Located in `apps/backend/src/modules/`
  - `auth/` - JWT authentication with passport-jwt
  - `users/` - User CRUD operations
  - `password-recovery/` - Password reset functionality
  - `prisma/` - Prisma client service (singleton pattern)

- **Database:** PostgreSQL with Prisma ORM
  - Schema: `apps/backend/prisma/schema.prisma`
  - Migrations: `apps/backend/prisma/migrations/`
  - Seeders: `apps/backend/prisma/seeders/`

- **API Documentation:** Swagger/OpenAPI available at `/api` endpoint when running

### Frontend Architecture (Vue 3)

Both frontend and dashboard use Vue 3 with Composition API:

- **Module-based structure:** Each feature is a module in `src/modules/`
  - Example: `modules/auth/`, `modules/user/`
  - Each module contains: `components/`, `views/`, services, types

- **Frontend (`apps/frontend`):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/LeohsPaixao) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
