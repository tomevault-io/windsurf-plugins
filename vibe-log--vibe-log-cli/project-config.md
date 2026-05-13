---
trigger: always_on
description: vibelog-cli is a TypeScript-based CLI application designed to track and analyze developer productivity sessions from Claude Code. It's distributed as an NPX package for cross-platform compatibility.
---

# Claude Code - vibe-log CLI Development Notes

## Project Architecture Overview

vibelog-cli is a TypeScript-based CLI application designed to track and analyze developer productivity sessions from Claude Code. It's distributed as an NPX package for cross-platform compatibility.

### Key Technologies
- **TypeScript 5.3+** - Type-safe development
- **Commander.js** - CLI command parsing and routing
- **Inquirer.js** - Interactive terminal UI components
- **Axios** - HTTP client for API communication
- **Better-SQLite3** - Fast SQLite database access for Cursor IDE integration
- **Chalk** - Terminal string styling
- **Ora** - Elegant terminal spinners
- **Conf** - Encrypted configuration storage
- **EventSource Polyfill** - SSE client for real-time auth
- **Vitest** - Modern testing framework
- **tsup** - Zero-config TypeScript bundler

## Core Directory Structure

```
vibelog-cli/
├── bin/              # NPX entry point
├── dist/             # Compiled JavaScript output
├── src/
│   ├── commands/     # CLI command implementations
│   ├── lib/          # Core business logic
│   │   ├── api-client.ts        # Secure API communication
│   │   ├── auth/                # Authentication modules
│   │   ├── readers/             # Session data readers
│   │   ├── sub-agents/          # Claude Code sub-agent management
│   │   └── ui/                  # Terminal UI components
│   ├── types/        # TypeScript type definitions
│   └── utils/        # Helper utilities
└── tests/            # Comprehensive test suite
```

## Main Components

### Command Layer (`/src/commands/`)
Each command is a standalone module that handles specific CLI operations:

- **`init.ts`** - First-time setup and authentication flow
- **`send.ts`** - Upload session data to Vibelog API (supports silent mode for hooks)
  - **IMPORTANT**: Always use `sendWithTimeout({ selectedSessions })` when uploading pre-selected sessions
  - Never call `apiClient.uploadSessions()` directly - the send command handles all session processing
  - The `selectedSessions` parameter expects `SelectedSessionInfo[]` from session-selector
- **`status.ts`** - Display user streak, points, and statistics (includes Cursor IDE stats)
- **`cursor-stats.ts`** - Display dedicated Cursor IDE statistics view
- **`auth.ts`** - Re-authentication for expired/invalid tokens
  - Supports `wizardMode` parameter to suppress menu-related messages during guided flows
- **`config.ts`** - Manage CLI configuration settings
- **`logout.ts`** - Clear stored authentication credentials
- **`install-hooks.ts`** - Legacy hook installation (redirects to hooks-manage)
- **`hooks-log.ts`** - View and manage hook execution logs
- **`hooks-manage.ts`** - Comprehensive hooks management interface
- **`verify-hooks.ts`** - Verify hook installation and configuration

### Core Libraries (`/src/lib/`)

#### API & Authentication
- **`api-client.ts`** - Centralized API client with:
  - Request rate limiting (60 req/min)
  - Automatic retry logic with exponential backoff
  - Request ID tracking for debugging
  - Secure token management
  - Session upload with privacy preservation

- **`auth/browser.ts`** - Browser-based OAuth flow:
  - Opens browser for authentication
  - Uses SSE for real-time auth status
  - Secure token exchange

- **`auth/sse-client.ts`** - Server-sent events client:
  - Real-time authentication monitoring
  - Automatic reconnection handling
  - Event parsing and error handling

#### Configuration & State
- **`config.ts`** - Encrypted configuration management:
  - AES-256-GCM encryption for sensitive data
  - Cross-platform config storage
  - Schema validation
  - Secure key generation and storage

- **`detector.ts`** - Setup state detection:
  - Detects installation state (FIRST_TIME, LOCAL_ONLY, CLOUD_AUTO, etc.)
  - Checks for sub-agents, hooks, and authentication
  - Provides detailed state information for UI

#### Data Processing
- **`message-sanitizer-v2.ts`** - Privacy-preserving sanitization:
  - Redacts sensitive information (credentials, paths, URLs)
  - Preserves semantic meaning for analysis
  - Consistent entity naming across sessions
  - Tracks redaction metadata

- **`readers/claude.ts`** - Claude Code session parser:
  - Reads JSONL session files from `~/.claude/projects/`
  - Extracts messages, timestamps, and metadata
  - Filters by date and project
  - Handles encoded project directory names

- **`readers/cursor.ts`** - Cursor IDE message counter:
  - Reads from Cursor's SQLite database (`state.vscdb`)
  - Supports both legacy and modern conversation formats
  - Counts total messages, user messages, and assistant messages
  - Cross-platform support (macOS, Windows, Linux)
  - Used in status command to show Cursor IDE statistics

#### Claude Code Integration
- **`sub-agents/manager.ts`** - Sub-agent lifecycle management:
  - Install/uninstall sub-agents to `~/.claude/agents/`
  - Check installation status
  - Batch installation support
  - Progress tracking

- **`sub-agents/templates.ts`** - Sub-agent configurations:
  - Defines 8 specialized vibe-log sub-agents
  - Each agent has specific analysis capabilities

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vibe-log/vibe-log-cli](https://github.com/vibe-log/vibe-log-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
