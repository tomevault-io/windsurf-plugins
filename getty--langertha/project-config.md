---
trigger: always_on
description: Langertha is a Perl LLM framework supporting 15+ engines via composable Moose roles. It provides chat, tool calling (MCP), streaming, embeddings, transcription, and an autonomous agent (Raider).
---

# Langertha — CLAUDE.md

## Overview

Langertha is a Perl LLM framework supporting 15+ engines via composable Moose roles. It provides chat, tool calling (MCP), streaming, embeddings, transcription, and an autonomous agent (Raider).

## Build System

Uses `[@Author::GETTY]` Dist::Zilla plugin bundle.

```bash
dzil test           # Build and test
prove -l t/         # Run tests directly
prove -lv t/60_tool_calling.t  # Single test, verbose
```

## Architecture

### Engine Hierarchy (lib/Langertha/Engine/)

```
Engine::Remote              url required, JSON + HTTP
  │
  ├── Engine::AnthropicBase /v1/messages format, x-api-key auth, SSE streaming
  │     │
  │     ├── Anthropic       Claude models, thinking blocks, tool_use
  │     ├── MiniMaxAnthropic MiniMax via legacy /anthropic/v1 shim endpoint
  │     └── LMStudioAnthropic LM Studio Anthropic-compatible endpoint
  │
  ├── Engine::OpenAIBase    /chat/completions format, Bearer auth, SSE streaming
  │     │
  │     │  Cloud providers (url has default, api_key from env)
  │     ├── OpenAI          gpt-4o, embeddings, whisper transcription, structured output
  │     ├── DeepSeek        deepseek-chat/reasoner, structured output
  │     ├── Groq            ultra-fast inference, whisper transcription, structured output
  │     ├── Mistral         EU-hosted, embeddings, structured output
  │     ├── MiniMax         Shanghai (default), 1M context window, M2.7
  │     ├── NousResearch    Hermes models, <tool_call> XML tool format
  │     ├── Cerebras        wafer-scale chips, fastest inference
  │     ├── OpenRouter      meta-provider, 300+ models, provider/model format
  │     ├── Replicate       thousands of open-source models, owner/model format
  │     ├── HuggingFace     Inference Providers, org/model format
  │     ├── Perplexity      search-augmented, citations — NO tool calling
  │     ├── AKIOpenAI       EU/Germany, GDPR-compliant
  │     ├── TSystems        T-Systems AIFS / LLM Hub, T-Cloud Germany + EU hyperscaler models
  │     ├── Scaleway        EU-hosted Generative APIs, drop-in OpenAI replacement
  │     │
  │     │  Self-hosted (url required, no api_key)
  │     ├── OllamaOpenAI    Ollama /v1 endpoint, embeddings
  │     ├── vLLM            high-throughput inference, single-model server
  │     ├── SGLang          SGLang OpenAI-compatible server, fast structured output
  │     ├── LlamaCpp        llama.cpp server, embeddings
  │     └── LMStudioOpenAI  LM Studio's OpenAI-compatible endpoint
  │
  ├── Engine::TranscriptionBase  Transcription-only OpenAI-shape base (no chat/tools)
  │     │
  │     └── Whisper         self-hosted faster-whisper-server etc.
  │
  │  Non-OpenAI formats (own request/response handling)
  ├── Gemini                ?key= auth, functionDeclarations, thought parts
  ├── Ollama                native /api/chat, NDJSON streaming, OpenAPI spec
  ├── AKI                   key-in-body auth, EU/Germany, /api/call/{model}
  └── LMStudio              LM Studio native API (non-OpenAI/non-Anthropic)
```

**LMStudio family** — LM Studio servers can expose three different
endpoints: `LMStudio` is the native API, `LMStudioOpenAI` is the
OpenAI-compatible endpoint, and `LMStudioAnthropic` is the
Anthropic-compatible endpoint. Pick whichever your LM Studio server is
configured to serve.

**AKI family** — `AKI` is the official AKI.IO native API (changes
often, breaks). `AKIOpenAI` is the more stable OpenAI-compatible
endpoint, but it sometimes lacks features. Both are provided so users
can pick their tradeoff; we don't endorse one over the other.

**Whisper / `->whisper` accessor** — `Whisper` no longer extends
`OpenAI` (since post-0.404 refactor). It extends the new
`TranscriptionBase` so it has only transcription functionality, no
chat / tools / embeddings / image generation. To get a transcription
handle from an existing `OpenAI` instance use the `whisper` attribute
— it returns a `TranscriptionBase` configured with the parent's
`api_key` and `url` so credentials don't have to be restated.

### Roles (lib/Langertha/Role/)

- **Capabilities** — `engine_capabilities` registry + `supports($cap)`
  helper. Composed by `Chat` (and indirectly via every other capability
  role). Mapping role→cap-flag lives in one map in `Role::Capabilities`;
  engines override via `around engine_capabilities` for wire-reality
  corrections (e.g. clearing `tool_choice_named` on string-only providers).
- **Chat** — sync/async chat (`simple_chat`, `simple_chat_f`); also
  `chat_f(messages => [...], tools => [...], tool_choice => ...,
  response_format => ...)` for single-turn structured calls.
- **Tools** — MCP tool calling loop (`chat_with_tools_f`, `mcp_servers`)
- **HermesTools** — XML-tag tool calling for models without native support
- **Streaming** — SSE / NDJSON streaming responses
- **Embedding** — Vector embeddings (`simple_embedding`)
- **Transcription** — Audio transcription
- **HTTP** — HTTP transport (sync + async via IO::Async)
- **JSON** — JSON encoding/decoding (`$self->json->encode/decode`)
- **SystemPrompt** — System prompt management

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Getty/langertha](https://github.com/Getty/langertha) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-11 -->
