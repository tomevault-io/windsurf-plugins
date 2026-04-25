---
trigger: always_on
description: Building a **thin Neovim client** for the Continue CLI (`cn serve`).
---

# Project Context: Continue Neovim Client

## What We're Doing

Building a **thin Neovim client** for the Continue CLI (`cn serve`).

**Approach**: HTTP client → `cn serve` backend (NOT porting TypeScript to Lua)
**Plugin Name**: continue.nvim - AI code assistant for Neovim  
**Status**: Phase 0 - Architecture Design

## Architecture Overview

```
┌─────────────────────────────────┐
│         Neovim                  │
│  ┌──────────────────────────┐   │
│  │  continue.nvim           │   │
│  │  (Lua - ~5-10K tokens)   │   │
│  │                          │   │
│  │  • Process manager       │   │  HTTP/JSON
│  │  • HTTP client           │───┼────────┐
│  │  • UI (buffers/windows)  │   │        │
│  │  • Commands & keymaps    │   │        │
│  └──────────────────────────┘   │        │
│           │                      │        ▼
│  ┌────────▼──────────┐           │  ┌─────────────────┐
│  │  Snacks Terminal  │───spawn───┼─▶│   cn serve      │
│  └───────────────────┘           │  │  (Node.js/TS)   │
└─────────────────────────────────┘  │                 │
                                     │  • Agent logic  │
                                     │  • LLM APIs     │
                                     │  • Tools/MCP    │
                                     │  • All Continue │
                                     │    features     │
                                     └─────────────────┘
```

## Quick Facts

- **Decision**: Use `cn serve` backend instead of porting 40-70K tokens
- **Savings**: 90% less code - reuse ALL Continue's logic
- **Plugin Name**: continue.nvim
- **Main Features**: Agent, Chat, Edit, Autocomplete (via Continue CLI)
- **Estimated effort**: 5-10K tokens (just the client wrapper)
- **Backend**: `@continuedev/cli` (installed via npm)
- **Protocol**: HTTP REST API (documented in source/extensions/cli/spec/wire-format.md)
- **Key components**:
  - Process lifecycle (spawn/stop `cn serve`)
  - HTTP polling client (state updates)
  - Buffer/window UI (display chat, tools)
  - Neovim command integration

## File Structure

```
repo/
├── source/                        # continuedev/continue submodule (reference)
│   └── extensions/cli/            # cn serve implementation (Node.js)
├── lua/
│   └── continue/
│       ├── init.lua               # Entry point & setup()
│       ├── config.lua             # Config schema (paths, ports, etc.)
│       ├── process.lua            # cn serve lifecycle management
│       ├── client.lua             # HTTP client (GET /state, POST /message)
│       ├── ui/
│       │   ├── chat.lua           # Chat buffer UI
│       │   ├── floating.lua       # Floating windows
│       │   └── render.lua         # Message rendering
│       ├── commands.lua           # Neovim commands
│       └── utils/
│           ├── http.lua           # HTTP helpers (vim.loop/curl)
│           └── json.lua           # JSON encode/decode
├── plugin/
│   └── continue.lua               # Auto-load commands
├── doc/
│   └── continue.txt               # Vim help docs
├── docs/                          # Development documentation
│   ├── PROJECT_KNOWLEDGE.md       # Architecture, HTTP protocol
│   ├── API_MAPPING.md             # HTTP API → Neovim integration
│   ├── QUICK_REFERENCE.md         # Common patterns
│   └── CHECKLIST.md               # Progress tracker
└── CLAUDE.md                      # This file
```

## HTTP Protocol Reference

From `source/extensions/cli/spec/wire-format.md`:

### Endpoints

- `GET /state` - Get current chat state (poll every 500ms)
  - Returns: `{ chatHistory, isProcessing, messageQueueLength, pendingPermission }`
- `POST /message` - Send user message
  - Body: `{ message: string }`
  - Returns: `{ queued, position }`
- `POST /permission` - Approve/reject tool permission
  - Body: `{ requestId, approved }`
- `POST /pause` - Interrupt current processing
- `GET /diff` - Get git diff from working tree
- `POST /exit` - Graceful shutdown

### Message Types

```json
{
  "role": "user" | "assistant" | "system",
  "content": "string",
  "isStreaming": boolean,
  "messageType": "tool-start" | "tool-result" | "tool-error" | "system",
  "toolName": "string",
  "toolResult": "string"
}
```

## When You're Asked To...

### Implement HTTP client
1. Check `docs/API_MAPPING.md` for endpoint patterns
2. Use `vim.loop` for HTTP requests (or curl via `vim.fn.system`)
3. Implement state polling with `vim.loop.new_timer()`
4. Handle JSON encode/decode with `vim.json` (Neovim 0.10+) or vendored lib

### Build UI components
1. Check `docs/QUICK_REFERENCE.md` for floating window patterns
2. Render chat messages in a buffer
3. Handle streaming updates (character-by-character)
4. Display tool calls and results

### Manage cn serve process
1. Use `vim.fn.jobstart()` to spawn `cn serve`
2. Track PID for graceful shutdown
3. Health check via `GET /state`
4. Auto-restart on crash (optional)

### Debug
1. Check `docs/PROJECT_KNOWLEDGE.md` > Debugging
2. Test HTTP endpoints with `curl` first
3. Use `:messages` to see errors
4. Log to `/tmp/continue-nvim.log`

### Test changes
```vim
" Hot reload without restarting Neovim:
:lua package.loaded['continue'] = nil
:lua require('continue').setup()

" Check process status:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Megatherium) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
