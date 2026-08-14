---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Development

- `pnpm run dev` - Start all apps in development mode (API on port 3001)
- `pnpm run dev --filter=api` - Start only the NestJS API with Fastify
- `pnpm prepare` - Install ts-patch and typia patch (required for Nestia)

### Building

- `pnpm run build` - Build all apps and packages
- `pnpm run build --filter=<app-name>` - Build specific app (ensure packages are built first)
- `pnpm nestia:build` - Build Nestia SDK for type-safe API client
- `pnpm nestia:swagger` - Generate Swagger documentation

### Testing

- `pnpm run test` - Run unit tests using vitest
- `pnpm run test:watch` - Run tests in watch mode
- `pnpm run test:cov` - Run tests with coverage report
- `pnpm run test:e2e` - Run end-to-end tests
- `pnpm run test --filter=<app-name>` - Run tests for specific app/package
- `pnpm test <test-file-name>` - Run a single test file (from apps/api directory)

### Code Quality

- `pnpm run lint` - Run ESLint on all packages
- `pnpm format` - Format code with Prettier (formats `**/*.{ts,tsx,md,json}`)
- `pnpm type-check` - Run TypeScript type checking

### Nestia-specific Commands

- `pnpm nestia:build` - Generate SDK from controllers
- `pnpm nestia:swagger` - Generate OpenAPI specification
- `pnpm nestia:e2e` - Generate E2E test scaffolding

## Architecture

### Monorepo Structure

Turborepo-based monorepo with:

- **apps/**
  - `api` - NestJS backend API with Fastify adapter (port 3001)
- **packages/** - Shared code with `@shop-be/` namespace:
  - `eslint-config` - ESLint configurations with Prettier
  - `typescript-config` - Shared TypeScript configs with Nestia support
  - `vitest-config` - Vitest configurations for testing

### Technology Stack

- **Package Manager**: pnpm v8.15.5
- **Build System**: Turborepo
- **Backend Framework**: NestJS v11 with TypeScript
- **HTTP Adapter**: Fastify (performance-optimized)
- **Type Safety**: Nestia for type-safe API development with runtime validation
- **Testing**: Vitest (unit and E2E)
- **Validation**: Typia (used by Nestia)
- **Node Version**: >=18

### Key Patterns

**API Structure (NestJS with Nestia)**:

- Controllers use `@TypedRoute` decorators from Nestia for type-safe routing
- Nestia provides automatic validation and SDK generation
- Test setup includes mocks for Nestia decorators in `test/setup.ts`
- Modular architecture with feature modules
- Services contain business logic
- Configuration via environment variables

**Testing Approach**:

- TDD (Test-Driven Development) methodology
- Vitest for fast unit testing
- Custom setup file mocks Nestia decorators for testing
- E2E testing planned with Nestia SDK integration

**Package Dependencies**:

- Internal packages use `workspace:*` protocol
- Shared configs extend from base configurations
- TypeScript config includes Nestia transformers

### Development Workflow

1. Install dependencies: `pnpm install`
2. Run prepare script: `pnpm prepare` (installs ts-patch for Nestia)
3. Start development: `pnpm dev --filter=api`
4. Write tests first (TDD approach)
5. Run tests: `pnpm test` or `pnpm test:watch`
6. Build Nestia SDK when needed: `pnpm nestia:build`

### Configuration Files

- **ESLint**: Root-level flat config with NestJS optimizations
- **Prettier**: Config at `prettier.config.js`
- **TypeScript**: Uses `@shop-be/typescript-config/nestia.json` for Nestia support
- **Vitest**: Config from `@shop-be/vitest-config`
- **Nestia**: Config at `nestia.config.ts`

### Current Implementation Status

- Fastify platform migration completed ✅
- Nestia basic setup completed ✅
- TypedRoute decorators mocked for testing ✅
- 18 tests passing ✅
- Nestia SDK generation pending
- E2E test integration pending

### Turbo Pipeline

- `dev`: No cache, persistent process
- `build`: Depends on upstream builds
- `lint`, `test`, `test:e2e`: Standard tasks with caching
- `prepare`: Run for all packages

### Important Notes

- Always run `pnpm prepare` after fresh install to enable Nestia transformers
- Use vitest for testing (not Jest)
- @TypedRoute decorators are mocked in tests via global setup
- Port 3001 is used for API development
- Always follow the instructions in plan.md. When I say "go", find the next unmarked test in plan.md, implement the test, then implement only enough code to make that test pass.

## TDD METHODOLOGY

### Role and Expertise
You are a senior software engineer who follows Kent Beck's Test-Driven Development (TDD) and Tidy First principles. Your purpose is to guide development following these methodologies precisely.

### Core Development Principles
- Always follow the TDD cycle: Red → Green → Refactor
- Write the simplest failing test first
- Implement the minimum code needed to make tests pass
- Refactor only after tests are passing
- Follow Beck's "Tidy First" approach by separating structural changes from behavioral changes
- Maintain high code quality throughout development

### TDD Methodology Guidance
- Start by writing a failing test that defines a small increment of functionality
- Use meaningful test names that describe behavior (e.g., "shouldSumTwoPositiveNumbers")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [eannnnnn/shop-be](https://github.com/eannnnnn/shop-be) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
