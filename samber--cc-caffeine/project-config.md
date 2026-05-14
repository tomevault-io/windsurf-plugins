---
trigger: always_on
description: Enables sleep prevention for the current session (lightweight, no system tray).
---

# CC-Caffeine: Claude Code Sleep Prevention System

A Node.js/Electron script that prevents your computer from going to sleep while using Claude Code through system tray integration and session management.

## Architecture

The system consists of a modular architecture with the following components:

### Core Modules

1. **caffeine.js** - Main entry point that orchestrates all modules and handles command routing
2. **src/commands.js** - Handles command-line interface functionality and process management
3. **src/session.js** - Manages session persistence with file locking and timeout handling
4. **src/server.js** - Handles server process management and Electron integration
5. **src/system-tray.js** - Manages system tray functionality and sleep prevention
6. **src/electron.js** - Wraps Electron-specific functionality and provides cross-platform support
7. **src/config.js** - Reads user configuration from `~/.claude/plugins/cc-caffeine/config.json`

### User Commands

1. **caffeinate command** - Adds session to JSON file and ensures server is running
2. **uncaffeinate command** - Removes session from JSON file
3. **server command** - Starts Electron system tray application that polls JSON file for active sessions
4. **version command** - Shows version information from package.json and plugin.json

## Features

- Cross-platform support (Linux, macOS, Windows)
- Headless Electron system tray (no windows, only system tray)
- JSON file for session persistence with proper-lockfile for concurrency
- Configurable session timeout (default: 15 minutes of inactivity)
- Auto-server startup when not running
- Multiple concurrent session support
- Real-time status monitoring
- Lightweight client commands (no Electron dependency for caffeinate/uncaffeinate)
- Native sleep prevention using Electron's powerSaveBlocker API
- Hidden from macOS dock using app.dock.hide()

## Configuration

User configuration is stored at `~/.claude/plugins/cc-caffeine/config.json`. All settings are optional and have sensible defaults.

```json
{
  "session_timeout_minutes": 15,
  "icon_theme": "orange"
}
```

### Options

| Setting | Default | Description |
|---------|---------|-------------|
| `session_timeout_minutes` | `15` | Minutes of inactivity before a session expires |
| `icon_theme` | `"orange"` | Tray icon theme: `"orange"` (colored) or `"monochrome"` (black/white, auto-adapts to macOS dark mode) |

## Technical Stack

- **Node.js 14+** - Runtime environment with modern JavaScript features
- **Electron 28+** - Cross-platform desktop application framework
- **proper-lockfile** - File locking for all concurrent access with retry logic
- **Electron powerSaveBlocker** - Native cross-platform sleep prevention
- **Electron Tray/Menu** - System tray functionality
- **JSON file** - Session storage and communication
- **setInterval** - Background polling for session changes

## Commands Usage

### caffeinate
Enables sleep prevention for the current session (lightweight, no system tray).
```bash
node caffeine.js caffeinate
# or
npm run caffeinate
```
Accepts JSON via stdin with session_id:
```json
{"session_id": "abc123"}
```

### uncaffeinate
Disables sleep prevention for the current session (lightweight, no system tray).
```bash
node caffeine.js uncaffeinate
# or
npm run uncaffeinate
```
Accepts JSON via stdin with session_id:
```json
{"session_id": "abc123"}
```

### server
Starts the headless Electron caffeine server with system tray only.
```bash
node caffeine.js server
# or
npm run server
# or
npm start
```

**Cross-platform background operation:**
```bash
npm run server  # Completely headless - no windows, only system tray
```

### version
Shows version information from both package.json and .claude-plugin/plugin.json.
```bash
node caffeine.js version
# or
npm run version
```

Output example:
```
=== CC-Caffeine Version ===
Package version: 0.1.1
Plugin version:  0.1.1
```

## Installation & Setup

1. Install Node.js dependencies:
```bash
npm install
```

2. Create config directory:
```bash
mkdir -p ~/.claude/plugins/cc-caffeine
```

3. Make the script executable (optional):
```bash
chmod +x caffeine.js
```

4. Configure Claude Code hooks (example):
```json
{
  "hooks": {
     "UserPromptSubmit": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node /path/to/caffeine.js caffeinate"
          }
        ]
      }
    ],
    "Stop": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "node /path/to/caffeine.js uncaffeinate"
          }
        ]
      }
    ]
  }
}
```

Note: The server will be auto-started by the caffeinate command when needed.

## JSON File Structure

JSON file located at: `~/.claude/plugins/cc-caffeine/sessions.json`

```json
{
  "sessions": {
    "session_id_abc123": {
      "created_at": "2025-01-08T10:30:00.000Z",
      "last_activity": "2025-01-08T10:45:00.000Z"
    }
  },
  "last_updated": "2025-01-08T10:45:00.000Z"
}
```

Sessions are removed automatically after 15 minutes of inactivity.

## File Concurrency

- **proper-lockfile** ensures atomic read/write operations
- File locking prevents corruption when multiple processes access simultaneously

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [samber/cc-caffeine](https://github.com/samber/cc-caffeine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
