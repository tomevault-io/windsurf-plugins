---
trigger: always_on
description: This document provides guidance for AI coding agents working with the LangChain.js codebase.
---

# AGENTS.md - AI Agent Guidelines for LangChain.js

This document provides guidance for AI coding agents working with the LangChain.js codebase.

## Project Overview

LangChain.js is a TypeScript framework for building LLM-powered applications. It provides standard interfaces for agents, models, embeddings, vector stores, and more, enabling developers to chain together interoperable components and third-party integrations.

### Supported Environments

- Node.js (ESM and CommonJS) - 20.x, 22.x, 24.x
- Cloudflare Workers
- Vercel / Next.js (Browser, Serverless and Edge functions)
- Supabase Edge Functions
- Browser
- Deno
- Bun

## Repository Structure

This is a **monorepo** managed with [pnpm workspaces](https://pnpm.io/) (v10.14.0) and [Turborepo](https://turbo.build/).

### Key Packages

| Package                    | Path                                  | Description                                                      |
| -------------------------- | ------------------------------------- | ---------------------------------------------------------------- |
| `langchain`                | `libs/langchain/`                     | Main LangChain package with agents, prompts, and orchestration   |
| `@langchain/core`          | `libs/langchain-core/`                | Core abstractions and interfaces (base classes, runnables, etc.) |
| `@langchain/textsplitters` | `libs/langchain-textsplitters/`       | Text splitting utilities                                         |
| `@langchain/openai`        | `libs/providers/langchain-openai/`    | OpenAI integration                                               |
| `@langchain/anthropic`     | `libs/providers/langchain-anthropic/` | Anthropic integration                                            |
| Other providers            | `libs/providers/langchain-*/`         | First-party provider integrations                                |

### Internal Packages

| Package                     | Path                             | Description                          |
| --------------------------- | -------------------------------- | ------------------------------------ |
| `@langchain/build`          | `internal/build/`                | Build utilities                      |
| `@langchain/tsconfig`       | `internal/tsconfig/`             | Shared TypeScript configuration      |
| `@langchain/standard-tests` | `libs/langchain-standard-tests/` | Standard test suite for integrations |

## Development Setup

### Prerequisites

- **Node.js v24.x** (check with `node -v`)
- **pnpm v10.14.0** (package manager)

### Initial Setup

```bash
# Install dependencies from root
pnpm install

# Build the core package first (required before other packages)
pnpm --filter @langchain/core build
```

## Common Commands

All commands can be run from the project root using `pnpm --filter <package>` to target specific workspaces.

### Package Filters

- `--filter langchain` - the main `langchain` package
- `--filter @langchain/core` - the core package
- `--filter @langchain/openai` - OpenAI integration (and similarly for other providers)

### Building

```bash
pnpm --filter langchain build
pnpm --filter @langchain/core build
```

### Linting

```bash
pnpm lint
```

### Formatting

```bash
pnpm format        # Fix formatting
pnpm format:check  # Check only
```

### Testing

```bash
# Unit tests
pnpm --filter langchain test
pnpm --filter @langchain/core test

# Integration tests (requires API keys)
pnpm --filter langchain test:integration

# Single test file
pnpm --filter <package> test:single <path-to-test>
```

## Coding Standards

### TypeScript Configuration

The project uses a shared TypeScript configuration from `internal/tsconfig/base.json`:

- Target: ES2022
- Module: ESNext with bundler resolution
- Strict mode enabled
- Source maps and declaration maps enabled

### Lint Rules

Lint rules are defined in `.oxlintrc.json`. Key rules to follow:

1. **No `process.env`** - Except in test files (`node/no-process-env: error`)
2. **No explicit `any`** - Use proper types (`typescript/no-explicit-any: error`)
3. **Prefer template literals** - Over string concatenation (`prefer-template: error`)
4. **File extensions required** - In imports (`import/extensions: error`)

### Import Conventions

```typescript
// Always include .js extension for local imports (ESM)
import { Something } from "./something.js";

// Use named exports, not default exports
export { MyClass, myFunction };
```

### Zod Schema Support

The codebase supports both Zod v3 and v4:

```typescript
import { z } from "zod/v3";
import { z as z4 } from "zod/v4";
```

## File Naming Conventions

### Source Files

- Regular modules: `my_module.ts` (snake_case)
- Index files: `index.ts`
- Type definitions: `types.ts`

### Test Files

- **Unit tests**: `*.test.ts` - Tests that don't require external APIs
- **Integration tests**: `*.int.test.ts` - Tests that call external APIs
- **Type tests**: `*.test-d.ts` - TypeScript type checking tests
- **Standard tests**: `*.standard.test.ts` / `*.standard.int.test.ts` - Standard test suite

Tests should be placed in a `tests/` folder alongside the module being tested.

## Core Abstractions

### Runnables


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [langchain-ai/langchainjs](https://github.com/langchain-ai/langchainjs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
