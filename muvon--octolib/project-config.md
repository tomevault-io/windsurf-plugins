---
trigger: always_on
description: Rust library providing a unified interface to multiple AI providers (OpenAI, Anthropic, OpenRouter, NVIDIA, Ollama, Google Vertex, Google Studio, Amazon, Cloudflare, DeepSeek, Moonshot, Z.ai, BytePlus, Alibaba, Groq, Cerebras, Together, Featherless, Fireworks, Hetzner, Inception, OpenCode Zen/Go, OctoHub, CLI proxies). Handles chat completions, embeddings, reranking, cost tracking, caching, structured output, vision, and tool calls. No panics, no `println!`, always `Result`. Copyright year is **
---

#! Octolib — AI Provider Library Guide

Rust library providing a unified interface to multiple AI providers (OpenAI, Anthropic, OpenRouter, NVIDIA, Ollama, Google Vertex, Google Studio, Amazon, Cloudflare, DeepSeek, Moonshot, Z.ai, BytePlus, Alibaba, Groq, Cerebras, Together, Featherless, Fireworks, Hetzner, Inception, OpenCode Zen/Go, OctoHub, CLI proxies). Handles chat completions, embeddings, reranking, cost tracking, caching, structured output, vision, and tool calls. No panics, no `println!`, always `Result`. Copyright year is **2026**.

## Project Structure

```
src/
├── lib.rs                          → Public re-exports (all public types live here)
├── errors.rs                       → ProviderError, ConfigError, MessageError, ToolCallError + ErrorContext trait
├── storage.rs                      → Cache dirs for FastEmbed/HuggingFace models
├── llm/
│   ├── traits.rs                   → AiProvider trait — THE interface every provider implements
│   ├── types.rs                    → Message, ChatCompletionParams, ProviderResponse, TokenUsage, ModelPricing, ReasoningEffort, etc.
│   ├── factory.rs                  → ProviderFactory::get_provider_for_model("provider:model")
│   ├── config.rs                   → CacheConfig, CacheTTL, CacheType
│   ├── strategies.rs               → ProviderStrategy trait + AnthropicStrategy/OpenAIStrategy (tool call formatting)
│   ├── tool_calls.rs               → GenericToolCall unified format
│   ├── retry.rs                    → Exponential backoff logic
│   ├── utils.rs                    → normalize_model_name, sanitize_model_name, is_model_in_pricing_table, calculate_cost_from_pricing_table
│   ├── reference_pricing.rs        → Baseline pricing for open/proxy models (substring-matched, specific-first)
│   ├── reference_capabilities.rs   → Baseline capabilities (vision, video, structured_output, context window)
│   └── providers/
│       ├── openai_compat.rs        → Shared OpenAI-compatible request/response layer; auto-passes reasoning_effort; also OpenAiCompatConfig, get_optional_api_key, get_api_url (internal, not pub)
│       ├── shared.rs               → HTTP client (arc-swap pool), cache control helpers, tool call parsers
│       ├── openai.rs               → Native — PRICING table, Responses API, OAuth support
│       ├── anthropic.rs            → Native — PRICING table, caching, thinking blocks, OAuth support
│       ├── google_vertex.rs        → Vertex AI — service-account auth, delegates to openai_compat; shared Gemini PRICING table + model-cache helpers
│       ├── google_studio.rs        → AI Studio (Gemini API) — API-key auth, delegates to openai_compat; reuses google_vertex PRICING
│       ├── amazon.rs               → Native — Bedrock
│       ├── deepseek.rs             → Native — PRICING table
│       ├── moonshot.rs             → Native — PRICING table
│       ├── minimax.rs              → Native — PRICING table
│       ├── zai.rs                  → Native — PRICING table
│       ├── byteplus.rs             → Native — PRICING table (ByteDance Seed models)
│       ├── alibaba.rs              → Native — PRICING table (Qwen + resold DeepSeek/GLM)
│       ├── groq.rs                 → Native — PRICING table
│       ├── nvidia.rs               → Proxy — delegates to openai_compat, reference pricing
│       ├── cerebras.rs             → Proxy — delegates to openai_compat, reference pricing
│       ├── together.rs             → Proxy — delegates to openai_compat, reference pricing
│       ├── ollama.rs               → Proxy — delegates to openai_compat, reference pricing
│       ├── local.rs                → Proxy — delegates to openai_compat, reference pricing
│       ├── cloudflare.rs           → Proxy — delegates to openai_compat
│       ├── openrouter.rs           → Proxy — delegates to openai_compat
│       ├── octohub.rs              → Proxy — delegates to openai_compat
│       ├── featherless.rs          → Proxy — delegates to openai_compat
│       ├── hetzner.rs              → Proxy — delegates to openai_compat (free, experimental, fixed MODELS table)
│       ├── inception.rs           → Native — PRICING table (Mercury diffusion LLMs, strict catalogue)
│       ├── tinker.rs               → Proxy — delegates to openai_compat (Inkling family + sampler checkpoints)
│       ├── opencode.rs             → Proxy — Zen + Go providers, delegates to openai_compat (shared OPENCODE_API_KEY)
│       ├── fireworks.rs            → Proxy — delegates to openai_compat (auto prefix-cache)
│       └── cli/                    → CLI proxy: claude, codex, cursor, gemini, generic backends
├── embedding/
│   ├── mod.rs                      → generate_embeddings(), generate_embeddings_batch(), count_tokens(), truncate_output()

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Muvon/octolib](https://github.com/Muvon/octolib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
