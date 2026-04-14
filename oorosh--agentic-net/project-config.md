---
trigger: always_on
description: This file provides essential information for agents working on this codebase.
---

# AGENTS.md - Agentic.NET Development Guide

This file provides essential information for agents working on this codebase.

## Project Overview

Agentic.NET is a .NET library for creating AI assistants with pluggable models, memory, middleware, and tools. The solution contains:
- **Agentic.NET** (main library) - `Agentic.NET.csproj`
- **Agentic.Tests** (unit tests) - `tests/Agentic.Tests/Agentic.Tests.csproj`
- **Samples** - `samples/*/` (BasicChat, MemoryAndMiddleware, ToolCalling, PersonalAssistant)

## Build Commands

```bash
# Restore dependencies
dotnet restore Agentic.NET.sln

# Build the library (Debug)
dotnet build src/Agentic.NET/Agentic.NET.csproj

# Build in Release mode
dotnet build src/Agentic.NET/Agentic.NET.csproj -c Release

# Pack NuGet package
dotnet pack src/Agentic.NET/Agentic.NET.csproj -c Release -o artifacts

# Build a sample
dotnet run --project samples/BasicChat/BasicChat.csproj
```

## Test Commands

```bash
# Run all tests
dotnet test src/Agentic.Tests/Agentic.Tests.csproj -c Release

# Run a single test by name
dotnet test src/Agentic.Tests/Agentic.Tests.csproj --filter "FullyQualifiedName~AgentBuilderTests.Build_throws_when_model_provider_missing"

# Run tests with verbose output
dotnet test src/Agentic.Tests/Agentic.Tests.csproj -c Release -v n
```

## Code Style Guidelines

### Language Features
- **Target Framework**: .NET 10.0 (preview)
- **LangVersion**: preview (enables latest C# features)
- **ImplicitUsings**: enabled
- **Nullable**: enabled

### Formatting Conventions
- Use **file-scoped namespaces**: `namespace Agentic.Core;` (no braces)
- Use **collection expressions**: `var list = new List<T>();` -> `List<T> list = [];`
- Use **target-typed new**: `new AgentBuilder()` instead of `new AgentBuilder()`
- Use **raw string literals** for multi-line strings
- Use **pattern matching**: `if (x is not null)` instead of `if (x != null)`

### Class Design
- Mark classes as **`sealed`** by default unless inheritance is explicitly needed
- Use **primary constructors** where appropriate
- Place `internal` constructors before `public` ones
- Use `private readonly` for fields that are set once

### Naming Conventions
- **Interfaces**: Prefix with `I` (e.g., `IAgentModel`, `ITool`)
- **Types**: PascalCase for all type names
- **Fields**: `_camelCase` (underscore prefix, camelCase)
- **Private fields**: `_fieldName`
- **Constants**: `PascalCase`
- **Parameters**: `camelCase`

### Import Organization
- Use **implicit usings** (enabled in csproj)
- Explicitly import namespaces only when needed to avoid ambiguity
- Group related imports together (Abstractions, Core, Middleware, Providers)

### Error Handling
- Throw **`InvalidOperationException`** for precondition failures
- Throw **`ArgumentNullException`** / **`ArgumentException`** for invalid arguments
- Use meaningful error messages: `"Tool '{name}' is not registered."`
- Avoid empty catch blocks; log or rethrow

### Async Patterns
- Use **`CancellationToken`** as the last parameter in async methods
- Always default to `= default` for CancellationToken parameters
- Return `Task` or `Task<T>`, avoid `void` except for event handlers

### Testing (xUnit)
- Use **`[Fact]`** attribute for test methods
- Use **`Assert.Throws<T>()`** for exception testing
- Use **`await`** for async test methods
- Create helper classes in the same test file as `private sealed class`
- Use descriptive test names: `Method_Scenario_ExpectedResult`

### Project Structure

```
agentic.net/
├── src/
│   ├── Agentic.NET/      # Library
│   │   ├── Abstractions/ # Interfaces and contracts (ITool, IMemoryService, etc.)
│   │   ├── Builder/      # AgentBuilder fluent API
│   │   ├── Core/         # Runtime types (Agent, AgentContext, ChatMessage)
│   │   ├── Loaders/      # Skill and SOUL document loaders
│   │   ├── Middleware/   # Middleware contracts and implementations
│   │   ├── Stores/       # Vector store implementations (PgVector, InMemory)
│   │   └── Agentic.NET.csproj
│   └── Agentic.Tests/    # Unit tests
├── samples/              # Usage examples
└── Agentic.NET.sln
```

### Key Interfaces
- **`IChatClient`** (Microsoft.Extensions.AI): Underlying LLM/chat model abstraction — bring any MEAI-compatible provider
- **`IEmbeddingGenerator<string, Embedding<float>>`** (Microsoft.Extensions.AI): Generates embeddings/vectors for semantic memory search
- **`IMemoryService`**: Memory storage/retrieval with optional embeddings
- **`IVectorStore`**: Pluggable storage for embeddings (pgvector, in-memory, etc.)
- **`ISkillLoader`**: Loads agent skills from filesystem
- **`ISoulLoader`**: Loads agent identity from SOUL.md
- **`IAssistantMiddleware`**: Pre/post-process conversation
- **`ITool`**: Executable function the model can invoke

> **Note:** `IAgentModel` is `internal` — it is an adapter wrapping `IChatClient` and is not part of the public API. Users work with `IChatClient` from `Microsoft.Extensions.AI`.

### Configuration
- Solution file: `Agentic.NET.sln`
- Main project: `Agentic.NET.csproj`
- Test project: `tests/Agentic.Tests/Agentic.Tests.csproj`

### Environment Variables (for samples)
- `OPENAI_API_KEY`: Required for OpenAI samples
- `OPENAI_MODEL`: Optional, defaults to `gpt-4o-mini`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oorosh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
