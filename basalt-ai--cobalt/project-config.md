---
trigger: always_on
description: > **💡 Collaboration First**: This project values collaborative decision-making. When facing decisions about architecture, developer experience, or anything with significant impact, **always present options and wait for user input** before proceeding. See [Decision-Making Philosophy](#decision-making-philosophy) for details.
---

# CLAUDE.md — Development Guide

> **💡 Collaboration First**: This project values collaborative decision-making. When facing decisions about architecture, developer experience, or anything with significant impact, **always present options and wait for user input** before proceeding. See [Decision-Making Philosophy](#decision-making-philosophy) for details.

## Project Overview

Cobalt is a TypeScript CLI testing framework for AI agents and LLM-powered applications. Think "Unit testing for AI Agents" — it provides experiment runners, evaluators, datasets, and result tracking.

### Tech Stack

- **Language**: TypeScript 5.7 with strict mode
- **Build Tool**: tsup for bundling
- **CLI Framework**: citty for command-line interface
- **Testing**: Vitest for unit and integration tests
- **Code Quality**: Biome for linting and formatting
- **Database**: better-sqlite3 for history tracking
- **HTTP Server**: Hono for dashboard API
- **Package Manager**: pnpm

## Architecture

Cobalt follows a clean, modular architecture:

```
packages/cobalt/
├── src/
│   ├── core/               # Core experiment runner logic
│   │   ├── experiment.ts   # Main experiment() function
│   │   ├── Evaluator.ts    # Evaluator class
│   │   └── config.ts       # Configuration system
│   ├── datasets/           # Dataset loading and transformation
│   │   └── Dataset.ts      # Dataset class
│   ├── evaluators/         # Evaluator implementations
│   │   ├── llm-judge.ts    # LLM-based evaluation
│   │   ├── function.ts     # Custom function evaluation
│   │   ├── exact-match.ts  # String matching
│   │   └── similarity.ts   # Embeddings (P2 - not implemented)
│   ├── cli/                # CLI commands
│   │   ├── index.ts        # CLI entry point
│   │   └── commands/       # Individual commands (run, init, etc.)
│   ├── dashboard/          # Dashboard server (P4)
│   │   ├── server.ts       # Hono server
│   │   └── routes.ts       # API routes
│   ├── mcp/                # Model Context Protocol (P3)
│   │   ├── server.ts       # MCP server
│   │   └── tools.ts        # MCP tool implementations
│   ├── storage/            # Data persistence
│   │   ├── results.ts      # JSON result files
│   │   ├── cache.ts        # LLM response cache
│   │   └── db.ts           # SQLite history database
│   ├── utils/              # Utilities
│   │   ├── cost.ts         # Token cost estimation
│   │   ├── stats.ts        # Statistical calculations
│   │   ├── template.ts     # Template rendering
│   │   └── hash.ts         # Hash generation
│   └── types/              # TypeScript types
└── tests/                  # Test suite
    ├── unit/               # Unit tests
    ├── integration/        # Integration tests
    └── helpers/            # Test helpers and mocks
```

## Development Guidelines

### 1. Adding New Features

When adding a feature, follow these steps:

#### For New Evaluator Types (e.g., similarity)

1. **Create evaluator implementation** in `src/evaluators/`
2. **Define types** in `src/types/index.ts`
3. **Update Evaluator class** dispatch logic in `src/core/Evaluator.ts`
4. **Write unit tests** in `tests/unit/evaluators/`
5. **Update documentation** in `.memory/documentation.md`

#### For New CLI Commands

1. **Create command file** in `src/cli/commands/`
2. **Register command** in `src/cli/index.ts`
3. **Write tests** (if feasible with mocked file system)
4. **Update README** with command usage

#### For New Dataset Loaders

1. **Add loader method** to `src/datasets/Dataset.ts`
2. **Write unit tests** in `tests/unit/Dataset.test.ts`
3. **Update documentation**

### 2. Writing Tests

**Unit tests are mandatory** for all core functionality (experiment runner, evaluators, datasets, utilities).

**Test Structure:**
```typescript
import { describe, it, expect, vi, beforeEach } from 'vitest'

describe('FeatureName', () => {
  beforeEach(() => {
    vi.clearAllMocks()
  })

  it('should do something', async () => {
    // Arrange
    const input = 'test'
    
    // Act
    const result = await functionUnderTest(input)
    
    // Assert
    expect(result).toBe('expected')
  })
})
```

**Mocking Guidelines:**
- **LLM APIs**: Mock `openai` and `@anthropic-ai/sdk` modules
- **File system**: Mock `node:fs` or use helper functions
- **External services**: Always mock, never make real API calls in tests

**Run Tests:**
```bash
pnpm test              # Run all tests
pnpm test:watch        # Watch mode
pnpm test:coverage     # With coverage report
```

### 3. Code Quality

**Before Committing (Required):**
```bash
pnpm test              # Run tests - MUST PASS
pnpm lint              # Check linting - MUST PASS
pnpm build             # Verify build works - MUST PASS
```

**Optional (but recommended):**
```bash
pnpm format            # Format code
pnpm check             # Auto-fix issues
```

**Code Standards:**
- Use TypeScript strict mode - no `any` types
- Write descriptive variable and function names
- Add comments only when logic isn't self-evident

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [basalt-ai/cobalt](https://github.com/basalt-ai/cobalt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
