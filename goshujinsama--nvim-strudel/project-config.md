---
trigger: always_on
description: **nvim-strudel** is a Neovim plugin that brings the [Strudel](https://strudel.cc/) live coding music environment to Neovim. It provides real-time visualization of active pattern elements using highlight groups and conceal characters, mirroring the web UI's visual feedback during playback.
---

# AGENTS.md - nvim-strudel

## Project Overview

**nvim-strudel** is a Neovim plugin that brings the [Strudel](https://strudel.cc/) live coding music environment to Neovim. It provides real-time visualization of active pattern elements using highlight groups and conceal characters, mirroring the web UI's visual feedback during playback.

### Goals
- Live code music patterns directly in Neovim
- Real-time visual feedback showing which code elements are currently producing sound
- Full playback control (start, pause, stop)
- Seamless integration with Neovim's editing experience

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                         Neovim                                  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │                    nvim-strudel (Lua)                      │  │
│  │  - Buffer management                                      │  │
│  │  - Highlight groups for active elements                   │  │
│  │  - Conceal characters for visualization                   │  │
│  │  - User commands (:StrudelPlay, :StrudelPause, :StrudelStop)│
│  │  - RPC/WebSocket client                                   │  │
│  └──────────────────────────┬────────────────────────────────┘  │
└─────────────────────────────┼───────────────────────────────────┘
                              │ WebSocket / JSON-RPC
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│                    strudel-server (Node.js)                     │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  - @strudel/core pattern evaluation                       │  │
│  │  - @strudel/webaudio for audio synthesis                  │  │
│  │  - Pattern event tracking (which elements are active)     │  │
│  │  - WebSocket server for Neovim communication              │  │
│  │  - Source map tracking (code position → sound events)     │  │
│  │  - LSP server for mini-notation (completions, hover, diag)│  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

## Directory Structure

```
nvim-strudel/
├── AGENTS.md                 # This file
├── README.md                 # User documentation
├── LICENSE                   # AGPL-3.0 (matching Strudel)
│
├── lua/
│   └── strudel/
│       ├── init.lua          # Plugin entry point, setup()
│       ├── config.lua        # Configuration defaults and validation
│       ├── client.lua        # WebSocket/RPC client to backend
│       ├── commands.lua      # User commands registration
│       ├── highlights.lua    # Highlight group definitions
│       ├── visualizer.lua    # Real-time highlight/conceal updates
│       ├── lsp.lua           # LSP client setup for mini-notation
│       ├── picker.lua        # Picker abstraction (Snacks/Telescope)
│       └── utils.lua         # Shared utilities
│
├── plugin/
│   └── strudel.vim           # Auto-load plugin registration
│
├── server/
│   ├── package.json          # Node.js dependencies
│   ├── tsconfig.json         # TypeScript configuration
│   └── src/
│       ├── index.ts          # Server entry point
│       ├── strudel-engine.ts # Strudel pattern evaluation wrapper
│       ├── websocket.ts      # WebSocket server for Neovim
│       ├── lsp.ts            # LSP server for mini-notation
│       ├── source-map.ts     # Track code positions to events
│       └── types.ts          # Shared TypeScript types
│
├── samples/                  # Example Strudel patterns
│   └── demo.strudel
│
└── tests/
    ├── lua/                  # Lua plugin tests (plenary.nvim)
    └── server/               # Node.js server tests (vitest)
```

## Key Components

### Neovim Plugin (Lua)

#### `lua/strudel/init.lua`
- Main entry point with `setup(opts)` function
- Lazy-loads other modules on demand
- Manages plugin lifecycle

#### `lua/strudel/client.lua`
- WebSocket client using `vim.loop` (libuv bindings)
- Handles connection, reconnection, and message parsing
- Sends code updates to server
- Receives active element events from server

#### `lua/strudel/visualizer.lua`
- Creates and manages extmarks for highlighting
- Uses `nvim_buf_set_extmark` with `hl_group` for active elements
- Optionally uses conceal to show playhead/activity indicators
- Updates at ~60fps or on server events

#### `lua/strudel/highlights.lua`
Defines highlight groups:
- `StrudelActive` - Currently sounding element
- `StrudelPending` - Element about to sound
- `StrudelMuted` - Muted/inactive element
- `StrudelPlayhead` - Current cycle position indicator

#### `lua/strudel/commands.lua`
User commands:
- `:StrudelPlay` - Start/resume playback
- `:StrudelPause` - Pause playback
- `:StrudelStop` - Stop and reset
- `:StrudelEval` - Evaluate current buffer/selection
- `:StrudelConnect` - Connect to server
- `:StrudelStatus` - Show connection/playback status
- `:StrudelSamples` - Browse available samples via picker

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Goshujinsama/nvim-strudel](https://github.com/Goshujinsama/nvim-strudel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
