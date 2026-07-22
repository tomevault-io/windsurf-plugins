---
trigger: always_on
description: This file provides guidance to AI agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI agents when working with code in this repository.

## What This Repo Is

Moss is a real-time semantic search runtime for AI agents targeting sub-10ms query latency. The runtime runs on-device using bundled embedding models (`moss-minilm`) no external embedding API calls are needed. The cloud layer handles project management and index distribution; the local layer handles querying.

This repository contains the **multi-language SDKs**, **framework integrations** (cookbooks), and **application examples**.

## Repository Layout

```
sdks/
  python/sdk/      — Python SDK (PyPI: moss), Python 3.10+
  javascript/sdk/  — JS/TS SDK (npm: @moss-dev/moss), ESM-only
  elixir/sdk/      — Elixir SDK (Hex: moss)
examples/
  python/          — Standalone Python usage examples
  javascript/      — Standalone TS usage examples
  javascript-web/  — Browser/Vite examples (no Node runtime)
  c/               — C binding examples
  go/              — Standalone Go SDK usage examples
  bun/             — Bun runtime example
  python-classification/ — Text classification with Moss
  voice-agents/    - End-to-end voice agents (LiveKit-based)
    airline-pnr/         - Ambient retrieval; per-caller Moss index, swap mid-call
    mortgage-lending/    - Multi-agent flow with shared session state
  cookbook/        — Framework integrations (one subdirectory per framework)
    autogen/       — AutoGen multi-agent e-commerce support
    crewai/        — CrewAI retrieval tool
    daytona/       — Log Ingestion Q&A Agent on Daytona sandboxes
    dspy/          — DSPy notebook
    haystack/      — Haystack RAG pipeline integration
    langchain/     — LangChain retriever + tool integration
    langflow/      — Langflow drag-and-drop retriever + search components
    langgraph/     - LangGraph stateful retrieval node
    mastra/        — Mastra agent createTool() integration
    pydantic-ai/   - Pydantic AI integration
    moss-cognee-daytona/ — Claude Code + Cognee + Moss on Daytona (shared memory)
apps/
  agora-moss/      — Agora Conversational AI voice agent (MCP server demo)
  docker/          — Dockerized Python + JS SDK examples (ECS/K8s pattern)
  elevenlabs-moss/ — ElevenLabs voice agent with Moss knowledge base
  livekit-moss-vercel/ — LiveKit voice agent + React frontend on Vercel
  moss-bun/        — Production Bun semantic search application
  moss-llamaindex/ — LlamaIndex + Liteparse full-stack PDF search demo
  next-js/         — Next.js 16 browser-based semantic search UI (@moss-dev/moss-web)
  pipecat-moss/    — Pipecat voice agent (three variants below)
    pipecat-quickstart/  — Cloud-deployable quickstart bot
    ollama-local/        — Local LLM + Moss + Pipecat via docker compose
    hume-ollama-local/   — Local LLM + Hume AI TTS + Moss + Pipecat
  ten-moss/        — TEN Framework voice agent with Moss session-scoped grounding
  vapi-moss/       — VAPI Custom Tool webhook server
packages/
  agora-moss/            — Agora Conversational AI MCP server package
  elevenlabs-moss/       — ElevenLabs integration package
  moss-cli/              — CLI for index/document management (no-code workflows)
  moss-data-connector/   — Database source connectors
    moss-connector-mongodb/  — MongoDB connector
    moss-connector-mysql/    — MySQL / MariaDB connector
    moss-connector-sqlite/   — SQLite connector
    moss-connector-supabase/ — Supabase (PostgREST) connector
  moss-md-indexer/       — Markdown docs → Moss index builder
  pipecat-moss/          — Pipecat Python integration package
  strands-agents-moss/   — AWS Strands Agents integration package
  ten-moss/              — TEN Framework Moss session manager (MossSessionManager)
  vapi-moss/             — VAPI Custom Knowledge Base webhook adapter
  n8n-nodes-moss/        — n8n community node for Moss index + query
  vercel-sdk/            — Vercel AI SDK tool wrappers (@moss-tools/vercel-sdk)
  vitepress-plugin-moss/ — VitePress search plugin (on-device fallback after cloud)
  zo-computer/           — Zo computer skill for Moss search
moss-live-labs/          - Experimental zone: prototypes and community demos (APIs can change)
  python/                - Minimal Python quickstart + advanced query example
  typescript/            - Minimal TypeScript quickstart + advanced query example
  examples/
    voice-agent/         - LiveKit + Moss voice assistant
    advanced-voice-agent/ - Persona impersonator built on a PDF knowledge base
    image-search/        - FastAPI + React image search over COCO data
  community-demos/
    voice-agents/
      bharat-benefits/      - Voice RAG over Indian public-benefit schemes (Sarvam STT/TTS)
      shoplabs-voice-agent/ - Pipecat WebRTC ecommerce support agent
```

**`moss-live-labs/` policy:** experimental code. Treat it as staging area for
ideas that may graduate into `sdks/`, `apps/`, `examples/`, or `packages/`, or
that may be deleted. Do not depend on it from stable code paths. When adding
new top-level features, prefer the main directories unless the user explicitly
asks for an experimental landing spot.

## Integrations & Cookbooks

### Framework Cookbooks (`examples/cookbook/`)

| Directory | Framework | What it demonstrates |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [usemoss/moss](https://github.com/usemoss/moss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
