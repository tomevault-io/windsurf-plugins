---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A minimal Node.js + TypeScript project template using modern tooling and strict TypeScript configuration.

## Requirements

- Node.js ≥ 24.7.0
- pnpm ≥ 10.28.0

## Development Commands

### Building
```bash
pnpm build
```
Uses tsdown for fast TypeScript bundling.

### Testing
```bash
pnpm test                 # Run all tests
pnpm test.watch           # Run tests in watch mode
```
Uses Node.js native test runner with tsx loader. Test files follow the pattern `src/**/*.test.{ts,tsx}`.

### Linting
```bash
pnpm lint            # Check code with Biome
pnpm lint.fix        # Auto-fix linting issues
```

**Important:** This project uses Biome for linting and formatting, NOT ESLint. Always use Biome commands and configuration. When adding lint ignore comments, use Biome's syntax (e.g., `// biome-ignore lint/...` not `// eslint-disable`).

## Architecture

### TypeScript Configuration

Strict TypeScript setup with:
- `noUncheckedIndexedAccess: true` - Array/object access returns possibly undefined
- `noImplicitReturns: true` - All code paths must return
- `strict: true` - All strict type-checking enabled
- `verbatimModuleSyntax: true` - Import/export syntax preserved
- `isolatedModules: true` - Each file must be transpilable independently
- `noUncheckedSideEffectImports: true` - Side-effect imports must be explicit

Path aliases:
- `@/*` maps to `./src/*`
- `~/*` maps to `./*`

### Code Style

**Linting Tool:** This project uses Biome (NOT ESLint) for all linting and formatting.

Biome enforces:
- Tab indentation
- Double quotes for strings
- Automatic import organization
- Recommended linting rules

When ignoring lint rules, use Biome syntax:
- ✅ Correct: `// biome-ignore lint/correctness/noChildrenProp: testing edge case`
- ❌ Wrong: `// eslint-disable-next-line`

### Project Structure

- `src/` - Source TypeScript files
- `dist/` - Build output (excluded from TypeScript compilation)
- `playground/` - Development playground and examples
  - `app.tsx` - Main playground entry point
  - `recipes/` - Standalone example recipes demonstrating patterns
- ES modules only (`"type": "module"` in package.json)

### Recipes

The `playground/recipes/` folder contains standalone examples demonstrating specific patterns or features.

**Rules for recipes:**
- Every recipe must have a co-located README file named `{recipe-name}.readme.md`
- Recipe files should be self-contained and runnable
- Include a JSDoc header comment in the `.tsx` file explaining the recipe's purpose
- READMEs should include: Overview, Problem, Solution, How It Works, and When to Use sections

## Coding Standards

### Architecture & Patterns
- Use a hybrid approach combining functional and object-oriented programming
- Effect (effect.website) is the core library - use its patterns throughout
- Prefer Effect's error handling over try/catch (except when it significantly hurts ergonomics)
- Use Services and Layers for dependency injection
- Prefer `pipe(effect, ...)` over `effect.pipe(...)`

### TypeScript Standards
- Type assertions (`as`, `!`) only when we're "smarter" than the compiler
- `any` is allowed for generic type params and library interop only
- Use explicit type guards over implicit checks
- Prefer generic constraints over flexibility
- Treat data structures as immutable - use `readonly` extensively
- Prefer `Option` > `undefined` > `null` for optional values
- All checks should be type-level when possible
- Use Schema for validation of unknowns and I/O

### Naming Conventions
- Files: kebab-case (e.g., `user-service.ts`)
- Variables/functions: camelCase, with `is*`, `has*`, `should*` prefixes for booleans
- Types/Interfaces: PascalCase, no `I` prefix for interfaces
- Constants (shared): UPPER_SNAKE_CASE
- Prefer named exports; default exports only if absolutely necessary

### Documentation
- All exported functions, types, and values must have JSDoc comments
- JSDoc `@type` annotations can be omitted (TypeScript handles types)
- Include text descriptions for parameters when not self-explanatory
- Inline comments only when needed - avoid commenting obvious code
- TODOs and FIXMEs are acceptable
- Effect Schemas should include descriptions/annotations when not self-explanatory

### Testing
- Follow Test-Driven Development workflow: spec → mock → test → implement
- Co-locate test files (`*.test.ts`) next to source code
- `__tests__/` directory allowed for compound/integration tests and shared fixtures/helpers
- `__type-tests__/` directory for compile-time type tests (see Type Tests section below)
- Write thorough tests against the API surface and specifications in co-located `specs.md` files
- Test naming conventions:
  - Use `describe` for test grouping, `it` or `test` for individual test cases
  - Test case names should match or reference acceptance criteria from specs.md
- Coverage requirements:
  - All acceptance criteria from specs.md must be covered
  - Cover both happy paths and error paths
  - Test all possible error types defined in the Effect error union (expected errors)
  - Include edge cases defined in specifications

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stefvw93/effect-ui](https://github.com/stefvw93/effect-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
