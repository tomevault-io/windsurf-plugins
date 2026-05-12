---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

TanStack AI is a type-safe, provider-agnostic AI SDK for building AI-powered applications. The repository is a **pnpm monorepo** managed with **Nx** that includes TypeScript, PHP, and Python packages, plus multiple framework examples.

## Package Manager & Tooling

- **Package Manager**: pnpm@10.17.0 (required)
- **Build System**: Nx for task orchestration and caching
- **TypeScript**: 5.9.3
- **Testing**: Vitest for unit tests
- **Linting**: ESLint with custom TanStack config
- **Formatting**: Prettier

## Common Commands

### Testing

```bash
# Run all tests (full CI suite)
pnpm test

# Run tests for affected packages only (for PRs)
pnpm test:pr

# Run specific test suites
pnpm test:lib              # Run unit tests for affected packages
pnpm test:lib:dev          # Watch mode for unit tests
pnpm test:eslint           # Lint affected packages
pnpm test:types            # Type check affected packages
pnpm test:build            # Verify build artifacts with publint
pnpm test:coverage         # Generate coverage reports
pnpm test:knip             # Check for unused dependencies
pnpm test:sherif           # Check pnpm workspace consistency
pnpm test:docs             # Verify documentation links

# E2E tests (required for all changes)
pnpm --filter @tanstack/ai-e2e test:e2e    # Run E2E tests
pnpm --filter @tanstack/ai-e2e test:e2e:ui # Run with Playwright UI
```

### Testing Individual Packages

```bash
# Navigate to package directory and run tests
cd packages/typescript/ai
pnpm test:lib              # Run tests for this package
pnpm test:lib:dev          # Watch mode
pnpm test:types            # Type check
pnpm test:eslint           # Lint
```

### Building

```bash
# Build affected packages
pnpm build

# Build all packages
pnpm build:all

# Watch mode (build + watch for changes)
pnpm watch
pnpm dev  # alias for watch
```

### Code Quality

```bash
pnpm format                # Format all files with Prettier
```

### Changesets (Release Management)

```bash
pnpm changeset             # Create a new changeset
pnpm changeset:version     # Bump versions based on changesets
pnpm changeset:publish     # Publish to npm
```

## Architecture

### Monorepo Structure

```
packages/
├── typescript/           # TypeScript packages (main implementation)
│   ├── ai/              # Core AI library (@tanstack/ai)
│   ├── ai-client/       # Framework-agnostic chat client
│   ├── ai-react/        # React hooks (useChat)
│   ├── ai-solid/        # Solid hooks
│   ├── ai-svelte/       # Svelte integration
│   ├── ai-vue/          # Vue integration
│   ├── ai-openai/       # OpenAI adapter
│   ├── ai-anthropic/    # Anthropic/Claude adapter
│   ├── ai-gemini/       # Google Gemini adapter
│   ├── ai-ollama/       # Ollama adapter
│   ├── ai-devtools/     # DevTools integration
│   ├── react-ai-devtools/ # React DevTools component
│   └── solid-ai-devtools/ # Solid DevTools component
├── php/                 # PHP packages (future)
└── python/              # Python packages (future)

testing/
├── e2e/                 # E2E tests (Playwright + aimock) — MANDATORY for all changes
└── panel/               # Vendor integration panel

examples/                # Example applications
├── ts-react-chat/       # React chat example
├── ts-solid-chat/       # Solid chat example
├── ts-vue-chat/         # Vue chat example
├── ts-svelte-chat/      # Svelte chat example
├── ts-group-chat/       # Multi-user group chat
├── vanilla-chat/        # Vanilla JS example
├── php-slim/            # PHP Slim framework example
└── python-fastapi/      # Python FastAPI example
```

### Core Architecture Concepts

#### 1. Adapter System (Tree-Shakeable)

The library uses a **tree-shakeable adapter architecture** where each provider (OpenAI, Anthropic, Gemini, Ollama) exports multiple specialized adapters:

- **Text adapters** (`openaiText`, `anthropicText`) - Chat/completion
- **Embedding adapters** (`openaiEmbed`) - Text embeddings
- **Summarize adapters** (`openaiSummarize`) - Summarization
- **Image adapters** (`openaiImage`) - Image generation

Each adapter is a separate import to minimize bundle size:

```typescript
import { openaiText } from '@tanstack/ai-openai/adapters'
import { ai } from '@tanstack/ai'

const textAdapter = openaiText()
const result = ai({ adapter: textAdapter, model: 'gpt-4o', messages: [...] })
```

#### 2. Core Functions

The `@tanstack/ai` package provides core functions:

- **`ai()`** / **`generate()`** - Unified generation function for any adapter type
- **`chat()`** - Chat completion with streaming, tools, and agent loops
- **`embedding()`** - Generate embeddings
- **`summarize()`** - Summarize text
- Legacy adapters (monolithic, deprecated) use `openai()`, `anthropic()`, etc.

#### 3. Isomorphic Tool System

Tools are defined once with `toolDefinition()` and can have `.server()` or `.client()` implementations:

```typescript
const tool = toolDefinition({
  name: 'getTodos',
  inputSchema: z.object({ userId: z.string() }),
  outputSchema: z.array(z.object({ id: z.string(), title: z.string() })),
})


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TanStack/ai](https://github.com/TanStack/ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
