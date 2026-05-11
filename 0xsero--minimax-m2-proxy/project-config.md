---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

MiniMax-M2 Proxy is a translation proxy that converts MiniMax-M2's custom XML-based tool calling format to standard OpenAI and Anthropic API formats. The proxy enables MiniMax-M2 (456B MoE model) to work seamlessly with any OpenAI/Anthropic-compatible tool or framework without modifications.

## Common Commands

### Development

```bash
# Setup environment
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt

# Run proxy (development mode with auto-reload)
uvicorn proxy.main:app --reload --port 8001

# Run proxy (simple script)
./start_proxy.sh

# Run proxy in background with logging
nohup uvicorn proxy.main:app --host 0.0.0.0 --port 8001 > proxy.log 2>&1 &
```

### Testing

```bash
# Run all unit tests
pytest tests/test_parsers.py -v

# Run specific test
pytest tests/test_parsers.py::TestToolCallParser::test_parse_single_tool_call -v

# Run with coverage
pytest tests/ --cov=. --cov-report=html

# End-to-end tests (requires TabbyAPI running on port 8000)
python tests/test_tool_calling.py
python tests/e2e_test.py
```

### Code Quality

```bash
# Format code
black . --line-length 100

# Lint code
ruff check .

# Auto-fix linting issues
ruff check . --fix
```

### Production Deployment

```bash
# Install systemd service
sudo cp minimax-m2-proxy.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl start minimax-m2-proxy
sudo systemctl enable minimax-m2-proxy

# Check service status
sudo systemctl status minimax-m2-proxy
sudo journalctl -u minimax-m2-proxy -f
```

## Architecture

### Request Flow

```
Client (OpenAI/Anthropic SDK)
    ↓
FastAPI endpoints (/v1/chat/completions or /v1/messages)
    ↓
Format conversion (Anthropic → OpenAI if needed)
    ↓
TabbyAPI/vLLM client (HTTP request to backend)
    ↓
MiniMax-M2 model (generates XML tool calls)
    ↓
ToolCallParser (XML → JSON conversion)
    ↓
Formatter (OpenAI or Anthropic response format)
    ↓
Client receives standard API response
```

### Core Components

**proxy/main.py** - FastAPI application with dual API endpoints
- `/v1/chat/completions` - OpenAI-compatible endpoint (streaming & non-streaming)
- `/v1/messages` - Anthropic-compatible endpoint (streaming & non-streaming)
- `/health` - Health check for proxy and backend
- `/v1/models`, `/v1/model` - Pass-through endpoints to backend
- Global instances: `tabby_client`, `tool_parser`, `openai_formatter`, `anthropic_formatter`
- Lifespan manager handles client initialization and cleanup

**proxy/client.py** - TabbyClient HTTP client wrapper
- Manages async HTTP connections to TabbyAPI/vLLM backend
- Provides `chat_completion()` for non-streaming requests
- Provides `extract_streaming_content()` for SSE streaming
- Handles health checks and connection management

**proxy/models.py** - Pydantic request/response models
- `OpenAIChatRequest` - OpenAI chat completion request schema
- `AnthropicChatRequest` - Anthropic messages request schema
- Conversion utilities: `anthropic_tools_to_openai()`, `anthropic_messages_to_openai()`
- Type validation for all API payloads

**proxy/config.py** - Environment configuration using pydantic-settings
- Loads settings from `.env` file
- Key settings: `TABBY_URL`, `TABBY_TIMEOUT`, `HOST`, `PORT`, `LOG_LEVEL`, `LOG_RAW_RESPONSES`

**parsers/tools.py** - XML tool call parser (adapted from vLLM)
- `ToolCallParser.parse_tool_calls()` - Extract tool calls from complete text
- Parses `<minimax:tool_call>` with `<invoke>` and `<parameter>` tags
- Type conversion based on tool schema (int, float, bool, JSON objects/arrays)
- Generates unique tool call IDs (`call_<uuid>`)
- Returns dict with `tools_called`, `tool_calls`, and `content` (without XML blocks)

**parsers/streaming.py** - Streaming state machine
- `SimpleStreamingParser.process_chunk()` - Process incremental text chunks
- Buffers text until complete tool calls detected
- Sends content deltas in real-time, tool calls when complete
- Separates content from `<minimax:tool_call>` XML blocks

**formatters/openai.py** - OpenAI format generator
- `format_complete_response()` - Non-streaming Chat Completion response
- `format_streaming_chunk()` - SSE chunks with proper delta format
- `format_tool_call_stream()` - Multi-chunk tool call streaming (id → name → arguments)
- Generates response IDs, timestamps, and usage statistics

**formatters/anthropic.py** - Anthropic format generator
- `format_complete_response()` - Messages API response with content blocks
- Event-based streaming: `message_start`, `content_block_start`, `content_block_delta`, `content_block_stop`, `message_delta`, `message_stop`
- Converts tool calls to `tool_use` blocks with proper structure
- Maps finish reasons: `stop` → `end_turn`, `tool_calls` → `tool_use`

### Key Design Decisions

**Stateless Proxy**: Each request is independent with no conversation history management. The proxy only translates formats in real-time.

**Think Block Preservation**: MiniMax-M2's `<think>` blocks are kept verbatim in responses to maintain transparency into the model's reasoning process.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xSero/minimax-m2-proxy](https://github.com/0xSero/minimax-m2-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
