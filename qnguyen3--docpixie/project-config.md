---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

DocPixie is a lightweight multimodal RAG library that uses vision AI instead of embeddings/vector databases. Documents are processed as images and analyzed using vision language models for both understanding and page selection.

## Development Commands

### Environment Setup
```bash
# Set up virtual environment with uv (recommended)
uv venv
source .venv/bin/activate

uv pip install docpixie
```

Start the CLI:
```bash
docpixie
```

## Core Architecture

### Provider System
The codebase uses a clean separation between **raw API operations** and **business logic**:

- **Providers** (`docpixie/providers/`): Handle only raw API calls with generic `process_text_messages()` and `process_multimodal_messages()` methods
- **AI Operations** (`docpixie/ai/`): Contain all business logic, prompt construction, and workflow orchestration

### Key Architectural Principles

1. **Provider-Agnostic Configuration**: Uses generic `flash_model`, `pro_model`, `vision_model` fields that work across all providers
2. **Automatic Provider Defaults**: `DocPixieConfig._set_provider_defaults()` sets appropriate models based on selected provider
3. **Image-Based Processing**: All documents converted to images via PyMuPDF, preserving visual information
4. **Adaptive RAG Agent**: Single adaptive mode that dynamically plans and re-evaluates tasks based on findings (replaces Flash/Pro modes in Phase 2)

### Provider Implementation Pattern
When adding new providers:
1. Inherit from `BaseProvider`
2. Implement only `process_text_messages()` and `process_multimodal_messages()`
3. Handle provider-specific message formatting (e.g., image_path → provider format)
4. Add to `providers/factory.py` and provider defaults in `config.py`

Example: OpenRouter provider uses OpenAI client with `base_url="https://openrouter.ai/api/v1"`

### Document Processing Flow
1. **PDF → Images**: PyMuPDF converts PDF pages to optimized JPEGs
2. **Storage**: Local filesystem or in-memory storage via pluggable backends
3. **Summarization**: Vision models analyze all page images in single API call for document summary
4. **Adaptive RAG Pipeline** (Phase 2): Vision-based page selection + dynamic task planning + conversation processing

### Configuration System
- Environment-first approach: API keys loaded from `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, `OPENROUTER_API_KEY`
- Provider-agnostic model configuration
- Agent-specific settings (max iterations, pages per task, conversation context)
- Test API key support: Use `"test-key"` to bypass validation during testing

### File Structure Significance

```
docpixie/
├── core/config.py          # Central configuration with provider defaults
├── providers/              # Raw API operations only
│   ├── base.py            # Generic message processing interface
│   ├── openai.py          # OpenAI API client
│   ├── anthropic.py       # Claude API client (handles different system message format)
│   ├── openrouter.py      # OpenRouter using OpenAI client + different base_url
│   └── factory.py         # Provider creation and validation
├── ai/                     # Business logic layer
│   ├── summarizer.py      # Page/document summarization workflows
│   ├── agent.py           # Main adaptive RAG agent orchestrator
│   ├── task_planner.py    # Dynamic task planning with document selection
│   ├── page_selector.py   # Vision-based page selection
│   ├── context_processor.py # Conversation summarization
│   ├── query_reformulator.py # Reference resolution
│   ├── query_classifier.py # Document need classification
│   ├── synthesizer.py     # Response synthesis
│   └── prompts.py         # All AI prompts
├── processors/             # Document-to-image conversion
│   ├── pdf.py             # PyMuPDF implementation
│   └── factory.py         # Auto-detection of processor type
├── storage/                # Pluggable storage backends
│   ├── local.py           # Filesystem storage
│   └── memory.py          # In-memory storage (for testing)
├── models/
│   ├── document.py        # Core data models without embeddings
│   └── agent.py           # Agent task/plan data models
├── exceptions.py          # Custom exception classes
└── __init__.py            # Main API entry point
```

## Important Implementation Details

### Configuration Testing
Never use test mode flags. Instead, use test API keys (`"test-key"`) which automatically bypass validation.

### Document Summarization
The critical architectural decision: document summaries use ALL page images in a single vision API call, not individual page summaries combined. This preserves visual context and document structure.

### Provider Message Format
All providers receive messages with `image_path` type, then convert to their specific format:
- OpenAI: `image_url` with data URL
- Anthropic: `image` with base64 data
- OpenRouter: Same as OpenAI

### Adaptive RAG Agent Implementation
The agent operates in a single adaptive mode with dynamic task planning:
1. **Context Processing**: Summarizes conversation when > 8 turns
2. **Query Reformulation**: Resolves references using context (outputs JSON)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qnguyen3/docpixie](https://github.com/qnguyen3/docpixie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
