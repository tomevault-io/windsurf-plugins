---
trigger: always_on
description: VSol Admin is a local-first, SQLite-backed web application that replaces the Excel-based "golden sheet" for managing monthly consultant payroll cycles. The system automates calculations, provides audit logging, and maintains data integrity while preserving the familiar workflow.
---

# VSol Admin - Cursor Rules

## Project Context

VSol Admin is a local-first, SQLite-backed web application that replaces the Excel-based "golden sheet" for managing monthly consultant payroll cycles. The system automates calculations, provides audit logging, and maintains data integrity while preserving the familiar workflow.

Key features:
- Monthly payroll cycle management with automated line item creation
- Real-time calculation of subtotals and totals using Excel formula logic
- Snapshot rates at cycle creation to preserve historical data
- Audit logging for all state changes with user attribution
- Consultant CRUD with termination date support
- Invoice and payment tracking integrated with cycles
- Anomaly detection for missing data, bonuses without dates, etc.

Tech stack:
- Monorepo: Turborepo + pnpm workspaces
- Backend: Node.js + Express, Drizzle ORM (SQLite), Zod validation, bcrypt for auth
- Frontend: React + Vite, React Router v6, TanStack Query, TanStack Table, Shadcn/ui, date-fns
- Database: SQLite (`file:./dev.db`)
- Auth: JWT with username/password (users: rommel, isabel, celiane)

Architecture:
- Turborepo monorepo structure: `apps/api`, `apps/web`, `packages/shared`
- Backend in Express with Drizzle ORM for type-safe queries
- Frontend as React SPA with TanStack Query for server state
- Shared Zod schemas and TypeScript types in packages/shared
- SQLite file-based database (local-first)

## Code Style & Patterns

TypeScript:
- Use strict mode for type safety
- Prefer type inference when obvious
- Define explicit interfaces for API request/response
- Use Zod schemas for runtime validation (create schemas in packages/shared)
- Avoid `any` - use `unknown` and narrow with type guards
- Use discriminated unions for polymorphic data (e.g., Payment kinds)
- Export types from .d.ts files when used across packages

JavaScript patterns:
- Always use async/await over Promises chains
- Use pnpm workspaces for package management
- Create services for business logic, keep routes thin
- Use middleware for cross-cutting concerns (auth, logging, validation)
- Prefer functional composition over class inheritance
- Use Drizzle for all database operations (never raw SQL)

Naming conventions:
- Files: kebab-case (e.g., `cycle-service.ts`, `line-item-table.tsx`)
- Components: PascalCase (e.g., `GoldenSheetPage.tsx`)
- Functions: camelCase (e.g., `createPayrollCycle()`)
- Constants: UPPER_SNAKE_CASE (e.g., `JWT_SECRET`)
- Database tables: camelCase (e.g., `payrollCycles`, `cycleLineItems`)
- React hooks: prefix with `use` (e.g., `useCycleData()`)

Import/export:
- Use named exports for services and utilities
- Use default exports for React components
- Group imports: external → workspace packages → relative
- Absolute imports for shared package: `@vsol-admin/shared`
- Absolute imports for web: `@/components`, `@/lib`, `@/hooks`

What to avoid:
- Do not mutate database records directly - use service layer
- Do not store computed values in database - calculate on the fly
- Do not use index.ts exports with re-exports except for package boundaries
- Do not create cycles without auto-creating line items for active consultants
- Do not allow editing snapshotted ratePerHour in line items

## Technology-Specific Guidelines

Express/Node.js:
- Use Express Router for route organization by entity
- Validate request body with Zod schemas before service calls
- Use try-catch blocks in async route handlers
- Return 400 for validation errors, 401 for auth failures, 500 for server errors
- Use .env for secrets, never hardcode credentials
- Enable CORS only for localhost:5173 (web dev server)

Drizzle ORM:
- Define schema in `apps/api/src/db/schema.ts`
- Use relations() for joins (e.g., cycle.lines)
- Use eq() for WHERE clauses (eq(t.column, value))
- Always use transactions for multi-step operations (create cycle + lines)
- Use Drizzle's migration system for schema changes
- Return plain objects from DB, not Drizzle model instances

React + Vite:
- Use functional components only, no class components
- Use TanStack Query for all server state (useQuery, useMutation)
- Use React Router v6 for routing with data loaders
- Store form state in controlled components (controlled inputs)
- Use Shadcn/ui components from shadcn/ui library
- Use date-fns for date formatting and manipulation

TanStack Query:
- Create custom hooks in `apps/web/src/hooks/`
- Use optimistic updates for inline editing on Golden Sheet
- Cache keys: `['cycles', cycleId]`, `['lines', cycleId]`
- Use invalidateQueries after mutations to refresh UI
- Add retry logic only for createCycle mutations
- Use staleTime: 0 for real-time data (cycles, lines)

TanStack Table:
- Define column types explicitly for type safety
- Use meta objects to pass cycleId, consultantId to inline editors
- Implement cell renderers for each editable field type
- Use row grouping for readability on large tables
- Enable column sorting only for dashboard lists (not Golden Sheet)

Shadcn/ui:
- Use Button, Input, Card, Calendar for consistent UI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DebonairSM) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
