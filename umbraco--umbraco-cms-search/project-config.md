---
trigger: always_on
description: Provides backoffice search using the Search API. Registers a backoffice search provider that queries the `Umb_Content` index.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Umbraco Search is a new search abstraction for Umbraco CMS v16+ that will eventually replace the current search implementation. It provides three main capabilities:
1. Frontend search via the `ISearcher` interface
2. Backoffice search
3. Delivery API querying

The project uses a **provider-based architecture** where search technology implementations (currently Examine/Lucene) plug into core abstractions.

## Development Philosophy

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

### 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

### 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

### 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

### 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

## Build & Test Commands

### Building the Solution

```bash
# Build entire solution
dotnet build src/Umbraco.Cms.Search.sln

# Build specific project
dotnet build src/Umbraco.Cms.Search.Core/Umbraco.Cms.Search.Core.csproj

# Build in Release mode
dotnet build src/Umbraco.Cms.Search.sln -c Release
```

### Running Tests

```bash
# Run all tests
dotnet test src/Umbraco.Cms.Search.sln

# Run unit tests only
dotnet test src/Umbraco.Test.Search.Unit/Umbraco.Test.Search.Unit.csproj

# Run integration tests (requires Examine provider)
dotnet test src/Umbraco.Test.Search.Integration/Umbraco.Test.Search.Integration.csproj
dotnet test src/Umbraco.Test.Search.Examine.Integration/Umbraco.Test.Search.Examine.Integration.csproj

# Run specific test by filter
dotnet test --filter "FullyQualifiedName~ContentExtensionsTests"
```

### Client Development (Backoffice UI)

The client code uses an **npm workspaces monorepo** rooted at `src/`. Two workspaces:
- **Core Client**: `src/Umbraco.Cms.Search.Core.Client/Client/` - Main backoffice UI (TypeScript + Vite, 3-bundle code-splitting)
- **Examine Client**: `src/Umbraco.Cms.Search.Provider.Examine/Client/` - Examine provider UI (TypeScript + Vite, single bundle)

Shared config lives in `src/`:
- `src/package.json` - Workspace root with aggregate scripts
- `src/tsconfig.json` - Shared TypeScript compiler options (both workspaces extend this)
- `src/.nvmrc` - Node.js version (24)
- `src/.prettierrc.json` - Shared Prettier config

```bash
cd src

# Install all workspaces
npm install

# Build all workspaces
npm run build

# Watch all workspaces
npm run watch

# Lint all workspaces (errors only)
npm run lint:errors-only

# Build a single workspace
npm run build --workspace=Umbraco.Cms.Search.Core.Client/Client
npm run build --workspace=Umbraco.Cms.Search.Provider.Examine/Client

# Generate OpenAPI client (Core Client only, requires test site at https://localhost:44324)
npm run generate-client --workspace=Umbraco.Cms.Search.Core.Client/Client
```

Requires **Node.js 24** (see `src/.nvmrc`).

### Test Site

Run the test site to manually test integration:

```bash
cd src/Umbraco.Web.TestSite.V17
dotnet run
```

The test site runs on .NET 10.0 and includes Umbraco CMS with Search packages pre-configured.

## Architecture

### Core Abstractions (Umbraco.Cms.Search.Core)

The core provides **provider-agnostic abstractions**:

**Key Interfaces:**
- `ISearcher` - Search operations (filtering, faceting, sorting, pagination)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umbraco/Umbraco.Cms.Search](https://github.com/umbraco/Umbraco.Cms.Search) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-27 -->
