---
trigger: always_on
description: Development guide for cc-hooks - a Claude Code hooks processing system with TTS announcements and
---

# CLAUDE.md

Development guide for cc-hooks - a Claude Code hooks processing system with TTS announcements and
contextual AI.

## Documentation Structure

- **[README.md](README.md)** - Plugin installation (recommended for most users)
- **[STANDALONE_README.md](STANDALONE_README.md)** - Standalone installation (for
  developers/contributors)
- **[MIGRATION.md](MIGRATION.md)** - Migration guide from standalone to plugin mode
- **[CLAUDE.md](CLAUDE.md)** - This file: Development and technical guide

## Best Practices

### PEP 723 for Standalone Scripts

**All test scripts, utilities, and one-off tools MUST use PEP 723** for dependency management:

```python
#!/usr/bin/env -S uv run --script
# /// script
# dependencies = ["aiosqlite", "python-dotenv"]
# ///
```

**Why:**

- Self-contained with explicit dependencies
- No global environment pollution
- Works across different machines
- Easy to run: `uv run script.py`

### Database Migrations

**CRITICAL RULES:**

1. **One SQL statement per migration** - Never combine multiple statements
2. **Append-only** - Never modify existing migrations that are deployed
3. **Sequential versions** - Always increment from the latest version

### Code Style

- Clean, readable code with minimal single-line comments
- Self-contained, reusable components
- Direct communication (no sugar-coating in error messages)
- Practical over perfect

## Quick Start

**Plugin Mode (Recommended)**:

```bash
# Set up alias first (add to .bashrc/.zshrc)
alias cld='~/.claude/plugins/marketplaces/cc-hooks-plugin/claude.sh'

# Start Claude Code with cc-hooks
cld

# Common configurations
cld --audio=gtts --language=id
cld --audio=elevenlabs --ai=full
cld --silent=announcements
```

**Standalone Mode**:

```bash
# From cc-hooks directory
./claude.sh

# Common configurations
./claude.sh --audio=gtts --language=id
./claude.sh --audio=elevenlabs --ai=full
./claude.sh --silent=announcements
```

## Project Overview

**Purpose**: Middleware that processes Claude Code hook events with audio feedback and contextual AI
messages.

**Core Flow**:

```
Claude Code → hooks.py → FastAPI server → Event processor → [TTS + Sound effects]
```

**Key Features**:

- Sequential event processing with SQLite queue
- TTS announcements (prerecorded/Google/ElevenLabs)
- Contextual AI completion messages via OpenRouter
- Per-instance servers for concurrent sessions
- Granular audio control (silent modes, provider selection)

## Data Directory

**All runtime data is stored in** `~/.claude/.cc-hooks/`

This directory is **shared across both installation modes** (standalone and plugin) to ensure
seamless migration and data persistence across updates.

**Directory Structure**:

```
~/.claude/.cc-hooks/
├── events.db           # SQLite database (events, sessions, migrations, version_checks)
├── logs/               # Per-session server logs
│   └── {claude_pid}.log
└── .tts_cache/         # TTS audio cache (when enabled)
    ├── prerecorded/
    ├── gtts/
    └── elevenlabs/
```

**Key Files**:

- **events.db**: Main SQLite database containing all events, sessions, and metadata
- **logs/**: Server logs named by Claude PID (e.g., `12345.log`)
- **.tts_cache/**: Generated TTS audio files organized by provider

**Note**: The data directory persists across updates and uninstallation, allowing you to:

- Switch between standalone and plugin modes without losing data
- Update cc-hooks while preserving event history and session state
- Manually inspect logs and database for debugging

## Installation Modes

cc-hooks supports two installation modes:

### 1. Standalone Mode

**Installation**: Can be installed anywhere on the filesystem (user's choice)

**Hooks Configuration**: Defined in `~/.claude/settings.json` (manual setup required)

**Use Case**: Development, testing, custom installation paths

**Setup**:

```json
{
  "hooks": {
    "SessionStart": {
      "type": "command",
      "command": "uv run /path/to/cc-hooks/hooks.py"
    }
    // ... other hooks
  }
}
```

### 2. Plugin Mode (Recommended)

**Installation**: Installed at `~/.claude/plugins/marketplaces/cc-hooks-plugin` (via Claude plugin
system)

**Hooks Configuration**: Defined in
`~/.claude/plugins/marketplaces/cc-hooks-plugin/hooks/hooks.json` (automatic)

**Use Case**: Production use, easy updates, no manual hook configuration

**Installation**:

```bash
# Via CLI (recommended)
claude plugin marketplace add https://github.com/husniadil/cc-hooks.git
claude plugin install cc-hooks-plugin@cc-hooks-plugin

# Or inside Claude REPL
/plugin marketplace add https://github.com/husniadil/cc-hooks.git
/plugin install cc-hooks-plugin@cc-hooks-plugin
```

**Key Differences**:

| Feature           | Standalone                         | Plugin                                           |
| ----------------- | ---------------------------------- | ------------------------------------------------ |
| Installation path | User-defined                       | `~/.claude/plugins/marketplaces/cc-hooks-plugin` |
| Hooks config      | Manual (`~/.claude/settings.json`) | Automatic (`hooks/hooks.json` in plugin)         |
| Updates           | Git pull                           | `/plugin update cc-hooks-plugin@cc-hooks-plugin` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [husniadil/cc-hooks](https://github.com/husniadil/cc-hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
