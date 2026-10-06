---
trigger: always_on
description: > **Audience**: AI Models (Claude, GPT, Gemini, Cursor, Copilot, Codex) and human developers working on the `llms.py` (`ServiceStack/llms` / `llmspy` / `AI.Chat`) repository.
---

# AGENTS.md — Codebase Blueprint & Architectural Guide

> **Audience**: AI Models (Claude, GPT, Gemini, Cursor, Copilot, Codex) and human developers working on the `llms.py` (`ServiceStack/llms` / `llmspy` / `AI.Chat`) repository.
> **Purpose**: Provides an authoritative blueprint of codebase architecture, design patterns, core invariants, current capabilities, and development conventions.

---

## 1. Project Overview & Philosophy

`llms.py` is a lightweight, privacy-centric CLI, OpenAI-compatible server, and ChatGPT/Open WebUI alternative for interacting with Large Language Models.

### Core Principles
1. **Zero Runtime Bloat**: Uses only Python standard library and `aiohttp`. No heavy AI frameworks (no LangChain, LlamaIndex, PyTorch, Transformers). Keep the runtime fast, portable, and minimal.
2. **Pure ESM Frontend**: The UI is written in Vue 3 using native browser ES Modules (ESM) and Tailwind CSS. There is **no compilation or bundler build step** (no Vite/Webpack required to run or extend).
3. **Local Privacy & Offline First**: SQLite database persistence per-user (`~/.llms/user/{username}/`), SHA-256 local file caching (`~/.llms/cache`), and first-class offline model support (Ollama, LMStudio).
4. **Modularity & Extensibility**: Features are packaged as ComfyUI-style extensions with server lifecycle hooks and frontend component slots.
5. **Durable Agent Architecture**: Background agent runs execute in bounded stages with database leasing, interrupt recovery, and non-destructive context compaction.

---

## 2. Codebase Directory Blueprint

```
ServiceStack/llms/
├── llms/                         # Main Python package
│   ├── main.py                   # Single-file functional core (~6k lines)
│   ├── db.py                     # Legacy / core database access helpers
│   ├── execution_context.py      # workspace_scope ContextVar: a durable run's project directories
│   ├── providers.json            # 530+ model definitions merged from models.dev
│   ├── providers-extra.json      # Provider overrides and custom provider definitions
│   ├── llms.json                 # User & default provider/model configuration
│   ├── index.html                # Single-page application entry point
│   ├── extensions/               # Pluggable modular feature extensions
│   │   ├── app/                  # Durable AgentScheduler, canonical chat_message schema, thread store,
│   │   │                         # project sidebar APIs, title generation (titles.py), DURABLE_AGENTS.md
│   │   ├── agents/               # Agent profiles (SYSTEM.md, templates, dynamic memory, footer actions)
│   │   ├── gemini/               # Gemini File Search Store RAG, bidirectional sync, assistants API
│   │   ├── core_tools/           # Sandboxed code execution (Python, JS, TS, C#), calc, grep, fetch_url
│   │   ├── computer/             # Anthropic computer-use tools (bash, edit, filesystem, screen)
│   │   ├── skills/               # Agent Skills standard (SKILL.md progressive disclosure)
│   │   ├── tools/                # Tool discovery, registry API, and function execution endpoint
│   │   ├── voice/                # Audio transcription (Whisper, Voxtral) and TTS support
│   │   ├── pdf/                  # PDF Studio: live Typst (.typ) template editing and PDF compilation
│   │   ├── gallery/              # Media gallery for generated images and audio
│   │   ├── projects/             # Project identity (stable ids), workspaces, sidebar visibility, publish paths
│   │   ├── publish/              # Public sharing of threads and media to ai.llmspy.org
│   │   ├── credentials/          # API key & secrets management
│   │   ├── github_auth/          # GitHub OAuth authentication & multi-user isolation
│   │   ├── katex/                # KaTeX LaTeX math rendering extension
│   │   └── analytics/            # Telemetry and query logging
│   └── ui/                       # Core frontend SPA (Vue 3 ESM)
│       ├── App.mjs               # Root layout, sidebar navigation, top bar, dynamic slot rendering
│       ├── ctx.mjs               # AppContext and ExtensionScope reactive state management
│       ├── ai.mjs                # API client (chat completions, SSE streaming, file uploads)
│       ├── modules/
│       │   ├── chat/             # Chat interface (ChatBody.mjs, ChatPrompt composer, draftStore.mjs,
│       │   │                     # ComposerContextBar.mjs project picker, SettingsDialog)
│       │   ├── model-selector.mjs# Composer model chip + searchable, filterable picker for 530+ models
│       │   └── layout.mjs        # Responsive layout and panel states
│       ├── components/           # Shared components usable by extensions (e.g. CheckBox.mjs)
│       └── lib/                  # Vendored frontend libraries (Vue 3, marked, highlight.js, chart.js, idb)
├── tests/                        # Comprehensive test suite (asyncio, scheduler, streaming, tools)
├── docs/                         # Technical documentation and specs
│   ├── AGENTS.md                 # Agent Profile directory and configuration specification

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ServiceStack/llms](https://github.com/ServiceStack/llms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
