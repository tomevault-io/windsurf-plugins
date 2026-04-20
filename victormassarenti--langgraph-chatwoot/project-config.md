---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a minimalist replica of a financial agent focused on message exchange with Chatwoot integration. It uses LangGraph for agent orchestration, Redis for message buffering with debouncing, and PostgreSQL for conversation persistence.

## Development Commands

### Running the Application

**Docker (Recommended):**
```bash
docker-compose up -d              # Start all services
docker-compose logs -f app        # View application logs
docker-compose logs app | grep ERROR  # View only errors
docker-compose restart postgres redis  # Restart dependencies
docker-compose down               # Stop all services
```

**Local Development:**
```bash
# Ensure PostgreSQL and Redis are running locally
source venv/bin/activate          # Activate virtual environment
python -m app.main                # Run the application
```

### Testing

**Manual Webhook Testing:**
```bash
curl -X POST http://localhost:8000/api/v1/chatwoot/webhook \
  -H "Content-Type: application/json" \
  -d '{
    "event": "message_created",
    "conversation": {"id": 1, "status": "pending"},
    "sender": {"id": 1, "type": "contact"},
    "content": "test message",
    "message_type": "incoming"
  }'
```

**Buffer Management:**
```bash
# Get buffer statistics for a conversation
curl http://localhost:8000/api/v1/chatwoot/buffer/stats/123

# Clear buffer for a conversation
curl -X DELETE http://localhost:8000/api/v1/chatwoot/buffer/clear/123
```

## Architecture

### Core Components

**LangGraph StateGraph** (`app/agent/graph.py`):
- Manages agent orchestration using state machines
- Three nodes: `agent` (LLM invocation), `tools` (function calling), `split_message` (message splitting)
- Routing logic: agent → tools (if tool calls) → agent → split (if long message) → END
- Uses `AsyncPostgresSaver` for conversation persistence with graceful fallback if unavailable

**Message Buffer** (`app/services/message_buffer.py`):
- Redis-based debouncing with 20-second sliding window (configurable)
- Aggregates rapid consecutive messages before processing
- Graceful fallback to immediate processing if Redis is unavailable
- Lock mechanism prevents duplicate processing
- Stores message metadata (content + timestamp) for debugging

**Webhook Handler** (`app/api/chatwoot.py`):
- Validates incoming webhooks (only `message_created` events, `incoming` messages, `pending` status)
- Returns 200 OK immediately to prevent Chatwoot timeout
- Processes messages asynchronously in background tasks
- Critical function: `_extract_assistant_messages_since_last_user()` extracts only NEW assistant messages (prevents duplication with persistence)

### Message Flow

1. **Incoming Message**: Chatwoot webhook → FastAPI endpoint
2. **Validation**: Check event type, message type, conversation status, contact info
3. **Buffering**: Add to Redis buffer, start/reset debounce timer (20s sliding window)
4. **Aggregation**: After debounce period, concatenate all buffered messages with `\n\n`
5. **Agent Processing**:
   - Convert to LangChain messages
   - Invoke StateGraph with conversation persistence (thread_id = `chatwoot_conv_{conversation_id}`)
   - Agent decides: call tools, split message, or end
6. **Message Splitting**: If response > 200 chars, use LLM with structured output to split intelligently
7. **Response Extraction**: Extract ONLY assistant messages since last user message (critical for avoiding duplicates)
8. **Sending**: Send messages to Chatwoot with 1-second delay between parts

### Critical Fixes Applied

This codebase has undergone 2 complete code reviews. See `CRITICAL_FIXES_FINAL.md` for details:

1. **Graph Compilation**: Checkpointer initialized async BEFORE graph compilation with graceful fallback
2. **Message Buffer Fallback**: Redis unavailability doesn't block processing - falls back to immediate processing
3. **Message Extraction**: `_extract_assistant_messages_since_last_user()` prevents massive message duplication by extracting only messages AFTER last user message (not all historical messages)
4. **Attachment Handling**: Sends user-friendly message when attachments received, silently ignores if text also present

## Configuration

All settings in `app/config.py` are controlled via environment variables:

**Required:**
- `OPENAI_API_KEY`: OpenAI API key
- `CHATWOOT_API_TOKEN`: Chatwoot bot token
- `CHATWOOT_ACCOUNT_ID`: Chatwoot account ID

**Common Modifications:**
- `MESSAGE_BUFFER_DEBOUNCE_SECONDS=20`: Debounce window duration
- `MESSAGE_SPLIT_MAX_LENGTH=200`: Max characters before splitting
- `LLM_MODEL=gpt-4o-mini`: OpenAI model (must support tool calling)
- `MESSAGE_BUFFER_ENABLED=true`: Enable/disable buffering
- `MESSAGE_SPLIT_ENABLED=true`: Enable/disable message splitting

## Adding New Tools

Tools are defined in `app/agent/tools.py`. Use the `@tool` decorator from LangChain:

```python
from langchain_core.tools import tool

@tool
def my_new_tool(param: str) -> str:
    """
    Brief description of what the tool does.

    Args:
        param: Description of parameter

    Returns:
        Description of return value
    """
    # Implementation here

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/VictorMassarenti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
