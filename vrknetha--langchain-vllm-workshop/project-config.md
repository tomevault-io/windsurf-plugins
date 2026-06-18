---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a LangChain v1 + vLLM + LangGraph workshop demonstrating a **Career Transition Advisor** agent. The project showcases:

- **LangGraph v1** state machine framework with simplified patterns
- **vLLM** for GPU-accelerated inference (deployed on RunPod)
- **Firecrawl MCP** tools for real-time web research
- **PostgreSQL persistence** with AsyncPostgresSaver for production-ready conversation storage
- **Thread-based memory** that persists across restarts
- **Dual UI options**: LangGraph Studio (built-in) + Next.js chat interface

Key demonstration: Multi-turn conversation agent that maintains context across turns, uses web research tools, and provides personalized career guidance.

## Common Commands

### Prerequisites

**Start PostgreSQL Database** (required):
```bash
# Start PostgreSQL in Docker
docker-compose up -d

# Verify PostgreSQL is running
docker-compose ps

# View logs if needed
docker-compose logs -f postgres
```

### Installation
```bash
# Recommended (uses uv for faster installs)
uv pip install -r requirements.txt

# Or standard pip
pip install -r requirements.txt
```

### Running the Agent

**Development Mode** (hot reload, in-memory storage):
```bash
langgraph dev --port 2024
# API: http://localhost:2024
# Studio UI: http://localhost:2024 (built-in chat interface)
# Note: Uses in-memory storage, conversation history lost on restart
```

**Production Mode** (PostgreSQL persistence):
```bash
langgraph up --port 2024
# API: http://localhost:2024
# Studio UI: http://localhost:2024 (built-in chat interface)
# Note: PostgreSQL must be running (docker-compose up -d)
# Note: Reads POSTGRES_URI from .env automatically
# Conversation history persists across restarts
```

**Alternative (explicit postgres-uri flag):**
```bash
langgraph up --port 2024 --postgres-uri "postgresql://langchain:langchain_dev_password@localhost:5432/langchain_db"
```

**Standalone Mode** (educational/testing):
```bash
python -m src.career_advisor.agent
# Initializes agent, creates tables, and prints info
# Note: This mode is for testing only
```

### Database Management

**Stop PostgreSQL**:
```bash
docker-compose down
# Data persists in postgres_data volume
```

**Reset Database** (clear all conversation history):
```bash
docker-compose down -v  # Removes volumes
docker-compose up -d    # Fresh database
```

**Access PostgreSQL CLI**:
```bash
docker exec -it langchain-postgres psql -U langchain -d langchain_db
# Then: \dt to list tables
```

### Frontend Chat UI
```bash
cd agent-chat-ui
pnpm install
pnpm dev
# Runs at http://localhost:3000
```

### vLLM Deployment (on RunPod pod)
```bash
./start_vllm.sh
# Starts OpenAI-compatible API on port 8000
# Model loads in 2-3 minutes
```

## Architecture: How Components Work Together

### High-Level Flow
```
User (Studio UI or Next.js)
  → LangGraph API Server (port 2024)
    → StateGraph with MessagesState
      → call_model() async node
        → ChatOpenAI client
          → RunPod vLLM endpoint
            → Hermes-2-Pro-Mistral-7B model
      → ToolNode (when LLM requests tools)
        → Firecrawl MCP Server (via npx)
          → firecrawl_search, firecrawl_scrape, etc.
```

### LangGraph v1 Simplifications

This codebase demonstrates **modern LangGraph v1 patterns** that replace older boilerplate:

1. **`MessagesState`** - No custom state class needed
   - Built-in message list management
   - Automatic thread-based conversation history
   - Replaces custom `class AgentState(TypedDict)` definitions

2. **`tools_condition`** - Built-in routing function
   - Automatically routes to tool node when LLM calls tools
   - Routes to END when LLM produces final response
   - Replaces ~18 lines of custom `should_continue()` logic

3. **`async call_model()`** - Async node for better I/O performance
   - Uses `ainvoke()` for non-blocking execution
   - Critical for web research tools that have network latency
   - System prompt injection handled within node

4. **PostgreSQL persistence** - Production-ready conversation storage
   - `AsyncPostgresSaver` configured in code with automatic table creation
   - Works with both `langgraph dev` and `langgraph up`
   - Requires PostgreSQL running (via Docker Compose)
   - Thread-based conversation history persists across restarts

### MCP (Model Context Protocol) Integration

The agent connects to **Firecrawl MCP server** via stdio transport:

```python
mcp_client = MultiServerMCPClient({
    "firecrawl": {
        "transport": "stdio",
        "command": "npx",
        "args": ["-y", "firecrawl-mcp"]
    }
})
```

**Available Tools:**
- `firecrawl_search` - Web search (query + limit)
- `firecrawl_scrape` - Extract content from URLs
- `firecrawl_map` - Crawl website structure
- `firecrawl_crawl` - Batch page crawling
- `firecrawl_check_crawl_status` - Monitor crawl jobs
- `firecrawl_extract` - Structured data extraction

**CRITICAL: Tool Usage Rules**
- **NEVER** use the `sources` parameter in `firecrawl_search` - it causes validation errors
- **ONLY** use `query` (required) and `limit` (optional) parameters

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vrknetha/langchain-vllm-workshop](https://github.com/vrknetha/langchain-vllm-workshop) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-18 -->
