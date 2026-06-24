---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Cascadia is an open-source, code-first Product Lifecycle Management (PLM) system built with Hono (API server) and Vite + TanStack Router (SPA frontend). It replaces traditional low-code PLM systems (like Aras Innovator) with a developer-centric, type-safe approach where all customization happens in code, not through UI configuration.

**Key Philosophy**: Code-first configuration, TypeScript everywhere, enterprise-ready PostgreSQL backend, Git-style versioning for engineering data.

The signature feature is "ECO-as-Branch" - each Engineering Change Order gets its own isolated branch for parallel development.

**See [cascadia-feature-list.md](./cascadia-feature-list.md) for comprehensive feature documentation.**

## Repository Context

This is the main Cascadia PLM application. Related repositories:

| Repository          | Purpose            |
| ------------------- | ------------------ |
| `../DocsSite/`      | Documentation site |
| `../MarketingSite/` | Marketing website  |

## Technology Stack

- **Frontend**: Vite SPA + TanStack Router (file-based routing) + TanStack Query
- **Backend**: Hono API server, TypeScript, Node.js
- **Database**: PostgreSQL 18+ with Drizzle ORM
- **UI**: Tailwind CSS 4 + Radix UI components
- **Auth**: @oslojs/crypto + @oslojs/encoding + Arctic (OAuth)
- **Validation**: TanStack Form + Zod
- **Graph Visualization**: React Flow (@xyflow/react) + Dagre for layout
- **AI Integration**: TanStack AI with Anthropic and OpenAI adapters
- **CAD Conversion**: Python worker with pythonocc-core (STEP/IGES → STL/GLB)
- **CAD Generation**: Zoo Text-to-CAD API + KCL for assemblies
- **Testing**: Vitest (unit) + Playwright (E2E)
- **Message Queue**: RabbitMQ
- **Containerization**: Docker, Docker Compose

## Project Structure

```
src/
├── components/       # React components (forms, tables, dialogs)
│   ├── ui/           # Base UI primitives (Button, Card, DataGrid, etc.)
│   ├── ai/           # AI chatbot panel
│   ├── design-engine/# Collaborative design workspace components
│   └── work-instructions/ # Work instruction authoring/execution
├── lib/
│   ├── auth/         # Authentication & authorization services
│   ├── db/           # Drizzle schema & database utilities
│   ├── items/        # Item services (Parts, Documents, etc.)
│   ├── services/     # Core services (Branch, Checkout, Commit, etc.)
│   ├── workflows/    # Workflow engine
│   ├── jobs/         # Background job dispatch, definitions & worker
│   ├── api/          # API utilities (apiHandler, response builders, schemas)
│   ├── vault/        # File storage system
│   ├── sysml/        # SysML v2 serialization
│   ├── ai/           # AI chatbot tools, adapters, session service
│   ├── design-engine/# Collaborative design engine (stages, tools, prompts, materialization)
│   └── cad-generation/ # CAD generation pipeline (Zoo API, KCL, assembly)
├── routes/           # TanStack Router routes & API endpoints
└── __tests__/        # Test utilities and fixtures
workers/
├── node/             # Node.js job worker Dockerfile
├── cad-converter/    # Python worker: STEP/IGES → STL/GLB (pythonocc)
└── cad-generator/    # Python worker: Parametric CAD (CadQuery)
tests/
├── e2e/              # Playwright E2E tests
│   ├── pages/        # Page object models
│   ├── workflows/    # Workflow-based E2E tests
│   └── fixtures/     # Test fixtures
docs/                 # Architecture & feature documentation
scripts/              # Database seeding, deployment scripts
```

## Development Commands

```bash
# Development
npm run dev           # Start dev server on port 3000
npm run build         # Build for production
npm run serve         # Preview production build

# Database
npm run db:generate   # Generate migrations from schema changes
npm run db:migrate    # Run pending migrations
npm run db:push       # Push schema directly to database (dev only)
npm run db:studio     # Open Drizzle Studio GUI
npm run db:seed       # Minimal seed (admin, roles, program, standard library)
npm run db:seed:catalog  # Generic component catalog (fasteners, raw stock)

# Database Reset (truncates all tables, then optionally reseeds)
npm run db:reset              # Truncate all tables only
npm run db:reset:seed         # Truncate + minimal seed

# Testing
npm run test          # Run Vitest tests
npm run test:watch    # Run tests in watch mode
npm run test:coverage # Run tests with coverage
npm run test:ui       # Open Vitest UI
npm run test:e2e      # Run Playwright E2E tests
npm run test:e2e:ui   # Run E2E tests with UI
npm run test:e2e:full # Reset database + run E2E tests (clean slate)

# Run a single test file
npx vitest run src/lib/services/BranchService.test.ts

# Run tests matching a pattern
npx vitest run -t "should create branch"

# Code Quality
npm run lint          # ESLint
npm run format        # Prettier
npm run check         # Format + lint fix

# Background Workers
npm run workers:dev   # Start RabbitMQ + all workers (Node.js + Python)
npm run workers:stop  # Stop Python workers
npm run workers:logs  # Tail Python worker logs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cascadia-PLM/Cascadia-App](https://github.com/Cascadia-PLM/Cascadia-App) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-24 -->
