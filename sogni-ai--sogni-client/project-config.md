---
trigger: always_on
description: This file provides guidance to AI coding assistants (Claude Code, Codex, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding assistants (Claude Code, Codex, etc.) when working with code in this repository.

## LLM Documentation Resources

For AI coding assistants working with this SDK, the following resources are available:

- **`llms.txt`** - Indexed quick reference with code examples
- **`llms-full.txt`** - Comprehensive documentation (~25KB) with complete API reference
- **`dist/index.d.ts`** - TypeScript type definitions (after build)
- **API Docs**: https://sdk-docs.sogni.ai

When helping users with Sogni SDK tasks, consult `llms-full.txt` for complete parameter references, especially for video generation where WAN 2.2 and LTX-2.3 models have different behaviors.

These are public agent-facing docs shipped with the npm package. They are kept aligned with:

- `package.json` for package version, runtime engines, npm scripts, exports, and published files.
- `src/index.ts` for public SDK namespaces and root exports.
- `src/Projects/types/index.ts`, `src/Projects/utils/index.ts`, and `src/Projects/createJobRequestMessage.ts` for media-generation parameters and video frame behavior.
- `src/Chat/types.ts`, `src/Chat/index.ts`, `src/Chat/tools.ts`, and `src/Chat/hostedToolValidation.generated.ts` for chat, hosted tools, structured outputs, and durable runs.
- `src/CreativeWorkflows/` and `src/Replay/` for durable workflow and RunRecord APIs.

Current public API anchors:

- The SDK runtime requirement is **Node.js >=22** (`package.json#engines`).
- Socket-native LLM chat uses `sogni.chat.completions.create()`.
- Durable creative workflows use `sogni.workflows`.
- Project cost helpers are `sogni.projects.estimateCost()`, `estimateVideoCost()`, and `estimateAudioCost()`.
- `checkAuth()` is only for cookie-auth browser flows. API-key auth auto-authenticates during `createInstance()`, and token auth uses `login()` or `setTokens()`.
- `ChatCompletionResult` is SDK-shaped (`content`, `role`, `finishReason`, `tool_calls`, `usage`, `cost`). Streaming chunks expose `chunk.content` and optional `chunk.tool_calls`.

## Sogni Intelligence APIs

The SDK wraps the public Sogni Intelligence endpoints used for text chat, hosted creative tools, durable chat turns, and deterministic multi-step workflows.

- `sogni.chat.completions.create()` maps to socket-native chat completions and supports text, streaming, vision input, custom function tools, Sogni tool injection, structured outputs, and `think` / `taskProfile` controls.
- `sogni.chat.hosted.create()` maps to `POST /v1/chat/completions`, the OpenAI-compatible REST chat endpoint. It can execute Sogni media-generation and composition tools server-side.
- `sogni.chat.runs` maps to `/v1/chat/runs`, a durable hosted-chat turn with persisted state, event replay, cancellation, and recovery across client disconnects.
- `sogni.workflows` maps to `/v1/creative-agent/workflows`, where callers submit exact multi-step creative plans and observe durable execution through snapshots, event logs, or SSE.
- `sogni.workflows.templates` maps to `/v1/creative-agent/workflows/templates`, the CRUD and fork API for saved, parameterized workflow recipes.
- `sogni.replay` maps to `/v1/replay/records`, the RunRecord write/list/get surface for replay viewers and audit tooling.

Public chat and workflow media rules:

- Vision chat accepts inline PNG or JPEG `data:` URIs through OpenAI-style `image_url` content parts.
- Durable chat runs and creative workflows use retrievable HTTP(S) media references, often produced by Sogni upload/download URL helpers.
- Request media references are addressable by media index in hosted tool and workflow calls, so later steps can reuse uploaded or generated images, videos, and audio without copying URLs into prompts.

## Overview

This is the **Sogni SDK for JavaScript/Node.js** - a TypeScript client library for the Sogni Supernet, a DePIN protocol for creative AI inference. The SDK supports image generation (Stable Diffusion, Flux, Z-Image, Qwen image-edit models, GPT Image 2), video generation (WAN 2.2, LTX-2.3, Seedance 2.0), audio generation (ACE-Step 1.5), LLM chat with tool calling, hosted creative tools, durable creative workflows, replay records, and multimodal vision chat (Qwen3.6 35B VLM, default `qwen3.6-35b-a3b-gguf-iq4xs`).

Runtime and packaging:

- Node.js `>=22` is required by `package.json`.
- CommonJS build: `dist/index.js`; ESM build: `dist-esm/index.js`; type declarations: `dist/index.d.ts`.
- Published package files include `README.md`, `AGENTS.md`, `llms.txt`, `llms-full.txt`, `dist/`, `dist-esm/`, and `src/`.

## Build & Development Commands

```bash
# Build the project (compiles TypeScript to dist/)
npm run build

# Watch mode for development
npm run watch

# Format code with Prettier
npm run prettier:fix

# Check formatting
npm run prettier

# Validate generated hosted-tool validation file is in sync
npm run check:hosted-tool-validation

# Validate generated hosted-tool manifest is in sync
npm run check:hosted-tools-manifest

# Build and run chat model-routing checks
npm run test:chat-routing

# Generate API documentation
npm run docs
```

Generated artifacts:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Sogni-AI/sogni-client](https://github.com/Sogni-AI/sogni-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
