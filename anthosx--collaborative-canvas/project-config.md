---
trigger: always_on
description: **Version 1.0.0** - Claude Code Plugin for Visual Collaboration
---

# Collaborative Canvas - Developer Documentation

**Version 1.0.0** - Claude Code Plugin for Visual Collaboration

> AI-powered visual collaboration powered by Excalidraw. Create diagrams, flowcharts, and architecture sketches with Claude's assistance. Or just draw with Claude for fun.

## Distribution Model

The MCP server is pre-bundled (`mcp-server/dist/bundle.cjs`) and electron source is pre-compiled (`electron-app/dist/`), both committed to git. The packaged Electron `.app` (282MB) is distributed via **GitHub Releases**.

**First use**: When a user opens a canvas for the first time, the MCP server auto-detects the missing Electron app and runs `setup.sh` to download it. No manual build step needed.

**Manual setup** (if auto-download fails):
```bash
./scripts/setup.sh
```

**For maintainers** (rebuilding artifacts):
```bash
cd mcp-server && npm install && npm run bundle   # Rebuild MCP bundle
cd electron-app && npm install && npm run build   # Rebuild electron dist
cd electron-app && npm run package:dir            # Rebuild .app for release
```

---

## System Overview

### Purpose

Collaborative Canvas enables visual collaboration between users and Claude. Users draw diagrams in a native Excalidraw window, click "Collaborate" for Claude's feedback, and Claude can analyze and add elements to the canvas.

### Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                        Claude Code                               │
│                                                                  │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐      │
│  │   Hooks      │    │  MCP Client  │    │   Plugin     │      │
│  │  (PostTool)  │◄──►│              │◄──►│   Config     │      │
│  └──────────────┘    └──────────────┘    └──────────────┘      │
│         │                   │                                    │
└─────────┼───────────────────┼────────────────────────────────────┘
          │                   │ stdio (JSON-RPC)
          │                   ▼
          │         ┌──────────────────┐
          │         │   MCP Server     │
          │         │   (Node.js)      │
          │         └──────────────────┘
          │                   │
          │                   ▼
          │         ┌──────────────────┐
          │         │  DrawingStorage  │◄─────────────────┐
          │         │  (File-based)    │                  │
          │         └──────────────────┘                  │
          │                   │                           │
          ▼                   │                           │
┌─────────────────┐           │                           │
│   Hook Queue    │◄──────────┼───────────────────────────┤
│  (JSON file)    │           │                           │
└─────────────────┘           │                           │
          ▲                   ▼                           │
          │         ┌──────────────────┐                  │
          └─────────┤  Electron App    │──────────────────┘
                    │  (Excalidraw)    │
                    └──────────────────┘
