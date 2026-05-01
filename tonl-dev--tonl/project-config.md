---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**TONL** (Token-Optimized Notation Language) is a TypeScript library that provides a text-first, LLM-friendly serialization format designed to reduce token costs in language model prompts while maintaining human readability. The project offers both a programmatic API and CLI tool for converting between JSON and TONL formats.

## Development Commands

```bash
# Build and test
npm run build              # TypeScript compilation
npm run build:browser      # Generate browser builds (ESM, UMD, IIFE)
npm run build:all          # Build both Node.js and browser versions
npm test                   # Build and run stable test suite
npm run test:all           # Build and run all tests including comprehensive
npm run test:examples      # Build and validate all feature examples
npm run dev                # Watch mode development
npm run clean              # Clean build artifacts

# Benchmarking and performance analysis
npm run bench              # Byte size compression analysis
npm run bench-tokens       # Token estimation across multiple LLM models
npm run bench-comprehensive # Full performance analysis
npm run bench-query        # Query system performance testing

# CLI development
npm run link               # Install tonl command locally for testing
```

## Testing

- **Framework**: Node.js built-in test runner (`node:test`)
- **Test files**: 46 comprehensive test suites covering all modules
- **Coverage**: 100% test coverage (791+ tests passing)
- **Test command**: `npm test` (runs stable test suite), `npm test:all` (runs all tests)
- **Feature coverage**: `npm run test:features` validates complete feature set

Run specific test categories:
- Core functionality: `node --test test/encode_decode_roundtrip.test.ts test/parser.test.ts`
- Schema validation: `node --test test/schema.test.ts test/schema-constraints.test.ts`
- Query system: `node --test test/query-*.test.ts`
- Modification API: `node --test test/modification-*.test.ts`
- Streaming: `node --test test/stream.test.ts`

## Architecture

### Core Components (`src/`)

- **`types.ts`** - Core type definitions and interfaces for the TONL format
- **`index.ts`** - Main entry point exporting public API (`encodeTONL`, `decodeTONL`, `encodeSmart`, `TONLDocument`)
- **`document.ts`** - TONLDocument class providing high-level document API (query, navigation, modification)
- **`encode.ts`** - JSON to TONL conversion with nested structure support
- **`decode.ts`** - TONL to JSON conversion (orchestrates modular parser)
- **`parser.ts`** - Low-level parsing utilities for TONL lines, headers, and delimiter detection
- **`infer.ts`** - Type inference and coercion utilities
- **`cli.ts`** - Command-line interface with `encode`, `decode`, `stats`, `format`, `validate`, `query`, `generate-types` commands

### Parser Modules (`src/parser/`)

- **`content-parser.ts`** - Document-level parsing orchestration
- **`block-parser.ts`** - Multi-line block and array parsing
- **`value-parser.ts`** - Single-line object and inline value parsing
- **`line-parser.ts`** - Primitive value parsing
- **`utils.ts`** - Parser helper functions
- **`index.ts`** - Public parser exports

### Feature Modules (`src/`)

- **`query/`** - JSONPath-like query system with filtering, caching, and performance optimization
- **`navigation/`** - Tree traversal API with iterators, walkers, and navigation utilities
- **`modification/`** - Complete CRUD operations, transactions, change tracking, and file editing
- **`indexing/`** - Performance indexing system (Hash, BTree, Compound) for fast queries
- **`schema/`** - Schema validation, type generation, and constraint enforcement
- **`stream/`** - Streaming API for large files with encode/decode streams and query pipelines
- **`repl/`** - Interactive REPL for exploring TONL documents
- **`errors/`** - Centralized error handling and custom error types

### Utility Modules (`src/utils/`)

- **`strings.ts`** - String handling, quoting, escaping, and indentation utilities
- **`metrics.ts`** - Token estimation across multiple LLM tokenizers (GPT-5, Claude 3.5, Gemini 2.0, Llama 4)

### Browser Support (`src/browser-*.ts`)

- **`browser.ts`** - Browser-optimized entry point
- **`document-browser.ts`** - Browser-compatible TONLDocument
- **`browser-core.ts`** - Core browser utilities
- **`browser-simple.ts`** - Lightweight browser API

### Key Features

- **TONLDocument API**: High-level document interface with query, navigation, and modification
- **Smart Encoding**: Automatically analyzes content to choose optimal delimiters and formatting
- **JSONPath Queries**: Advanced query system with filtering and caching
- **Complete CRUD**: Full modification API with transactions and change tracking
- **Performance Indexing**: Multiple index types for optimized queries
- **Schema Validation**: Comprehensive validation with 13 constraint types
- **Streaming Support**: Handle large files with minimal memory usage
- **Multi-delimiter Support**: `,`, `|`, `\t`, `;` with auto-detection
- **Type Hints**: Optional schema information for validation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tonl-dev/tonl](https://github.com/tonl-dev/tonl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
