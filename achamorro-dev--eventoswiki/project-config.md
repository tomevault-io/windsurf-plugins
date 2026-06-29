---
trigger: always_on
description: This is an Astro-based event management platform built with TypeScript, React, and a clean architecture pattern. The project follows Domain-Driven Design (DDD) principles with a modular structure.
---

# EventosWiki Agent Guidelines

## Project Overview

This is an Astro-based event management platform built with TypeScript, React, and a clean architecture pattern. The project follows Domain-Driven Design (DDD) principles with a modular structure.

## Project Structure & Module Organization

Astro handles routing from `src/pages`, with shared layout primitives in `src/layouts` and global styles in `src/styles`. Domain logic is grouped under `src/modules` (e.g. `events`, `meetups`, `organizations`) while cross-cutting utilities live in `src/modules/shared`. Database schemas, content migrations, and seeds live in `db/`, and static assets are split between `public/` for runtime delivery and `docs/` for marketing collateral.

### Clean Architecture Structure

- **Domain Layer**: Contains business logic, entities, value objects, and domain services
- **Application Layer**: Contains use cases, commands, and application services
- **Infrastructure Layer**: Contains database repositories and external service implementations
- **Presentation Layer**: Contains UI components, forms, and server actions

### Module Organization

Each feature module follows this structure:

```
src/modules/{module-name}/
??? domain/           # Business logic, entities, value objects
??? application/      # Use cases, commands, queries
??? infrastructure/   # Database repositories, external services
??? presentation/     # UI components, forms, server actions
```

### Shared Module

The shared module contains:

- `src/modules/shared/domain/` - Common domain utilities (DateTime, Primitives, ValueObject)
- `src/modules/shared/presentation/ui/` - Reusable UI components (shadcn/ui based)
- `src/modules/shared/application/` - Common use case patterns

## Build, Test, and Development Commands

Install dependencies with `pnpm install --frozen-lockfile` to stay aligned with CI. Use `pnpm dev` for hot-reload development, `pnpm build` for production bundles, and `pnpm preview` to inspect the compiled output. Run `pnpm astro check` whenever you touch Astro collections or TypeScript types to catch regressions early.

### Development Workflow

- **Git Hooks**: Use simple-git-hooks for pre-commit formatting
- **Code Formatting**: Use Prettier with Tailwind plugin
- **Linting**: Use commitlint for conventional commits
- **Type Checking**: Run `pnpm astro check` regularly

## Coding Standards

### TypeScript Configuration

- Use strict TypeScript configuration
- Follow the established path aliases:
  - `@/ui/*` ? `./src/modules/shared/presentation/ui/*`
  - `@/shared/*` ? `./src/modules/shared/*`
  - `@/{module}/*` ? `./src/modules/{module}/*`
  - `@/*` ? `./src/*`

### Coding Style & Naming Conventions

Prettier enforces two-space indentation, single quotes, trailing commas, and no semicolons; format large sets of changes with `pnpm exec prettier --write .`. Tailwind utility order is normalized by `prettier-plugin-tailwindcss`, so keep class lists descriptive rather than rearranging manually.

#### Clean Code Principles

- **No Line-by-Line Comments**: Write self-documenting code that explains intent through clear naming and structure
- **Semantic Code**: Use descriptive variable, function, and class names that explain purpose without comments
- **Single Responsibility**: Each function/class should have one reason to change
- **Meaningful Names**: Use pronounceable, searchable names that reveal intent
- **Small Functions**: Keep functions under 20 lines with descriptive names
- **No Magic Numbers**: Extract constants with meaningful names
- **Explicit Intent**: Code should read like well-written prose

**Example of Clean Code:**
```typescript
// ❌ Avoid this
const calculate = (a: number, b: number): number => {
  // Calculate the total price including tax
  const tax = 0.21 // 21% tax rate
  return a * b * (1 + tax)
}

// ✅ Prefer this
const calculateTotalPriceWithTax = (unitPrice: number, quantity: number): number => {
  const STANDARD_TAX_RATE = 0.21
  return unitPrice * quantity * (1 + STANDARD_TAX_RATE)
}
```

**File Naming Conventions:**

- **Components**: PascalCase (e.g., `EventEditForm.tsx`)
- **Hooks**: camelCase starting with 'use' (e.g., `useUploadFile.ts`)
- **Utilities**: camelCase (e.g., `datetime.ts`)
- **Types**: PascalCase (e.g., `EventData.ts`)
- **Constants**: UPPER_SNAKE_CASE (e.g., `DATE_FORMATS.ts`)
- **Route-level `.astro` files**: kebab-case to match Astro expectations
- **React components**: PascalCase
- **Hooks and utilities**: camelCase

### Domain Layer Patterns

- **Entities**: Use private constructors with static factory methods
- **Value Objects**: Extend the base `ValueObject<T>` class
- **Primitives**: Use the `Primitives<T>` type for serialization/deserialization
- **Validation**: Implement domain validators for business rules
- **IDs**: Use dedicated ID classes (e.g., `EventId`, `OrganizationId`)

Example Entity Pattern:

```typescript
export class Event implements EventProps {
  private constructor(props: EventProps) {
    /* ... */
  }

  static fromPrimitives(primitives: Primitives<Event>): Event {
    /* ... */
  }
  toPrimitives(): Primitives<Event> {
    /* ... */
  }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [achamorro-dev/eventoswiki](https://github.com/achamorro-dev/eventoswiki) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
