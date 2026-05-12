---
trigger: always_on
description: A DankMaterialShell (DMS) launcher plugin for executing shell commands directly from the launcher with history tracking and preset shortcuts.
---

# AGENTS.md - DMS Command Runner

## Project Overview
A DankMaterialShell (DMS) launcher plugin for executing shell commands directly from the launcher with history tracking and preset shortcuts.

**Language**: QML (Qt Modeling Language)
**Type**: Launcher plugin for DankMaterialShell
**Default Trigger**: `>`
**Version**: 1.1.2

## Recent Maintenance Notes (2026-02-18)
- The `Always Active`/`noTrigger` setting has been removed from settings UI.
- Trigger configuration is always visible and trigger-based usage is now the expected path.
- Legacy empty trigger values are normalized to `>` in settings initialization.

## Architecture Overview

```
┌─────────────────────────────────────────────────────┐
│  User Input                                          │
│  "> htop"                                           │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  Command Processing                                  │
│  - Parse command                                     │
│  - Check history for suggestions                    │
│  - Generate action items:                           │
│    • Run in terminal                                │
│    • Run in background                              │
│    • Copy to clipboard                              │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  Execution Methods                                   │
│  ┌──────────────┐  ┌──────────────┐  ┌───────────┐ │
│  │   Terminal   │  │  Background  │  │ Clipboard │ │
│  │              │  │              │  │           │ │
│  │ terminal -e  │  │  sh -c cmd   │  │ wl-copy   │ │
│  │  + wrapper   │  │   (detached) │  │           │ │
│  └──────────────┘  └──────────────┘  └───────────┘ │
└─────────────────────────────────────────────────────┘
                      ↓
┌─────────────────────────────────────────────────────┐
│  History Tracking                                    │
│  - Add to history (most recent first)               │
│  - Limit to maxHistoryItems                         │
│  - Persist to DMS settings                          │
└─────────────────────────────────────────────────────┘
```

## File Structure

### Core Files
- **plugin.json** - Plugin metadata, version, trigger, capabilities
- **CommandRunner.qml** - Main component (~165 lines)
  - Command parsing and execution
  - History management
  - Terminal/background/clipboard actions
- **CommandRunnerSettings.qml** - Settings UI (~200 lines)
  - Terminal configuration
  - History management
  - Trigger configuration

## Key Concepts

### Execution Modes

#### 1. Terminal Execution (Default)
Runs command in user's configured terminal with a wrapper:
```javascript
const wrappedCommand = command + "; echo '\nPress Enter to close...'; read";
Quickshell.execDetached([terminal.cmd, terminal.execFlag, "sh", "-c", wrappedCommand]);
```

**Wrapper purpose**:
- Keeps terminal open after command completes
- Prevents terminal from closing immediately
- Allows user to review output

#### 2. Background Execution
Runs command silently without opening terminal:
```javascript
Quickshell.execDetached(["sh", "-c", command]);
```

**Use cases**: Fire-and-forget commands, system services, background tasks

#### 3. Clipboard Copy
Copies command to clipboard using wl-copy:
```javascript
Quickshell.execDetached(["sh", "-c", "echo -n '" + text + "' | wl-copy"]);
```

### History Management

**Storage**: DMS plugin settings (`commandRunner.history`)
**Max items**: Configurable (1-100, default 20)
**Order**: Most recent first (LIFO)

**Deduplication**: When command is re-executed, it's moved to top:
```javascript
function addToHistory(command) {
    const index = commandHistory.indexOf(command);
    if (index > -1) {
        commandHistory.splice(index, 1);  // Remove old entry
    }
    commandHistory.unshift(command);       // Add to front
    // Trim to max size
    if (commandHistory.length > maxHistoryItems) {
        commandHistory = commandHistory.slice(0, maxHistoryItems);
    }
}
```

### Terminal Configuration

Users must configure their terminal before first use:

**Common configurations**:
- `kitty` with `-e` flag
- `alacritty` with `-e` flag
- `foot` with `-e` flag
- `wezterm` with `start` flag
- `gnome-terminal` with `--` flag

**Storage**:
- `commandRunner.terminal` - Terminal command (e.g., "kitty")
- `commandRunner.execFlag` - Execution flag (e.g., "-e")

## Development Workflow

### 1. Modifying Command Execution

**Location**: `CommandRunner.qml` lines 102-114

**Terminal execution** (runCommand):
- Wraps command with pause/read
- Launches in configured terminal
- Adds to history

**Background execution** (runBackground):
- No terminal, direct shell execution
- Adds to history

**Security consideration**: Commands are passed to `sh -c`, which can execute arbitrary code. This is intentional but means:
- No validation/sanitization
- Full shell features available (pipes, redirects, etc.)
- User responsibility for command safety

### 2. Adding Common Command Shortcuts


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devnullvoid/dms-command-runner](https://github.com/devnullvoid/dms-command-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
