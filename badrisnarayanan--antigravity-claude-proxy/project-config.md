---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Antigravity Claude Proxy is a Node.js proxy server that exposes an Anthropic-compatible API backed by Antigravity's Cloud Code service. It enables using Claude models (`claude-sonnet-4-6-thinking`, `claude-opus-4-6-thinking`) and Gemini models (`gemini-3-flash`, `gemini-3.1-pro-low`, `gemini-3.1-pro-high`) with Claude Code CLI.

The proxy translates requests from Anthropic Messages API format → Google Generative AI format → Antigravity Cloud Code API, then converts responses back to Anthropic format with full thinking/streaming support.

## Commands

```bash
# Install dependencies (automatically builds CSS via prepare hook)
npm install

# Start server (runs on port 8080)
npm start

# Start with specific account selection strategy
npm start -- --strategy=sticky      # Cache-optimized (stays on same account)
npm start -- --strategy=round-robin # Load-balanced (rotates every request)
npm start -- --strategy=hybrid      # Smart distribution (default)

# Start with model fallback enabled (falls back to alternate model when quota exhausted)
npm start -- --fallback

# Start with developer mode (debug logging + dev tools)
npm start -- --dev-mode

# Start with debug logging (legacy alias, also enables dev mode)
npm start -- --debug

# Development mode (file watching)
npm run dev              # Watch server files only
npm run dev:full         # Watch both CSS and server files (recommended for frontend dev)

# CSS build commands
npm run build:css        # Build CSS once (minified)
npm run watch:css        # Watch CSS files for changes

# Account management
npm run accounts         # Interactive account management
npm run accounts:add     # Add a new Google account via OAuth
npm run accounts:add -- --no-browser  # Add account on headless server (manual code input)
npm run accounts:list    # List configured accounts
npm run accounts:verify  # Verify account tokens are valid

# Run all tests (server must be running on port 8080)
npm test

# Run individual tests
npm run test:signatures    # Thinking signatures
npm run test:multiturn     # Multi-turn with tools
npm run test:streaming     # Streaming SSE events
npm run test:interleaved   # Interleaved thinking
npm run test:images        # Image processing
npm run test:caching       # Prompt caching
npm run test:crossmodel    # Cross-model thinking signatures
npm run test:oauth         # OAuth no-browser mode
npm run test:cache-control # Cache control field stripping

# Run strategy unit tests (no server required)
node tests/test-strategies.cjs
```

## Architecture

**Request Flow:**
```
Claude Code CLI → Express Server (server.js) → CloudCode Client → Antigravity Cloud Code API
```

**Directory Structure:**

```
src/
├── index.js                    # Entry point
├── server.js                   # Express server
├── constants.js                # Configuration values
├── errors.js                   # Custom error classes
├── fallback-config.js          # Model fallback mappings and helpers
│
├── cloudcode/                  # Cloud Code API client
│   ├── index.js                # Public API exports
│   ├── session-manager.js      # Session ID derivation for caching
│   ├── rate-limit-parser.js    # Parse reset times from headers/errors
│   ├── request-builder.js      # Build API request payloads
│   ├── sse-parser.js           # Parse SSE for non-streaming
│   ├── sse-streamer.js         # Stream SSE events in real-time
│   ├── message-handler.js      # Non-streaming message handling
│   ├── streaming-handler.js    # Streaming message handling
│   └── model-api.js            # Model listing and quota APIs
│
├── account-manager/            # Multi-account pool management
│   ├── index.js                # AccountManager class facade
│   ├── storage.js              # Config file I/O and persistence
│   ├── rate-limits.js          # Rate limit tracking and state
│   ├── credentials.js          # OAuth token and project handling
│   └── strategies/             # Account selection strategies
│       ├── index.js            # Strategy factory (createStrategy)
│       ├── base-strategy.js    # Abstract base class
│       ├── sticky-strategy.js  # Cache-optimized sticky selection
│       ├── round-robin-strategy.js  # Load-balanced rotation
│       ├── hybrid-strategy.js  # Smart multi-signal distribution
│       └── trackers/           # State trackers for hybrid strategy
│           ├── index.js        # Re-exports trackers
│           ├── health-tracker.js    # Account health scores
│           ├── token-bucket-tracker.js  # Client-side rate limiting
│           └── quota-tracker.js     # Quota-aware account selection
│
├── auth/                       # Authentication
│   ├── oauth.js                # Google OAuth with PKCE
│   ├── token-extractor.js      # Legacy token extraction from DB
│   └── database.js             # SQLite database access
│
├── webui/                      # Web Management Interface
│   └── index.js                # Express router and API endpoints
│
├── modules/                    # Feature modules
│   └── usage-stats.js          # Request tracking and history persistence
│

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [badrisnarayanan/antigravity-claude-proxy](https://github.com/badrisnarayanan/antigravity-claude-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
