---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Building the project

```bash
npm run build        # Build the project using tsup
npm run dev          # Build in watch mode for development (--watch)
npm run build:docs   # Generate TypeDoc documentation
```

### Running tests

```bash
npm test                   # Run all unit tests using Vitest
npm run test:watch        # Run tests in watch mode
npm run test:coverage     # Run tests with coverage report
npm run test:e2e          # Run end-to-end tests
npm run test:e2e:watch    # Run e2e tests in watch mode
npm run test:integration  # Run integration tests (requires NOVEUM_API_KEY)
npm run test:all          # Run all tests (unit + e2e + integration)
```

### Code quality and formatting

```bash
npm run lint             # Run ESLint and auto-fix issues
npm run lint:check      # Run ESLint without fixing
npm run format          # Format code with Prettier
npm run format:check    # Check code formatting without fixing
npm run check-types     # Run TypeScript type checking
```

### CI/CD and automation

The project includes comprehensive GitHub Actions workflows:

- **CI/CD Pipeline** (`.github/workflows/ci.yml`) - Runs on push/PR to main/develop
  - Code quality checks (lint, format, typecheck)
  - Unit tests across Node.js 18, 20, 22
  - Build verification and package testing
  - Integration tests (if NOVEUM_API_KEY available)
  - Security audits and dependency checks
  - Bundle size analysis and performance tests
  - Documentation generation and validation

- **Release Automation** (`.github/workflows/release.yml`) - Runs on releases
  - Automated changelog generation
  - GitHub release creation
  - npm package publishing
  - Docker image building and deployment
  - Documentation deployment to GitHub Pages

- **Documentation Updates** (`.github/workflows/docs-update.yml`) - Runs on main branch changes
  - Automatic CLAUDE.md statistics updates
  - API documentation generation with TypeDoc
  - Example validation and README completeness checks
  - Scripts documentation synchronization

### Release and maintenance

```bash
npm run release        # Create release with automatic version bump using standard-version
```

### Environment setup

1. Copy `.env.example` to `.env` and configure API keys for testing:

   ```bash
   # Required for tracing functionality
   NOVEUM_API_KEY=your_noveum_api_key_here

   # Optional: For OpenAI integration tests
   OPENAI_API_KEY=your_openai_api_key_here

   # Other provider keys as needed for testing
   ANTHROPIC_API_KEY=your_anthropic_api_key_here
   ```

2. Run integration tests to verify setup:
   ```bash
   npm run test:integration
   ```

## Important Implementation Details

### Core Architecture

The Noveum Trace TypeScript SDK follows a modular architecture designed for performance, extensibility, and ease of use:

#### Client Management (`src/core/client.ts`)

- **NoveumClient**: Central client class managing configuration, HTTP transport, and lifecycle
- **Configuration system**: Both new Python-compatible config (`src/types/config.ts`) and legacy options
- **Batch processing**: Intelligent batching with configurable size and flush intervals via `BatchProcessor`
- **Error handling**: Robust retry logic with exponential backoff
- **Resource management**: Automatic cleanup and graceful shutdown

#### Tracing Core (`src/core/`)

- **Trace Management**: Full trace lifecycle with attributes, events, and relationships
- **Span Operations**: Comprehensive span creation, modification, and completion
- **Context Propagation**: Advanced context management for span relationships
- **Standalone Support**: Self-contained trace/span operations for edge cases

#### Transport Layer (`src/transport/`)

- **HTTP Transport** (`http-transport.ts`): Robust HTTP client with retry logic and error handling
- **Batch Processor** (`batch-processor.ts`): Efficient batching system with configurable thresholds
- **Batch Serialization**: Efficient JSON serialization with Python-compatible timestamps
- **Network Resilience**: Configurable timeouts, retries, and backoff strategies

#### Integration Framework (`src/integrations/`)

- **Express.js** (`express.ts`): Complete middleware with request/response tracing
- **Next.js** (`nextjs.ts`): App Router support with `withNoveumTracing` wrapper
- **Hono** (`hono.ts`): Modern framework integration with middleware and handler wrapping
- **Framework Agnostic**: Manual tracing support for any TypeScript/JavaScript application

#### Developer Experience (`src/decorators/`)

- **Comprehensive Decorators**: `@trace`, `@traceLLM`, `@traceAgent`, `@traceRetrieval`, `@traceTool` for automatic instrumentation
- **Specialized Decorators**: Domain-specific decorators matching Python SDK functionality
- **Base Decorator** (`base.ts`): Core decorator functionality with flexible options
- **Type Safety**: Full TypeScript support with excellent IntelliSense
- **Debug Support**: Comprehensive logging and error reporting

#### LLM Utilities (`src/llm/`)

- **Cost Estimation** (`cost-estimation.ts`): Token-based cost calculation for LLM providers

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Noveum/noveum-trace-ts](https://github.com/Noveum/noveum-trace-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
