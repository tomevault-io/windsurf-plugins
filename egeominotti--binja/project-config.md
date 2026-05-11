---
trigger: always_on
description: This file provides guidance to Claude Code when working with the binja template engine.
---

# CLAUDE.md - binja

This file provides guidance to Claude Code when working with the binja template engine.

## Project Overview

**binja** is a high-performance Jinja2/Django Template Language (DTL) engine for Bun/JavaScript. It provides 100% compatibility with Django templates while running on the Bun runtime for maximum performance.

## Quick Commands

```bash
# Install dependencies
bun install

# Run tests
bun test

# Run specific test file
bun test test/filters.test.ts

# Build
bun run build

# Type check
bun run typecheck
```

## Project Structure

```
binja/
├── src/
│   ├── index.ts          # Main entry point, Environment class
│   ├── cli.ts            # CLI tool (binja compile/check/watch/lint)
│   ├── lexer/
│   │   ├── index.ts      # Lexer - tokenizes template strings
│   │   └── tokens.ts     # Token types and interfaces
│   ├── parser/
│   │   ├── index.ts      # Parser - generates AST from tokens
│   │   └── nodes.ts      # AST node type definitions
│   ├── runtime/
│   │   ├── index.ts      # Runtime - executes AST (with inline filter optimization)
│   │   └── context.ts    # Context class with forloop/loop support
│   ├── compiler/
│   │   ├── index.ts      # AOT compiler - generates JS functions
│   │   └── flattener.ts  # Template flattener for AOT inheritance
│   ├── filters/
│   │   └── index.ts      # 80+ built-in filters
│   ├── tests/
│   │   └── index.ts      # 28 built-in tests (is operator)
│   ├── ai/               # AI-powered linting (optional)
│   │   ├── index.ts      # Entry point for binja/ai
│   │   ├── types.ts      # LintResult, Issue types
│   │   ├── lint.ts       # Main lint function
│   │   ├── prompt.ts     # AI prompt engineering
│   │   └── providers/    # AI provider implementations
│   │       ├── index.ts  # Auto-detect and provider factory
│   │       ├── anthropic.ts  # Claude provider
│   │       ├── openai.ts     # GPT-4 provider
│   │       ├── ollama.ts     # Local Ollama provider
│   │       └── groq.ts       # Groq provider (free tier)
│   ├── engines/          # Multi-engine support
│   │   ├── index.ts      # Unified MultiEngine API
│   │   ├── handlebars/   # Handlebars engine
│   │   │   ├── index.ts  # Entry point
│   │   │   ├── lexer.ts  # Handlebars tokenizer
│   │   │   └── parser.ts # Handlebars → common AST
│   │   └── liquid/       # Liquid (Shopify) engine
│   │       ├── index.ts  # Entry point
│   │       ├── lexer.ts  # Liquid tokenizer
│   │       └── parser.ts # Liquid → common AST
│   └── debug/
│       ├── index.ts      # Debug panel exports
│       ├── collector.ts  # DebugCollector for timing/context
│       └── panel.ts      # HTML panel generator
├── test/
│   ├── lexer.test.ts     # Lexer tests
│   ├── parser.test.ts    # Parser tests
│   ├── filters.test.ts   # Filter tests
│   ├── filters-extended.test.ts # Extended filters tests
│   ├── runtime.test.ts   # Runtime/core tags tests
│   ├── inheritance.test.ts # Template inheritance tests
│   ├── aot-inheritance.test.ts # AOT with extends/include tests
│   ├── raw.test.ts       # Raw/verbatim tag tests
│   ├── engines.test.ts   # Multi-engine tests (Handlebars, Liquid)
│   ├── debug.test.ts     # Debug panel tests
│   └── ...
├── examples/             # Usage examples
│   ├── 01-basic-usage.ts
│   ├── ...
│   ├── 07-complete-reference.ts  # All features reference
│   └── 09-multi-engine.ts # Multi-engine usage
├── website/              # Demo website with debug panel
│   ├── server.ts         # Hono server
│   └── templates/        # Demo templates
├── package.json
├── tsconfig.json
└── CLAUDE.md
```

## Architecture

### Template Processing Pipeline

```
Template String → Lexer → Tokens → Parser → AST → Runtime → Output String
```

1. **Lexer** (`src/lexer/`): Tokenizes template into tokens (TEXT, VAR_START, BLOCK_START, etc.)
   - Pure TypeScript implementation optimized for Bun
   - 2-4x faster than Nunjucks
2. **Parser** (`src/parser/`): Converts tokens into Abstract Syntax Tree (AST)
3. **Runtime** (`src/runtime/`): Executes AST with context to produce output
   - Inline filter optimization for ~70 common filters (10-15% faster)

### Performance

Binja is **2-4x faster** than Nunjucks in runtime mode:

| Benchmark | binja | Nunjucks | Speedup |
|-----------|-------|----------|---------|
| Simple Template | 371K ops/s | 96K ops/s | **3.9x** |
| Complex Template | 44K ops/s | 23K ops/s | **2.0x** |
| Nested Loops | 76K ops/s | 26K ops/s | **3.0x** |
| Conditionals | 84K ops/s | 25K ops/s | **3.4x** |
| HTML Escaping | 985K ops/s | 242K ops/s | **4.1x** |

### Key Classes

| Class | File | Purpose |
|-------|------|---------|
| `Environment` | `src/index.ts` | Main API, template loading, configuration, debug |
| `Lexer` | `src/lexer/index.ts` | Pure TypeScript lexer |
| `Parser` | `src/parser/index.ts` | Generates AST from tokens |
| `Runtime` | `src/runtime/index.ts` | Executes AST with inline filter optimization |
| `Context` | `src/runtime/context.ts` | Variable scope management |
| `Compiler` | `src/compiler/index.ts` | AOT compilation to JS functions |
| `TemplateFlattener` | `src/compiler/flattener.ts` | Resolves extends/include at compile-time |
| `DebugCollector` | `src/debug/collector.ts` | Collects timing and context data |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [egeominotti/binja](https://github.com/egeominotti/binja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
