---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Intent is a TypeScript library that uses LLMs to rerank arbitrary items based on relevance to a query. It scores items 0-10, filters by threshold, and returns results sorted by score with stable ordering on ties.

## Core Principles

**Quality First**: This library maintains strict quality standards:

- ✅ 100% test coverage (unit tests)
- ✅ Zero TypeScript errors
- ✅ Clean linting (Prettier + ESLint)
- ✅ Comprehensive JSDoc on all functions

**Code Style Fundamentals**:

- **Small Functions**: Target 20-30 lines, maximum 50 lines per function
- **Small Files**: Under 300 lines per file
- **Early Returns**: Always prefer early returns over nested conditionals
- **Always Use Braces**: No single-line if statements, even for simple returns
- **Minimal Nesting**: Maximum 2-3 levels of nesting depth
- **Document Everything**: Every function (public and private) must have JSDoc
- **Many Small Helpers**: Prefer many focused functions over few large ones

## Commands

### Build & Type Checking

- `npm run build` - Compile TypeScript to dist/
- `npm run typecheck` - Type check without emitting files
- `npm run clean` - Remove dist/ folder

### Testing

- `npm test` - Run all tests with coverage (unit + integration)
- `npm run test:unit` - Unit tests only (100% coverage enforced)
- `npm run test:int` - Integration tests only (requires `GROQ_API_KEY`)
- Test files are co-located: `*.unit.test.ts` for unit, `*.int.test.ts` for integration

### Linting

- `npm run lint` - Auto-fix linting and formatting issues
- `npm run lint:check` - Check without fixing
- **Line Length**: Prettier configured for 100-character line width (`printWidth: 100`)

## Architecture

### Core Reranking Flow

The `Intent` class orchestrates the entire ranking process:

1. **Preparation** (`prepareCandidates`): Normalizes items into a consistent shape with key/summary/index
2. **Batching** (`batchProcess` in batches.ts): Splits candidates into batches, merges tiny trailing batches to avoid inefficient LLM calls
3. **Batch Processing** (`processBatch`):
   - Ensures unique keys by appending index to duplicates: `"Key (idx)"`
   - Builds JSON schema (`buildRelevancySchema`) with required properties for each candidate key
   - Builds chat messages (`buildMessages`) with system prompt + user query/candidates
   - Calls LLM via pluggable `LlmClient` interface
   - Filters by relevancy threshold and sorts (score desc, then input order)
4. **Fallback**: On any error, preserves original order for affected batch/entire input

### LLM Client Architecture

- **Interface**: `LlmClient` (types.ts) - single `call()` method accepting messages, JSON schema, config, userId
- **Default Provider**: Groq adapter (providers/groq.ts) with JSON schema response format and retry logic for validation failures
- **Selection Logic**: `selectLlmClient()` chooses ctx.llm if provided, else creates Groq client if `GROQ_API_KEY` exists, else undefined

### Configuration System

All config lives in config.ts using lib/config helpers:

- Environment variables: `INTENT_MODEL`, `INTENT_TIMEOUT_MS`, `INTENT_RELEVANCY_THRESHOLD`, `INTENT_BATCH_SIZE`, `INTENT_TINY_BATCH_FRACTION`
- Groq-specific: `GROQ_API_KEY`, `GROQ_DEFAULT_MODEL`, `GROQ_DEFAULT_TEMPERATURE`
- Config is loaded automatically via `dotenv/config` import at top of config.ts
- Intent constructor accepts options as a single optional object

**Config Naming Convention**: By design, config keys in `CONFIG` use `UPPER_SNAKE_CASE` to mirror their environment variable names (e.g., `CONFIG.INTENT.MODEL` matches `INTENT_MODEL`). The user-facing API uses `camelCase` (e.g., `options.model`). This intentional distinction keeps config keys aligned with environment variables while providing an ergonomic API. Internal code converts between these formats as needed.

### Key Design Patterns

- **Stable fallbacks**: Any failure (LLM error, timeout, invalid response) returns items in original order
- **Duplicate key handling**: Internal disambiguation using `"Key (idx)"` suffix
- **Strict typing**: Uses TypeScript strict mode with `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`
- **Generic item support**: Intent is `Intent<T>` with user-provided key/summary extractors
- **Zero/one-item fast path**: Avoids LLM calls when unnecessary

## Testing Strategy

- **Unit tests**: Mock all LLM calls, enforce 100% coverage
- **Integration tests**: Live Groq API calls, no coverage requirements
- **Test scope control**: Set `TEST_SCOPE=unit|int|all` environment variable
- Coverage excludes: index.ts, types.ts, test files, type definitions

## Code Style Guidelines

### Naming Conventions

- **Descriptive Names**: Use purpose-driven names for all variables, functions, and types
  - PascalCase for classes and types
  - camelCase for functions and variables
  - Avoid generic names like "default", "temp", "main"
  - Avoid ambiguous abbreviations without context
- **Clarity and Context**: Include context in variable names to distinguish similar entities
  - Example: `batchSize` vs `tinyBatchFraction` instead of `size1` and `size2`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [with-logic/intent](https://github.com/with-logic/intent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
