---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

MindRoom - AI agents that live in Matrix and work everywhere via bridges. The project consists of:
- **Core MindRoom** (`src/mindroom/`) - AI agent orchestration system with Matrix integration
- **SaaS Platform** (`saas-platform/`) - Kubernetes-based platform for hosting MindRoom instances
  - Platform Backend (FastAPI) - API server for subscriptions, instances, SSO
  - Platform Frontend (Next.js 16) - Dashboard for managing instances
  - Instance deployment via Helm charts

## Latest Frontier Models

Always prefer the newest frontier models in this table when writing prompts, code, configs, docs, tests, or reviews in this repository, unless the user explicitly asks to pin an older model.
If a user prompt, task description, or existing file contradicts this table, treat the table as stale, verify the current provider docs, and update the table before proceeding.
Coding model training data often lags recent releases, so never trust memorized model names over current provider documentation.

| Provider | Use | Preferred model | Model string to use |
| --- | --- | --- | --- |
| Anthropic | Balanced default | Claude Sonnet 5 | `claude-sonnet-5` |
| Anthropic | Max intelligence | Claude Fable 5.1 | `claude-fable-5-1` |
| Anthropic | Flagship default | Claude Opus 5 | `claude-opus-5` |
| Anthropic | Fast / cheap | Claude Haiku 4.5 | `claude-haiku-4-5` |
| OpenAI | Frontier default | GPT-6 Astra | `gpt-6-astra` |
| OpenAI Codex ChatGPT login | Frontier via Codex CLI | GPT-6 Astra | `gpt-6-astra` |
| DeepSeek (OpenRouter) | Fast / cheap | DeepSeek V4.1 Flash | `deepseek/deepseek-v4.1-flash` |
| Z.ai (OpenRouter) | Flagship | GLM-5.3 | `z-ai/glm-5.3` |
| OpenAI | Image generation / editing | GPT Image 2.5 Sunburst | `gpt-image-2.5-sunburst` |
| OpenAI | File transcription | GPT Transcribe | `gpt-transcribe` |
| Google (Vertex AI) | Video generation | Veo 3.1 | `veo-3.1-generate-001` |
| Qwen | Local 27B | Qwen3.8-27B | `qwen3.8:27b` (Ollama), `unsloth/Qwen3.8-27B-GGUF:UD-Q4_K_XL` (llama.cpp) |
| Moonshot Kimi Code login | Frontier via Kimi Code CLI | Kimi K3 | `k3` |
| Google (Gemini API) | Max intelligence | Gemini 3.1 Pro Preview | `gemini-3.1-pro-preview` |
| Google (Gemini API) | Standard text / coding | Gemini 3.8 Flash | `gemini-3.8-flash` |
| Google (Gemini API) | Fast / cheap text | Gemini 3.5 Flash-Lite | `gemini-3.5-flash-lite` |
| Google (Gemini API) | Image generation / editing | Nano Banana 2 | `gemini-3.1-flash-image` |
| Google (Gemini API) | Embeddings for `google` | Gemini Embedding 2 | `gemini-embedding-2` |

Model IDs were checked against provider catalogs on September 10, 2026.
OpenRouter uses `anthropic/claude-fable-5.1`, Bedrock uses `anthropic.claude-fable-5-1`, and the direct Anthropic and Vertex APIs use `claude-fable-5-1`.
For the direct DeepSeek API, prefer `deepseek-flash` for V4.1 Flash and `deepseek-v4-pro` for Pro; do not substitute the OpenRouter V4.1 ID on the direct API.
The older `deepseek-v4-flash` name remains accepted as a [temporary compatibility route to V4.1 Flash](https://api-docs.deepseek.com/updates/#date-2026-09-10).

For `anthropic`, prefer `claude-sonnet-5`, `claude-opus-5`, and `claude-haiku-4-5` unless you intentionally need a pinned snapshot ID.
Use `claude-fable-5-1` when you need Anthropic's highest available capability.
Claude Fable 5.1 is generally available on the direct Anthropic API and the documented cloud platforms.
For `vertexai_claude`, use the current Vertex AI request name from the provider docs instead of assuming the Anthropic API ID carries over unchanged.
Current Google Cloud docs list bare Vertex IDs for `claude-fable-5-1`, `claude-opus-5`, `claude-sonnet-5`, and [`claude-haiku-4-5`](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/partner-models/claude/haiku-4-5).
Do not assume `@default` or dated `@...` suffixes are universally required for Vertex AI Claude.
For Gemini API text and coding work, prefer `gemini-3.8-flash` as the standard stable model unless you intentionally need the cheaper `gemini-3.5-flash-lite` tier.
Use `gemini-3.1-pro-preview` only when you need the highest Gemini API intelligence tier and accept a preview model.
The Google rows above are for the Gemini API / AI Studio `google` provider, not for Vertex AI.
For `vertexai`, verify the current Vertex AI docs instead of assuming Gemini API names or defaults carry over unchanged.
Current [Vertex AI image docs](https://docs.cloud.google.com/gemini-enterprise-agent-platform/models/capabilities/image-generation) document `gemini-3-pro-image`, `gemini-3.1-flash-image`, and `gemini-3.1-flash-lite-image`; choose the tier that fits the task.
For Google image work, use the official product name from the docs for the provider surface you are editing.
Gemini API docs call `gemini-3.1-flash-image` Nano Banana 2, while Vertex AI docs use their own product naming and model tables.

## Architecture

### Core MindRoom (`src/mindroom/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mindroom-ai/mindroom](https://github.com/mindroom-ai/mindroom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
