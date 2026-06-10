---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Node.js REST API that serves as a metadata provider for Audiobookshelf, acting as a bridge between Audiobookshelf and Storytel's API. The service fetches and processes book/audiobook metadata with intelligent title processing and multi-region support.

## Development Commands

**Start development server (with auto-reload):**
```bash
npm run dev
```

**Start production server:**
```bash
npm start
```

**Docker development:**
```bash
docker-compose up -d
```

**No testing framework is currently configured** - tests would need to be set up if needed.

## Architecture

### Core Components

**src/server.js** - Express server with three main API endpoints:
- `/:region/search` - General search (all media types)  
- `/:region/book/search` - E-book specific search (filters out audiobooks)
- `/:region/audiobook/search` - Audiobook specific search with enhanced metadata

**src/provider.js** - `StorytelProvider` class containing:
- Storytel API integration (`baseSearchUrl`) - all data comes from the search endpoint
- Metadata processing and cleanup with extensive regex patterns
- Multi-language title/series processing (20+ language patterns)
- Cover image URL enhancement (upgrades to 640x640 resolution)
- 10-minute TTL caching via node-cache

### Key Features

**Authentication**: Optional via `AUTH` environment variable, checked in `checkAuth` middleware
**Input Validation**: Region and query parameter validation in middleware
**Caching**: 10-minute cache to reduce API calls to Storytel
**Internationalization**: Extensive regex patterns for title cleanup across multiple languages/regions
**Smart Processing**: 
- Removes series information from titles and extracts to separate field
- Handles subtitle extraction
- Processes narrator, duration, and audiobook-specific metadata

### External Dependencies

**Storytel API Integration:**
- Search: `https://www.storytel.com/api/search.action` (returns all needed metadata directly)

**Audiobookshelf Integration:**
- Designed as custom metadata provider
- Returns standardized metadata format expected by Audiobookshelf

### Environment Variables

- `PORT` (default: 3000) - Server port
- `AUTH` - Optional authentication token for API access

### Development Notes

- No linting or type checking commands are currently configured
- The provider has extensive language-specific title processing regex patterns
- Cover images are automatically upgraded from 320x320 to 640x640 resolution
- Series information is intelligently extracted and formatted as "Series Name, Number"
- Maximum 5 results per search as per Storytel API limitations

<!-- GSD:project-start source:PROJECT.md -->
## Project

**abs-storytel-provider Hardening**

A Node.js REST API that serves as metadata provider for Audiobookshelf, bridging Audiobookshelf and Storytel's search API. Fetches and processes book/audiobook metadata with intelligent title processing and multi-region support. Deployed as Docker container on Synology NAS.

**Core Value:** Reliable, fast metadata search results from Storytel for Audiobookshelf — every search must return correct results without silent failures.

### Constraints

- **Verification**: Every change must be tested against the live Storytel API — no mocks
- **Downtime**: Minimal — runs as Docker container, restarts should be fast
- **Compatibility**: Must remain compatible with Audiobookshelf's expected response format
- **Dependencies**: Keep dependency count low — no large frameworks for simple improvements
<!-- GSD:project-end -->

<!-- GSD:stack-start source:codebase/STACK.md -->
## Technology Stack

## Languages
- JavaScript (Node.js) - Server-side API implementation in `src/server.js` and `src/provider.js`
## Runtime
- Node.js 22 (Alpine Linux base)
- Specified in `Dockerfile` as `node:22-alpine`
- npm
- Lockfile: `package-lock.json` present (v3)
## Frameworks
- Express.js 4.18.2 - HTTP API server framework
- nodemon 3.0.3 - Auto-reload during development (`npm run dev`)
- better-sqlite3 12.6.2 - Persistent SQLite cache layer
## Key Dependencies
- axios 1.13.6 - HTTP client for Storytel API requests
- express 4.18.2 - Web framework (see Frameworks section)
- better-sqlite3 12.6.2 - Local caching (see Frameworks section)
- cors 2.8.5 - CORS middleware for cross-origin API requests
## Configuration
- Configuration via environment variables:
- Region parameter passed as URL path parameter (required in middleware `validateRegion`)
- Dockerfile: Multi-stage pattern
## Platform Requirements
- Node.js 22+
- npm for dependency management
- Build tools for native modules (python3, make, g++)
- Docker with Alpine Linux support
- 3000 port available for Express server
- Writable filesystem for SQLite cache directory (`/app/data`)
- External network access to `https://www.storytel.com/api/search.action`
<!-- GSD:stack-end -->

<!-- GSD:conventions-start source:CONVENTIONS.md -->
## Conventions

## Naming Patterns
- Lowercase with `.js` extension: `server.js`, `provider.js`
- No hyphens or underscores in file names
- One class per file when appropriate

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Revisor01/abs-storytel-provider](https://github.com/Revisor01/abs-storytel-provider) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
