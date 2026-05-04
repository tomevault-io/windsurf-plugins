---
trigger: always_on
description: **Bucketwise Planner** is an open-source, community-driven personal budget management app implementing the **Barefoot Investor** method by Scott Pape (https://www.barefootinvestor.com/). Features fortnightly budgeting cycles, debt snowball payoff tracking, and bucket-based spending allocations.
---

# Copilot Instructions

## Project Overview

**Bucketwise Planner** is an open-source, community-driven personal budget management app implementing the **Barefoot Investor** method by Scott Pape (https://www.barefootinvestor.com/). Features fortnightly budgeting cycles, debt snowball payoff tracking, and bucket-based spending allocations.

**Attribution**: This project is inspired by Scott Pape's Barefoot Investor methodology. We are not affiliated with or endorsed by Scott Pape, but credit his work prominently in all documentation.

**Tech Stack:**
- Backend: Node.js 18+ + Express v5 + TypeScript (ESM), PostgreSQL 14+ via node-postgres
- Frontend: React 18 + Vite 7 + TypeScript + Mantine v8.3.10 + Tabler Icons
- Tests: Vitest for unit/integration tests
- Deployment: Docker + Docker Compose (multi-user per self-hosted instance)
- Optional: Google Gemini 2.5 Flash AI via @google/genai (disabled by default)
- Monorepo: pnpm workspaces (backend + frontend folders)

## Architecture

### Backend (Domain-Driven Design)

**Multi-User by Default**: Each self-hosted instance is multi-user with JWT authentication. No single-user limitation.

**Layers**:
- **Domain Layer**: Pure business logic (Money, Debt, Fortnight, Allocation) — framework-free, no dependencies on Express/Zod/database
- **Application Layer**: Use cases implementing workflows, Zod schemas for DTOs, error mapping, business workflow coordination
- **Infrastructure Layer**: PostgreSQL repositories, in-memory adapters for testing, database persistence implementations
- **Presentation Layer**: Express v5 HTTP controllers, middleware, route definitions, request validation
- **Authentication Layer**: JWT tokens (refresh + access), bcryptjs password hashing, session management

**Key Principles:**
- **Domain purity**: No Express, Zod, or database concerns in domain entities — just business rules and invariants
- **Thin controllers**: Controllers delegate to use cases, never contain business logic
- **Envelope pattern**: All HTTP responses use `ApiResponse<T>` wrapper via `ResponseFormatter`
- **Error mapping**: Throw domain errors (`ValidationError`, `DomainError`); map to HTTP via `ErrorMapper`
- **Validation**: Zod schemas in `application/dtos/schemas`, applied via middleware (never re-parsed in controllers)
- **Dependency injection**: Manual constructor injection, no global singletons or static state
- **Testing**: Vitest unit tests for domain/use cases, deterministic and fast, mock repositories for isolation
- **Optional AI**: AI routes only registered if `AI_ENABLED=true` and `GEMINI_API_KEY` is set; graceful no-op if disabled
- **OOP-first**: Prefer composition, inheritance, and abstraction to model behaviors; use base classes/interfaces for shared workflows

### Frontend (React + Mantine)
- **Views**: DashboardView, FortnightView, TransactionsView, DebtsView, ProfileView
- **Components**: Reusable UI components (FortnightSelector, HelpDrawer, Badges, EmptyState)
- **State**: API client with typed fetch wrapper, minimal local state
- **Theme**: Dark custom Mantine theme (navy/slate with teal/amber accents)
- **Help System**: Global HelpDrawer with context provider, keyboard shortcuts (⌘/), searchable content
- **UX Patterns**: Tooltips on complex controls, loading/error/empty states, consistent formatting
- **OOP where useful**: Extract shared UI behavior into helper classes/adapters, avoid repeated logic in views

## Coding Conventions

### TypeScript
- Strict mode: `verbatimModuleSyntax`, `exactOptionalPropertyTypes` on
- Use type-only imports for types: `import type { Express, Request } from 'express'`
- Prefer `interface` for public contracts, `type` for unions/intersections
- Keep files ASCII, avoid emoji/unicode in code

### Backend Patterns
- Controllers: thin, validation via middleware, delegate to use cases
- Use cases: implement IUseCase interface, accept single request object, return typed response
- Repositories: conform to interfaces, implementations swappable (Postgres/in-memory)
- Value objects: immutable (Money uses integer cents), guard invariants in constructors
- Routes: wire with `asyncHandler`, validate with Zod middleware first, centralized error handling
- Logging: use request logging middleware, avoid ad-hoc console logs in domain/application
- OOP helpers: prefer base use-case/repository abstractions or mappers to reduce repetition

### Frontend Patterns
- Date handling: use `formatDateToISO()` utility to normalize dates to YYYY-MM-DD (prevents timezone drift)
- API calls: unwrap `ApiResponse` envelope, handle errors via ErrorAlert component
- Forms: Mantine components, controlled inputs, validation feedback
- Help triggers: "?" button + `mod+/` hotkey pattern on all pages
- Tooltips: use Mantine Tooltip on complex buttons/fields/headers
- Components: small, focused, prop-driven; avoid prop drilling with context when needed
- OOP helpers: encapsulate shared logic in adapters/services to avoid repetition across views

### Testing

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [solid-logic-studios/bucketwise-planner](https://github.com/solid-logic-studios/bucketwise-planner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
