---
trigger: always_on
description: This document gives AI coding assistants (like GitHub Copilot) essential context to work effectively in the SharpVector repo: architecture overview, key entry points, docs locations, conventions, and safe practices for adding features or fixing bugs.
---

# Project Guide for AI Coding Agents (Copilot)

This document gives AI coding assistants (like GitHub Copilot) essential context to work effectively in the SharpVector repo: architecture overview, key entry points, docs locations, conventions, and safe practices for adding features or fixing bugs.

## Overview

- Primary package: `Build5Nines.SharpVector` (NuGet: Build5Nines.SharpVector) targeting .NET 8+.
- Optional integrations:
  - `Build5Nines.SharpVector.OpenAI` — embeddings via OpenAI/Azure OpenAI.
  - `Build5Nines.SharpVector.Ollama` — embeddings via local Ollama server.
- Playground and samples are provided for demos and manual testing.

- Branches: active development occurs on `dev`; confirm before broad changes.
- CI: GitHub Actions workflow `build-release.yml` builds and releases NuGet packages.

## Documentation Locations

- Public docs site sources: `docs/` (MkDocs)
  - Index: `docs/docs/index.md`
  - Get Started: `docs/docs/get-started/`
  - Concepts, Persistence, Text Chunking, Samples, etc. under `docs/docs/`
- Root README: `README.md` — high-level intro and NuGet info.
- Project-specific docs inside src:
  - `src/Build5Nines.SharpVector/docs/` — internal docs snippets.

When adding features, update both code and related docs (MkDocs under `docs/docs/...`). Keep docs concise with examples and cross-links.

- `src/SharpVector.sln` — solution file.
- `src/Build5Nines.SharpVector/` — core library
  - Embeddings interfaces: `Embeddings/IEmbeddingsGenerator.cs`, `Embeddings/IBatchEmbeddingsGenerator.cs`
  - Core DB abstractions:
    - `IVectorDatabase.cs` — main interface.
    - `VectorDatabaseBase.cs` — common logic.
    - `MemoryVectorDatabaseBase.cs`, `MemoryVectorDatabase.cs`, `BasicMemoryVectorDatabase.cs` — in-memory implementations.
    - Disk persistence: `BasicDiskMemoryVectorDatabaseBase.cs`, `BasicDiskVectorDatabase.cs`, `DatabaseFile.cs`, `DatabaseInfo.cs`
  - Vector comparison (search metrics): `VectorCompare/`
    - `IVectorComparer.cs`
    - `CosineSimilarityVectorComparerAsync.cs` (default)
    - `EuclideanDistanceVectorComparerAsync.cs`
  - Preprocessing & Vectorization pipeline: `Preprocessing/`, `Vectorization/`, `Vocabulary/`, `VectorStore/`, `Id/`
  - Extensions: `IVectorDatabaseExtensions.cs`
- `src/Build5Nines.SharpVector.OpenAI/` — OpenAI embeddings
  - `Embeddings/OpenAIEmbeddingsGenerator.cs`
  - Memory DB wrappers using OpenAI: `OpenAIMemoryVectorDatabase*.cs`
- `src/Build5Nines.SharpVector.Ollama/` — Ollama embeddings
  - `Embeddings/OllamaEmbeddingsGenerator.cs`
  - Memory DB wrappers using Ollama: `OllamaMemoryVectorDatabase*.cs`
- Playground & samples
  - `src/Build5Nines.SharpVector.Playground/` — demo app, configurable via `appsettings.json`
  - `samples/` and `src/*ConsoleTest`, `*Test` projects — usage examples and tests.

## Typical Usage (Core Library)

- Create DB: `var vdb = new BasicMemoryVectorDatabase();`
- Add text: `vdb.AddText("some text", metadata);` (sync/async variants)
- Search: `var results = vdb.Search("query text");` (uses cosine similarity by default)
- Custom embeddings: Provide your own `IEmbeddingsGenerator` or use OpenAI/Ollama packages.
- Change comparison metric: Supply an `IVectorComparer` (e.g., Euclidean distance) to the DB.

Minimal example:

```csharp
using Build5Nines.SharpVector;

var vdb = new BasicMemoryVectorDatabase();
vdb.AddText("Hello SharpVector", metadata: "sample");
var results = vdb.Search("Hello");
```

## Key Design Concepts

- In-memory first: Default DB stores vectors in memory for speed. Disk-backed options exist for persistence.
- Pluggable pipeline:
  - Embeddings generation — interfaces allow external providers.
  - Preprocessing — text normalization/tokenization configurable under `Preprocessing/`.
  - Vector comparison — swapable similarity metrics via `IVectorComparer`.
- Metadata support: Store arbitrary metadata alongside each text entry.
- Async support: Async APIs exist for scalable operations.

## Conventions & Coding Guidelines

- Language/Runtime: C#, .NET 8+. Use async/await where appropriate.
- Style: Match existing patterns. Avoid wide refactors; make minimal, focused changes.
- Naming: Prefer descriptive names; avoid single-letter variables.
- Comments: Keep code clear; avoid inline comments unless necessary for clarity.
- Errors/exceptions: Use specific exception types like `DatabaseFileException` where applicable.
- Tests: When adding/altering behavior, include or update tests in `src/*Test` projects.

- API stability: Prefer additive changes; avoid breaking public types/methods.
- Nullability: Follow existing project settings; respect nullable context in projects.
- Performance: Avoid allocations in tight loops; prefer spans/arrays where safe.

## How to Add Features Safely

1. Identify extension point:
   - New similarity metric → implement `IVectorComparer` under `src/Build5Nines.SharpVector/VectorCompare/` and wire via constructor/config.
   - New embeddings provider → implement `IEmbeddingsGenerator` (and optionally `IBatchEmbeddingsGenerator`) under a new package or existing OpenAI/Ollama.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Build5Nines/SharpVector](https://github.com/Build5Nines/SharpVector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
