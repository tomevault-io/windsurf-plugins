---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`use-ai` is a TypeScript monorepo for building AI-powered React applications. It enables React components to expose tools (functions) to Claude AI via a Socket.IO server, allowing users to control UI through natural language.

**Key Innovation**: Tools execute client-side where app state lives. Server coordinates between Claude API and client tools.

**Monorepo Structure:**
- `packages/core/` - AG-UI protocol types and events
- `packages/client/` - React hooks (`useAI`)
- `packages/server/` - Socket.IO server with plugin support
- `packages/plugin-workflows/` - Headless workflow execution (server plugin)
- `packages/plugin-workflows-client/` - Client hooks for workflows (`useAIWorkflow`)
- `apps/example/` - Example todo app

## Development Commands

**Never use `bun test` to run tests, always `bun run test`**.

```bash
# Setup
bun install
export ANTHROPIC_API_KEY=...

# Development (server must run first)
bun run start:server           # Port 8081
bun run dev                    # Port 3000

# Building
bun run build                  # All packages
bun run build:client           # Client only
bun run build:server           # Server only

# Testing
bun run test                   # Unit tests
bun run test:e2e               # E2E tests (requires ANTHROPIC_API_KEY)
bun run test:e2e:ui            # Interactive E2E UI

# Single test file
bun run test packages/server/src/agents/AISDKAgent.test.ts
cd apps/example && bunx playwright test test/chat-history.e2e.test.ts

# Utilities
bun run kill                   # Kill processes on ports 3000, 3002, 8081
```

### Environment Variables

**Server:**
- `ANTHROPIC_API_KEY` (required)
- `LOG_FORMAT` (optional): `pretty` (default) or `json`
- `RATE_LIMIT_MAX_REQUESTS` (optional): Max requests per window (0 = unlimited)
- `RATE_LIMIT_WINDOW_MS` (optional): Window in ms (default: 60000)
- `LANGFUSE_PUBLIC_KEY`, `LANGFUSE_SECRET_KEY`, `LANGFUSE_BASE_URL` (optional): LLM observability

### WebSocket Protocol

Use `wss://your-domain.com` for secure WebSocket connections. For local development without SSL, use `ws://localhost:8081`.

## Core Architecture

### Data Flow

```
1. Client registers tools → Server stores definitions
2. User sends prompt + component state → Server forwards to Claude API
3. Claude returns tool_use blocks → Server sends tool_call to client
4. Client executes tool + waits for re-render → Sends tool_response back
5. Server sends results to Claude → Claude generates final response
```

### Key Patterns

#### Tool Definition

```typescript
import { defineTool } from '@meetsmore-oss/use-ai-client';
import { z } from 'zod';

const addTodo = defineTool(
  'Add a new todo item to the list',
  z.object({ text: z.string() }),
  (input) => ({ success: true, message: 'Todo added' })
);

// No arguments
const logout = defineTool('Log the user out', () => { /* ... */ });

// Requires user approval (destructive action)
const deleteAccount = defineTool(
  'Delete account permanently',
  () => { /* ... */ },
  { annotations: { destructiveHint: true } }
);
```

#### Component Integration

```typescript
useAI({
  tools: { addTodo, deleteTodo },
  prompt: `Todo List: ${JSON.stringify(todos)}`,  // State AI sees
  suggestions: ['Add a todo to buy groceries'],    // Empty chat suggestions (optional)
  invisible: true,                                  // For non-rendering components (optional)
  id: `Row ${rowIndex}`,                           // For multiple instances (optional)
});
```

**State Management:**
- Component state provided via `prompt` argument (serialized to string)
- Library waits for re-render before sending tool response to get updated state
- Use `invisible: true` for components that don't re-render (e.g., providers)
- Use `id` to differentiate multiple instances (prefixes tool names: `Row 1/updateLabel`)

**Suggestions:**
- Aggregated from all mounted hooks
- Up to 4 randomly selected suggestions shown in empty chat
- Click to send as message

#### Socket.IO Protocol

**Client → Server messages:**
- `run_agent`: User prompt with tools, state, conversation history (chat)
- `run_workflow`: Trigger headless workflow (use-ai extension)
- `tool_result`: Tool execution result
- `abort_run`: Cancel current run

**Server → Client events:**
- `TEXT_MESSAGE_*`: Streaming text responses
- `TOOL_CALL_*`: Tool execution requests
- `RUN_*`: Lifecycle events (started, finished, error)
- `STATE_SNAPSHOT`: Current app state
- `MESSAGES_SNAPSHOT`: Conversation history

### Plugin Architecture

Server supports plugins for extensibility. All plugins implement `UseAIServerPlugin` interface.

**Built-in Plugin:** `WorkflowsPlugin` for headless workflow execution.

```typescript
import { UseAIServer, AISDKAgent } from '@meetsmore-oss/use-ai-server';
import { WorkflowsPlugin, DifyWorkflowRunner } from '@meetsmore-oss/use-ai-plugin-workflows';
import { anthropic } from '@ai-sdk/anthropic';

const server = new UseAIServer({
  agents: { claude: new AISDKAgent({ model: anthropic('claude-3-5-sonnet-20241022') }) },
  defaultAgent: 'claude',
  plugins: [
    new WorkflowsPlugin({
      runners: new Map([

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [meetsmore/use-ai](https://github.com/meetsmore/use-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
