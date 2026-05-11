---
trigger: always_on
description: **⚠️ macOS Primary, Windows Secondary** - This project primarily supports macOS with full auto-trigger functionality. Windows support is partially implemented but may require additional configuration.
---

# FL Studio Direct Integration - Documentation

## Platform Support

**⚠️ macOS Primary, Windows Secondary** - This project primarily supports macOS with full auto-trigger functionality. Windows support is partially implemented but may require additional configuration.

## Overview

This system allows Claude to interact with FL Studio's piano roll through direct file-based communication. It provides seamless integration between Claude's AI capabilities and FL Studio's music production environment with built-in timing delays to ensure reliable operation.

## Installation and Setup

### 1. Install MCP Server for Claude
```bash
./install_mcp_for_claude.sh
```
This registers the MCP server with Claude Code. The server is named `fl-studio-mcp`.

### 2. Usage Workflow

**For each session:**
1. **Open FL Studio** and open a piano roll
2. **Run ComposeWithLLM** once (Tools → Scripting → ComposeWithLLM) to initialize
3. **Start Claude** - the trigger mechanism is built into the MCP server
4. **No need to run separate scripts** - everything is handled automatically!

**Note:** Claude Code needs Accessibility permissions (System Settings → Privacy & Security → Accessibility) to send keystrokes to FL Studio.

## Getting Started (For AI Assistants)

If you're an LLM helping the user with this system, here's what you need to know:

### 1. Check Available Tools

First, verify you have access to these MCP tools (they'll have the `mcp__fl_studio_mcp__` prefix):
- `mcp__fl_studio_mcp__get_piano_roll_state` - Read the piano roll
- `mcp__fl_studio_mcp__send_notes` - Add or replace notes (including chords)
- `mcp__fl_studio_mcp__delete_notes` - Remove specific notes
- `mcp__fl_studio_mcp__clear_piano_roll` - Clear all notes
- `mcp__fl_studio_mcp__clear_queue` - Clear pending requests

If these aren't available, the MCP server needs to be started or reconnected. The server is registered as `fl-studio-mcp` in Claude Code.

### 2. Auto-Trigger System

The trigger mechanism is now **built into the MCP server**:
- When you send notes/requests, the MCP server automatically triggers FL Studio
- A 2-second delay ensures FL Studio has time to receive focus and process the request
- No separate auto-trigger script needed
- Works with both detached and docked piano roll windows

### 3. Always Start With State

Before making ANY changes, get the current piano roll state:

```python
state = mcp__fl_studio_mcp__get_piano_roll_state()
```

This tells you:
- What notes already exist
- The PPQ (timing resolution)
- What you're working with

### 4. Key Concepts for LLMs

**Built-in Auto-Trigger System:**
- MCP server writes requests to `mcp_request.json`
- MCP server automatically sends Cmd+Opt+Y (macOS) or Ctrl+Alt+Y (Windows)
- 2-second delay ensures proper focus handling
- FL Studio re-runs the last script (ComposeWithLLM)
- Notes appear automatically after the delay
- Cross-platform trigger support with focus management

**Time is Always in Quarter Notes:**
- `time=0` = beat 1
- `time=4` = beat 5 (measure 2 in 4/4)
- `duration=1` = quarter note
- `duration=4` = whole note
- Never use ticks - the bridge converts automatically

**Chords Are Just Multiple Notes:**
- A chord = multiple notes with the same `time` value
- Send them all in one `send_notes` call with matching times
- For example: C major = MIDI 60, 64, 67 all at same time

### 5. Common Interaction Patterns

**Pattern: Add a chord progression**
```python
# User: "Add a I-IV-V progression in C"

# Start by getting state (good practice)
state = mcp__fl_studio_mcp__get_piano_roll_state()

# Send each chord
mcp__fl_studio_mcp__send_notes([
    {"midi": 60, "duration": 4, "time": 0},
    {"midi": 64, "duration": 4, "time": 0},
    {"midi": 67, "duration": 4, "time": 0}
], mode="add")  # C major

mcp__fl_studio_mcp__send_notes([
    {"midi": 65, "duration": 4, "time": 4},
    {"midi": 69, "duration": 4, "time": 4},
    {"midi": 72, "duration": 4, "time": 4}
], mode="add")  # F major

mcp__fl_studio_mcp__send_notes([
    {"midi": 67, "duration": 4, "time": 8},
    {"midi": 71, "duration": 4, "time": 8},
    {"midi": 74, "duration": 4, "time": 8}
], mode="add")  # G major

# Notes will appear automatically!
```

**Pattern: Modify existing notes**
```python
# User: "Change that G note to an A"

# Get state to see what's there
state = mcp__fl_studio_mcp__get_piano_roll_state()

# Find the G note in the state (look for number: 67)
# Delete it
mcp__fl_studio_mcp__delete_notes([{"midi": 67, "time": 0}])

# Add replacement
mcp__fl_studio_mcp__send_notes([{"midi": 69, "duration": 4, "time": 0}])

# Changes appear automatically!
```

**Pattern: Start fresh**
```python
# User: "Clear everything and give me a C major scale"

# Use replace mode to clear and add
mcp__fl_studio_mcp__send_notes([
    {"midi": 60, "duration": 0.5, "time": 0},   # C
    {"midi": 62, "duration": 0.5, "time": 0.5}, # D
    {"midi": 64, "duration": 0.5, "time": 1},   # E
    {"midi": 65, "duration": 0.5, "time": 1.5}, # F

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [calvinw/fl-studio-mcp](https://github.com/calvinw/fl-studio-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
