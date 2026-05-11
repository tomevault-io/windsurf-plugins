---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**PromptObjects** is a Ruby framework where markdown files with LLM-backed behavior act as first-class autonomous entities. The core insight: **everything is a capability**—primitives (Ruby code) and Prompt-Objects (markdown files) share the same interface, differing only in interpretation complexity.

**Current Status**: v0.5.0 — The core framework is fully implemented and functional. The original 6-phase implementation plan is complete. Active development is focused on visualization, developer experience, and exploring new primitives. See `CHANGELOG.md` for release history and `docs/archive/` for original design context.

## Architecture

```
RUNTIME (Environment)
├── CAPABILITY REGISTRY
│   ├── PRIMITIVES (Ruby) - deterministic interpretation
│   ├── PROMPT-OBJECTS (Markdown) - semantic interpretation via LLM
│   └── UNIVERSAL CAPABILITIES - available to all POs automatically
├── MESSAGE BUS - routes messages, logs to SQLite for replay
├── SESSION STORE (SQLite) - persistent conversation threads, delegation tracking
├── HUMAN QUEUE - non-blocking ask_human requests
├── WEB SERVER (Falcon + async-websocket) - serves React frontend
└── MCP SERVER - exposes POs as tools via Model Context Protocol
```

### Unified Capability Interface
Both primitives and Prompt-Objects implement:
```ruby
class Capability
  def name          # string identifier
  def description   # what this capability does
  def receive(message, context:)  # handle message, return response
end
```

### Prompt-Object Structure
Markdown files with two parts:
- **Frontmatter (YAML)**: Configuration (name, description, capabilities)
- **Body (Markdown)**: Identity and behavior (becomes LLM system prompt)

Example:
```markdown
---
name: reader
description: Helps people understand files
capabilities:
  - read_file
  - list_files
---

# Reader
## Identity
You are a careful, thoughtful file reader...
```

### Universal Capabilities
Available to all Prompt-Objects automatically (no frontmatter declaration needed):
- `ask_human` - pause for human input/confirmation
- `think` - internal reasoning (not shown to human)
- `create_capability` / `add_capability` / `remove_capability` - self-modification
- `list_capabilities` / `list_primitives` - introspection
- `create_primitive` / `add_primitive` / `delete_primitive` / `verify_primitive` / `modify_primitive` / `request_primitive` - primitive management
- `modify_prompt` - rewrite own system prompt at runtime
- `store_env_data` / `get_env_data` / `list_env_data` / `update_env_data` / `delete_env_data` - thread-scoped shared key-value store for delegation chains

### PO-to-PO Delegation
When a PO calls another PO, the system creates an isolated delegation thread in the target PO. The caller's context is tracked so messages show correct provenance. Delegation start/complete events are broadcast via WebSocket for real-time UI updates.

## Technology Stack

- **Ruby** (>= 3.2, tested through Ruby 4) - core implementation
- **LLM APIs** - OpenAI, Anthropic, Gemini, Ollama, OpenRouter (adapter pattern via `LLM::Factory`)
- **Falcon** - async HTTP server for REST API and static file serving
- **async-websocket** - real-time WebSocket communication
- **React + TypeScript** - web frontend (Smalltalk System Browser-inspired multi-pane UI)
- **Three.js** - spatial canvas visualization (force-directed PO graph)
- **SQLite** - session persistence and event log storage
- **MCP** - Model Context Protocol server mode

## File Structure

```
prompt_objects/
├── exe/prompt_objects              # CLI entrypoint
├── lib/
│   ├── prompt_objects.rb           # Main entry, requires all modules
│   └── prompt_objects/
│       ├── environment.rb          # Runtime container (registry, bus, LLM, sessions)
│       ├── capability.rb           # Base capability interface
│       ├── prompt_object.rb        # PO implementation (LLM conversation loop)
│       ├── primitive.rb            # Primitive tool wrapper
│       ├── loader.rb               # Parses frontmatter + body from .md files
│       ├── registry.rb             # Capability registration and lookup
│       ├── message_bus.rb          # Message routing, logging, SQLite persistence
│       ├── human_queue.rb          # Non-blocking human interaction queue
│       ├── cli.rb                  # CLI command definitions
│       ├── server.rb               # Web server setup
│       ├── server/
│       │   ├── app.rb              # Sinatra application
│       │   ├── api/routes.rb       # REST API endpoints
│       │   ├── websocket_handler.rb # WebSocket event handling
│       │   └── file_watcher.rb     # Live .md file change detection
│       ├── llm/
│       │   ├── factory.rb          # Provider/model selection
│       │   ├── response.rb         # Unified response object
│       │   ├── pricing.rb          # Token cost calculation
│       │   ├── openai_adapter.rb   # OpenAI + Ollama + OpenRouter
│       │   ├── anthropic_adapter.rb
│       │   └── gemini_adapter.rb
│       ├── primitives/             # Built-in: read_file, list_files, write_file, http_get

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [works-on-your-machine/prompt_objects](https://github.com/works-on-your-machine/prompt_objects) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
