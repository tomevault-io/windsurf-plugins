---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a unified Python library for streaming structured JSON outputs from OpenAI, Anthropic (Claude), and Google Gemini models. The library leverages **native model capabilities** for structured JSON generation—avoiding tool-based approaches entirely to deliver superior performance, reliability, and efficiency compared to traditional function calling methods.

## Development Commands

### Environment Setup
```bash
# Install dependencies with uv
uv sync

# Run tests
uv run pytest

# Run specific test file
uv run pytest tests/test_providers.py
```

### Testing
The project uses pytest with pytest-asyncio for async testing. Tests are organized in:
- `tests/test_providers.py` - Base provider tests
- `tests/test_factory.py` - Factory pattern tests
- `tests/test_openai_integration.py` - OpenAI-specific tests
- `tests/test_anthropic_integration.py` - Anthropic-specific tests
- `tests/test_anthropic_provider_unit.py` - Anthropic unit tests
- `tests/test_google_integration.py` - Google-specific tests
- `tests/test_mock_provider.py` - Mock provider for testing

## Architecture

### Core Components

1. **Base Provider** ([`llm_json_streaming/base.py`](llm_json_streaming/base.py))
   - `LLMJsonProvider` abstract class defining the streaming interface
   - Provides utility methods for JSON parsing and validation
   - Key methods:
     - `_safe_parse_json()`: Attempts to parse accumulated JSON using the schema
     - `_get_best_partial_json()`: Returns both parsed object and raw JSON text

2. **Factory Pattern** ([`llm_json_streaming/factory.py`](llm_json_streaming/factory.py))
   - `create_provider()` function for instantiating providers
   - Supports "openai", "anthropic", "claude", or "google" as provider names

3. **Provider Implementations**
   - **OpenAI Provider** ([`llm_json_streaming/providers/openai/provider.py`](llm_json_streaming/providers/openai/provider.py))
     - Uses **native structured outputs** via `client.beta.chat.completions.stream`
     - **Performance**: 2-3x faster than function calling, guaranteed schema compliance
     - **Reliability**: No tool call failures or parsing overhead
     - Default model: `gpt-4o-2024-08-06`

   - **Anthropic Provider** ([`llm_json_streaming/providers/anthropic/provider.py`](llm_json_streaming/providers/anthropic/provider.py))
     - Configurable strategy selection with three modes:
       - `"auto"`: Auto-detect based on model capabilities (default)
       - `"structured"`: Force native structured outputs mode
       - `"prefill"`: Force schema-aware prefill mode
     - **Native Advantage**: No function calling or tool overhead
     - **Performance**: Direct JSON generation eliminates tool call latency
     - Priority: constructor mode > method parameter > auto-detection
     - Uses specialized streaming classes:
       - `StructuredOutputStreamer` ([`llm_json_streaming/providers/anthropic/structured.py`](llm_json_streaming/providers/anthropic/structured.py)) - **Native structured outputs**
       - `PrefillJSONStreamer` ([`llm_json_streaming/providers/anthropic/prefill.py`](llm_json_streaming/providers/anthropic/prefill.py)) - **Schema-aware prefill, no tools**

   - **Google Provider** ([`llm_json_streaming/providers/google/provider.py`](llm_json_streaming/providers/google/provider.py))
     - Uses **native structured outputs** via Google GenAI SDK with `response_mime_type="application/json"`
     - **Performance**: Direct JSON streaming without function call delays
     - **Reliability**: Eliminates tool-based failure modes and inconsistencies
     - Default model: `gemini-2.5-flash`
     - Includes JSON repair functionality for enhanced partial object support
     - Requires `GEMINI_API_KEY` environment variable

### Streaming Interface

All providers implement the `stream_json()` method that yields dictionaries with:
- `partial_object`: Current best parsed object with progressive enhancement:
  - Available from the beginning of streaming in all modes
  - Early stage: Partial dictionaries for incomplete JSON
  - Later stage: Validated Pydantic model instances for complete/repairable JSON
- `delta`: Real-time text updates during streaming
- `final_object`: Complete, validated Pydantic model when streaming finishes
- `partial_json`: Current accumulated JSON text string
- `final_json`: Complete JSON text string when streaming finishes

**Best Practice**: Use `partial_object` for real-time UI updates as it provides the most reliable partial parsing. Handle both dictionary and Pydantic types gracefully for consistent user experience across all providers and models.

### Configuration

Set API keys in environment variables:
- `OPENAI_API_KEY` and `OPENAI_BASE_URL`
- `ANTHROPIC_API_KEY` and `ANTHROPIC_BASE_URL`
- `GEMINI_API_KEY` and `GOOGLE_BASE_URL` (optional)

## Key Design Patterns

1. **Strategy Pattern**: Anthropic provider dynamically chooses between native structured outputs and prefill strategies based on model capabilities
2. **Factory Pattern**: Centralized provider instantiation with consistent interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniel-style/llm-json-streaming](https://github.com/daniel-style/llm-json-streaming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
