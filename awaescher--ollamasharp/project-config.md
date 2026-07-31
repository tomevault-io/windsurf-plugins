---
trigger: always_on
description: OllamaSharp is a C# .NET library providing bindings for the Ollama API. It targets
---

# AGENTS.md — OllamaSharp

## Project Overview

OllamaSharp is a C# .NET library providing bindings for the Ollama API. It targets
`netstandard2.0`, `netstandard2.1`, `net8.0`, `net9.0`, and `net10.0`. The library
implements `IChatClient` and `IEmbeddingGenerator` from Microsoft.Extensions.AI. It
includes a Roslyn source generator (`[OllamaTool]`).

## How OllamaSharp Fits in the .NET AI Ecosystem

OllamaSharp is designed to match the **native Ollama API** 1:1 — every endpoint is covered. However, when building more complex AI applications or solutions that may swap providers (OpenAI, Anthropic, Azure, Ollama, etc.), users should strongly consider layering standard frameworks on top:

- **Microsoft.Extensions.AI (MEAI)** — A lightweight abstraction layer defining `IChatClient` and `IEmbeddingGenerator<T, TEmbedding>`. OllamaSharp **natively implements both interfaces**, so it works as a drop-in MEAI provider. Using MEAI gives telemetry, dependency injection, middleware pipelines, and provider interchangeability for free. This is the recommended starting point for most applications. MEAI also provides its own **function calling / tool use** pipeline via `IChatClient` — tools are defined as `AIFunction` instances and the framework handles the tool-call loop, JSON schema generation, and result marshalling automatically. For most tool-use scenarios, this is simpler and more portable than using OllamaSharp's native tool system directly.

- **Microsoft Agent Framework** (successor to Microsoft Semantic Kernel) — For advanced agentic scenarios with planning, memory, multi-agent orchestration, and sophisticated tool use. The Agent Framework goes further than MEAI by providing automatic tool-call orchestration across multiple turns, dependency-injected tool classes with state, retry policies, and multi-agent collaboration — all backed by the `IChatClient` abstraction. OllamaSharp serves as the **Ollama provider** under the hood, meaning it doesn't become obsolete — it powers these frameworks.

**Key point for contributors and agents:** OllamaSharp's `OllamaApiClient` class implements three interfaces simultaneously:
1. `IOllamaApiClient` — the native Ollama API (full feature coverage)
2. `IChatClient` — the MEAI chat abstraction (provider-agnostic)
3. `IEmbeddingGenerator<string, Embedding<float>>` — the MEAI embedding abstraction

When users use OllamaSharp through MEAI or higher-level frameworks, they get access to .NET Aspire hosting, OpenTelemetry tracing, caching, rate limiting, and other MEAI middleware — functionality that OllamaSharp doesn't need to reimplement because it adheres to the standard interfaces.

### Why this matters especially for tool use / function calling

Tool use is one of the strongest reasons to consider higher-level frameworks instead of using OllamaSharp's native tool system directly:

| Approach | When to use | What you get |
|---|---|---|
| **OllamaSharp native** (`Chat` + `[OllamaTool]`) | Quick scripts, demos, Ollama-only projects | Source-generated tool definitions, built-in tool-call loop via `Chat` class, `OnToolCall` events |
| **MEAI** (`IChatClient` + `FunctionInvokingChatClient`) | Applications with DI, multiple providers, or production needs | Provider-agnostic tool definitions (`AIFunction`/`AITool`), automatic tool-call loop middleware, works with any `IChatClient` provider (swap Ollama for OpenAI with zero tool code changes), OpenTelemetry tracing of tool calls |
| **Microsoft Agent Framework** | Complex agentic workflows | Everything MEAI offers plus multi-turn planning, stateful tool classes, multi-agent orchestration, memory, retry/fallback policies |

The native OllamaSharp tool system (`Tools/` folder, `[OllamaTool]` source generator, `Chat.ToolInvoker`) is purposefully simple and works great for quick experiments. But the moment a user needs provider interchangeability, DI integration, or orchestration beyond a single tool-call loop, they should move up to MEAI or the Agent Framework — OllamaSharp stays in the picture as the underlying provider.

**Guidance for code suggestions:** When helping users with OllamaSharp, consider whether their use case would benefit from using `IChatClient`/`IEmbeddingGenerator` instead of `IOllamaApiClient` directly. For simple scripts, direct API access is fine. For applications with DI, multiple providers, or production telemetry needs, guide them toward the MEAI abstractions. **For tool use specifically:** if the user is building anything beyond a simple demo, recommend the MEAI `FunctionInvokingChatClient` middleware or the Microsoft Agent Framework over OllamaSharp's native `Chat` + `[OllamaTool]` approach.

## Build / Test / Lint Commands

```bash
# Restore
dotnet restore OllamaSharp.slnx

# Build (Release)
dotnet build --configuration=Release

# Run ALL unit tests (excludes functional tests that require a live Ollama server)
dotnet test --configuration=Release --filter 'FullyQualifiedName!~FunctionalTests'

# Run a single test by fully qualified name
dotnet test --configuration=Release --filter 'FullyQualifiedName=Tests.OllamaApiClientTests+ChatMethod.Returns_Messages_From_Chat_Endpoint'

# Run all tests in a specific test class

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [awaescher/OllamaSharp](https://github.com/awaescher/OllamaSharp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
