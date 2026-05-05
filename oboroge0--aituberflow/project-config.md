---
trigger: always_on
description: AITuberFlow is a visual workflow editor for creating AI VTuber streaming setups. Users connect nodes in a graph to build pipelines that handle:
---

# AITuberFlow Development Guide

## Project Overview

AITuberFlow is a visual workflow editor for creating AI VTuber streaming setups. Users connect nodes in a graph to build pipelines that handle:
- Chat input (YouTube, Twitch)
- LLM responses (OpenAI, Anthropic, Google, Ollama)
- Text-to-Speech (VOICEVOX, COEIROINK, Style-Bert-VITS2)
- Avatar control (VRM models, expressions, lip sync)
- Audio playback and subtitle display

## Architecture

```
AITuberFlow/
├── apps/
│   ├── server-ts/       # TypeScript Bun+Hono backend
│   │   ├── src/
│   │   │   ├── engine/  # Workflow execution engine
│   │   │   ├── routes/  # API endpoints
│   │   │   ├── db/      # Drizzle ORM + bun:sqlite
│   │   │   ├── websocket/ # Native WebSocket handler
│   │   │   └── index.ts # Server entry point
│   │   └── package.json
│   └── web/             # Next.js frontend
│       ├── app/         # Pages (editor, overlay)
│       ├── components/  # React components
│       └── stores/      # Zustand state management
├── packages/
│   └── sdk-ts/          # TypeScript SDK for node development
│       └── src/
├── plugins/             # Node plugins (each in own directory)
│   ├── {node-name}/
│   │   ├── manifest.json
│   │   └── node.ts      # TypeScript implementation
├── tests/               # Test suites
│   ├── engine/          # TypeScript engine tests (bun:test)
│   └── routes/          # TypeScript route tests (bun:test)
└── templates/           # Workflow templates (JSON)
```

## Tech Stack

| 項目 | 技術 |
|------|------|
| Runtime | Bun |
| Framework | Hono |
| Database | bun:sqlite + Drizzle ORM |
| WebSocket | Native WebSocket (Hono/Bun) |
| Validation | Zod |
| Frontend | Next.js + React + Tailwind CSS |

## Node Development

### Plugin Structure

Each node is a plugin in `plugins/{node-name}/`:
- `manifest.json` - Node metadata, inputs, outputs, config schema
- `node.ts` - TypeScript implementation extending `BaseNode`

### BaseNode Methods

```typescript
import { BaseNode, NodeContext, Event } from "@aituber-flow/sdk";

class MyNode extends BaseNode {
  async setup(config: Record<string, unknown>, context: NodeContext): Promise<void> {
    // Called once when workflow starts
  }

  async execute(inputs: Record<string, unknown>, context: NodeContext): Promise<Record<string, unknown>> {
    // Called each time the node runs, returns outputs
    return { outputId: value };
  }

  async onEvent(event: Event, context: NodeContext): Promise<Record<string, unknown> | null> {
    // Handle WebSocket events (optional)
    return null;
  }

  async teardown(): Promise<void> {
    // Called when workflow stops
  }
}
```

### NodeContext API

```typescript
await context.log(message, "info");  // Log to frontend
await context.emitEvent({ type: "event.name", payload: {} });  // WebSocket event
context.createTask(promise);  // Background task
context.cancelBackgroundTasks();  // Cancel all tasks
```

### Frontend Registration (v1.2.0+)

プラグインのUI設定は `manifest.json` の `ui` セクションで定義するだけで自動登録されます。
フロントエンドのコード編集は不要です。

```json
{
  "ui": {
    "label": "My Node",
    "icon": "Cpu",
    "color": "#10B981",
    "bgColor": "rgba(16, 185, 129, 0.1)"
  }
}
```

## Node Categories

- `input` - Data sources (chat, timer, manual input)
- `output` - Endpoints (console, subtitle, audio player)
- `process` - Data transformation (LLM, text transform, emotion analyzer)
- `avatar` - Avatar control (avatar-controller, lip-sync)
- `control` - Flow control (switch, delay, loop)
- `llm` - LLM providers
- `tts` - Text-to-Speech engines

## Event System

Events enable real-time communication between nodes and frontend:

```typescript
await context.emitEvent({
  type: "avatar.expression",
  payload: { expression: "happy", intensity: 0.8 },
});
```

Common events:
- `avatar.expression` - Change avatar expression
- `avatar.mouth` - Lip sync mouth value (0.0-1.0)
- `avatar.motion` - Trigger animation
- `audio.play` - Play audio file
- `audio.stop` - Stop audio playback
- `subtitle` - Display subtitle text

## Overlay System

OBS-compatible overlay at `/overlay/[workflowId]`:
- Unified overlay with avatar, subtitles, and audio playback
- Transparent background for OBS Browser Source
- URL parameters: model, animation, scale, x, y, subtitle, subPosition, subFontSize, volume, debug

## Commands

```bash
# Full stack (backend + frontend)
npm run dev

# Individual services
npm run dev:web         # Frontend only
npm run dev:api         # Backend only

# Install all sub-project dependencies
npm run setup

# Run tests
npm test

# Lint
npm run lint

# Create a new plugin
npm run create-node
```

## Development Tips

1. **Single Responsibility**: Each node should do one thing well
2. **Events for Real-time**: Use events for time-sensitive data (lip sync, expressions)
3. **Outputs for Data Flow**: Use outputs for data that flows to next node
4. **Pass-through Outputs**: Include input data in outputs when downstream nodes need it
5. **Async/Await**: All node methods are async - use `await` properly
6. **Type Safety**: Match manifest types with TypeScript implementations

## Running the Project

```bash
# Backend
cd apps/server-ts && bun run dev

# Frontend
cd apps/web && npm run dev


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [oboroge0/AITuberFlow](https://github.com/oboroge0/AITuberFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
