---
trigger: always_on
description: Detailed guidance for AI coding assistants working with the Ocean DataView monorepo.
---

# AGENTS.md

Detailed guidance for AI coding assistants working with the Ocean DataView monorepo.

This file provides comprehensive instructions for maintaining code quality, architectural consistency, and project-specific requirements.

## 1. Repository Structure

This is a Turborepo monorepo containing a full-stack data visualization platform.

### Apps

- **apps/web** - Next.js 16 frontend (port 3001)
  - Page routes for each view type (table, list, gallery, board, charts)
  - Demo modules showcasing dataview components
  - Uses React 19, TailwindCSS 4, shadcn/ui

- **apps/server** - Hono API server (port 3000)
  - Lightweight HTTP server with tRPC router
  - Handles data fetching, filtering, pagination

### Packages

- **packages/dataview** - Core React components (`@sparkyidea/dataview`)
  - View components: TableView, ListView, GalleryView, BoardView
  - Hooks: pagination, filtering, sorting, grouping
  - Providers: DataViewProvider, QueryBridge, ToolbarContext
  - UI components: toolbar, filters, pagination controls

- **packages/db** - Database layer (`@sparkyidea/db`)
  - Drizzle ORM schema and migrations
  - PostgreSQL connection setup
  - Seed data utilities

- **packages/shared** - Shared utilities (`@sparkyidea/shared`)
  - Type definitions (filter, sort, group, pagination)
  - URL DSL parsers for query state
  - Configuration utilities

- **packages/trpc** - tRPC router (`@sparkyidea/trpc`)
  - Product router with getMany, getGroup, getManyByColumn
  - SQL builders for filter/sort/group operations
  - Cursor-based pagination logic

- **packages/ui** - UI component library (`@sparkyidea/ui`)
  - shadcn/ui components
  - Shared styles and themes

- **packages/env** - Environment variables (`@sparkyidea/env`)
- **packages/config** - Build configuration (`@sparkyidea/config`)

### Prerequisites

- Bun >= 1.3.9
- Node.js >= 20 (for some tooling)
- PostgreSQL >= 15

## 2. Core Development Principles

### Philosophy

- **Move fast while maintaining high standards** - prioritize clarity over cleverness
- **Read the relevant code first** - follow existing patterns and naming
- **Keep solutions small and simple** - favor functions over classes
- **Prefer immutability** - don't mutate inputs; return new values
- **Handle errors up-front** - use guard clauses and early returns

### Separation of Concerns

- **Keep business logic separate from UI components**
- Extract logic into `lib/`, `utils/`, or hooks
- UI components should orchestrate, not implement complex logic
- Views consume hooks, hooks consume providers

### Fail-Fast, No Fallbacks

- **No Silent Fallbacks**: Code must fail immediately when conditions aren't met
- **Explicit Error Messages**: Clear messages explaining what failed
- **Data Mapping**: Handle all known values explicitly, throw for unknown

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Files/directories | kebab-case | `use-page-pagination.ts` |
| Components | PascalCase | `TableView`, `FilterPicker` |
| Hooks | camelCase with `use` | `usePagePagination` |
| Functions | camelCase verbs | `buildWhere`, `parseGroupBy` |
| Constants | UPPER_SNAKE_CASE | `LIMIT_OPTIONS` |
| Types/Interfaces | PascalCase | `DataViewProperty` |
| Booleans | is/has/can/should prefix | `isLoading`, `hasNextPage` |
| ENV vars | UPPER_SNAKE_CASE | `DATABASE_URL` |

### Code Organization

- Keep functions short and single-purpose (<20 statements)
- Keep files focused (<300 lines ideal)
- Avoid `let` - make a new function that returns the value
- Prefer early exits over deep nesting
- Use map/filter for iteration, avoid forEach

## 3. TypeScript Standards

### Type Safety

- **No `any`** - use `unknown` and narrow it down
- **Prefer `Record<string, unknown>` over `object`**
- **Do not disable ESLint/TypeScript rules** - fix the root cause
- **Constrain generics** with `extends` - avoid overly broad `<T>`
- **Use discriminated unions** for mutually exclusive states

### Type Inference

- **Do not add unnecessary type annotations** when inference works
- **Let TypeScript infer return types** when obvious
- **Avoid type casts (`as`)** unless unavoidable
- **Use `satisfies`** to check object literals match a type

### Async/Await

- **Any function returning a Promise must be `async`**
- **Always use `await`** when calling async functions
- **Avoid `.then()`/`.catch()`** - prefer try/catch
- **Avoid IIFEs** as workarounds for async

### Control Flow

- **Avoid nested ternaries** - use if/else or switch
- **Use `switch` for multiple values** - leverage exhaustiveness checking
- **Prefer string methods over RegEx** when possible

## 4. Frontend Development (React + Next.js)

### Component Architecture

- **Functional components only** - no classes
- **TypeScript everywhere** - interfaces for props
- **Components consume context** - don't prop-drill excessively

### DataView Component Pattern

Each view follows this pattern:

```typescript
export function TableView<TData, TProperties extends readonly DataViewProperty<TData>[]>({
  pagination,
  onRowClick,
  // ...
}: TableViewProps<TData>) {
  // 1. Get context
  const { data, properties, counts } = useDataViewContext<TData, TProperties>();

  // 2. Setup view (grouping, validation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jingerpie/ocean-dataview](https://github.com/jingerpie/ocean-dataview) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
