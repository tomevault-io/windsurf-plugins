---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is **OpenAI Scala Client** - an async Scala client for OpenAI API and multiple LLM providers. It's a multi-module Scala project that supports Scala 2.12, 2.13, and 3, providing comprehensive coverage of OpenAI endpoints plus adapters for Anthropic, Google (Gemini/Vertex AI), Perplexity, and other LLM providers.

The library is designed to be self-contained with minimal dependencies and uses a Play WS backend for HTTP calls. It's published as `io.cequence:openai-scala-client` on Maven Central.

## Build & Test Commands

The project uses SBT with custom command aliases defined in build.sbt:

### Core Commands
- **Build & test**: `sbt clean test` or `sbt ++2.13.11 clean test` for a specific Scala version
- **Test with coverage**: `sbt testWithCoverage` (alias for `coverage; test; coverageReport`)
- **Format code**: `sbt formatCode` (runs scalafmt, scalafmtSbt, and Test/scalafmt)
- **Validate code**: `sbt validateCode` (runs scalafix and scalafmt checks - this is what CI runs)
- **Compile specific module**: `sbt core/compile` or `sbt client/compile` etc.
- **Run specific test**: `sbt "testOnly *YourTestClassName"`
- **Run example**: `sbt "examples/runMain io.cequence.openaiscala.examples.YourExampleClass"`

### Cross-Build
The project cross-compiles for Scala 2.12.18, 2.13.11, and 3.2.2:
- `sbt ++2.12.18 test`
- `sbt ++2.13.11 test`
- `sbt ++3.2.2 test`

### Running Examples
The `Example` trait uses `System.exit()` which sbt's TrapExit mechanism intercepts, causing output to be swallowed. For reliable output when running examples from sbt, either:
- Run from IntelliJ directly (recommended)
- Write standalone `main` methods using `Await.result` instead of extending `Example`

## Module Architecture

The codebase is organized into multiple SBT subprojects with clear dependency relationships:

### Core Modules
- **openai-core** (`core`): Core domain models, JSON formats, service interfaces (OpenAIService, OpenAICoreService, OpenAIChatCompletionService), adapters, retry helpers, and base exception types. This is the foundation that other modules depend on.
- **openai-client** (`client`): Main client implementation with factories (OpenAIServiceFactory, OpenAIChatCompletionServiceFactory) and concrete service implementations. Depends on and aggregates openai-core.
- **openai-client-stream** (`client_stream`): Streaming extensions providing OpenAIStreamedServiceExtra and streaming factories for SSE-based completions. Depends on openai-client.

### Provider-Specific Clients
- **anthropic-client** (`anthropic_client`): Anthropic/Claude API client with OpenAI-compatible adapter. Supports MCP toolsets, extended thinking, fast mode, tools (bash, code execution, computer use, web search/fetch, text editor, memory). Also supports x-api-key, static bearer/OAuth token (`ANTHROPIC_AUTH_TOKEN` → `CLAUDE_CODE_OAUTH_TOKEN_ALTERNATIVE` → `CLAUDE_CODE_OAUTH_TOKEN`; the `_ALTERNATIVE` var is safe to export persistently since the real `claude` CLI never reads it), and `ant auth` OAuth profiles with auto-refresh (`forAuthToken` / `forOAuthProfile` / `forAuthTokenProvider` / `customInstance`).
- **google-vertexai-client** (`google_vertexai_client`): Google Vertex AI client (Gemini models on GCP). Supports tools (function declarations, Google search, code execution) with ToolConfig.
- **google-gemini-client** (`google_gemini_client`): Google Gemini API client (direct Gemini API). Supports tools, prompt caching, thinking levels, and has its own exception hierarchy (GeminiScalaClientException) with error code handling.
- **perplexity-sonar-client** (`perplexity_sonar_client`): Perplexity Sonar search-based AI client.
- **claude-agent-client** (`claude_agent_client`): Subprocess transport wrapping the `claude` CLI (Claude Agent SDK-compatible NDJSON protocol over stdin/stdout) - full bidirectional sessions with tool-permission callbacks and interrupt support, distinct from the HTTP-based `anthropic-client`. Requires the `claude` CLI installed and authenticated separately (API key or Claude subscription).

All provider clients depend on openai-core and provide `asOpenAI()` adapters to work with the standard OpenAI interfaces, with one exception: `claude-agent-client` is a fundamentally different subprocess/NDJSON transport (not an `OpenAIChatCompletionService`) and does NOT provide an `asOpenAI()` adapter.

### Utility Modules
- **openai-all** (`all`): Envelope module aggregating all clients (except guice) into a single dependency: `openai-scala-all`.
- **openai-count-tokens** (`count_tokens`): Token counting utilities (OpenAICountTokensHelper) using jtokkit for estimating API costs before making calls.
- **openai-guice** (`guice`): Dependency injection support using scala-guice.
- **openai-examples** (`examples`): Comprehensive examples demonstrating all features.

### Module Dependency Graph
```
openai-core
    ├── openai-client (aggregates core)
    │   ├── openai-client-stream (aggregates client)
    │   └── openai-count-tokens
    ├── anthropic-client (aggregates core + client + client-stream)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cequence-io/openai-scala-client](https://github.com/cequence-io/openai-scala-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
