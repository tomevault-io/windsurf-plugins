---
trigger: always_on
description: Manages subagent visualizations:
---

# Vibecraft - Technical Documentation

This document explains the Vibecraft codebase for future AI assistants working on this project.

## Project Purpose

Vibecraft visualizes Claude Code's activity in real-time as a 3D workshop. When Claude uses tools (Read, Edit, Bash, etc.), a character moves to corresponding workstations in a Three.js scene. The user can also send prompts to Claude from the browser via tmux integration.

## Architecture Overview

```
Claude Code → Hook Script → WebSocket Server → Browser (Three.js)
                  ↓
            events.jsonl (persistent log)
```

### Data Flow

1. **Claude Code** executes tools (Read, Edit, Bash, etc.)
2. **Hook script** (`hooks/vibecraft-hook.sh`) receives JSON via stdin from Claude Code's hook system
3. Hook writes to `~/.vibecraft/data/events.jsonl` AND POSTs to `http://localhost:4003/event`
4. **WebSocket server** (`server/index.ts`) broadcasts events to connected clients
5. **Browser** (`src/main.ts`) receives events and moves the Claude character

**Important:** Both hook and server use `~/.vibecraft/data/` as the data directory. This ensures they share the same files regardless of how vibecraft was installed (npx, global npm, or local dev).

### EventBus Architecture

Events are handled via a decoupled EventBus pattern, separating concerns into focused handler modules:

```
handleEvent(event)
    ↓
eventBus.emit(type, event, context)
    ↓
┌─────────────────────────────────────────────────────────┐
│  soundHandlers.ts      → Tool sounds, lifecycle sounds  │
│  notificationHandlers.ts → Zone floating text           │
│  characterHandlers.ts  → Movement, states               │
│  subagentHandlers.ts   → Task spawn/remove              │
│  zoneHandlers.ts       → Zone attention/status          │
│  feedHandlers.ts       → Thinking indicator             │
└─────────────────────────────────────────────────────────┘
    ↓
main.ts continues       → UI updates (DOM), special cases
```

**Files:**
- `src/events/EventBus.ts` - Core event dispatch with typed handlers
- `src/events/handlers/` - Handler modules (6 files)
- `src/events/handlers/index.ts` - Barrel export with `registerAllHandlers()`

**Adding a new event handler:**
```typescript
// In src/events/handlers/myHandlers.ts
import { eventBus } from '../EventBus'
import type { PreToolUseEvent } from '../../../shared/types'

export function registerMyHandlers(): void {
  eventBus.on('pre_tool_use', (event: PreToolUseEvent, ctx) => {
    if (!ctx.session) return
    // Handle the event...
  })
}

// Then add to src/events/handlers/index.ts:
import { registerMyHandlers } from './myHandlers'
export function registerAllHandlers(): void {
  // ... existing
  registerMyHandlers()
}
```

**EventContext** provides access to:
- `scene` - WorkshopScene for 3D updates
- `feedManager` - Activity feed
- `timelineManager` - Timeline strip
- `soundEnabled` - Sound toggle state
- `session` - Current session's claude, subagents, zone, stats

**Design principle:** EventBus handlers update 3D scene state. main.ts handles DOM UI updates (updateActivity, updateStats, etc.) and special cases (modals).

## Key Files

### `shared/types.ts`
Defines all TypeScript types used across server and client:

- `ClaudeEvent` - Union type of all event types (pre_tool_use, post_tool_use, stop, etc.)
- `TOOL_STATION_MAP` - Maps tool names to station names (Read→bookshelf, Bash→terminal)
- `StationType` - Valid station identifiers
- `ServerMessage` / `ClientMessage` - WebSocket protocol types

**Important**: When adding new tools, update `TOOL_STATION_MAP` to assign them to stations.

### `hooks/vibecraft-hook.sh`
Bash script that captures Claude Code events. The source lives in `hooks/vibecraft-hook.sh` but `npx vibecraft setup` copies it to `~/.vibecraft/hooks/vibecraft-hook.sh` (stable location).

**What it does:**
- Reads JSON from stdin (Claude Code pipes hook data)
- Transforms to our event format with `jq`
- Writes to `~/.vibecraft/data/events.jsonl` (append-only log)
- POSTs to server for real-time updates

**Cross-platform support:**
- Adds common tool paths to PATH (`/opt/homebrew/bin`, `/usr/local/bin`, etc.)
- Uses `find_tool()` function to locate `jq` and `curl` defensively
- Handles macOS timestamp differences (no `date +%N`)

**Known issue fixed**: Timestamp calculation used `$(date +%N)` which returns "087" etc. This was interpreted as octal. Fixed with `10#$ms_part` to force decimal.

**Compact JSON**: Must use `jq -n -c` (not just `jq -n`) to avoid multi-line output breaking JSONL format.

### Setup Process (`npx vibecraft setup`)

The setup command:
1. Copies `hooks/vibecraft-hook.sh` to `~/.vibecraft/hooks/vibecraft-hook.sh`
2. Creates `~/.vibecraft/data/` directory
3. Configures all 8 hooks in `~/.claude/settings.json`:
   - PreToolUse, PostToolUse, Stop, SubagentStop
   - SessionStart, SessionEnd, UserPromptSubmit, Notification
4. Backs up existing settings

**Why ~/.vibecraft/hooks/?** The hook path must be stable. If hooks pointed to the npm package location, they'd break when the package updates or npx cache clears.

### `server/index.ts`
Node.js WebSocket server:

- Watches `events.jsonl` with chokidar for file-based events
- Accepts POST `/event` for real-time hook notifications
- Broadcasts to WebSocket clients

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nearcyan/vibecraft](https://github.com/Nearcyan/vibecraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
