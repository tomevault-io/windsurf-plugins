---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Apple Dev MCP (Model Context Protocol) server that provides complete Apple development guidance, combining Human Interface Guidelines (design principles) with Technical Documentation (API reference) for all Apple platforms. It serves comprehensive content through both traditional MCP integration and modern Desktop Extensions (.dxt) for AI assistants like Claude.

## Development Commands

### Build and Test
- `npm run build` - Compile TypeScript to JavaScript in `dist/`
- `npm run clean:build` - Clean and rebuild the project
- `npm test` - Run Jest test suite
- `npm run test:watch` - Run tests in watch mode
- `npm run lint` - Run ESLint on TypeScript files
- `npm run lint:fix` - Fix linting issues automatically

### Development
- `npm run dev` - Start development server using tsx
- `npm start` - Run compiled server from `dist/`
- `npm run test:automation` - Automated MCP server testing

### Testing with MCP Inspector
```bash
npx @modelcontextprotocol/inspector dist/server.js
```

## Architecture Overview

The project uses a static content architecture with pre-built markdown files and optimized search indices. All content is pre-processed and cached for instant access without external dependencies:

### Core Components

1. **AppleHIGMCPServer** (`src/server.ts`) - Main MCP server entry point
   - Coordinates all components and handles MCP protocol communication
   - Sets up request handlers for resources and tools
   - Manages graceful startup/shutdown
   - Initializes static content services

2. **StaticContentSearchService** (`src/services/static-content-search.service.ts`) - Primary content search engine
   - Fast search across 113+ pre-processed Apple HIG sections
   - Smart keyword matching with synonym expansion
   - Platform-specific filtering and relevance scoring
   - No external dependencies required

3. **HIGCache** (`src/cache.ts`) - Smart caching layer
   - TTL-based caching with graceful degradation
   - Backup cache entries for offline resilience 
   - Two-tier caching: fresh data + stale fallback data
   - Methods: `getWithGracefulFallback()`, `setWithGracefulDegradation()`

4. **HIGResourceProvider** (`src/resources.ts`) - MCP Resources implementation
   - Serves structured content via URIs like `hig://ios`, `hig://ios/buttons`
   - Platform-specific and category-specific resource organization
   - Uses static content from pre-built markdown files
   - Generates comprehensive content with proper Apple attribution

5. **HIGToolsService** (`src/services/tools.service.ts`) - MCP Tools implementation
   - Interactive search with advanced keyword matching and intent recognition
   - 4 focused tools for design guidelines and technical documentation
   - Multi-factor relevance scoring (keyword + structure + context + synonym expansion)
   - Enhanced keyword search with synonym expansion and intelligent matching
   - Optimized for fast response times without external model dependencies

6. **ContentProcessor** (`src/services/content-processor.service.ts`) - Content processing pipeline
   - Markdown content structuring and validation
   - Quality assurance for pre-processed content
   - Apple-specific content pattern recognition and enhancement
   - Structured content organization (overview, guidelines, examples, specifications)

7. **AppleDevAPIClient** (`src/services/apple-dev-api-client.service.ts`) - Technical documentation integration
   - Provides access to Apple's API documentation and technical references
   - Integrates with design guidelines for comprehensive development guidance
   - Caches technical content for performance optimization

8. **Desktop Extension Support** (`scripts/build-extension.js`, `manifest.json`) - Modern distribution
   - Builds DXT-compliant Desktop Extensions for one-click installation
   - Packages server and dependencies in portable .dxt format
   - Includes proper manifest, icon, and validation for Claude Desktop integration

### Data Flow

```
MCP Client → AppleHIGMCPServer → HIGResourceProvider/HIGToolsService
                                            ↓
                                   StaticContentSearchService → HIGCache → Pre-built Content

Search Flow:
Query → HIGToolsService → Advanced Keyword Matching + Synonym Expansion
                            ↓
                    Multi-factor Scoring (keyword + synonym + structure + context)
                            ↓
                    Ranked Results (with intent recognition and boost factors)
```

### Content Processing and Delivery

```
User Request → StaticContentSearchService → Search Index Lookup
                        ↓
                Pre-built Markdown Files (113+ sections) → Content Filtering
                        ↓
                Relevance Scoring + Context Matching
                        ↓
                HIGCache (with graceful degradation)
                        ↓
                Structured Content Response
```

### Key Patterns


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tmaasen/apple-dev-mcp](https://github.com/tmaasen/apple-dev-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
