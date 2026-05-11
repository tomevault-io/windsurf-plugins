---
trigger: always_on
description: This document provides guidelines for AI coding assistants working on this codebase.
---

# AGENTS.md - Guidelines for AI Coding Assistants

This document provides guidelines for AI coding assistants working on this codebase.
The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be
interpreted as described in [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt).

## Architecture Overview

This project follows **Clean Architecture** principles. Code is organized into layers
with strict dependency rules.

```
┌─────────────────────────────────────────────────────────────┐
│                      Presentation                           │
│                      (src/app/cli/)                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                      Application                            │
│                   (src/app/)                                │
│                   Orchestration (indexer, search)           │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┴───────────────┐
              ▼                               ▼
┌─────────────────────────────┐     ┌─────────────────────────────┐
│      Domain Layer           │     │    Infrastructure Layer     │
│    (src/domain/)            │     │    (src/infrastructure/)    │
│                             │     │                             │
│  ├── entities/              │     │  ├── config/                │
│  │   Pure data types        │     │  │   Config loading/saving  │
│  │                          │     │  │                          │
│  ├── ports/                 │◄────│  ├── embeddings/            │
│  │   Interfaces             │     │  │   Transformers.js        │
│  │                          │     │  │                          │
│  ├── services/              │     │  ├── filesystem/            │
│  │   Pure algorithms        │     │  │   Node.js fs             │
│  │                          │     │  │                          │
│  └── usecases/              │     │  └── storage/               │
│      Business logic         │     │      Index file I/O         │
└─────────────────────────────┘     └─────────────────────────────┘
```

## Layer Rules

### Domain Layer (`src/domain/`)

The domain layer MUST contain only pure business logic with NO external dependencies.

#### `src/domain/entities/`

- MUST contain only data types (interfaces, types, classes)
- MUST NOT import from `infrastructure/`, `app/`, or external packages
- MUST NOT perform I/O operations
- MAY import from other domain entities

**Examples of what belongs here:**

- `Chunk`, `FileIndex`, `SearchResult` - Core data structures
- `Config`, `ModuleConfig` - Configuration types
- `FileSummary`, `FileIntrospection` - Index metadata types

#### `src/domain/ports/`

- MUST define interfaces for external dependencies
- MUST NOT contain implementations
- SHOULD be named as capabilities (e.g., `IFileSystem`, `IEmbeddingProvider`)

#### `src/domain/services/`

- MUST contain pure algorithms with no I/O
- MUST NOT import from `infrastructure/`
- MAY import from `domain/entities/` and `domain/ports/`
- SHOULD be stateless functions or classes

**Examples of what belongs here:**

- `BM25Index` - BM25 search algorithm
- `extractKeywords()` - Keyword extraction logic
- `cosineSimilarity()` - Vector similarity calculation
- `introspection.ts` - File metadata extraction and keyword generation
- `conventions/` - File convention pattern matching (entry points, config files, frameworks)

#### `src/domain/usecases/`

- MUST contain business logic use cases
- MAY depend on domain entities, ports, and services
- SHOULD accept dependencies through parameters (dependency injection)
- MUST NOT perform I/O directly (use injected dependencies)

**Examples of what belongs here:**

- `indexDirectory()` - Orchestrates indexing a directory
- `searchIndex()` - Orchestrates searching the index
- `cleanupIndex()` - Removes stale entries

### Infrastructure Layer (`src/infrastructure/`)

The infrastructure layer implements domain ports using external technologies.

#### General Rules

- MUST implement interfaces defined in `domain/ports/`
- MAY import external packages (fs, path, @xenova/transformers, etc.)
- MAY import from `domain/entities/` for type definitions
- MUST NOT contain business logic

#### `src/infrastructure/config/`

- MUST contain configuration loading and saving
- Currently: `configLoader.ts` with path utilities and config I/O

#### `src/infrastructure/embeddings/`

- MUST contain embedding provider implementations
- Currently: `XenovaTransformersEmbeddingProvider` and `HuggingFaceTransformersEmbeddingProvider` using `@xenova/transformers` / `@huggingface/transformers`; `createEmbeddingProvider()` selects the adapter; global API: `getEmbedding()`, `getEmbeddings()`, `configureEmbeddings()`

#### `src/infrastructure/filesystem/`

- MUST contain file system operations
- Currently: `NodeFileSystem` using Node.js fs

#### `src/infrastructure/storage/`

- MUST contain index persistence logic

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [conradkoh/raggrep](https://github.com/conradkoh/raggrep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
