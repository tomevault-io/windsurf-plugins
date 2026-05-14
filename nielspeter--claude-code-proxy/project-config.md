---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Claude Code Proxy is an HTTP proxy that translates Claude API requests to OpenAI-compatible format, enabling Claude Code to work with 200+ alternative models through OpenRouter, OpenAI Direct (o1/o3), and Ollama (local). The proxy runs as a daemon, performs bidirectional API format conversion, and maintains full Claude Code feature compatibility including tool calling, extended thinking blocks, and streaming.

## Build Commands

```bash
# Build the binary
go build -o claude-code-proxy cmd/claude-code-proxy/main.go
# Or use make
make build

# Build for all platforms (creates dist/ folder)
make build-all

# Run tests
go test ./...

# Run specific test file
go test -v ./internal/converter

# Run single test
go test -v ./internal/converter -run TestConvertMessagesWithComplexContent

# Run tests with coverage
make test-coverage

# Format code
go fmt ./...

# Compile and start proxy in simple log mode
go build -o claude-code-proxy cmd/claude-code-proxy/main.go && ./claude-code-proxy -s
```

## Architecture

### Core Request Flow

1. **Claude Code** → sends Claude API format request to `localhost:8082`
2. **handlers.go** → receives `/v1/messages` POST request
3. **converter.go** → transforms Claude format → OpenAI format
   - Detects provider type (OpenRouter/OpenAI/Ollama) via `cfg.DetectProvider()`
   - Applies provider-specific parameters (reasoning format, tool_choice)
   - Maps Claude model name to target provider model using pattern-based routing
4. **handlers.go** → forwards OpenAI request to configured provider
5. **Provider** → returns OpenAI-format response (streaming or non-streaming)
6. **converter.go** → transforms OpenAI format → Claude format
7. **handlers.go** → returns Claude-format response to Claude Code

### Provider-Specific Behavior

The proxy applies different request parameters based on `OPENAI_BASE_URL`:

**OpenRouter** (`https://openrouter.ai/api/v1`):
- Adds `reasoning: {enabled: true}` for thinking support
- Uses `usage: {include: true}` for token tracking
- Extracts `reasoning_details` array → converts to Claude `thinking` blocks

**OpenAI Direct** (`https://api.openai.com/v1`):
- Adds `reasoning_effort: "medium"` for GPT-5 reasoning models
- Uses standard `stream_options: {include_usage: true}`

**Ollama** (`http://localhost:*`):
- Sets `tool_choice: "required"` when tools are present (forces tool usage)
- No API key validation (localhost endpoints skip auth)

### Format Conversion Details

**Tool Calling** (`convertMessages` in converter.go):
- Claude `tool_use` content blocks → OpenAI `tool_calls` array
- OpenAI `tool_calls` → Claude `tool_use` blocks
- Maintains `tool_use.id` ↔ `tool_result.tool_use_id` correspondence
- Preserves JSON arguments as strings during conversion

**Thinking Blocks** (`ConvertResponse` in converter.go):
- OpenRouter `reasoning_details` → Claude `thinking` block with `signature` field
- `signature` field is REQUIRED for Claude Code to hide/show thinking properly
- Without signature, thinking appears as regular text in chat

**Streaming** (`streamOpenAIToClaude` in handlers.go):
- Converts OpenAI SSE chunks (`data: {...}`) → Claude SSE events
- Generates proper event sequence: `message_start`, `content_block_start`, `content_block_delta`, `content_block_stop`, `message_delta`, `message_stop`
- Tracks content block indices to maintain proper ordering
- Handles tool call deltas by accumulating function arguments across chunks

### Pattern-Based Model Routing

The `mapModel()` function in converter.go implements intelligent routing:

```go
// Haiku tier → lightweight models
"*haiku*" → gpt-5-mini (or ANTHROPIC_DEFAULT_HAIKU_MODEL)

// Sonnet tier → version-aware
"*sonnet-4*" or "*sonnet-5*" → gpt-5
"*sonnet-3*" → gpt-4o
(or ANTHROPIC_DEFAULT_SONNET_MODEL)

// Opus tier → flagship models
"*opus*" → gpt-5 (or ANTHROPIC_DEFAULT_OPUS_MODEL)
```

Override via environment variables to route to alternative models (Grok, Gemini, DeepSeek-R1, etc.).

###  Adaptive Per-Model Capability Detection

**Core Philosophy**: Support all provider quirks automatically - never burden users with advance configs.

The proxy uses a fully adaptive system that automatically learns what parameters each model supports through error-based retry and caching. This eliminates ALL hardcoded model patterns (~100 lines removed in v1.3.0).

**How It Works:**

1. **First Request (Cache Miss)**:
   - `ShouldUseMaxCompletionTokens()` checks cache for `CacheKey{BaseURL, Model}`
   - Cache miss → defaults to trying `max_completion_tokens` (correct for reasoning models)
   - If provider returns "unsupported parameter" error, `retryWithoutMaxCompletionTokens()` is called
   - Retry succeeds → cache `{UsesMaxCompletionTokens: false}`
   - Original request succeeds → cache `{UsesMaxCompletionTokens: true}`

2. **Subsequent Requests (Cache Hit)**:
   - `ShouldUseMaxCompletionTokens()` returns cached value immediately
   - No trial-and-error needed
   - ~1-2 second first request penalty, instant thereafter

**Cache Structure** (`internal/config/config.go:29-48`):

```go
type CacheKey struct {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nielspeter/claude-code-proxy](https://github.com/nielspeter/claude-code-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
