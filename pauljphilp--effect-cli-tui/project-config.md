---
trigger: always_on
description: This project, `effect-cli-tui`, is a TypeScript library for building powerful and interactive command-line interfaces (CLIs). It is built on top of the Effect-TS library, which provides a powerful a functional, type-safe, and composable way to write asynchronous and concurrent code.
---

# GEMINI.md

## Project Overview

This project, `effect-cli-tui`, is a TypeScript library for building powerful and interactive command-line interfaces (CLIs). It is built on top of the Effect-TS library, which provides a powerful a functional, type-safe, and composable way to write asynchronous and concurrent code.

The library provides a comprehensive set of tools for building CLIs, including:

*   **Interactive Prompts:** For gathering user input.
*   **Display Utilities:** For printing formatted output to the console.
*   **CLI Wrapper:** For executing external commands.
*   **Tables, Boxes, and Spinners:** For creating rich terminal user interfaces.

## Building and Running

The project uses `bun` as its package manager. The following commands are used for building, running, and testing the project:

*   **Install Dependencies:**
    ```bash
    bun install
    ```
*   **Build:**
    ```bash
    bun run build
    ```
*   **Run Tests:**
    ```bash
    bun run test
    ```
*   **Lint:**
    ```bash
    bun run lint
    ```
*   **Type Check:**
    ```bash
    bun run type-check
    ```

## Development Conventions

*   **Code Style:** The project uses Biome for code linting and formatting. (Note: Biome configuration files, such as `biome.json` or `.biomeignore`, were not found in the initial analysis. If you intend to use Biome, please ensure its configuration is present.)
*   **Testing:** The project uses `vitest` for testing. Test files are located in the `__tests__` directory.
*   **Commits:** The project follows the Conventional Commits specification for commit messages.
*   **Branching:** Feature branches should be created from the `main` branch.
*   **Pull Requests:** Pull requests should be opened against the `main` branch.


# Ultracite Code Standards

This project uses **Ultracite**, a zero-config Biome preset that enforces strict code quality standards through automated formatting and linting.

## Quick Reference

- **Format code**: `npx ultracite fix`
- **Check for issues**: `npx ultracite check`
- **Diagnose setup**: `npx ultracite doctor`

Biome (the underlying engine) provides extremely fast Rust-based linting and formatting. Most issues are automatically fixable.

---

## Core Principles

Write code that is **accessible, performant, type-safe, and maintainable**. Focus on clarity and explicit intent over brevity.

### Type Safety & Explicitness

- Use explicit types for function parameters and return values when they enhance clarity
- Prefer `unknown` over `any` when the type is genuinely unknown
- Use const assertions (`as const`) for immutable values and literal types
- Leverage TypeScript's type narrowing instead of type assertions
- Use meaningful variable names instead of magic numbers - extract constants with descriptive names

## TypeScript Rules & Effect-TS Patterns

### Type Safety & Configuration

- Enable `strict: true` in tsconfig.json with:
  - `noImplicitAny: true`
  - `strictNullChecks: true`
  - `strictFunctionTypes: true`
  - `exactOptionalPropertyTypes: true`
- Use `--noEmitOnError` compiler flag
- Never use `// @ts-ignore` without explanatory comments

### Type Definitions

- **Never use `any`** - use `unknown` with type guards instead
- **Never use Enums** - use union types: `type Status = "active" | "inactive"`
- Use `readonly` for immutable properties and arrays
- Explicitly type function parameters and return types
- Use discriminated unions with exhaustiveness checking

### Interfaces vs Types

- **Use `interface` for object contracts** (all `api.ts` files)
- Use `type` only for: unions, intersections, branded types
- Add `// biome-ignore lint: Interface preferred for object contracts` when needed

```typescript
// Good
export interface HttpClientApi {
  readonly request: <T>(path: string) => Effect.Effect<T, HttpClientError>;
}

// Bad - Don't use type for object contracts
export type HttpClientApi = { ... }
```

### Service Pattern

Use `Effect.Service` with `Effect.fn()`:

```typescript
export class ServiceName extends Effect.Service<ServiceName>()("ServiceName", {
  effect: Effect.fn(function* (config: ConfigType) {
    return { ... } satisfies ServiceApi;
  }),
}) {}
```

### Error Types

Use `Data.TaggedError` for discriminated errors:

```typescript
export class MemoryNotFoundError extends Data.TaggedError("MemoryNotFoundError")<{
  readonly key: string;
}> {}

export type MemoryError = MemoryNotFoundError | MemoryValidationError;
```

### Branded Types

Use `Brand.refined` for validated types:

```typescript
export type Namespace = string & Brand.Brand<"Namespace">;
export const Namespace = Brand.refined<Namespace>(
  (ns) => ns.length > 0 && /^[a-zA-Z0-9_-]+$/.test(ns),
  (ns) => Brand.error("Invalid namespace")
);
```

### File Structure

```
services/[name]/
├── api.ts       # Interface contract (use interface, not type)
├── errors.ts    # Data.TaggedError classes
├── types.ts     # Branded types, config types
├── service.ts   # Effect.Service implementation
├── helpers.ts   # Pure utility functions

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PaulJPhilp/effect-cli-tui](https://github.com/PaulJPhilp/effect-cli-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
