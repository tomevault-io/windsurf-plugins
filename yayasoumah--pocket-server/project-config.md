---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pocket Server is a Node.js + Hono WebSocket server that provides AI agent capabilities, file system operations, terminal management, and background agent orchestration. Built with TypeScript and ESM modules, it serves as the backend for the Pocket Agent mobile app.

## Architecture

### Core Design Principles
- **Event-driven mailbox pattern** - Each session has its own AgentRunner with strict ordering
- **Server-authoritative storage** - All conversation data persisted to `data/sessions/`
- **Streaming-first** - Optimized for real-time WebSocket streaming with frame aggregation
- **Module isolation** - Clean separation between agent, auth, file system, and terminal modules

### Module Structure

```
src/
├── agent/                       # AI agent orchestration
│   ├── anthropic/              # Anthropic Claude implementation
│   │   ├── anthropic.ts        # Main agent logic and streaming
│   │   ├── prompt.ts           # System prompts and context
│   │   ├── streaming.ts        # Stream event processing
│   │   ├── title.ts            # Title generation
│   │   ├── tools/              # Tool implementations
│   │   │   ├── bash.ts         # Terminal command execution
│   │   │   ├── editor.ts       # File editing operations
│   │   │   └── web-search.ts   # Web search capability
│   │   └── types.ts            # Anthropic API types
│   ├── store/                  # Session persistence
│   │   └── session-store-fs.ts # File-based session storage
│   └── index.ts                # Agent module registration
│
├── auth/                       # Authentication & authorization
│   ├── device-registry.ts     # Device management
│   ├── middleware.ts           # Auth verification middleware
│   ├── pairing.ts             # Local PIN pairing window
│   ├── routes.ts              # Auth HTTP endpoints
│   └── token.ts               # Token generation/validation
│
├── background-agent/           # Cloud agent integrations
│   └── cursor/                # Cursor IDE integration
│       ├── client.ts          # Cursor API client
│       ├── github.ts          # GitHub integration
│       ├── routes.ts          # Cursor HTTP endpoints
│       ├── tracker.ts         # Agent status tracking
│       └── types.ts           # Cursor-specific types
│
├── file-system/               # File operations
│   ├── handlers.ts           # WebSocket handlers
│   ├── service.ts            # Core file operations
│   ├── telescope-search.ts   # Fast file search
│   ├── terminal.ts           # Terminal command execution
│   └── types.ts              # File system types
│
├── notifications/             # Push notifications
│   └── index.ts              # Expo push integration
│
├── server/                    # Core server infrastructure
│   ├── router.ts             # Custom Hono router adapter
│   └── websocket.ts          # WebSocket management
│
├── shared/                    # Shared utilities
│   ├── logger.ts             # Structured logging
│   ├── paths.ts              # Path resolution utilities
│   ├── public-url.ts         # Public URL management
│   └── types/
│       └── api.ts            # Shared API types
│
├── terminal/                  # Terminal management
│   └── terminal-manager.ts   # PTY session management
│
├── tunnel/                    # Remote access
│   └── cloudflare.ts         # Cloudflare quick tunnel
│
├── cli.ts                     # CLI entry point
└── index.ts                   # Server entry point
```

## Development Commands

```bash
# Development with hot reload
npm run dev              # tsx watch mode on port 3000

# Production build
npm run build            # Bundle with esbuild to dist/
npm start                # Run production server

# Testing
npm test                 # Run all tests
npm test -- src/foo.test.ts  # Run specific test

# Linting (gates releases)
npm run lint             # Biome linting with strict rules

# CLI commands
npm run run start              # Start server
npm run run start --remote     # Start with Cloudflare tunnel
npm run run pair               # Open pairing window
npm run run stop               # Stop server
npm run run -- --help          # Show CLI usage
npm run run update       # Update to latest version
```

## Key APIs and Protocols

### WebSocket Protocol

All WebSocket messages follow this envelope structure:
```typescript
{
  v: number;              // Protocol version (currently 1)
  type: string;           // Message type (namespaced)
  id: string;             // Unique message ID
  sessionId: string;      // Conversation/session scope
  correlationId?: string; // Request/response pairing
  ts: string;             // ISO timestamp
  seq: number;            // Sequence number per session
  payload?: unknown;      // Type-specific payload
}
```

### Message Namespaces
- `agent:*` - AI agent lifecycle and streaming
- `fs:*` - File system operations
- `term:*` - Terminal I/O
- `ws:*` - WebSocket meta events
- `server:*` - Server status
- `discovery:*` - Server discovery

### Authentication Flow

1. **Pairing**:
   - Local (default):
     ```
     POST /auth/pair { deviceId, pin }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yayasoumah/pocket-server](https://github.com/yayasoumah/pocket-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
