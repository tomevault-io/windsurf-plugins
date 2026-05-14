---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with this repository.

## How to Contribute

This project is developed entirely by AI agents. Follow these contribution guidelines to maintain quality and consistency.

### Documentation First

- Keep documentation concise - reference ADRs for architectural details
- Update documentation before implementing features
- Remove temporary files, scripts, and test data before committing

### Consistency & Quality

- Follow existing code patterns and styles (check `package.json` for libraries)
- Run `bun run typecheck` before all commits
- Validate sources and provide references for decisions
- Use established patterns found in neighboring files

### Architectural Decisions

Always reference or create ADRs (docs/04-Architecture/ADRs/) for technical decisions:

- **ADR-001**: Monorepo structure with shared packages
- **ADR-013**: TypeScript project references for type safety
- **ADR-012**: Database schema evolution strategy
- **ADR-019**: Dashboard security (critical: DASHBOARD_API_KEY required)
- **ADR-021**: E2E testing with Playwright

### Contribution Style Guide

To ensure consistency, all contributions should adhere to the following key style points, which are derived from our established architecture and best practices. For deeper context, please refer to the linked documentation.

#### Monorepo & Code Structure

**1. Centralize Shared Logic in `packages/shared`**

- **Rationale**: Enforces a single source of truth for types and utilities shared between the `proxy` and `dashboard` services, preventing code duplication and type mismatches.
- **Reference**: [ADR-001: Monorepo Structure](docs/04-Architecture/ADRs/adr-001-monorepo-structure.md)

**2. Use TypeScript Project References for Type Checking**

- **Rationale**: Ensures correct build order and enables incremental type-checking across the monorepo, which is critical for both local development and CI performance.
- **Reference**: [ADR-013: TypeScript Project References](docs/04-Architecture/ADRs/adr-013-typescript-project-references.md)

**3. Use `kebab-case` for All New File Names**

- **Rationale**: Maintains a consistent and readable file structure across all operating systems and avoids issues with case-sensitivity in version control.
- **Reference**: [Development Guide](docs/01-Getting-Started/development.md)

#### TypeScript & Naming Conventions

**4. Adhere to Standard TypeScript Naming Conventions**

- **Rationale**: Use `PascalCase` for types and interfaces, and `camelCase` for variables and functions to maintain idiomatic and predictable code.
- **Reference**: [Development Guide: Adding a New API Endpoint](docs/01-Getting-Started/development.md#adding-a-new-api-endpoint)

**5. Use `snake_case` for API and Database Boundaries**

- **Rationale**: Creates a clear distinction between internal code (`camelCase`) and external data contracts (API JSON payloads and database columns), simplifying serialization.
- **Reference**: [API Reference](docs/02-User-Guide/api-reference.md)

#### API & Server Logic

**6. Return Standardized JSON Error Objects**

- **Rationale**: Provides consistent, predictable error feedback for all API clients, which simplifies front-end error handling and debugging.
- **Reference**: [API Reference: Error Responses](docs/02-User-Guide/api-reference.md#error-responses)

**7. Implement `/health` Endpoints for All Services**

- **Rationale**: Provides a standard, essential mechanism for monitoring service health in development, staging, and production environments.
- **Reference**: [Architecture Overview](docs/00-Overview/architecture.md#services)

**8. Externalize All Configuration via Environment Variables**

- **Rationale**: Prevents hardcoding of secrets and configuration, allowing for flexible and secure deployments across different environments.
- **Reference**: [Environment Variables Reference](docs/06-Reference/environment-vars.md)

#### Database

**9. Write Idempotent, Numerically-Prefixed Database Migrations**

- **Rationale**: Ensures that database schema changes are deterministic, repeatable, and can be safely applied without causing errors on subsequent runs.
- **Reference**: [Technical Debt Register: Database Migration Strategy](docs/04-Architecture/technical-debt.md#7--database-migration-strategy-resolved)

**10. Prevent N+1 Queries with Efficient Data Loading**

- **Rationale**: Avoids critical performance bottlenecks by fetching related data in a single query using joins or window functions instead of making multiple sequential queries.
- **Reference**: [Technical Debt Register: N+1 Query Pattern](docs/04-Architecture/technical-debt.md#2--n1-query-pattern-in-conversations-api-resolved)

#### Testing

**11. Use `data-testid` Selectors for All E2E Tests**

- **Rationale**: Decouples tests from fragile UI implementation details like CSS classes or text content, making them more resilient to refactoring and style changes.
- **Reference**: [ADR-021: E2E Testing Strategy](docs/04-Architecture/ADRs/adr-021-e2e-testing-strategy.md)

**12. Separate Test Files by Suffix: `.spec.ts` vs. `.test.ts`**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Moonsong-Labs/agent-prompttrain](https://github.com/Moonsong-Labs/agent-prompttrain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
