---
trigger: always_on
description: This document provides guidelines for AI coding agents working in the aws-sdk-vitest-mock repository.
---

# AWS SDK Vitest Mock - Agent Guidelines

This document provides guidelines for AI coding agents working in the aws-sdk-vitest-mock repository.

<!-- nx configuration start-->
<!-- Leave the start & end comments to automatically receive updates. -->

## General Guidelines for working with Nx

- When running tasks (for example build, lint, test, e2e, etc.), always prefer running the task through `nx` (i.e. `nx run`, `nx run-many`, `nx affected`) instead of using the underlying tooling directly
- You have access to the Nx MCP server and its tools, use them to help the user
- When answering questions about the repository, use the `nx_workspace` tool first to gain an understanding of the workspace architecture where applicable.
- When working in individual projects, use the `nx_project_details` mcp tool to analyze and understand the specific project structure and dependencies
- For questions around nx configuration, best practices or if you're unsure, use the `nx_docs` tool to get relevant, up-to-date docs. Always use this instead of assuming things about nx configuration
- If the user needs help with an Nx configuration or project graph error, use the `nx_workspace` tool to get any errors

<!-- nx configuration end-->

## Project Overview

This is a TypeScript library that provides type-safe mocking utilities for AWS SDK v3 clients with Vitest. It uses Nx as a build system and monorepo tool.

## Build, Test, and Lint Commands

### Running Tests

- Run all tests: `bun nx test`
- Run tests in watch mode: `vitest --watch`
- Run a single test file: `bun nx test --testFile=src/lib/mock-client-basic.spec.ts`
- Run a specific test: `vitest run -t "test name pattern"`
- Run tests with coverage: `bun nx test --coverage`

### Building

- Build the library: `bun nx build`
- Build output goes to `./dist` directory
- Build creates both ES modules (.js) and CommonJS (.cjs) formats

### Linting

- Lint all files: `bun nx lint`
- Auto-fix linting issues: `bun nx lint --fix`
- Lint runs on TypeScript files in `./src` and `./package.json`

### Other Commands

- Generate documentation: `bun run docs` (uses TypeDoc)
- Watch documentation: `bun run docs:watch`
- Install dependencies: `bun install --frozen-lockfile`
- Format code: `prettier --write <files>`

## Code Style Guidelines

### TypeScript Configuration

- Target: ES2020 (updated from ES2015 for modern JavaScript features)
- Lib: es2020, es2022.array (for Array.at() support)
- Module resolution: nodenext
- Source files in: `src/`
- Use `.ts` extension for source files
- Use `.spec.ts` extension for test files
- Use `.mts` extension for config files (vite.config.mts, eslint.config.mts)

### Import Style

- Import order is enforced and auto-fixed by ESLint
- Order: builtin → external → internal → parent → sibling → index
- No newlines between import groups
- Alphabetize imports (case-insensitive, ascending)
- Always use file extensions in imports (.js, not .ts) due to ES modules

Example:

```typescript
import type { MetadataBearer } from "@smithy/types";
import { type Mock, vi } from "vitest";
import { createDebugLogger } from "./utils/debug-logger.js";
import { loadFixture } from "./utils/file-helpers.js";
```

### Naming Conventions

- Files: kebab-case (e.g., `mock-client.ts`, `debug-logger.spec.ts`)
- Types/Interfaces: PascalCase (e.g., `AwsClientStub`, `CommandInputType`)
- Functions/Variables: camelCase (e.g., `mockClient`, `getEffectiveDebugState`)
- Constants: camelCase (e.g., `globalDebugEnabled`)
- Test files: `*.spec.ts`

### Type Usage

- Use TypeScript strict type checking (enabled via tsconfig)
- Prefer `type` imports for type-only imports: `import type { Foo } from "..."`
- Use explicit return types for public API functions
- Use `DeepPartial<T>` utility type for partial mock responses
- Disable `@typescript-eslint/no-explicit-any` only in test files
- Use generics for command constructors and inputs/outputs

Example:

```typescript
export function mockClient<TClient extends object>(
  clientCtor: new (config?: unknown) => TClient,
): AwsClientStub<TClient>;
```

### Formatting

- Prettier is used with minimal config
- Double quotes for strings (singleQuote: false)
- Auto-formatting happens via lint-staged on commit
- Run `prettier --write` for manual formatting

### Testing Patterns

- Use Vitest for all tests
- Import test utilities: `import { expect, test, beforeEach, afterEach, describe } from "vitest"`
- Use `beforeEach` for mock setup, `afterEach` for cleanup
- Always restore mocks: `mock.restore()` in afterEach
- Use descriptive test names: `test("should mock S3Client send method", ...)`
- Use custom matchers: `expect(mock).toHaveReceivedCommand(Command)`
- Extend matchers in setup: `import "./vitest-setup.js"`

### Code Quality

- ESLint plugins enabled: security, import, unicorn, sonarjs, vitest
- Security rules from eslint-plugin-security
- SonarJS rules for code smells
- Unicorn plugin with some rules disabled (prevent-abbreviations, no-this-assignment)
- Report unused ESLint disable directives as errors
- Vitest recommended rules for test files

### Documentation

- Use JSDoc comments for public API functions
- Include `@example` blocks in JSDoc

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sudokar/aws-sdk-vitest-mock](https://github.com/sudokar/aws-sdk-vitest-mock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
