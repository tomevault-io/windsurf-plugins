---
trigger: always_on
description: Manages the Graphiti client singleton:
---

# Graphiti MCP Server - Codebase Guide for AI Agents

This document explains the modular structure of the Graphiti MCP Server codebase to help AI agents navigate and understand the code organization.

## Overview

The Graphiti MCP Server exposes Graphiti (a knowledge graph memory service for AI agents) through the Model Context Protocol (MCP). The codebase has been refactored from a single monolithic file into a well-organized Python package with focused modules.

## Entry Points

There are two ways to run the server:

```bash
# As a Python package (recommended)
python -m graphiti_mcp_server [options]

# Legacy wrapper script
python run_server.py [options]
```

Both entry points call `graphiti_mcp_server.main:main()`.

**Important**: The wrapper script is named `run_server.py` (not `graphiti_mcp_server.py`) to avoid name conflicts with the package directory.

## Package Structure

```
graphiti_mcp_server/
├── __init__.py              # Package exports
├── __main__.py              # Entry point for `python -m graphiti_mcp_server`
├── main.py                  # CLI parsing, server startup, signal handling
│
├── config/                  # Configuration classes
│   ├── __init__.py          # Exports all config classes
│   ├── settings.py          # Global constants (DEFAULT_LLM_MODEL, SEMAPHORE_LIMIT, etc.)
│   ├── llm.py               # GraphitiLLMConfig - LLM client configuration
│   ├── embedder.py          # GraphitiEmbedderConfig - Embedding configuration
│   ├── neo4j.py             # Neo4jConfig - Database connection settings
│   ├── redis.py             # RedisConfig - Queue connection settings
│   └── graphiti.py          # GraphitiConfig, MCPConfig - Main configuration classes
│
├── models/                  # Data models and type definitions
│   ├── __init__.py          # Exports all models
│   ├── entities.py          # Pydantic models: Requirement, Preference, Procedure, ENTITY_TYPES
│   ├── responses.py         # TypedDict definitions for API responses
│   └── queue.py             # QueuedEpisode model for Redis queue
│
├── auth/                    # Authentication and authorization
│   ├── __init__.py          # Exports auth components
│   ├── nonce.py             # Nonce token validation (ALLOWED_NONCE_TOKENS, is_nonce_valid)
│   ├── principal.py         # Principal authentication (get_authenticated_principal)
│   └── middleware.py        # AuthenticationMiddleware - ASGI middleware for auth
│
├── group_id/                # Group ID management
│   ├── __init__.py          # Exports context functions
│   └── context.py           # Context variable functions for group_id allowlisting
│                            # (get_allowed_group_ids, get_effective_group_id, etc.)
│
├── queue/                   # Redis-based episode queue
│   ├── __init__.py          # Exports queue components
│   ├── state.py             # Global state variables (redis_client, queue_manager, etc.)
│   ├── manager.py           # RedisQueueManager class - queue operations
│   ├── worker.py            # process_episode_queue - background worker
│   └── init.py              # initialize_redis, shutdown_redis functions
│
├── client/                  # Graphiti client management
│   ├── __init__.py          # Exports client functions
│   └── graphiti.py          # Graphiti client initialization and singleton access
│                            # (get_graphiti_client, initialize_graphiti, etc.)
│
├── tools/                   # MCP Tool implementations (11 tools)
│   ├── __init__.py          # Exports all tools
│   ├── memory.py            # add_memory - Add episodes to the knowledge graph
│   ├── search.py            # search_memory_nodes, search_memory_facts
│   ├── episodes.py          # get_episodes, delete_episode
│   ├── edges.py             # get_entity_edge, delete_entity_edge
│   ├── groups.py            # list_group_ids, delete_everything_by_group_id
│   ├── queue_status.py      # get_queue_status - Monitor processing queues
│   └── admin.py             # clear_graph - Administrative operations
│
├── resources/               # MCP Resources
│   ├── __init__.py          # Exports resources
│   └── status.py            # get_status - Server and Neo4j connection status
│
└── utils/                   # Utility functions
    ├── __init__.py          # Exports utilities
    └── formatters.py        # format_fact_result - Format EntityEdge for output
```

## Key Components

### Configuration (`config/`)

All configuration is managed through Pydantic models that can be initialized from:
1. Environment variables (`.env` file)
2. CLI arguments (which override environment variables)

Key classes:
- `GraphitiConfig`: Main configuration aggregating all sub-configs
- `GraphitiLLMConfig`: OpenAI API settings (model, temperature, API key)
- `Neo4jConfig`: Database connection (uri, user, password)
- `RedisConfig`: Queue connection and key prefixes

### Authentication (`auth/`)

The server supports nonce-based authentication via query parameters:
- Set `MCP_SERVER_NONCE_TOKENS` environment variable with comma-separated tokens
- Requests must include `?nonce=<token>` for authentication
- `AuthenticationMiddleware` is a pure ASGI middleware that handles auth

### Group ID Management (`group_id/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [michabbb/graphiti-mcp-but-working](https://github.com/michabbb/graphiti-mcp-but-working) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
