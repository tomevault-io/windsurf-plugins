---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an **OpenClaw Channel Plugin** for WeCom (企业微信 / WeChat Work). It enables AI bot integration with enterprise WeChat through a dual-mode architecture.

- **Package**: `@mocrane/wecom`
- **Type**: ES Module (NodeNext)
- **Entry**: `index.ts`

## Architecture

### Dual-Mode Design (Bot + Agent)

The plugin implements a unique dual-mode architecture:

| Mode | Purpose | Webhook Path | Capabilities |
|------|---------|--------------|--------------|
| **Bot** (智能体) | Real-time streaming chat | `/wecom`, `/wecom/bot` | Streaming responses, low latency, text/image only |
| **Agent** (自建应用) | Fallback & broadcast | `/wecom/agent` | File sending, broadcasts, long tasks (>6min) |

**Key Design Principle**: Bot is preferred for conversations; Agent is used as fallback when Bot cannot deliver (files, timeouts) or for proactive broadcasts.

### Core Components

```
index.ts              # Plugin entry - registers channel and HTTP handlers
src/
  channel.ts          # ChannelPlugin implementation, lifecycle management
  monitor.ts          # Core webhook handler, message flow, stream state
  runtime.ts          # Runtime state singleton
  http.ts             # HTTP client with undici + proxy support
  crypto.ts           # AES-CBC encryption/decryption for webhooks
  media.ts            # Media file download/decryption
  outbound.ts         # Outbound message adapter
  target.ts           # Target resolution (user/party/tag/chat)
  dynamic-agent.ts    # Dynamic agent routing (per-user/per-group isolation)
  agent/
    api-client.ts     # WeCom API client with AccessToken caching
    handler.ts        # XML webhook handler for Agent mode
  config/
    schema.ts         # Zod schemas for configuration
  monitor/
    state.ts          # StreamStore and ActiveReplyStore with TTL pruning
  types/constants.ts  # API endpoints and limits
```

### Stream State Management

The plugin uses a sophisticated stream state system (`src/monitor/state.ts`):

- **StreamStore**: Manages message streams with 6-minute timeout window
- **ActiveReplyStore**: Tracks `response_url` for proactive pushes
- **Pending Queue**: Debounces rapid messages (500ms default)
- **Message Deduplication**: Uses `msgid` to prevent duplicate processing

### Token Management

Agent mode uses automatic AccessToken caching (`src/agent/api-client.ts`):
- Token cached with 60-second refresh buffer
- Automatic retry on expiration
- Thread-safe refresh deduplication

## Development Commands

### Testing

This project uses **Vitest**. Tests extend from a base config at `../../vitest.config.ts`:

```bash
# Run all tests
npx vitest --config vitest.config.ts

# Run specific test file
npx vitest --config vitest.config.ts src/crypto.test.ts

# Run tests matching pattern
npx vitest --config vitest.config.ts --testNamePattern="should encrypt"

# Watch mode
npx vitest --config vitest.config.ts --watch
```

Test files are located alongside source files with `.test.ts` suffix:
- `src/crypto.test.ts`
- `src/monitor.integration.test.ts`
- `src/monitor/state.queue.test.ts`
- etc.

### Type Checking

```bash
npx tsc --noEmit
```

### Build

The plugin is loaded directly as TypeScript by OpenClaw. No build step is required for development, but type checking is recommended.

## Configuration Schema

Configuration is validated via Zod (`src/config/schema.ts`):

```typescript
{
  enabled: boolean,
  bot: {
    token: string,              // Bot webhook token
    encodingAESKey: string,     // AES encryption key
    receiveId: string?,         // Optional receive ID
    streamPlaceholderContent: string?,  // "Thinking..."
    welcomeText: string?,
    dm: { policy, allowFrom }
  },
  agent: {
    corpId: string,
    corpSecret: string,
    agentId: number,
    token: string,              // Callback token
    encodingAESKey: string,     // Callback AES key
    welcomeText: string?,
    dm: { policy, allowFrom }
  },
  network: {
    egressProxyUrl: string?     // For dynamic IP scenarios
  },
  media: {
    maxBytes: number?           // Default 25MB
  },
  dynamicAgents: {
    enabled: boolean?           // Enable per-user/per-group agents
    dmCreateAgent: boolean?     // Create agent per DM user
    groupEnabled: boolean?      // Enable for group chats
    adminUsers: string[]?       // Admin users (bypass dynamic routing)
  }
}
```

### Dynamic Agent Routing

When `dynamicAgents.enabled` is `true`, the plugin automatically creates isolated Agent instances for each user/group:

```bash
# Enable dynamic agents
openclaw config set channels.wecom.dynamicAgents.enabled true

# Configure admin users (use main agent)
openclaw config set channels.wecom.dynamicAgents.adminUsers '["admin1","admin2"]'
```

**Generated Agent ID format**: `wecom-{type}-{peerId}`
- DM: `wecom-dm-zhangsan`
- Group: `wecom-group-wr123456`

Dynamic agents are automatically added to `agents.list` in the config file.

## Key Technical Details

### Webhook Security

- **Signature Verification**: HMAC-SHA256 with token
- **Encryption**: AES-CBC with PKCS#7 padding (32-byte blocks)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TencentCloud-Lighthouse/openclaw-wecom](https://github.com/TencentCloud-Lighthouse/openclaw-wecom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
