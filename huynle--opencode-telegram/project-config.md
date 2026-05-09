---
trigger: always_on
description: Manages the lifecycle of OpenCode instances:
---

# AGENTS.md - OpenCode Telegram Integration

## Project Overview

This project creates a **Telegram bot that orchestrates multiple OpenCode instances** through forum topics. Each forum topic in a Telegram supergroup gets its own dedicated OpenCode instance, enabling multi-user/multi-project AI assistance.

### Key Capabilities
- **Forum Topic → OpenCode Instance**: Each topic gets a dedicated OpenCode session
- **Real-time Streaming**: SSE events from OpenCode are streamed to Telegram as editable messages
- **Instance Lifecycle Management**: Auto-start, health checks, crash recovery, idle timeout
- **Persistent State**: SQLite databases track topic mappings and instance state across restarts

## Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    Telegram Supergroup (Forum)                       │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐                          │
│  │ Topic #1 │  │ Topic #2 │  │ Topic #3 │  ...                     │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘                          │
└───────┼─────────────┼─────────────┼─────────────────────────────────┘
        │             │             │
        ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      Integration Layer                               │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐                 │
│  │ grammY Bot  │  │TopicManager │  │StreamHandler│                 │
│  └─────────────┘  └─────────────┘  └─────────────┘                 │
└─────────────────────────────────────────────────────────────────────┘
        │             │             │
        ▼             ▼             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    Instance Manager (Orchestrator)                   │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐              │
│  │ Instance #1  │  │ Instance #2  │  │ Instance #3  │  ...         │
│  │ Port 4100    │  │ Port 4101    │  │ Port 4102    │              │
│  │ opencode     │  │ opencode     │  │ opencode     │              │
│  │ serve        │  │ serve        │  │ serve        │              │
│  └──────────────┘  └──────────────┘  └──────────────┘              │
└─────────────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
src/
├── index.ts              # Entry point - starts the bot
├── config.ts             # Configuration from environment variables
├── integration.ts        # Wires all components together
├── api-server.ts         # External instance registration API
├── bot/
│   └── handlers/
│       └── forum.ts      # Telegram message/command handlers
├── forum/
│   ├── index.ts          # Exports
│   ├── topic-manager.ts  # Topic → Session mapping logic
│   └── topic-store.ts    # SQLite persistence for topic mappings
├── opencode/
│   ├── index.ts          # Exports
│   ├── client.ts         # OpenCode REST API client
│   ├── discovery.ts      # Discover running OpenCode instances
│   ├── stream-handler.ts # SSE → Telegram message bridging
│   ├── telegram-markdown.ts # Markdown conversion for Telegram
│   └── types.ts          # OpenCode-related types
├── orchestrator/
│   ├── index.ts          # Exports
│   ├── manager.ts        # Manages multiple instances
│   ├── instance.ts       # Single OpenCode instance lifecycle
│   ├── port-pool.ts      # Port allocation
│   └── state-store.ts    # SQLite persistence for instance state
└── types/
    ├── forum.ts          # Forum/topic types
    └── orchestrator.ts   # Orchestrator types

data/                     # Runtime data (gitignored)
├── orchestrator.db       # Instance state
└── topics.db             # Topic mappings
```

## Key Components

### 1. Integration Layer (`src/integration.ts`)
The main orchestration point that:
- Creates and configures the grammY bot
- Sets up event handlers for orchestrator events
- Manages OpenCode clients and SSE subscriptions
- Routes messages between Telegram and OpenCode

### 2. Instance Manager (`src/orchestrator/manager.ts`)
Manages the lifecycle of OpenCode instances:
- Creates instances on-demand for new topics
- Handles health checks and crash recovery
- Implements idle timeout for resource cleanup
- Persists state to SQLite for restart recovery

### 3. Stream Handler (`src/opencode/stream-handler.ts`)
Bridges SSE events from OpenCode to Telegram:
- Shows "Thinking..." progress messages
- Streams text responses with throttling
- Handles tool execution status
- Edits messages in-place for clean UX

### 4. Topic Manager (`src/forum/topic-manager.ts`)
Maps forum topics to OpenCode sessions:
- Creates sessions for new topics automatically
- Routes messages to correct instances
- Handles both new and existing topics

## Running the Bot

### Prerequisites
1. Telegram bot token from @BotFather
2. Telegram supergroup with **Topics enabled**
3. Bot added as **admin** to the supergroup
4. Bun runtime installed

### Environment Variables
```bash
# Required
TELEGRAM_BOT_TOKEN=your-bot-token
TELEGRAM_CHAT_ID=-100xxxxxxxxxx  # Supergroup ID (negative number)

# Optional
PROJECT_BASE_PATH=/path/to/projects  # Where topic directories are created

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huynle/opencode-telegram](https://github.com/huynle/opencode-telegram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
