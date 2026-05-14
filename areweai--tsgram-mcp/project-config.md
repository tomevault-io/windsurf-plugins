---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

TSGram MCP (`tsgram-mcp`) is a TypeScript/Node.js system that enables communication between Claude Code sessions and Telegram. It includes a Telegram bot with automatic AI responses, MCP server for Claude Code integration, and CLI-to-Telegram forwarding capabilities. The system runs in Docker containers and provides secure, filtered communication between Claude Code CLI and Telegram.

**Note**: The repository name `signal-aichat` reflects the project's original scope that included Signal integration, but Signal functionality has been archived due to platform limitations.

## Architecture

### Core Components

#### Telegram System (Primary)
- **src/telegram-mcp-webhook-server.ts**: Main Telegram MCP server with AI auto-responses
- **src/telegram-bot-ai-powered.ts**: Main AI-powered Telegram bot (deployed in Docker)
- **src/telegram/bot-client.ts**: Telegram Bot API client implementation
- **src/types/telegram.ts**: Telegram-specific type definitions
- **src/mcp-docker-proxy.ts**: MCP proxy for Claude Code → Docker communication

#### Bridge System
- **src/cli-telegram-bridge.ts**: Secure CLI response forwarding to Telegram
- **src/telegram-to-claude-bridge.ts**: Bridge for Telegram messages to Claude
- **src/telegram-claude-queue.ts**: Queue management for Telegram-Claude communication
- **claude-with-telegram.sh**: Bash wrapper for Claude Code CLI
- **claude-tg**: Global command (symlinked to `/usr/local/bin/claude-tg`)

#### Development Dashboard
- **src/spa/**: React-based web dashboard (runs on port 3000)
- **vite.spa.config.ts**: Dashboard build configuration
- Real-time monitoring of bots and services

#### Signal System (Archived)
- See `/docs/signal/BLOCKED_UNTIL_SIGNAL_FIXES_MANUAL_QR.md` for details
- Signal integration abandoned due to platform limitations

#### Shared Components
- **src/models/**: AI model implementations
  - `ChatModel.ts`: Factory class for AI model instances
  - `OpenAIAPI.ts`, `OpenRouterAPI.ts`: Model-specific implementations
- **src/utils/ChatHistory.ts**: Conversation history management
- **src/types/**: TypeScript type definitions
- **scripts/**: Deployment and testing automation scripts

### Message Flow

1. Telegram message received → `handleUpdate()` in bot
2. Check for `:h` commands or process with AI
3. Execute workspace commands or generate AI response
4. Return formatted response to Telegram chat

### Model Integration

Each AI model implements the `AIModelAPI` interface with a unified `send()` method. Models are dynamically loaded based on environment configuration and available credentials.

## CLI-to-Telegram Forwarding

### Global Command Setup

**IMPORTANT: The global `claude-tg` command has been installed:**
```bash
# This was executed by the user during setup:
sudo ln -sf /Users/edunc/Documents/gitz/tsgram-mcp/claude-tg /usr/local/bin/claude-tg
```

### Usage

Instead of using `claude` directly, use `claude-tg` to forward responses to Telegram:

```bash
# Normal Claude Code usage:
claude "What's the weather?"

# With Telegram forwarding (recommended):
claude-tg "What's the weather?"
claude-tg mcp list
claude-tg --help
```

### Security Filtering

All CLI responses are automatically filtered to remove:
- API keys (`sk-`, `pk-`, tokens, `OPENROUTER_API_KEY`, etc.)
- Environment variables (`API_KEY=`, `TOKEN=`, `SECRET=`)
- Database URLs (`postgres://`, `mysql://`, `mongodb://`)
- File paths containing sensitive info (`.env` files)
- Telegram bot tokens (`TELEGRAM_BOT_TOKEN`)

### Target Configuration

Responses are sent to:
- **Chat ID**: Configured via `AUTHORIZED_CHAT_ID` environment variable (numeric user ID from @userinfobot)
- **Format**: Professional MCP-branded messages
- **Security**: Uses numeric user ID instead of username for authorization (more secure)

### Integration with Claude Code

**For manual CLI forwarding**, users can replace `claude` with `claude-tg`:
```bash
# Instead of:
claude "analyze this codebase"

# Use:
claude-tg "analyze this codebase"
```

**For automated forwarding from Claude Code sessions**, the system is ready but would require Claude Code to use the `claude-tg` wrapper instead of the direct `claude` command. This allows all Claude Code CLI responses to be automatically forwarded to Telegram with security filtering.

## Development Commands

### Docker Deployment (Primary)

```bash
# Build and start main services
npm run docker:build
npm run docker:start

# Check service status
npm run docker:health
docker ps --filter name=tsgram

# View logs
npm run docker:logs

# Stop services
npm run docker:stop

# Rebuild after changes
npm run docker:rebuild
```

### Development Services

```bash
# Start web dashboard
npm run dashboard            # Runs on http://localhost:3000

# Start individual services
npm run mcp                  # MCP server (port 4040)
npm run cli-bridge          # CLI-to-Telegram bridge
npm run tg-claude-bridge     # Telegram-to-Claude bridge
npm run telegram-queue       # Queue management

# Health checks
npm run health-check         # Check ports 4040 and 4041
curl http://localhost:4040/health
```

### Build and Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [areweai/tsgram-mcp](https://github.com/areweai/tsgram-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
