---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

claude-canvas is a shared visual canvas that runs as a local server + browser client. Claude Code can send draw commands (shapes, arrows, text, freehand paths) via CLI or HTTP API, and users can also draw interactively in the browser. Communication between server and browser happens over WebSocket.

The canvas also serves as a **visual Q&A tool**: Claude can send structured questions alongside canvas drawings, and users answer by picking options, typing text, or drawing on the canvas.

## Commands

- **Dev (full stack):** `npm run dev` — runs the server via tsx with hot reload
- **Dev (client only):** `npm run dev:client` — Vite dev server on :5173, proxies API/WS to :7890
- **Build:** `npm run build` — builds both client (Vite) and server (esbuild)
- **Build client only:** `npm run build:client`
- **Build server only:** `npm run build:server`
- **Run tests:** `npm test` (or `npx vitest run`)
- **Run single test:** `npx vitest run tests/protocol.test.ts`
- **Run E2E tests:** `npx playwright test` (requires `npm run build` first)
- **Run all tests:** `npm test && npx playwright test`

## CLI Usage

### Session management
Each `start` creates an isolated canvas session with its own server and browser tab:
```bash
claude-canvas start                    # → { sessionId, port, url, pid }
claude-canvas list                     # List all running sessions
claude-canvas stop -s <sessionId>      # Stop a specific session
claude-canvas stop --all               # Stop all sessions
```

### Basic drawing
All commands accept `-s, --session <id>`. Omit if only one session is running.
```bash
claude-canvas draw '<DrawPayload>'     # Send draw commands
claude-canvas draw -                   # Read DrawPayload from stdin
claude-canvas clear                    # Clear all canvas objects
claude-canvas clear --layer claude     # Clear only Claude's objects
claude-canvas status '<text>'          # Update status badge in browser
claude-canvas screenshot               # Returns JSON: { path, answers }
```

### Visual Q&A (ask command)
Send structured questions with per-question canvas drawings in a single batch:
```bash
claude-canvas ask '<AskPayload>'       # Send all questions at once
claude-canvas ask -                    # Read AskPayload from stdin
```

**AskPayload format:**
```json
{
  "questions": [
    {
      "id": "q1",
      "text": "Which layout do you prefer?",
      "type": "single",
      "options": ["Layout A", "Layout B"],
      "commands": [
        {"type": "rect", "x": 80, "y": 80, "width": 200, "height": 150, "label": "Layout A", "fill": false},
        {"type": "rect", "x": 350, "y": 80, "width": 200, "height": 150, "label": "Layout B", "fill": false}
      ]
    },
    {
      "id": "q2",
      "text": "What should the title be?",
      "type": "text",
      "commands": [
        {"type": "text", "x": 300, "y": 100, "content": "Your Title Here", "fontSize": 24, "textAlign": "center"}
      ]
    }
  ]
}
```

**Question types:**
- `single` — user picks one option (radio-style pill buttons)
- `multi` — user picks multiple options (toggle pill buttons)
- `text` — user types free text
- `canvas` — user draws on the canvas as their answer

**Flow:**
1. Claude sends `ask` with all questions
2. A floating panel appears at the bottom — user navigates and answers
3. User clicks Done when all answered
4. Claude calls `screenshot` — response includes `answers` array alongside the PNG path

**Screenshot response with answers:**
```json
{
  "ok": true,
  "path": "/tmp/claude-canvas/canvas-xxx.png",
  "answers": [
    {"questionId": "q1", "value": "Layout A"},
    {"questionId": "q2", "value": "My Custom Title"},
    {"questionId": "q3", "value": ["Option A", "Option C"]},
    {"questionId": "q4", "value": "see canvas", "canvasSnapshot": "/tmp/claude-canvas/canvas-q4-xxx.png"}
  ]
}
```

### DrawCommand reference

All shapes support `label?: string` (floating text above the shape).

```typescript
// Shapes (fillStyle defaults to "hachure"; set fillStyle: "none" for wireframes)
// fillStyle: "hachure" | "solid" | "zigzag" | "cross-hatch" | "dots" | "dashed" | "zigzag-line" | "none"
{ type: "rect", x, y, width, height, label?, fillStyle? }
{ type: "circle", x, y, radius, label?, fillStyle? }
{ type: "ellipse", x, y, width, height, label?, fillStyle? }

// Lines
{ type: "line", x1, y1, x2, y2, label? }
{ type: "arrow", x1, y1, x2, y2, label? }

// Text (textAlign: "left" | "center" | "right")
{ type: "text", x, y, content, fontSize?, textAlign? }

// Freehand
{ type: "freehand", points: [[x,y], ...] }

// Grouping
{ type: "group", id, commands: DrawCommand[] }
```

## Architecture

### Protocol (`src/protocol/types.ts`)
Central type definitions shared by server and client. `DrawCommand` is a discriminated union (rect, circle, ellipse, line, arrow, text, freehand, group, connector). `Question`, `Answer`, `AskPayload` define the Q&A system. `WsMessage` wraps draw/clear/ask/screenshot messages.

### Server (`src/server/`)
Express + WebSocket server. Key files:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [uditalias/claude-canvas](https://github.com/uditalias/claude-canvas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
