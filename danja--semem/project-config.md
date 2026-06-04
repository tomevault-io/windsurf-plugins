---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

There should be no inline fallbacks as this leads to indeterminate code. If the value is not successfully retrieved from config then that is an error that needs fixing.

## Project Overview

Semem (Semantic Memory) is a Node.js library for intelligent agent memory management that integrates large language models (LLMs) with semantic web technologies (RDF/SPARQL). It provides a memory system for AI applications with multiple storage backends and LLM provider integrations.

It uses ES modules and Vitest for tests.

Only look at docs when requested. Always ignore all files under docs/ignore

## Architectural Notes

- The memory and json storage backends are being phased out, sparql storage should be used throughout
- The MCP (Model Context Protocol) system has been restructured around 13 core verbs instead of the previous complex tool hierarchy

## Development Guidelines

When creating new code follow the patterns described in `docs/manual/infrastructure.md`

- scripts should be run from the server root
- Mocking is only allowed for trivial arithmetic-style unit checks (e.g. “does a + b equal c?”); every other test must assume the live services defined in `config/config.json` are reachable and interact with them directly.

## Configuration Management

Configuration constants are centralized in `config/preferences.js` to avoid hardcoded values throughout the codebase. This file contains:

- **SEARCH_CONFIG**: Quality thresholds, scoring weights, and boost factors for AdaptiveSearchEngine
- **SPARQL_CONFIG**: Similarity search defaults and data health assessment thresholds  
- **MEMORY_CONFIG**: Decay factors and persistence settings

The configuration uses detailed comments explaining each constant's purpose. When adding new configurable values, add them to the appropriate section in `preferences.js` rather than hardcoding in source files.

## Remote Server Deployment

The production instance of Semem runs on Docker with the following endpoints:

- **Workbench UI**: https://semem.tensegrity.it/
- **Fuseki SPARQL**: https://semem-fuseki.tensegrity.it/
- **MCP Server**: https://mcp.tensegrity.it/
- **API Server**: https://api.tensegrity.it/

This deployment provides a fully functional remote instance for testing and demonstration purposes.

## Blog Guidelines

- Progress reports and plans should be saved as md files under docs/postcraft/content/raw/entries/ 
- Naming scheme follows format: `YYYY-MM-DD_claude_title.md`
- Main title should start with "# Claude :"
- Document should be rendered in the style of a development worklog
- If the document exceeds a page or two, or if the primary topic of activity changes, create a new document


## Architecture

Semem has a layered architecture with the following key components:

1. **Memory Management Layer**
   - `MemoryManager`: Core class that coordinates memory operations
   - `ContextManager`: Manages context retrieval and window sizing
   - `ContextWindowManager`: Handles text chunking and window management

2. **Storage Layer**
   - `BaseStore`: Abstract base class for all storage backends
   - `MemoryStore`: In-process memory management
   - `InMemoryStore`: Transient in-memory storage
   - `JSONStore`: Persistent storage using JSON files
   - `SPARQLStore`: RDF-based storage with SPARQL endpoints
   - `CachedSPARQLStore`: Optimized version with caching

3. **Handlers Layer**
   - `EmbeddingHandler`: Manages vector embeddings generation and processing
   - `LLMHandler`: Orchestrates language model interactions
   - `CacheManager`: Provides caching for improved performance

4. **API Layer**
   - Multiple interface types (HTTP, CLI, REPL)
   - `APIRegistry`: Central service registry
   - Request handling via active/passive handlers

5. **Connector Layer**
   - `ClientConnector`: Base connector class
   - Provider-specific connectors (Ollama, Claude, etc.)

6. **Ragno Layer (Knowledge Graph Integration)**
   - `Entity`: RDF-based entities extracted from text
   - `Unit`: Independent semantic units from corpus decomposition
   - `Relationship`: First-class relationship nodes between entities
   - `RDFGraphManager`: Manages RDF graph operations
   - `decomposeCorpus`: Main function for text-to-RDF decomposition
   - Uses Ragno vocabulary (http://purl.org/stuff/ragno/) for RDF modeling

7. **MCP Layer (Model Context Protocol)**
   - 13 core verbs: `tell`, `ask`, `augment`, `zoom`, `pan`, `tilt`, `inspect`, `remember`, `forget`, `recall`, `project_context`, `fade_memory`, `train-vsom`
   - HTTP server: `src/mcp/http-server.js` - provides REST API endpoints
   - STDIO server: `src/mcp/index.js` - provides MCP protocol communication
   - Unified validation using Zod schemas for all verb parameters
   - Direct HTTP endpoints: `/tell`, `/ask`, `/augment`, `/zoom`, `/pan`, `/tilt`, `/inspect`, `/remember`, `/forget`, `/recall`

8. **VSOM Layer (Visual Self-Organizing Map)**
   - Standalone server: `src/frontend/vsom-standalone/server.js` - serves VSOM UI and proxies to MCP
   - API proxy configuration routes requests to appropriate backend services
   - ZPT (Zoom, Pan, Tilt) 3-dimensional navigation system

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [danja/semem](https://github.com/danja/semem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
