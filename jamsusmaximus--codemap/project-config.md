---
trigger: always_on
description: Real-time pixel-art visualization of Claude Code and Cursor agents. Shows agents as characters moving around a hotel, working at desks that represent files.
---

# CodeMap Hotel

Real-time pixel-art visualization of Claude Code and Cursor agents. Shows agents as characters moving around a hotel, working at desks that represent files.

## Agent Quick Start

**Working on hooks?** → Edit `hooks/thinking-hook.sh` or `hooks/file-activity-hook.sh`
**Working on visualization?** → Edit `client/src/drawing/agent.ts` (sprites) or `client/src/components/HabboRoom.tsx` (main loop)
**Working on server?** → Edit `server/src/index.ts` (endpoints) or `server/src/types.ts` (data structures)

Key files to understand first:
1. `server/src/types.ts` - All shared interfaces
2. `client/src/drawing/types.ts` - Client-side types
3. `hooks/thinking-hook.sh` - How hook data flows in

Run tests: `cd server && npm test` and `cd client && npm test`

## Quick Reference

| Resource | URL |
|----------|-----|
| Server | `http://localhost:5174` |
| Client | `http://localhost:5173/hotel` |
| Start both | `npm run dev` |
| Run tests | `npm test` (from client/ or server/) |
| Hook logs | `tail -f /tmp/codemap-hook.log` |

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        DATA FLOW                                 │
├─────────────────────────────────────────────────────────────────┤
│                                                                  │
│  Claude Code / Cursor                                            │
│       │                                                          │
│       ▼                                                          │
│  ┌─────────────────┐                                             │
│  │  hooks/ (bash)  │  file-activity-hook.sh, thinking-hook.sh   │
│  └────────┬────────┘                                             │
│           │ POST /api/activity, /api/thinking                    │
│           ▼                                                      │
│  ┌─────────────────┐                                             │
│  │ server/ (node)  │  Express + WebSocket on port 5174          │
│  └────────┬────────┘                                             │
│           │ WebSocket broadcast                                  │
│           ▼                                                      │
│  ┌─────────────────┐                                             │
│  │ client/ (react) │  Canvas visualization on port 5173         │
│  └─────────────────┘                                             │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
codemap/
├── bin/
│   └── setup.js           # Universal setup script for any project
├── hooks/
│   ├── file-activity-hook.sh   # Tracks Read/Write/Edit operations
│   └── thinking-hook.sh        # Tracks agent thinking state
├── server/
│   └── src/
│       ├── index.ts            # Express server + API endpoints
│       ├── activity-store.ts   # File tree and activity tracking
│       ├── git-activity.ts     # Git-based folder ranking
│       ├── websocket.ts        # WebSocket client management
│       ├── types.ts            # Shared TypeScript interfaces
│       └── *.test.ts           # Test files (126 tests)
├── client/
│   └── src/
│       ├── components/
│       │   └── HabboRoom.tsx   # Main visualization (1200+ lines)
│       ├── drawing/            # Pixel art rendering modules
│       │   ├── agent.ts        # Character sprites
│       │   ├── furniture.ts    # Desks, monitors
│       │   ├── decorations.ts  # Themed room items
│       │   └── ...
│       ├── hooks/
│       │   └── useFileActivity.ts  # WebSocket connection
│       ├── utils/
│       │   ├── agent-movement.ts   # Movement calculations
│       │   ├── screen-flash.ts     # Activity indicators
│       │   └── *.test.ts           # Test files (122 tests)
│       └── layout/
│           └── multi-floor.ts      # Floor layout algorithm
├── CLAUDE.md              # This file (agent reference)
├── README.md              # User documentation
└── package.json           # Workspace root
```

## Server API Reference

### Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/api/activity` | Receive file read/write events |
| POST | `/api/thinking` | Receive agent thinking state |
| GET | `/api/thinking` | Return all agent states (JSON) |
| GET | `/api/graph` | Return file tree data |
| GET | `/api/hot-folders` | Git-ranked folders |
| GET | `/api/health` | Health check |
| GET | `/api/debug` | Debug info (uptime, clients, activity) |

### WebSocket

- Path: `ws://localhost:5174/ws`
- Messages: `{ type: 'activity' | 'graph' | 'thinking', data: ... }`

### Event Types

```typescript
// Activity events (from file-activity-hook.sh)
type ActivityEvent = {
  type: 'read-start' | 'read-end' | 'write-start' | 'write-end' | 'search-start' | 'search-end';
  filePath: string;      // Relative to project root
  agentId?: string;      // UUID of the agent
  source?: 'claude' | 'cursor';
  timestamp: number;
};

// Thinking events (from thinking-hook.sh)
type ThinkingEvent = {
  type: 'thinking-start' | 'thinking-end';
  agentId: string;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JamsusMaximus/codemap](https://github.com/JamsusMaximus/codemap) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