```

### Components

| Component | Technology | Purpose |
|-----------|------------|---------|
| MCP Server | Node.js/TypeScript | Provides tools to Claude via MCP protocol |
| Electron App | Electron/React | Native Excalidraw editor window |
| Hook Scripts | Node.js | PostToolUse hooks for collaboration workflow |
| Storage | JSON files | Persists drawings and queue state |

## Directory Structure

```
collaborative-canvas/
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest (name, version, etc.)
├── .mcp.json                    # MCP server configuration
├── hooks/
│   ├── hooks.json               # Hook event configuration
│   └── scripts/
│       ├── open-canvas-choice.js    # Force listen/save after open
│       ├── save-canvas-listen.js    # Auto-trigger listen after save
│       ├── collaboration-poll.js    # 30-min polling for buttons
│       └── get-canvas-state-decision.js  # Force close/save decision
├── skills/
│   └── canvas/
│       └── SKILL.md             # Auto-activating skill definition
├── commands/
│   └── canvas.md                # /canvas slash command
├── mcp-server/
│   ├── package.json
│   ├── tsconfig.json
│   ├── src/
│   │   ├── index.ts             # Entry point
│   │   ├── server.ts            # MCP server (11 tools, 3 resources)
│   │   ├── storage/
│   │   │   └── DrawingStorage.ts    # File-based CRUD
│   │   ├── tools/
│   │   │   ├── openCanvas.ts    # Create/open + launch Electron
│   │   │   ├── saveCanvas.ts    # Persist elements
│   │   │   ├── getCanvasState.ts    # Compact JSON format
│   │   │   ├── addToCanvas.ts   # Add elements
│   │   │   ├── listen.ts        # Wait for collaboration
│   │   │   ├── closeWidget.ts   # Close Electron window
│   │   │   └── ...              # Other tools
│   │   ├── collaboration/       # Strategy pattern (future)
│   │   ├── api/                 # HTTP client (browser fallback)
│   │   └── types/               # TypeScript definitions
│   └── dist/                    # Compiled output
├── electron-app/
│   ├── package.json
│   ├── main/
│   │   ├── main.ts              # Electron entry point
│   │   ├── WindowManager.ts     # Window lifecycle
│   │   ├── ipc-handlers.ts      # IPC + direct hook queue writes
│   │   └── storage/             # Bundled DrawingStorage
│   ├── renderer/
│   │   ├── App.tsx              # React main component
│   │   └── components/
│   │       ├── ExcalidrawCanvas.tsx
│   │       ├── Toolbar.tsx
│   │       └── SaveConfirmDialog.tsx
│   ├── dist/                    # Built output
│   └── release/                 # Packaged app (.app, .exe)
├── scripts/
│   └── setup.sh                 # Build script
├── package.json                 # Root package.json
├── README.md                    # User documentation
└── CLAUDE.md                    # This file
```

## Storage

### XDG-Compliant Path

```
~/.local/share/collaborative-canvas/
├── drawings/
│   ├── {uuid}.excalidraw        # Drawing files (JSON)
│   └── {uuid}.meta.json         # Metadata (name, dates, tags)
├── hooks-queue.json             # Collaboration request queue
├── listen-state-{id}.json       # Per-drawing listen state
├── collaboration-status.json    # Retry tracking
├── logs/                        # Log files
└── screenshots/                 # Captured canvas images
```

### Key Files

| File | Purpose |
|------|---------|
| `drawings/*.excalidraw` | Excalidraw scene (elements, appState) |
| `drawings/*.meta.json` | Metadata (name, timestamps, elementCount) |
| `hooks-queue.json` | Queue for Collaborate/Finish button clicks |
| `listen-state-{id}.json` | Tracks active listen state per drawing |

## Data Flows

### 1. Opening a Canvas

```
User: "Open a new canvas called Architecture"
    │
    ▼
Claude calls open_canvas({ name: "Architecture" })
    │
    ▼
MCP Server:
    ├── DrawingStorage.createDrawing()
    ├── Save to ~/.local/share/collaborative-canvas/drawings/
    └── spawn Electron app with drawing ID
    │
    ▼
Electron opens with drawing ID in URL
    │
    ▼
PostToolUse hook: open-canvas-choice.js
    │
    ▼
Returns additionalContext: "Call listen or save_canvas"
    │
    ▼
Claude calls listen({ drawingId: "..." })
    │
    ▼
PostToolUse hook: collaboration-poll.js starts polling
```

### 2. Collaboration Flow

```
User draws in Excalidraw, clicks "Collaborate"
    │
    ▼
Electron IPC: writeToHooksQueue()
    ├── Acquires file lock
    ├── Writes to ~/.local/share/collaborative-canvas/hooks-queue.json
    └── { drawingId, elementCount, timestamp, type: 'collaborate' }
    │
    ▼
MCP listen tool (polling every 3 seconds internally)
    ├── Detects request in queue
    ├── Removes request from queue
    └── Returns tool result with next-step instructions
    │
    ▼
Claude calls get_canvas_state({ drawingId: "..." })
    │
    ▼
Returns compact JSON of elements (~4.9x compression)
    │
    ▼
Claude analyzes diagram, optionally calls save_canvas to add elements
    │
    ▼
Claude calls listen() again to wait for next interaction
```

### 3. Element Update Flow (Claude → Widget)

```
Claude calls save_canvas({ drawingId, elements: [...] })
    │
    ▼
MCP Server:
    ├── Normalizes elements (auto-generates IDs)
    ├── Sets customData.createdBy = 'claude'
    └── DrawingStorage.updateDrawing()
    │
    ▼
Writes to ~/.local/share/collaborative-canvas/drawings/{id}.excalidraw
    │
    ▼
Electron polling (every 2 seconds)
    ├── Compares file modification time
    ├── If newer: IPC load-drawing
    └── React state update
    │
    ▼
Excalidraw re-renders with Claude's additions
```

### 4. Finish Flow

```
User clicks "Finish" button
    │
    ▼
Electron IPC: writeToHooksQueue({ type: 'finished' })
    │
    ▼
collaboration-poll.js detects { type: 'finished' }
    │
    ▼
Returns additionalContext: "Call close_widget..."
    │
    ▼
Claude calls close_widget({ drawingId: "..." })
    │
    ▼
MCP Server writes close-signal-{id}.json
    │
    ▼
Electron polling detects close signal
    │
    ▼
Window closes gracefully
```

## MCP Tools

| Tool | Purpose |
|------|---------|
| `open_canvas` | Create/open drawing, launch Electron |
| `save_canvas` | Save elements with optional Mermaid |
| `get_canvas_state` | Get compact JSON of elements |
| `add_to_canvas` | Add elements (supports compact format) |
| `update_canvas` | Update metadata (name, tags) |
| `list_canvases` | List all drawings with filtering |
| `delete_canvases` | Two-step delete with preview |
| `listen` | **Blocking** - polls hooks-queue.json until Collaborate/Finish |
| `close_widget` | Close Electron window |
| `capture_screenshot` | Capture canvas as PNG |
| `export_canvas` | Export (placeholder for Phase 3) |

## MCP Resources

| Resource | Description |
|----------|-------------|
| `drawing://list` | List all drawings with metadata |
| `drawing://{id}` | Get specific drawing state |
| `drawing://active` | Get current active drawing |

## Hook Configuration

### hooks/hooks.json

```json
{
  "hooks": {
    "PostToolUse": [
      { "matcher": "mcp__plugin_collaborative-canvas_canvas__open_canvas", "hooks": [...] },
      { "matcher": "mcp__plugin_collaborative-canvas_canvas__save_canvas", "hooks": [...] },
      { "matcher": "mcp__plugin_collaborative-canvas_canvas__get_canvas_state", "hooks": [...] }
    ]
  }
}
```

### Hook Scripts

| Script | Trigger | Timeout | Purpose |
|--------|---------|---------|---------|
| `open-canvas-choice.js` | After open_canvas | 10s | Force listen/save decision |
| `save-canvas-listen.js` | After save_canvas | 10s | Auto-trigger listen |
| `get-canvas-state-decision.js` | After get_canvas_state | 10s | Force close/save decision |

> **Note**: Collaboration polling is handled by the MCP `listen` tool itself (not a hook).
> PostToolUse hooks are non-blocking in Claude Code — they run briefly and return.
> Long-running blocking must happen inside the MCP tool call.

## Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Storage path | `~/.local/share/collaborative-canvas/` | XDG compliance, Electron standard |
| MCP server name | `canvas` | Plugin tool prefix: `mcp__plugin_collaborative-canvas_canvas__*` |
| Direct hook queue | Electron writes to hooks-queue.json | Avoids race conditions with multiple MCP instances |
| Compact JSON | ~4.9x compression | Prevents token limit issues |
| Listen timeout | 60 minutes | Long collaboration sessions |
| Listen blocking | MCP tool polls internally | PostToolUse hooks are non-blocking; MCP tool calls block |
| File locking | proper-lockfile | Atomic queue operations |
| Distribution | Pre-bundled MCP + GitHub Release Electron | Zero-build install, auto-downloads .app on first use |
| MCP bundling | esbuild → single CJS file | No node_modules needed at runtime (828KB) |

## Development

### Building

```bash
# Rebuild MCP bundle (after changing server code)
cd mcp-server && npm install && npm run bundle

# Rebuild Electron dist (after changing app code)
cd electron-app && npm install && npm run build

# Package Electron for release
cd electron-app && npm run package:dir
```

### Development Mode

```bash
# Enable hot reload
export EXCALIDRAW_DEV=1

# Start Vite dev server
cd electron-app && npm run dev:renderer

# Then use Claude Code normally
claude
```

### Testing

```bash
# After plugin is installed
/canvas Test Diagram
```

## Troubleshooting

### Electron not opening

1. Check packaged app exists: `ls electron-app/release/mac/`
2. Verify CANVAS_PLUGIN_ROOT is set in MCP env
3. Check MCP server logs for spawn errors

### Buttons not working

1. Verify `listen` was called (creates listen-state file)
2. Check hooks-queue.json for pending requests
3. Look for stale requests (>60 seconds old)

### Elements not appearing

1. Check drawing file exists in storage
2. Verify element IDs are unique
3. Check for sanitization filtering in Electron

## Version History

- **1.0.0** - Initial plugin release
  - Full MCP server with 11 tools
  - Electron desktop app
  - PostToolUse hooks for collaboration
  - XDG-compliant storage
  - Pre-bundled MCP server (esbuild, zero-dependency runtime)
  - GitHub Releases for Electron app distribution
  - Auto-download on first use

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anthosx)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/anthosx)
<!-- tomevault:4.0:windsurf_rules:2026-04-08 -->
