---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Lava Stew is a geospatial analyst agent demonstrating building an agent with the Claude Agent SDK.

**References to magma_soup**: If you see references to Magma Soup: Magma Soup was the application built to demonstrate MCP servers. If you need to reference it, magma soup is in a sibling directory to this project: cd ../magma_soup

## Important information about SDK documentation

The most recent documentation for the SDK is under the context/agent_sdk_documentation directory. Consult it as needed.

## Why This Architecture?

### The Agent SDK is Fundamentally Stateful

Unlike simple API calls with message history, the [Claude Agent SDK](https://docs.claude.com/en/docs/agent-sdk/overview) maintains stateful sessions:

- **Session IDs** preserve full conversation context across interactions
- **Automatic context compaction** prevents context window exhaustion
- **File operations and sandboxes** tied to session state
- **Resume/fork capabilities** for conversation branching

The SDK documentation explicitly recommends [containerized deployment patterns](https://docs.claude.com/en/docs/agent-sdk/hosting):

1. **Ephemeral Sessions** - One-off tasks (destroy container after completion)
2. **Long-Running Sessions** - Persistent containers for chat bots requiring rapid response
3. **Hybrid Sessions** - Ephemeral containers hydrated with saved state
4. **Single Containers** - Multiple SDK processes for agent collaboration

**This project implements Pattern 2 (Long-Running Sessions)** - maintaining stateful agent worker processes that hold SDK session state in memory.

### Why Not Just Send Message Arrays?

You _could_ build a stateless system that sends full conversation history with each request, but you'd lose:

- SDK's automatic context management and compaction
- Session-based file operations and execution sandboxes
- Ability to resume/fork conversations efficiently
- The performance benefits of SDK's internal state caching

The complexity in this architecture isn't over-engineering - it's the minimum viable implementation of the [Agent SDK's intended deployment pattern](https://docs.claude.com/en/docs/agent-sdk/sessions).

## Architecture

### Design Principles

1. **Stateful agent workers** - Long-running containerized processes maintain Agent SDK session state (following SDK's recommended Pattern 2)
2. **Python tools via TypeScript agent worker** - Demonstrate TypeScript agent worker invoking Python geospatial scripts (GIS professionals often prefer Python)
3. **SSE streaming** - Server-Sent Events for real-time response streaming
4. **TypeScript** - Type safety for API server and worker coordination

### System Components

- **API Server** (Express/TypeScript) - HTTP endpoint, SSE streaming, RabbitMQ RPC client
- **Agent Worker** (TypeScript) - Agent SDK integration, session management, tool execution
- **RabbitMQ** - Message broker with RPC pattern using exclusive reply queues
- **Python Tools** - Geocoding and distance calculation scripts invoked via uv

### Communication Flow

1. Client sends HTTP POST to API server with conversationId + message
2. API server creates exclusive reply queue, publishes to `chat.requests`
3. Agent worker consumes from `chat.requests`, processes with Agent SDK
4. Agent worker publishes streaming events to reply queue
5. API server consumes from reply queue, streams to client via SSE
6. Reply queue auto-deletes when API server disconnects

## Development Commands

### Prerequisites

- Node.js 20+
- Docker and Docker Compose
- Python 3.11+ with uv (`curl -LsSf https://astral.sh/uv/install.sh | sh`)
- Anthropic API key (from [Anthropic Console](https://console.anthropic.com/))
- Google Maps API key (from [Google Cloud Console](https://console.cloud.google.com/))
- Flutter

### Local Development

```bash
# Start services (API server, Agent Worker)
docker compose up -d

# View logs
docker compose logs -f

# Test the agent with curl
curl -X POST http://localhost:3001/chat \
  -H "Content-Type: application/json" \
  -d '{"conversationId": "test-123", "message": "What is the distance between Seattle and Portland?"}'
```

# Flutter client

```bash
cd flutter_client
flutter pub get
flutter run
```

## Tech Stack

- **Server**: Node.js, Express, TypeScript
- **Agent Worker**: [Anthropic Agent SDK (TypeScript)](https://docs.claude.com/en/docs/agent-sdk/overview)
- **Client**: Flutter, Flutter Map, BLoC state management
- **APIs**: Google Maps API (geocoding)
- **Python tooling**: uv for dependency management
- **Geospatial**: Turf.js for TypeScript, Google Maps API for Python

## Custom Tools

Tools are defined in the Anthropic SDK format. All tools are invoked by the TypeScript agent worker via child_process, demonstrating how TypeScript infrastructure can leverage multiple language's ecosystems.

**Current Tools:**

1. **geocode** - Convert city name to coordinates via Python + Google Maps API
2. **calculate_distance** - Calculate geodesic distance between two points via Python + geopy

**An Example Tool Implementation Pattern:**

Agent worker defines tool schema and invokes Python via uv:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ecopony/lava_stew](https://github.com/ecopony/lava_stew) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-26 -->
