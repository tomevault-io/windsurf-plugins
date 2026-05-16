---
trigger: always_on
description: **pi.nvim** is a Neovim plugin that provides a native UI for the Pi coding agent. It uses Pi's RPC protocol for communication and provides real-time visualization of agent operations, file changes, and conversation history.
---

# AGENTS.md - Pi.nvim Project Guide

## Project Overview

**pi.nvim** is a Neovim plugin that provides a native UI for the Pi coding agent. It uses Pi's RPC protocol for communication and provides real-time visualization of agent operations, file changes, and conversation history.

### Key Features
- Native RPC communication with Pi agent via TCP/JSON-RPC
- Real-time status panel and log streaming
- Side-by-side diff viewer for proposed changes
- File change approval/rejection workflow
- Session persistence and management
- Chat interface with the agent

---

## Working with This Project

### Project Structure

```
pi.nvim/
├── lua/pi/
│   ├── init.lua              # Main entry point
│   ├── state.lua             # Global state management
│   ├── events.lua            # Event system
│   ├── config.lua            # Configuration
│   ├── rpc/                  # RPC client and wrappers
│   │   ├── client.lua        # Core RPC client
│   │   ├── agent.lua         # Agent control methods
│   │   ├── files.lua         # File operations
│   │   ├── logs.lua          # Log streaming
│   │   ├── conversation.lua  # Chat interface
│   │   └── session.lua       # Session management
│   ├── ui/                   # UI components
│   │   ├── control_panel.lua # Status/control UI
│   │   ├── diff_viewer.lua   # Side-by-side diff
│   │   ├── logs_viewer.lua   # Structured log view
│   │   ├── chat.lua          # Conversation UI
│   │   ├── file_tree.lua     # File modification tree
│   │   └── approval.lua      # Approve/reject UI
│   └── watcher.lua           # File change monitoring
├── plugin/pi.lua             # Vim commands
└── doc/pi.txt                # Help documentation
```

---

## Task Tracking with PLAN.md

The `PLAN.md` file contains the complete implementation plan with **checkboxes** for tracking progress. **Always update these checklists as you work.**

### How to Use the Checklists

Each phase in PLAN.md has testing checklists like this:

```markdown
**Testing Checklist:**
- [ ] Can connect to Pi RPC server
- [ ] Handles connection failures gracefully
- [ ] Can send requests and receive responses
```

#### When Starting a Task:
1. **Read the relevant section** in PLAN.md
2. **Check if prerequisites are complete** - previous phase checkboxes should be checked
3. **Create/update the corresponding file(s)** in the lua/ directory
4. **Mark tasks in progress** by leaving them unchecked but noting progress

#### When Completing a Task:
1. **Test the implementation** against the checklist items
2. **Update PLAN.md** - change `[ ]` to `[x]` for completed items
3. **Commit your work** with a descriptive message
4. **Move to the next task** in the phase

#### Example Workflow:

```bash
# 1. Check current status in PLAN.md
# Look for unchecked items in the current phase

# 2. Implement the file
nvim lua/pi/rpc/client.lua
# ... write code based on PLAN.md examples ...

# 3. Test manually in Neovim
:lua require("pi.rpc.client").new({port=43863})

# 4. Update PLAN.md - mark checklist items complete
nvim PLAN.md
# Change: - [ ] Can connect to Pi RPC server
# To:     - [x] Can connect to Pi RPC server

# 5. Commit
git add lua/pi/rpc/client.lua PLAN.md
git commit -m "feat(rpc): implement client connection handling

- TCP socket connection to Pi RPC server
- JSON message parsing with buffering
- Request/response correlation with id tracking
- Event emission for server notifications

From PLAN.md Phase 1.1"
```

---

## Implementation Phases

Work through these phases **in order**. Each phase builds on the previous.

### Phase 1: Foundation (Week 1)
**Files to create:**
- `lua/pi/rpc/client.lua` - RPC client implementation
- `lua/pi/events.lua` - Event system
- `lua/pi/state.lua` - State management

**Key checklists in PLAN.md:**
- 1.1 Basic RPC Client testing checklist
- 1.2 Event System events list
- 1.3 State Management testing checklist

**Success criteria:**
- [ ] All three foundation files created and working
- [ ] Can connect/disconnect from Pi RPC
- [ ] Events propagate correctly
- [ ] State updates trigger UI notifications

---

### Phase 2: RPC Wrappers (Week 1-2)
**Files to create:**
- `lua/pi/rpc/agent.lua` - Agent control (start/pause/resume/stop)
- `lua/pi/rpc/files.lua` - File operations (read/write/list/stat)
- `lua/pi/rpc/logs.lua` - Log streaming
- `lua/pi/rpc/session.lua` - Session management
- `lua/pi/rpc/conversation.lua` - Chat interface

**Key checklists in PLAN.md:**
- Phase 2 testing checklist (after all wrapper sections)

**Success criteria:**
- [ ] All RPC wrappers implemented
- [ ] Each wrapper updates state correctly
- [ ] Errors are handled gracefully
- [ ] Events emitted for state changes

---

### Phase 3: Core UI Components (Week 2-3)
**Files to create:**
- `lua/pi/ui/control_panel.lua` - Floating status window
- `lua/pi/ui/diff_viewer.lua` - Side-by-side diff
- `lua/pi/ui/logs_viewer.lua` - Bottom panel for logs

**Key checklists in PLAN.md:**
- 3.1 Control Panel testing checklist
- (Diff viewer and logs viewer sections)

**Success criteria:**
- [ ] Control panel shows real-time status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0xburden/pi.nvim](https://github.com/0xburden/pi.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-16 -->
