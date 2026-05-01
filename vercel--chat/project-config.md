---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build Commands

```bash
# Install dependencies
pnpm install

# Build all packages (uses Turborepo)
pnpm build

# Type-check all packages
pnpm typecheck

# Check all packages (linting and formatting)
pnpm -w run check

# Auto-fix linting and formatting issues
pnpm -w run check --write

# Check for unused exports/dependencies
pnpm knip

# Run all tests
pnpm test

# Run full validation. ALWAYS do this before declaring a task to be done.
pnpm validate


# Run dev mode (watch for changes)
pnpm dev

# Build a specific package
pnpm --filter chat build
pnpm --filter @chat-adapter/slack build
pnpm --filter @chat-adapter/gchat build
pnpm --filter @chat-adapter/teams build

# Run tests for a specific package
pnpm --filter chat test
pnpm --filter @chat-adapter/integration-tests test

# Run a single test file
pnpm --filter @chat-adapter/integration-tests test src/slack.test.ts
```

## Code Style

- Install dependencies with `pnpm add` rather than manually editing package.json
- `sample-messages.md` files in adapter packages contain real-world webhook logs as examples

## Architecture

This is a **pnpm monorepo** using **Turborepo** for build orchestration. All packages use ESM (`"type": "module"`), TypeScript, and **tsup** for bundling.

### Package Structure

- **`packages/chat-sdk`** - Core SDK (`chat` package) with `Chat` class, types, and markdown utilities (mdast-based)
- **`packages/adapter-slack`** - Slack adapter using `@slack/web-api`
- **`packages/adapter-gchat`** - Google Chat adapter using `googleapis`
- **`packages/adapter-teams`** - Microsoft Teams adapter using `botbuilder`
- **`packages/state-memory`** - In-memory state adapter (for development/testing)
- **`packages/state-redis`** - Redis state adapter (for production)
- **`packages/adapter-whatsapp`** - WhatsApp adapter using Meta Cloud API
- **`packages/integration-tests`** - Integration tests against real platform APIs
- **`examples/nextjs-chat`** - Example Next.js app showing how to use the SDK

### Core Concepts

1. **Chat** (`packages/chat-sdk/src/chat.ts` in `chat` package) - Main entry point that coordinates adapters and handlers
2. **Adapter** - Platform-specific implementations (Slack, Teams, Google Chat). Each adapter:
   - Handles webhook verification and parsing
   - Converts platform-specific message formats to/from normalized format
   - Provides `FormatConverter` for markdown/AST transformations
3. **StateAdapter** - Persistence layer for subscriptions and distributed locking
4. **Thread** - Represents a conversation thread with methods like `post()`, `subscribe()`, `startTyping()`
5. **Message** - Normalized message format with `text`, `formatted` (mdast AST), and `raw` (platform-specific)

### Thread ID Format

All thread IDs follow the pattern: `{adapter}:{channel}:{thread}`

- Slack: `slack:C123ABC:1234567890.123456`
- Teams: `teams:{base64(conversationId)}:{base64(serviceUrl)}`
- Google Chat: `gchat:spaces/ABC123:{base64(threadName)}`

### Message Handling Flow

1. Platform sends webhook to `/api/webhooks/{platform}`
2. Adapter verifies request, parses message, calls `chat.handleIncomingMessage()`
3. Chat class acquires lock on thread, then:
   - Checks if thread is subscribed -> calls `onSubscribedMessage` handlers
   - Checks for @mention -> calls `onNewMention` handlers
   - Checks message patterns -> calls matching `onNewMessage` handlers
4. Handler receives `Thread` and `Message` objects

### Formatting System

Messages use **mdast** (Markdown AST) as the canonical format. Each adapter has a `FormatConverter` that:

- `toAst(platformText)` - Converts platform format to mdast
- `fromAst(ast)` - Converts mdast to platform format
- `renderPostable(message)` - Renders a `PostableMessage` to platform string

## Testing

### Test Utilities

The `packages/chat/src/mock-adapter.ts` file provides shared test utilities:

- `createMockAdapter(name)` - Creates a mock Adapter with vi.fn() mocks for all methods
- `createMockState()` - Creates a mock StateAdapter with working in-memory subscriptions, locks, and cache
- `createTestMessage(id, text, overrides?)` - Creates a test Message object
- `mockLogger` - A mock Logger that captures all log calls

Example usage:

```typescript
import { createMockAdapter, createMockState, createTestMessage } from "./mock-adapter";

const adapter = createMockAdapter("slack");
const state = createMockState();
const message = createTestMessage("msg-1", "Hello world");
```

## Recording & Replay Tests

Production webhook interactions can be recorded and converted into replay tests:

1. **Recording**: Enable `RECORDING_ENABLED=true` in deployed environment. Recordings are tagged with git SHA.
2. **Export**: Use `pnpm recording:list` and `pnpm recording:export <session-id>` from `examples/nextjs-chat`
3. **Convert**: Extract webhook payloads and create JSON fixtures in `packages/integration-tests/fixtures/replay/`
4. **Test**: Write replay tests using helpers from `replay-test-utils.ts`

See `packages/integration-tests/fixtures/replay/README.md` for detailed workflow.

### Downloading and Analyzing Recordings


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vercel/chat](https://github.com/vercel/chat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
