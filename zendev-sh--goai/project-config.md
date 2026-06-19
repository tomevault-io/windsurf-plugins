---
trigger: always_on
description: > Instructions for AI coding agents helping developers use the GoAI SDK.
---

# GoAI SDK - AI Agent Skill

> Instructions for AI coding agents helping developers use the GoAI SDK.
> Load this file when a user's project imports `github.com/zendev-sh/goai`.

## Overview

GoAI is a Go SDK for AI applications. One unified API across 25+ LLM providers. Inspired by the Vercel AI SDK, adapted to Go idioms (generics, interfaces, channels).

- **Package**: `github.com/zendev-sh/goai`
- **Go version**: 1.25+
- **Dependencies**: stdlib + `golang.org/x/oauth2` for Vertex AI. Optional `observability/otel` submodule adds OTel SDK (separate go.mod, not pulled unless imported).
- **Docs**: https://goai.sh
- **GoDoc**: https://pkg.go.dev/github.com/zendev-sh/goai

## Quick Reference

```
go get github.com/zendev-sh/goai@latest
```

```go
import (
    "github.com/zendev-sh/goai"
    "github.com/zendev-sh/goai/provider/openai"     // or any provider
)
```

## Core API

### 7 Top-Level Functions

| Function                                      | Purpose                         | Returns                     |
| --------------------------------------------- | ------------------------------- | --------------------------- |
| `goai.GenerateText(ctx, model, opts...)`      | Non-streaming text generation   | `(*TextResult, error)`      |
| `goai.StreamText(ctx, model, opts...)`        | Streaming text via channels     | `(*TextStream, error)`      |
| `goai.GenerateObject[T](ctx, model, opts...)` | Typed structured output (JSON)  | `(*ObjectResult[T], error)` |
| `goai.StreamObject[T](ctx, model, opts...)`   | Streaming structured output     | `(*ObjectStream[T], error)` |
| `goai.Embed(ctx, model, text, opts...)`       | Single text embedding           | `(*EmbedResult, error)`     |
| `goai.EmbedMany(ctx, model, texts, opts...)`  | Batch embeddings (auto-chunked) | `(*EmbedManyResult, error)` |
| `goai.GenerateImage(ctx, model, imgOpts...)`  | Image generation                | `(*ImageResult, error)`     |

### Model Constructors

Each provider has `Chat()`, and optionally `Embedding()` and `Image()`:

```go
// Language models
openai.Chat("gpt-4o")
anthropic.Chat("claude-sonnet-4-20250514")
google.Chat("gemini-2.5-flash")
bedrock.Chat("anthropic.claude-sonnet-4-20250514-v1:0")
azure.Chat("gpt-4o", azure.WithEndpoint("https://my-resource.openai.azure.com"))
vertex.Chat("gemini-2.5-flash", vertex.WithProject("my-project"), vertex.WithLocation("us-central1"))
groq.Chat("llama-3.3-70b-versatile")
ollama.Chat("llama3.2")

// Embedding models
openai.Embedding("text-embedding-3-small")
google.Embedding("text-embedding-004")
cohere.Embedding("embed-english-v3.0")
ollama.Embedding("nomic-embed-text")

// Image models
openai.Image("gpt-image-1")
google.Image("imagen-4.0-generate-001")
```

### Auth - Auto-Resolved from Environment

Providers auto-read API keys from env vars. No explicit config needed:

| Provider  | Env Var                                                      |
| --------- | ------------------------------------------------------------ |
| OpenAI    | `OPENAI_API_KEY`                                             |
| Anthropic | `ANTHROPIC_API_KEY`                                          |
| Google    | `GOOGLE_GENERATIVE_AI_API_KEY` or `GEMINI_API_KEY`           |
| Bedrock   | `AWS_ACCESS_KEY_ID` + `AWS_SECRET_ACCESS_KEY` + `AWS_REGION` |
| Azure     | `AZURE_OPENAI_API_KEY`                                       |
| Vertex AI | Application Default Credentials (ADC)                        |
| xAI       | `XAI_API_KEY`                                                |
| Groq      | `GROQ_API_KEY`                                               |
| Cohere    | `COHERE_API_KEY`                                             |
| Mistral   | `MISTRAL_API_KEY`                                            |
| DeepSeek  | `DEEPSEEK_API_KEY`                                           |

Or set explicitly:

```go
model := openai.Chat("gpt-4o", openai.WithAPIKey("sk-..."))
```

Most providers support these options (Bedrock uses AWS credential options; Ollama requires no auth):

```go
provider.WithAPIKey(key)         // static API key
provider.WithTokenSource(ts)     // dynamic auth (OAuth, service accounts)
provider.WithBaseURL(url)        // override endpoint (Azure uses WithEndpoint)
provider.WithHeaders(h)          // custom HTTP headers
provider.WithHTTPClient(c)       // custom HTTP transport
```

---

## Patterns and Examples

### 1. Basic Text Generation

```go
result, err := goai.GenerateText(ctx, openai.Chat("gpt-4o"),
    goai.WithSystem("You are a helpful assistant."),
    goai.WithPrompt("What is Go?"),
)
if err != nil {
    return err
}
fmt.Println(result.Text)
```

### 2. Streaming

```go
stream, err := goai.StreamText(ctx, openai.Chat("gpt-4o"),
    goai.WithPrompt("Write a poem about Go."),
)
if err != nil {
    return err
}
// Option A: text-only channel
for text := range stream.TextStream() {
    fmt.Print(text)
}
// Option B: raw chunks (mutually exclusive with TextStream)
// for chunk := range stream.Stream() { ... }

// Always available after streaming completes:
result := stream.Result()
fmt.Printf("\nTokens: %d in, %d out\n", result.TotalUsage.InputTokens, result.TotalUsage.OutputTokens)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zendev-sh/goai](https://github.com/zendev-sh/goai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
