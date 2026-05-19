---
trigger: always_on
description: A TypeScript CLI application using citty for command handling and consola for terminal output, following Test-Driven Development, Clean Architecture, and strict validation workflows.
---


# CLI Application

A TypeScript CLI application using citty for command handling and consola for terminal output, following Test-Driven Development, Clean Architecture, and strict validation workflows.

## Commands

Run `nvm use` before any npm command. During development, use file-scoped commands for faster feedback, and run the full validation suite (`npx biome check && npm test && npm run build`) before commits.

```bash
# ALWAYS run first
nvm use

# Core commands
npm install              # Install deps (updates package-lock.json)
npm test                 # Run tests (vitest)
npm run test:e2e         # Run end-to-end tests (vitest e2e config)
npm run build            # Production build
npm run dev              # Start development with hot reload
npx biome check          # Lint check
npx biome check --write  # Auto-fix lint/format
npm run lint:fix         # Auto-fix lint/format (alias for biome check --write)

# File-scoped (faster feedback)
npx biome check path/to/file.ts
npm test path/to/file.test.ts

# Validation suite (run before commits)
npx biome check && npm test && npm run build

# Other
npm audit                # Security check
npm run lint:workflows   # Validate GitHub Actions (actionlint)
npm run lint:yaml        # Validate YAML (yamllint)
```

## Workflow: TDD Required

Follow this exact sequence for ALL code changes. Work in small increments — make one change at a time and validate before proceeding.

1. **Research**: Search codebase for existing patterns, commands, utilities. Use Context7 MCP tools for library/API documentation.
2. **Write failing test**: Create test describing desired behavior
3. **Verify failure**: Run `npm test` — confirm clear failure message
4. **Implement minimal code**: Write just enough to pass
5. **Verify pass**: Run `npm test` — confirm pass
6. **Refactor**: Clean up, remove duplication, keep tests green
7. **Validate**: `npx biome check && npm test && npm run build`

Task is NOT complete until all validation passes.

## Tech Stack

- **Framework**: citty — lightweight CLI framework with command definitions and argument parsing
- **Language**: TypeScript (strict mode)
- **Terminal Output**: consola — elegant console logging with levels and formatting
- **Validation**: Zod for runtime validation of external data
- **Testing**: Vitest (never Jest), Chance.js for test fixtures
- **Linting**: Biome (never ESLint/Prettier separately)
- **HTTP**: fetch API only (for external service calls)
- **Architecture**: Clean Architecture principles

## Project Structure

```
.github/           GitHub Actions workflows
src/               Application source code
  entities/        Layer 1: Business domain entities
  use-cases/       Layer 2: Application business rules
  gateways/        Layer 3: External system adapters
  commands/        Layer 3: CLI command handlers
  lib/             Utilities and helpers
  index.ts         Application entry point
tests/             Test files (mirror src/ structure)
.nvmrc             Node.js version (latest LTS)
```

## Code Style

```typescript
import { defineCommand } from 'citty';
import { consola } from 'consola';
import { z } from 'zod';

// Define schema for runtime validation
const ConfigSchema = z.object({
  name: z.string(),
  version: z.string(),
});

type Config = z.infer<typeof ConfigSchema>;

// ✅ Good - small, typed, single purpose, descriptive names, runtime validation
async function loadConfig(filePath: string): Promise<Config> {
  if (!filePath) {
    throw new Error('File path required');
  }

  const content = await readFile(filePath, 'utf-8');
  const data: unknown = JSON.parse(content);
  return ConfigSchema.parse(data);
}

// ❌ Bad - untyped, no validation, multiple responsibilities
async function doStuff(x) {
  console.log('loading');
  const data = JSON.parse(await readFile(x));
  return data as Config;
}
```

**Rules:**
- Always use TypeScript type hints
- Use descriptive names for variables, functions, and modules
- Functions must be small and have single responsibility
- Avoid god functions and classes — break into smaller, focused units
- Avoid repetitive code — extract reusable functions
- Extract functions when there are multiple code paths
- Favor immutability and pure functions
- Avoid temporal coupling
- Keep cyclomatic complexity low
- Remove all unused imports and variables
- Validate external data at runtime with Zod — never use type assertions (`as Type`) on API responses
- Run lint and tests after EVERY change

## Testing Standards

Tests are executable documentation. Use Arrange-Act-Assert pattern. Generate test fixtures with Chance.js.

```typescript
import Chance from 'chance';
import { describe, it, expect, vi } from 'vitest';
import { createConfigLoader } from './config-loader';

const chance = new Chance();

// ✅ Good - describes behavior, uses generated fixtures, mocks dependencies
describe('Config Loader', () => {
  describe('given a valid config file path', () => {
    it('loads and validates the configuration', async () => {
      // Arrange
      const configPath = chance.word() + '.json';
      const expectedConfig = {
        name: chance.word(),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zpratt/lousy-iam](https://github.com/zpratt/lousy-iam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
