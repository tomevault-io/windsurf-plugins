---
trigger: always_on
description: This file provides context for AI coding assistants working with the Yantra repository.
---

# AGENTS.md

This file provides context for AI coding assistants working with the Yantra repository.

## Project Overview

**Yantra** is a minimalist collection of `@ai-yantra/` scoped extensions for the AI SDK. We don't just build tools—we craft the invisible threads that connect intelligence to action.

- **Repository**: https://github.com/dumbmachine/ai-sdk-clothes
- **Inspiration**: Anthropic's advanced tool use engineering (https://www.anthropic.com/engineering/advanced-tool-use)
- **License**: MIT (assumed)

## Repository Structure

This is a **monorepo** using pnpm workspaces.

### Key Directories

| Directory               | Package Name           | Description                                      |
| ----------------------- | ---------------------- | ------------------------------------------------ |
| `packages/pg-fs`        | `@ai-yantra/pg-fs`        | PostgreSQL-backed filesystem with AI SDK tools   |
| `packages/memory`       | `@ai-yantra/memory`       | AI SDK Memory Tools backed by SQLite via pg-fs   |
| `packages/skills`       | `@ai-yantra/skills`       | Skill discovery + loading for AI SDK agents      |
| `packages/tool-search`  | `@ai-yantra/tool-search`  | Tool Search utility package                      |
| `packages/ptc`          | `@ai-yantra/ptc`          | Programmable Tool Calling (live)                 |
| `packages/demo`         | `@ai-yantra/demo`         | Demo application (private)                       |

## Development Setup

### Requirements

- **Node.js**: v18 or higher
- **pnpm**: v8+ (`npm install -g pnpm`)

### Initial Setup

```bash
pnpm install        # Install all dependencies
pnpm build          # Build all packages
```

## Development Commands

### Root-Level Commands

| Command            | Description                    |
| ------------------ | ------------------------------ |
| `pnpm install`     | Install dependencies           |
| `pnpm build`       | Build all packages             |
| `pnpm lint`        | Run linting across workspace   |

### Package-Level Commands

Run these from within a package directory (e.g., `packages/pg-fs`):

| Command              | Description                                      |
| -------------------- | ------------------------------------------------ |
| `pnpm build`         | Build the package (TypeScript compilation)       |
| `pnpm test`          | Run all tests                                     |
| `pnpm test:watch`    | Run tests in watch mode                          |
| `pnpm test:ui`       | Run tests with UI interface (Vitest)             |
| `pnpm lint`          | Type-check with TypeScript (noEmit mode)         |
| `pnpm test -- <pattern>` | Run single test file (e.g., `pnpm test -- utils.test.ts`) |
| `pnpm test --run --reporter=verbose <pattern>` | Run specific test with verbose output |

### Database Commands (pg-fs package)

| Command              | Description                                      |
| -------------------- | ------------------------------------------------ |
| `pnpm db:generate`   | Generate database migrations                     |
| `pnpm db:push`       | Push schema changes to database                  |
| `pnpm db:studio`     | Open Drizzle Studio for database management      |

## Coding Standards

### TypeScript Configuration

- **Target**: ES2022 modules with NodeNext module resolution
- **Strict Mode**: Enabled for all packages
- **Declaration Files**: Generated with source maps
- **Imports**: Use named imports, relative paths for local modules
- **File Extensions**: Use `.js` extensions in imports for ESM compatibility

### Code Style Guidelines

#### Imports and Exports
```typescript
// ✅ Good: Named imports, relative paths
import { Pool } from "pg";
import { drizzle } from "drizzle-orm/node-postgres";
import * as schema from "./schema.js";
import { PgFileSystem } from "./db-fs.js";

// ❌ Bad: Default imports, absolute paths
import pg from "pg";
import drizzle from "drizzle-orm/node-postgres";
import * as schema from "../../schema.js";
```

#### Naming Conventions
- **Variables/Functions**: camelCase (`normalizePath`, `createFileSystemTools`)
- **Types/Interfaces/Classes**: PascalCase (`PgFsConfig`, `FileSystemUtils`)
- **Constants**: UPPER_SNAKE_CASE (`MAX_FILE_SIZE`)
- **Private Members**: Prefix with underscore (`_internalMethod`)

#### Error Handling
```typescript
// ✅ Good: Specific error types, descriptive messages
try {
  await fsOperation();
} catch (error) {
  if (error instanceof ValidationError) {
    throw new PgFsError(`Invalid path format: ${error.message}`);
  }
  throw new PgFsError(`Filesystem operation failed: ${error.message}`);
}

// ❌ Bad: Generic error handling
try {
  await fsOperation();
} catch (error) {
  throw new Error("Something went wrong");
}
```

#### Type Safety
- Use `strict: true` TypeScript configuration
- Leverage Zod for runtime validation of external inputs
- Prefer union types over `any`
- Use branded types for domain-specific strings

```typescript
// ✅ Good: Branded types and validation
import { z } from "zod";

const PathSchema = z.string().refine(path => path.startsWith("/"), "Path must be absolute");
type Path = z.infer<typeof PathSchema> & { readonly __brand: "Path" };


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DumbMachine/ai-yantra](https://github.com/DumbMachine/ai-yantra) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
