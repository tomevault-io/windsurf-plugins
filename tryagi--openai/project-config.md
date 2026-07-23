---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

C# SDK for the [OpenAI](https://openai.com/) API, auto-generated from the official OpenAI OpenAPI specification using [AutoSDK](https://github.com/tryAGI/AutoSDK). Published as the `tryAGI.OpenAI` NuGet package. Includes support for custom providers (Azure, DeepSeek, Groq, Ollama, OpenRouter, Together, and many more), a source generator for native C# function/tool definitions, pricing constants, and structured output helpers.

## Build Commands

```bash
# Build the solution
dotnet build OpenAI.slnx

# Build for release (also produces NuGet package)
dotnet build OpenAI.slnx -c Release

# Run integration tests (requires OPENAI_API_KEY env var)
dotnet test src/tests/OpenAI.IntegrationTests/OpenAI.IntegrationTests.csproj

# Run snapshot tests for CSharpToJsonSchema
dotnet test src/tests/CSharpToJsonSchema.SnapshotTests/CSharpToJsonSchema.SnapshotTests.csproj

# Run unit tests for CSharpToJsonSchema
dotnet test src/tests/CSharpToJsonSchema.UnitTests/CSharpToJsonSchema.UnitTests.csproj

# Regenerate SDK from OpenAPI spec
cd src/libs/tryAGI.OpenAI && ./generate.sh
```

## Architecture

### Code Generation Pipeline

The SDK code in `Generated/` is **auto-generated** -- do not manually edit files in `src/libs/tryAGI.OpenAI/Generated/`.

1. `src/libs/tryAGI.OpenAI/openapi.yaml` -- the OpenAI OpenAPI spec (fetched from Stainless)
3. `src/libs/tryAGI.OpenAI/generate.sh` -- orchestrates: download spec, fix spec, run AutoSDK CLI, output to `Generated/`
4. CI auto-updates the spec and creates PRs if changes are detected

### Hand-Written Extensions

Unlike most other tryAGI SDKs, this repo has significant **hand-written code** alongside the generated code:

| Path | Purpose |
|------|---------|
| `Metadata/` | Pricing constants and model metadata (chat, embedding, images, TTS, STT) |
| `Conversions/` | Implicit conversions for `ChatCompletionRequestMessage`, `CreateMessageRequest`, etc. |
| `Extensions/StringExtensions.cs` | Helper extension methods |
| `CustomProviders.cs` | Factory methods for Azure, DeepSeek, Groq, Ollama, OpenRouter, etc. |
| `ChatClient.CreateChatCompletion.AsStream.cs` | Streaming chat completion extensions |
| `ChatClient.CreateChatCompletion.As.cs` | Structured output (`CreateChatCompletionAsAsync<T>`) |
| `AssistantClient.CreateRun.AsStream.cs` | Streaming assistant run extensions |
| `RealtimeConversationClient.cs` | Realtime API WebSocket client |
| `OpenAiApi.Headers.cs` | Custom header helpers |
| `TypeToSchemaHelpers.cs` | JSON schema generation helpers for tools |
| `Image.Bytes.cs` | Image byte conversion helpers |

### Project Layout

| Project | Purpose |
|---------|---------|
| `src/libs/tryAGI.OpenAI/` | Main SDK library (`OpenAiClient` / `OpenAiApi`) |
| `src/tests/OpenAI.IntegrationTests/` | Integration tests against real OpenAI API and custom providers |
| `src/tests/CSharpToJsonSchema.SnapshotTests/` | Snapshot tests for schema generation |
| `src/tests/CSharpToJsonSchema.UnitTests/` | Unit tests for schema generation |

### Documentation Generation

Tests in `src/tests/OpenAI.IntegrationTests/Examples` are the single source of truth for both test coverage and documentation:
- Each file has a JSDoc header (`order`, `title`, `slug`) consumed by `autosdk docs sync .`
- Comments prefixed with `////` become prose paragraphs in generated docs
- CI workflow (`.github/workflows/mkdocs.yml`) auto-generates `docs/examples/` and populates `EXAMPLES:START/END` markers in README.md, docs/index.md, and mkdocs.yml
- Config: `autosdk.docs.json` points to `src/tests/OpenAI.IntegrationTests/Examples`
### Key Dependencies

- `CSharpToJsonSchema` -- source generator for defining tools/functions via C# interfaces
- `Tiktoken` -- token counting for pricing calculations
- `System.Net.ServerSentEvents` -- SSE parsing for streaming endpoints

### Build Configuration

- **Target:** `net10.0` (single target)
- **Language:** C# preview with nullable reference types
- **Signing:** Strong-named assemblies via `src/key.snk`
- **Versioning:** Semantic versioning from git tags (`v` prefix) via MinVer
- **Analysis:** All .NET analyzers enabled, AOT/trimming compatibility enforced
- **Testing:** MSTest + AwesomeAssertions
- **CLS Compliant:** Assembly marked as CLS-compliant

### CI/CD

- Uses shared workflows from `HavenDV/workflows` repo
- Dependabot updates NuGet packages weekly (auto-merged)
- Documentation deployed to GitHub Pages via MkDocs Material

## Key Conventions

- The NuGet package name is `tryAGI.OpenAI` (not just `OpenAI`) to avoid conflicts with the official OpenAI package
- The main client class is `OpenAiApi` (for backward compatibility) with `OpenAiClient` as the generated variant
- Custom providers are accessed via static factory methods on `CustomProviders` class (e.g., `CustomProviders.Azure(...)`)

## CustomProviders — Environment Variables for Tests

Each custom provider requires specific environment variables to run its integration tests. Tests skip (not fail) if the key is unset.

| Provider | Env Var(s) | Default Model | Capabilities |
|----------|-----------|---------------|--------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tryAGI/OpenAI](https://github.com/tryAGI/OpenAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
