---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`mcpc` is a universal command-line client for the [Model Context Protocol (MCP)](https://modelcontextprotocol.io/),
which maps MCP to intuitive CLI commands for shell access, scripts, and AI coding agents.

`mcpc` can connect to any MCP server over Streamable HTTP or stdio transports,
securely login via OAuth credentials and store credentials,
and keep long-term sessions to multiple servers in parallel.
It supports all major MCP features, including tools, resources, prompts, asynchronous tasks, and notifications.

`mcpc` is handy for manual testing of MCP servers, scripting,
and AI coding agents to use MCP in ["code mode"](https://www.anthropic.com/engineering/code-execution-with-mcp),
for better accuracy and lower tokens compared to traditional tool function calling.
After all, UNIX-compatible shell script is THE most universal coding language, for both people and LLMs.

**Key capabilities:**

- Universal MCP client - Works with any MCP server over Streamable HTTP or stdio
- Persistent sessions - Keep multiple server connections alive simultaneously
- Zero setup - Connect to remote servers instantly with just a URL
- Full protocol support - Tools, resources, prompts, dynamic discovery, and async notifications
- `--json` output - Easy integration with `jq`, scripts, and other CLI tools
- AI-friendly - Designed for code generation and automated workflows
- Secure - OS keychain integration for credentials, encrypted auth storage

## Build and Development Commands

```bash
# Install dependencies
npm install

# Build the project
npm run build

# Run tests
npm test

# Test locally after building
npm link
mcpc --help

# Run linter/formatter (if configured)
npm run lint
npm run format
```

## Quick Start Examples

```bash
# List all active sessions and saved authentication profiles
mcpc

# Login to OAuth-enabled MCP server and save authentication for future use
mcpc login mcp.apify.com

# Create a persistent session
mcpc connect mcp.apify.com @test
mcpc @test                               # show session info
mcpc @test tools-list                    # list available tools
mcpc @test tools-call search-actors query:="web crawler"
mcpc @test shell                         # interactive shell

# Use JSON mode for scripting
mcpc --json @test tools-list

# Use a local server package referenced by MCP config file
mcpc connect ~/.vscode/mcp.json:filesystem @fs
mcpc @fs tools-list
```

## Design Principles

- Delightful for humans and AI agents alike (interactive + scripting)
- Avoid unnecessary interaction loops, provide sufficient context, yet be concise (save tokens)
- One clear way to do things (orthogonal commands, no surprises)
- Do not ask for user input (except `shell` and `login`, no unexpected OAuth flows)
- Be forgiving, always help users make progress (great errors + guidance)
- Be consistent with the [MCP specification](https://modelcontextprotocol.io/specification/latest), with `--json` strictly
- Minimal and portable (few deps, cross-platform)
- No slop!

## Architecture

### High-Level Structure

The project is organized as a single TypeScript package with internal modules:

```
mcpc/
├── src/
│   ├── core/           # Core MCP protocol implementation (runtime-agnostic)
│   ├── bridge/         # Bridge process logic for persistent sessions
│   ├── cli/            # CLI interface and command parsing
│   └── lib/            # Shared utilities
│       ├── auth/       # Authentication management (OAuth, bearer tokens, profiles)
│       └── ...         # Other utilities
├── bin/
│   ├── mcpc            # Main CLI executable
│   └── mcpc-bridge     # Bridge process executable
└── test/
    └── e2e/
        └── server/     # Test MCP server for E2E tests
```

### Core Components

**1. Core Module (`src/core/`)**

- Runtime-agnostic MCP protocol implementation (works with Node.js ≥18 and Bun ≥1)
- Transport abstraction: Streamable HTTP and stdio
- Protocol state machine: initialization handshake, version negotiation, session management
- Request/response correlation using JSON-RPC style with request IDs
- Multiplexing: supports up to 10 concurrent requests, queues up to 100
- Streamable HTTP connection management with reconnection (exponential backoff: 1s → 30s max)
- Event emitter for async notifications (tools/resources/prompts list changes, progress, logging)
- Uses native `fetch` API (no external HTTP libraries needed)
- **Note**: Only supports Streamable HTTP transport (current standard). The deprecated HTTP with SSE transport is not supported.

**2. Bridge Process (`src/bridge/`)**

- Separate executable (`mcpc-bridge`) that maintains persistent MCP connections
- Session persistence via `~/.mcpc/sessions.json` with file locking (`proper-lockfile` package)
- Process lifecycle management for local package servers (stdio transport)
- Unix domain socket server for CLI-to-bridge IPC (named pipes on Windows)
- Socket location: `~/.mcpc/bridges/<session-name>.sock`
- Heartbeat mechanism for health monitoring
- Orphaned process cleanup on startup
- Atomic writes for session file (write to temp, then rename)
- Lock timeout: 5 seconds


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apify/mcpc](https://github.com/apify/mcpc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
