---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GraphZep is a TypeScript implementation of the Zep temporal knowledge graph memory system for AI agents. Based on the Graphzep framework and enhanced with Zep memory capabilities, it provides episodic, semantic, and procedural memory types with advanced search and retrieval functionality.


Key features:

- **Zep Memory System**: Episodic, semantic, and procedural memory types with session management
- **Advanced Search**: Semantic, keyword, hybrid search with MMR reranking and fact extraction
- **Bi-temporal Data Model**: Explicit tracking of event occurrence times and memory validity
- **Hybrid Retrieval**: Semantic embeddings, keyword search (BM25), and graph traversal
- **RDF Support**: Complete RDF triple store with SPARQL 1.1 queries and ontology management
- **Type Safety**: Full TypeScript implementation with Zod schema validation
- **Multiple Backends**: Neo4j, FalkorDB, and optimized RDF in-memory store with external endpoint support
- **Production Ready**: HTTP server, MCP integration, and Docker deployment

## Development Commands

### Main Development Commands (run from project root)

```bash
# Install dependencies
npm install

# Build the core library
npm run build

# Format code (Prettier formatting)
npm run format

# Lint code (ESLint + TypeScript checking)
npm run lint

# Run tests (comprehensive tests including RDF functionality using Node.js built-in test runner)  
npm test

# Run all checks (format, lint, typecheck, test)
npm run check

# Development mode with watch
npm run dev

# Type checking only
npm run typecheck

# Clean build artifacts
npm run clean
```

### HTTP Server Development (run from server/ directory)

```bash
cd server/
# Install server dependencies
npm install

# Run server in development mode (Hono with hot reload)
npm run dev

# Build production server
npm run build

# Start production server
npm start

# Format, lint, test server code
npm run format
npm run lint
npm test
```

### MCP Server Development (run from mcp_server/ directory)

```bash
cd mcp_server/
# Install MCP server dependencies
npm install

# Run in development mode
npm run dev

# Build MCP server
npm run build

# Start production MCP server
npm start

# Run with Docker Compose (TypeScript version)
docker-compose -f docker-compose.yml up --build
```

### Examples and Tutorials (run from examples/ directory)

```bash
cd examples/
# Install example dependencies
npm install

# Run quickstart examples
npm run quickstart:neo4j      # Basic Neo4j operations
npm run quickstart:falkordb   # FalkorDB backend
npm run quickstart:rdf        # RDF triple store with SPARQL
npm run quickstart:neptune    # Amazon Neptune support

# Run advanced examples
npm run ecommerce            # Product catalog search demo
npm run podcast              # Conversation analysis
npm run langgraph-agent      # AI sales agent with memory
```

## Code Architecture

### Core Library (`src/`)

- **Main Entry Point**: `src/index.ts` - Main exports for the GraphZep library
- **Zep Memory System**: `src/zep/` - Complete Zep memory implementation
  - `memory.ts` - Memory management with fact extraction
  - `session.ts` - Session management and isolation
  - `retrieval.ts` - Advanced search and retrieval system
  - `types.ts` - Zep-specific type definitions
- **Orchestration**: `src/graphzep.ts` - Main `Graphzep` class for underlying graph operations
- **Graph Storage**: `src/drivers/` - Database drivers for Neo4j, FalkorDB, optimized RDF driver, and future Neptune support
- **LLM Integration**: `src/llm/` - Clients for OpenAI, Anthropic with TypeScript interfaces
- **Embeddings**: `src/embedders/` - Embedding clients with type-safe configurations
- **Graph Elements**: `src/core/nodes.ts`, `src/core/edges.ts` - Core graph data structures with Zod validation
- **Type Definitions**: `src/types/` - Comprehensive TypeScript type definitions
- **RDF System**: `src/rdf/` - Complete RDF support with SPARQL queries and ontology management
  - `namespaces.ts` - RDF namespace management with Zep ontology
  - `memory-mapper.ts` - Convert Zep memories to/from RDF triples
  - `sparql-interface.ts` - SPARQL 1.1 query interface with Zep extensions
  - `ontology-manager.ts` - Load and validate custom domain ontologies
  - `ontologies/` - Default Zep ontology and custom ontology support
- **Utilities**: `src/utils/` - Date/time handling, validation utilities

### HTTP Server (`server/`)

- **Hono Service**: `server/src/standalone-main.ts` - High-performance HTTP server using Hono framework
- **Configuration**: `server/src/config/` - Environment-based configuration with Zod validation
- **DTOs**: `server/src/dto/` - Data transfer objects with Zod schemas for API contracts
- **API Endpoints**: All FastAPI endpoints preserved with identical functionality

### MCP Server (`mcp_server/`)

- **MCP Implementation**: `mcp_server/src/graphzep-mcp-server.ts` - TypeScript Model Context Protocol server
- **Tool Handlers**: Complete MCP tool implementation for AI assistants
- **Docker Support**: TypeScript-based containerized deployment with health checks

### Examples (`examples/`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aexy-io/graphzep](https://github.com/aexy-io/graphzep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
