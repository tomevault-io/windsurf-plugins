---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Build and Test
```bash
# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run tests with race detection
go test -race ./...

# Run specific test
go test -run TestChatComplete

# Run tests in verbose mode
go test -v ./...

# Build the package
go build ./...

# Format code
go fmt ./...

# Run go vet for static analysis
go vet ./...

# Install dependencies
go mod download

# Update dependencies
go mod tidy
```

### Running Examples
```bash
# Set API key
export OPENROUTER_API_KEY="your-api-key"

# Run examples
go run examples/basic/main.go
go run examples/streaming/main.go
go run examples/list-models/main.go
go run examples/advanced/main.go
go run examples/structured-output/main.go
go run examples/tool-calling/main.go
go run examples/web_search/main.go
go run examples/transforms/main.go
go run examples/app-attribution/main.go
go run examples/image-inputs/main.go
go run examples/audio-inputs/main.go
go run examples/pdf-inputs/main.go
go run examples/text-file-inputs/main.go
go run examples/mcp-tools/main.go
go run examples/responses/main.go
go run examples/embedding-chunking/main.go
go run examples/rerank/main.go
go run examples/broadcast-webhook/main.go
```

### Running E2E Tests
```bash
# Set API key
export OPENROUTER_API_KEY="your-api-key"

# Run all tests
go run cmd/openrouter-test/main.go -test all

# Run specific test
go run cmd/openrouter-test/main.go -test models
go run cmd/openrouter-test/main.go -test chat
go run cmd/openrouter-test/main.go -test streaming
go run cmd/openrouter-test/main.go -test image
go run cmd/openrouter-test/main.go -test audio
go run cmd/openrouter-test/main.go -test audiobuilder
go run cmd/openrouter-test/main.go -test audioformats
go run cmd/openrouter-test/main.go -test pdf
go run cmd/openrouter-test/main.go -test pdfengine
go run cmd/openrouter-test/main.go -test pdfannotations
go run cmd/openrouter-test/main.go -test base64pdf
go run cmd/openrouter-test/main.go -test pdfcomparison
go run cmd/openrouter-test/main.go -test textfile
go run cmd/openrouter-test/main.go -test multipletextfiles
go run cmd/openrouter-test/main.go -test textbuilder
go run cmd/openrouter-test/main.go -test textformats
go run cmd/openrouter-test/main.go -test mcp
go run cmd/openrouter-test/main.go -test responses
go run cmd/openrouter-test/main.go -test responses-reasoning
go run cmd/openrouter-test/main.go -test responses-tools
go run cmd/openrouter-test/main.go -test responses-websearch
go run cmd/openrouter-test/main.go -test responses-stream
go run cmd/openrouter-test/main.go -test zdr-endpoints
go run cmd/openrouter-test/main.go -test models-user
go run cmd/openrouter-test/main.go -test chunking
go run cmd/openrouter-test/main.go -test chunkedembedding
go run cmd/openrouter-test/main.go -test rerank
go run cmd/openrouter-test/main.go -test guardrails

# Run with verbose output
go run cmd/openrouter-test/main.go -test models -v
```

## Architecture Overview

This is a zero-dependency Go client library for the OpenRouter API that follows idiomatic Go patterns.

### Core Components

**Client (`client.go`)**: The main client struct that manages HTTP communication, authentication, and request routing. Uses functional options pattern for configuration.

**Request/Response Types (`models.go`)**: Defines all data structures for API requests and responses, including messages, tools, and completion parameters.

**Functional Options (`options.go`)**: Implements the options pattern for flexible configuration of both client and request parameters. This allows clean API usage like `WithModel()`, `WithMessages()`, etc.

**Streaming (`stream.go`)**: Implements Server-Sent Events (SSE) parsing for streaming responses without external dependencies. Handles reconnection and error recovery.

**Error Handling (`errors.go`)**: Custom error types that preserve OpenRouter API error details including rate limits, retry information, and moderation flags.

**Retry Logic (`retry.go`)**: Implements exponential backoff with jitter for handling transient failures and rate limits.

### API Endpoints

**Chat Completions (`chat.go`)**: Modern chat API supporting messages, tools, structured outputs, and streaming.

**Legacy Completions (`completions.go`)**: Support for older prompt-based completion API.

**Models Listing (`models_endpoint.go`)**: Retrieve available models with filtering by category and detailed model information.

**Web Search (`web_search.go`)**: Integration with OpenRouter's web search plugin for augmented responses.

**MCP Tool Conversion (`mcp.go`)**: Utilities for converting MCP (Model Context Protocol) tool definitions to OpenAI-compatible format for use with OpenRouter.

**Responses API (`responses.go`, `responses_models.go`, `responses_options.go`)**: **[BETA - UNSTABLE]** OpenAI-compatible Responses API with support for reasoning, tool calling, web search, and streaming. This is a stateless API where each request is independent. **WARNING: This API is in beta and may have breaking changes at any time. Do not use in production workloads.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hra42) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
