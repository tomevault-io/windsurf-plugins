---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

C# implementation of the LangChain framework for building applications with LLMs through composability. Provides chains, memory, RAG (Retrieval-Augmented Generation), document loaders, text splitters, and a serving layer. Distributed as multiple NuGet packages under the `LangChain` namespace. The meta-package `LangChain` bundles the most commonly used providers (OpenAI, Anthropic, Google, Ollama, Azure, etc.) with the core library.

## Build and Test Commands

```bash
# Build the entire solution
dotnet build LangChain.slnx

# Run all integration tests
dotnet test src/Meta/test/LangChain.IntegrationTests.csproj

# Run core unit tests
dotnet test src/Core/test/UnitTests/LangChain.Core.UnitTests.csproj

# Run splitter tests
dotnet test src/Splitters/Abstractions/test/LangChain.Splitters.Abstractions.Tests.csproj

# Run a specific test
dotnet test src/Meta/test/LangChain.IntegrationTests.csproj --filter "FullyQualifiedName~WikiTests"

# Validate trimming/NativeAOT compatibility (requires: dotnet tool install -g autosdk.cli --prerelease)
autosdk trim src/libs/*//*.csproj
```

Integration tests require API keys via environment variables (e.g., `OPENAI_API_KEY`). Tests skip (not fail) if keys are unset.

## Architecture

### Project Structure

```
src/
├── Core/src/                  # LangChain.Core — chains, memory, prompts, retrievers, schema
├── Meta/src/                  # LangChain (meta-package) — bundles Core + popular providers
├── Meta/test/                 # Integration tests (WikiTests, ReadmeTests)
├── DocumentLoaders/
│   ├── Abstractions/          # LangChain.DocumentLoaders.Abstractions
│   ├── Pdf/                   # LangChain.DocumentLoaders.Pdf (PdfPig-based)
│   ├── WebBase/               # LangChain.DocumentLoaders.Html
│   └── Word/                  # LangChain.DocumentLoaders.Word
├── Splitters/
│   ├── Abstractions/          # LangChain.Splitters.Abstractions (CharacterTextSplitter, etc.)
│   └── CSharp/                # LangChain.Splitters.CSharp
├── Extensions/
│   └── Docker/                # LangChain.Extensions.Docker
├── Serve/
│   ├── Abstractions/          # LangChain.Serve.Abstractions
│   ├── src/                   # LangChain.Serve (ASP.NET Core middleware)
│   └── OpenAI/                # LangChain.Serve.OpenAI (OpenAI-compatible serving)
├── Utilities/
│   ├── Pollyfils/             # LangChain.Polyfills (framework polyfills)
│   ├── Sql/                   # LangChain.Utilities.Sql
│   └── Postgres/              # LangChain.Utilities.Postgres
├── Cli/                       # LangChain.Cli
    # Note: Non-MEAI provider interfaces (TTS, Image, ImageToText) are in Core/src/Providers/
examples/                      # Sample projects (OpenAI, Azure, Memory, Serve, LocalRAG, etc.)
```

### Core Abstractions (src/Core/src/)

**Chains** (`Chains/`):
- `Chain` static class — fluent API for composing stackable chains using the `|` operator
- Key chain factory methods: `Set()`, `Template()`, `LLM()`, `RetrieveSimilarDocuments()`, `CombineDocuments()`, `LoadMemory()`, `UpdateMemory()`, `TTS()`, `STT()`, `ReActAgentExecutor()`, `GroupChat()`
- `StackableChains/` — individual chain implementations (LLM, ReAct agents, Crew agents, file chains, image generation)
- Chains are composed via the pipe `|` operator and executed with `chain.RunAsync("key")`

**Memory** (`Memory/`):
- `ConversationBufferMemory` — stores full conversation history
- `ConversationSummaryMemory` — stores summarized conversation history
- `ConversationSummaryBufferMemory` — hybrid (recent messages + summarized history)
- `ConversationWindowBufferMemory` — sliding window of recent messages

**Prompts** (`Prompts/`):
- Prompt template system with variable substitution via `{variable_name}` placeholders

**Base Classes** (`Base/`):
- `BaseChain` — abstract base for all chains
- `BaseCallbackHandler` — callback system for tracing and debugging

### Dependencies

LangChain.Core depends on:
- `LangChain.Databases.Abstractions` (NuGet) — message history (BaseChatMessageHistory, ChatMessageHistory)
- `Microsoft.Extensions.AI` (NuGet) — MEAI interfaces (IChatClient, IEmbeddingGenerator, ISpeechToTextClient, ChatMessage, ChatRole)
- `Microsoft.Extensions.VectorData.Abstractions` (NuGet) — MEVA vector store abstractions (`VectorStore`, `VectorStoreCollection<TKey, TRecord>`)
- `LangChain.DocumentLoaders.Abstractions` (project reference)
- `LangChain.Splitters.Abstractions` (project reference)
- Non-MEAI provider interfaces (ITextToSpeechModel, ITextToImageModel, IImageToTextModel) are defined locally in `src/Core/src/Providers/`

The meta-package additionally references:
- `tryAGI.OpenAI`, `tryAGI.Anthropic`, `Ollama` — SDK packages implementing MEAI interfaces natively
- `Google_Gemini` — conditional on net10.0 (only TFM it supports)
- `Microsoft.SemanticKernel.Connectors.InMemory` — in-memory MEVA vector store

### Vector Store (MEVA) Architecture


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tryAGI/LangChain](https://github.com/tryAGI/LangChain) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
