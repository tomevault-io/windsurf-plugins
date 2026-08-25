---
trigger: always_on
description: <!-- doc-status: current | updated: 2026-07-05 -->
---

# CLAUDE.md

<!-- doc-status: current | updated: 2026-07-05 -->

This file provides context to Claude Code when working with this repository.

## Project Overview

This is a **multi-provider MCP server** bridging Claude Code with Google Gemini AI and 400+ models via OpenRouter. Claude can access Gemini's unique capabilities (1M context, video, TTS, Deep Research, RAG) plus any model available on OpenRouter (GPT-4o, Llama, Mistral, Claude, etc.) through a single unified interface.

**Version:** 4.5.0
**SDK:** google-genai >= 2.0.0 (Interactions API, 'steps' schema) + FastMCP + filelock
**Architecture:** Modular package structure with SQLite persistence, dynamic model registry, and multi-provider routing

See also: [CHANGELOG.md](CHANGELOG.md) for release notes, and `DEVELOPMENT_ROADMAP.md` for future plans (internal file, git-ignored — exists only in local checkouts, so no markdown link: it would 404 on GitHub).

## Architecture (v4.5.0)

**Production-grade MCP server** with FastMCP SDK:

```
omni-ai-mcp/
├── run.py                    # Entry point wrapper
├── pyproject.toml            # Package configuration
├── app/
│   ├── __init__.py          # Package init, exports main(), __version__
│   ├── server.py            # FastMCP server (20 tools with @mcp.tool())
│   ├── cli.py               # Setup wizard CLI (omni-ai-mcp-setup)
│   │
│   ├── core/                # Infrastructure & cross-cutting concerns
│   │   ├── __init__.py      # Core exports
│   │   ├── config.py        # Configuration (env vars, defaults, version, model IDs)
│   │   ├── logging.py       # StructuredLogger, activity logging, JSON format
│   │   └── security.py      # PathValidator, SecretsSanitizer, SafeFileWriter, cross-platform file locking
│   │
│   ├── services/            # External service integrations
│   │   ├── __init__.py      # Service exports
│   │   ├── gemini.py        # Gemini client wrapper, generate_with_fallback()
│   │   ├── model_registry.py # Dynamic model discovery, cache, fallback (NEW v4.0)
│   │   ├── openrouter.py    # OpenRouter client, 400+ models (NEW v4.0)
│   │   └── persistence.py   # SQLite conversation storage + conversation index
│   │
│   ├── tools/               # MCP tool implementations (by domain)
│   │   ├── __init__.py      # Tool registration, get_tools_list()
│   │   ├── registry.py      # ToolRegistry with @tool decorator
│   │   ├── text/            # Text/reasoning tools
│   │   │   ├── ask_gemini.py    # Text generation with thinking + dual mode
│   │   │   ├── ask_model.py     # Multi-provider routing: Gemini + OpenRouter (NEW v4.0)
│   │   │   ├── models.py        # Live model catalog with deprecation warnings (NEW v4.0)
│   │   │   ├── brainstorm.py    # Creative ideation (6 methodologies)
│   │   │   ├── challenge.py     # Critical thinking / Devil's Advocate
│   │   │   ├── code_review.py   # Code analysis
│   │   │   └── conversations.py # Conversation management (list, delete)
│   │   ├── code/            # Code tools
│   │   │   ├── analyze_codebase.py # Large-scale analysis (1M context, 5MB limit)
│   │   │   └── generate_code.py    # Structured generation with dry-run & XML sanitization
│   │   ├── media/           # Media tools
│   │   │   ├── analyze_image.py  # Vision analysis
│   │   │   ├── generate_image.py # Imagen image generation
│   │   │   ├── generate_video.py # Veo video generation
│   │   │   └── text_to_speech.py # TTS with 30 voices
│   │   ├── web/             # Web tools
│   │   │   ├── web_search.py     # Google-grounded search
│   │   │   └── deep_research.py  # Deep Research Agent (Interactions API)
│   │   └── rag/             # RAG tools
│   │       ├── file_store.py    # Create/list file stores
│   │       ├── file_search.py   # Query documents
│   │       └── upload.py        # Upload to stores
│   │
│   ├── utils/               # Helper functions
│   │   ├── __init__.py
│   │   ├── file_refs.py     # @file expansion, expand_file_references()
│   │   └── tokens.py        # Token estimation, size limits
│   │
│   ├── schemas/             # Pydantic v2 validation
│   │   ├── __init__.py
│   │   └── inputs.py        # Tool input schemas (7 validated tools)
│   │
│   └── middleware/          # Request processing
│       └── __init__.py
│
└── tests/                   # Test suite (file counts: see «Test Structure» below)
    ├── conftest.py          # Pytest fixtures
    ├── unit/                # Unit tests
    └── integration/         # Integration tests
```

### Key Components

| Component | Location | Purpose |
|-----------|----------|---------|
| Entry Point | `run.py` | Wrapper that imports and runs `app.main()` |
| FastMCP Server | `app/server.py` | FastMCP server with 20 `@mcp.tool()` registrations |
| Config | `app/core/config.py` | Environment variables, defaults, version, model IDs |
| Logging | `app/core/logging.py` | StructuredLogger with JSON support |
| Security | `app/core/security.py` | Sandboxing, sanitization, safe writes, cross-platform file locking |
| Tool Registry | `app/tools/registry.py` | @tool decorator, tool discovery |
| Gemini Client | `app/services/gemini.py` | API wrapper with generate_with_fallback() |
| Model Registry | `app/services/model_registry.py` | Dynamic discovery of 44+ models, cache TTL 1h, auto-fallback |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmyx77/omni-ai-mcp](https://github.com/marmyx77/omni-ai-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
