---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

sttp-ai is a Scala library providing a non-official client wrapper for OpenAI, Claude (Anthropic), and OpenAI-compatible APIs. Built on sttp HTTP client with support for sync/async operations and various effect systems (cats-effect, ZIO, Akka/Pekko Streams, Ox).

**Key Features:**
- Native OpenAI API support (Chat, Completions, Embeddings, Audio, Images, etc.)
- Native Claude (Anthropic) API support with dedicated module
- OpenAI-compatible API support (Ollama, Grok, OpenRouter, etc.)
- Streaming support for all major effect systems
- Cross-platform: Scala 2.13.16 and Scala 3.3.6
- Agent loop tools loadable from [MCP](https://modelcontextprotocol.io) servers (`mcp` module, Scala 3 only, via [chimp](https://github.com/softwaremill/chimp)), in addition to manually defined `AgentTool`s

## Development Commands

### Essential Commands
```bash
# Compile
sbt compile                              # All modules
sbt openai/compile                       # OpenAI module
sbt claude/compile                       # Claude module
sbt mcp3/compile                         # MCP module (Scala 3 only; sbt-projectmatrix suffixes the Scala 3 row with "3")

# Test
sbt test                                 # Unit tests (excludes integration)
sbt "testOnly *OpenAIIntegrationSpec"   # OpenAI integration (requires OPENAI_API_KEY)
sbt "testOnly *ClaudeIntegrationSpec"   # Claude integration (requires ANTHROPIC_API_KEY)
./run-integration-tests.sh              # All integration tests

# Format (CRITICAL - always run after changes!)
sbt scalafmtAll                         # Format all code
sbt scalafmtCheck                       # Verify formatting
sbt Test/scalafmtCheck                  # Verify test formatting

# Documentation
sbt compileDocumentation                # Compile mdoc documentation
```

**If `jetbrains` MCP is available, USE `mcp__jetbrains__reformat_file` tool instead of running `sbt scalafmtAll` command.**

### Model Update Scripts
```bash
# Update OpenAI model definitions (automated workflow)
scala-cli model_update_scripts/scrape_models.scala                        # 1. Scrape models
scala-cli model_update_scripts/update_code_with_new_models.scala --apply  # 2. Update code
sbt scalafmtAll                                                           # 3. Format
```

## Architecture Patterns

### Dual API Support (OpenAI + Claude)

**Core Differences:**

| Aspect | OpenAI (`openai/`) | Claude (`claude/`) |
|--------|-----------------|-------------------|
| **Client** | `OpenAI` / `OpenAISyncClient` | `ClaudeClient` / `ClaudeSyncClient` |
| **Return Type** | `Either[OpenAIException, A]` | `Either[ClaudeException, A]` |
| **Message Content** | Simple strings | `ContentBlock` arrays (rich content) |
| **System Messages** | Role-based in messages array | Separate `system` parameter |
| **Authentication** | `Authorization: Bearer <key>` | `x-api-key: <key>` + `anthropic-version` |
| **Package Structure** | `sttp.ai.openai.*` | `sttp.ai.claude.*` |

**Shared Patterns:**
- Both use **uPickle with SnakePickle** for JSON (snake_case conversion)
- Both have **comprehensive exception hierarchies** for API errors
- Both support **streaming via SSE** with same effect systems

### Streaming Architecture

Each streaming module (`streaming/{effect-system}/`) provides extensions for **both** APIs:

| Effect System | Module Location | OpenAI Extension | Claude Extension | Scala Version |
|--------------|----------------|-----------------|------------------|---------------|
| **fs2** | `streaming/fs2/` | `sttp.ai.openai.streaming.fs2.*` | `sttp.ai.claude.streaming.fs2.*` | 2.13, 3 |
| **zio** | `streaming/zio/` | `sttp.ai.openai.streaming.zio.*` | `sttp.ai.claude.streaming.zio.*` | 2.13, 3 |
| **akka** | `streaming/akka/` | `sttp.ai.openai.streaming.akka.*` | `sttp.ai.claude.streaming.akka.*` | 2.13 only |
| **pekko** | `streaming/pekko/` | `sttp.ai.openai.streaming.pekko.*` | `sttp.ai.claude.streaming.pekko.*` | 2.13, 3 |
| **ox** | `streaming/ox/` | `sttp.ai.openai.streaming.ox.*` | `sttp.ai.claude.streaming.ox.*` | 3 only |

**Pattern:** Extension methods add `.parseSSE` + `.parseClaudeStreamResponse`/`.parseOpenAIStreamResponse`

### Key Navigation Tips

- **OpenAI API endpoints**: `openai/src/main/scala/sttp/ai/openai/requests/{api-category}/`
- **Claude API code**: `claude/src/main/scala/sttp/ai/claude/`
- **OpenAI models**: Search for `ChatCompletionModel`, `EmbeddingModel` in `openai/` request bodies
- **Claude models**: `claude/src/main/scala/sttp/ai/claude/models/ClaudeModel.scala`
- **Streaming implementations**: `streaming/{effect-system}/src/main/scala/`
- **MCP tool loading**: `mcp/src/main/scala/sttp/ai/core/agent/mcp/McpTools.scala` (Scala 3 only; depends on `core` and chimp's `chimp-client`, not on `openai`/`claude`)
- **Examples**: `examples/src/main/scala/examples/` (runnable with scala-cli)
- **Tests**: Each module has `{module}/src/test/` following same package structure

**Request package mirrors OpenAI API structure**: `requests/{api-category}/` contains endpoint-specific request/response models.

## Code Style & Formatting


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [softwaremill/sttp-ai](https://github.com/softwaremill/sttp-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
