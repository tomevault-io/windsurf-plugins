---
trigger: always_on
description: **atomic-monorepo** is the development repository for **Atomic Agents**, a lightweight and modular Python framework for building Agentic AI applications. The framework is built around the principle of **atomicity** - creating single-purpose, reusable, and composable components for AI pipelines.
---

# Atomic Agents Monorepo

## What is this Repository?

**atomic-monorepo** is the development repository for **Atomic Agents**, a lightweight and modular Python framework for building Agentic AI applications. The framework is built around the principle of **atomicity** - creating single-purpose, reusable, and composable components for AI pipelines.

### Core Philosophy

Atomic Agents bridges the gap between flexibility and reliability in production AI applications by providing:

- **Predictable AI Behavior**: Controlled, schema-driven agent construction vs. autonomous but unpredictable multi-agent systems
- **Modular Development**: Build AI applications using familiar software engineering principles (LEGO-like composability)
- **Type Safety**: Consistent input/output contracts through Pydantic schemas
- **Developer Control**: Full visibility and control over AI behavior with no hidden abstractions

Built on top of [Instructor](https://github.com/jxnl/instructor) (for structured LLM outputs) and [Pydantic](https://docs.pydantic.dev/) (for data validation).

---

## Monorepo Structure

This repository contains four main packages/projects:

```
atomic-monorepo/
├── atomic-agents/          # Core framework library (main package)
├── atomic-assembler/       # CLI tool for managing components
├── atomic-examples/        # Example projects and use cases
├── atomic-forge/           # Collection of downloadable tools
├── docs/                   # Sphinx documentation
├── guides/                 # Development guides
├── .github/workflows/      # CI/CD pipelines
├── pyproject.toml          # Project configuration
└── README.md              # Main documentation
```

---

## Package Details

### 1. atomic-agents/ - Core Framework

**Published as:** `atomic-agents` on PyPI (version lives in the root `pyproject.toml`)
**Purpose:** Main Python package containing all core framework components

**Key Components:**
- `agents/` - AtomicAgent class and agent configuration
- `base/` - Base abstractions (BaseIOSchema, BaseTool, BaseToolConfig)
- `context/` - ChatHistory, SystemPromptGenerator, dynamic context providers
- `connectors/` - External integrations (MCP support)
- `utils/` - Shared utilities (token counting, formatting)

**Installation:**
```bash
pip install atomic-agents
```

### 2. atomic-assembler/ - CLI Tool

**Command:** `atomic`
**Purpose:** Terminal UI application for browsing, downloading, and managing tools from atomic-forge
**Built With:** [Textual](https://textual.textualize.io/) (TUI framework)

**Features:**
- Interactive tool exploration
- Download tools directly into your project
- Manage tool dependencies

**Usage:**
```bash
atomic  # Launch interactive TUI
```

### 3. atomic-examples/ - Example Projects

**Purpose:** Complete, runnable example implementations demonstrating framework capabilities

**Available Examples:**
- `quickstart/` - Basic getting started examples
- `deep-research/` - Research agent implementations
- `web-search-agent/` - Web search integration
- `rag-chatbot/` - RAG (Retrieval Augmented Generation)
- `youtube-summarizer/` - Video transcript summarization
- `orchestration-agent/` - Multi-agent orchestration patterns
- `mcp-agent/` - Model Context Protocol integration
- `fastapi-memory/` - API with memory/state management
- And 5+ more examples...

Each example is self-contained and demonstrates specific patterns and capabilities.

### 4. atomic-forge/ - Tool Repository

**Not a Python package** - This is a downloadable tool collection that users integrate into their projects.

**Philosophy:** Tools are NOT bundled with the framework. Instead, users download individual tools for full control and customization.

**Available Tools:**
- `arxiv_search/` - Academic paper search via the public arXiv API
- `bocha_search/` - BoCha web search
- `calculator/` - Mathematical computation tool
- `datetime_tool/` - Timezone-aware now / parse / convert / shift / diff
- `fia_signals/` - Crypto market intelligence (regime, signals, yields, gas, trending, wallet risk)
- `hackernews_search/` - Hacker News search via the free Algolia API
- `pdf_reader/` - PDF text + metadata extraction (local file or URL, page-range support)
- `searxng_search/` - Privacy-focused search integration
- `tavily_search/` - Tavily API search tool
- `weather/` - Current conditions and daily/hourly forecast via Open-Meteo (no key)
- `webpage_scraper/` - Web scraping capabilities
- `wikipedia_search/` - Wikipedia search in any language edition (no key)
- `youtube_transcript_scraper/` - YouTube transcript extraction

**Benefits:**
- Only install what you need (reduces dependency bloat)
- Full control to modify tools for your specific use case
- Tools become part of your codebase (no black boxes)

---

## Core Architecture

### Agent Anatomy

Every Atomic Agent consists of:

1. **System Prompt** - Defines agent behavior and purpose
2. **Input Schema** (Pydantic model) - Validates and structures input
3. **Output Schema** (Pydantic model) - Ensures consistent output format

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eigenwise/atomic-agents](https://github.com/Eigenwise/atomic-agents) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
