---
trigger: always_on
description: This file contains project context for AI agents working on SemanticSlicer. It consolidates the memory bank content for agent reference.
---

# SemanticSlicer - Agent Memory

This file contains project context for AI agents working on SemanticSlicer. It consolidates the memory bank content for agent reference.

## Brief

SemanticSlicer is a lightweight .NET library and tooling suite for recursively splitting large documents into semantically meaningful, LLM-ready chunks. It preserves structure using configurable separators (text, Markdown, HTML), counts tokens via tiktoken encoders (cl100k_base, o200k), and supports optional HTML stripping, metadata passthrough, and chunk headers.

### Key Features
- Recursive, centermost-aware splitting with prefix/suffix/remove behaviors
- Token-aware chunk sizing with min-percentage safeguards and stable ordering
- CLI for one-off or daemon mode with stdin/pipe processing
- Minimal Web API service for POST /slice chunking
- NuGet package for direct library use across Windows, macOS, Linux

### Technologies
- .NET (netstandard2.1, .NET 8 tooling), C#
- HtmlAgilityPack, Tiktoken (token encoders)

### Significance
- Produces high-quality chunks for embeddings and RAG pipelines, reducing token waste and improving retrieval relevance while offering flexible deployment: library, CLI, daemon, and REST service.

---

## Product

### Why This Project Exists
SemanticSlicer exists to split large documents into semantically meaningful, LLM-ready chunks to optimize embeddings and retrieval. It preserves document structure (text, Markdown, HTML), controls chunk size via token-aware limits, and supports multiple deployment/use modes: library (NuGet), CLI (one-shot and daemon), and minimal REST service.

### Problems It Solves
- Excessive token usage in naive chunking leading to poor embeddings and higher costs
- Loss of semantic boundaries causing degraded retrieval relevance
- Need for consistent, repeatable chunking across different content types (text, Markdown, HTML)
- Operational flexibility: single-run CLI, in-memory daemon, and REST service for integration
- Metadata passthrough and optional chunk headers for RAG pipelines

### How It Should Work
- Input content is normalized and optionally HTML-stripped for token efficiency
- Token counting is performed using tiktoken encoders cl100k_base and o200k
- Recursive splitting uses centermost-aware matching across configured separators
- Ensures chunk size <= MaxChunkTokenCount and safeguards with MinChunkPercentage
- Supports overlapping chunks via an OverlapPercentage (0-100) carry-forward of previous chunk tokens
- Stable ordering with Index assigned per chunk for reassembly
- Each chunk includes StartOffset and EndOffset character positions relative to normalized input
- Optional metadata dictionary is preserved on each chunk
- Optional per-chunk header is prepended to content and accounted for in tokens

### Usage Modes
- **Library (NuGet)**: `drittich.SemanticSlicer` for direct integration in .NET apps
- **CLI**: One-off execution for files or piped input; daemon mode keeps slicer in memory
- **Service**: Minimal Web API with POST /slice accepting content, metadata, and chunkHeader

### Document Types and Separators
- **Text**: Sentence/end punctuation and whitespace-aware splitting
- **Markdown**: Headers prioritized to preserve structure
- **HTML**: Element-aware prefixes to align splits with block boundaries

### Success Criteria
- Chunks consistently within token limits with minimal semantic breakage
- Reduced token consumption relative to naive splitting
- Reliable operation across library, CLI, daemon, and REST service modes
- Clear, minimal API surface with sane defaults and optional extensibility via custom separators

---

## Architecture

### System Overview
SemanticSlicer provides three operation modes over a shared core:
- **Library**: Direct use of the slicer class in .NET apps
- **CLI**: Command-line interface wrapping the slicer
- **Service**: Minimal REST API for in-memory slicing

### Core Responsibilities
- Normalize and prepare input
- Token-aware chunk sizing
- Recursive splitting via separators
- Stable chunk ordering and optional metadata/header passthrough

### Components

#### Core
- **Slicer** (`SemanticSlicer/Slicer.cs`)
  - `GetDocumentChunks()` - Standard preprocessing + splitting
  - `SplitDocumentChunksRaw()` - Advanced API: splitting without preprocessing
  - `PrepareContentForChunking()` - Separate preprocessing step
  - `CountTokens()` - Public token counting
  - `RemoveNonBodyContent()` - Public HTML-to-text
  - `ExtractTitle()` - Public title extraction
- **TextUtilities** (`SemanticSlicer/TextUtilities.cs`) - Public static preprocessing helpers
  - `NormalizeLineEndings()`
  - `CollapseWhitespace()`
- **Options** (`SemanticSlicer/SlicerOptions.cs`)
  - Defaults: MaxChunkTokenCount=1000, MinChunkPercentage=10, Encoding=Cl100K, Separators=Text
  - Toggle: StripHtml
- **Separators** (`SemanticSlicer/Separators.cs`)
  - Text/Markdown/Html ordered regex lists

#### Models
- `DocumentChunk` - Output chunk model with content, tokens, metadata, index, offsets
- `Separator` - Separator definition with regex and behavior
- `Encoding` enum - Token encoding options (Cl100K, O200K)

#### CLI
- Entry and mode parsing (`SemanticSlicer.Cli/Program.cs`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [drittich/SemanticSlicer](https://github.com/drittich/SemanticSlicer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
