---
trigger: always_on
description: A native desktop application that provides real-time usage analytics for Claude Code. Currently available for macOS only.
---

# Claudit - Claude Code Usage Monitor

A native desktop application that provides real-time usage analytics for Claude Code. Currently available for macOS only.

> **Note:** This project was mostly coded by Claude Code itself.

## Tech Stack

- **Framework**: Tauri 2 (Rust backend + webview)
- **Frontend**: React 18, Vite, TypeScript
- **Styling**: Tailwind CSS 3
- **Charts**: Recharts
- **Data Fetching**: TanStack Query
- **Animations**: Motion (import from "motion/react")
- **Icons**: Lucide React

## Commands

```bash
pnpm install        # Install dependencies
pnpm dev           # Run in development mode
pnpm build         # Build for production
```

## Project Structure

```
claudit/
├── packages/
│   └── frontend/           # React frontend
│       └── src/
│           ├── domains/    # Feature-based organization
│           │   ├── analytics/   # Dashboard with charts & stats
│           │   ├── settings/    # App preferences UI
│           │   ├── projects/    # Project browser & AI suggestions
│           │   ├── agents/      # Claude agents browser
│           │   ├── plugins/     # Claude plugins browser
│           │   ├── config/      # Claude config viewer
│           │   ├── analysis/    # Deep usage analysis
│           │   ├── backup/      # Backup management
│           │   └── shared/      # Shared components
│           ├── components/      # Reusable UI components
│           └── lib/             # Utilities
├── src-tauri/              # Rust backend
│   └── src/
│       ├── services/       # Core services
│       │   ├── usage.rs    # JSONL parsing
│       │   ├── analytics.rs # Stats calculation
│       │   ├── hooks.rs    # HTTP server for hooks
│       │   ├── settings.rs # Preferences
│       │   └── config.rs   # Claude config & project management
│       ├── tray.rs         # System tray menu
│       └── lib.rs          # Main entry & Tauri commands
├── landing/                # Landing page (claudit.cloud.neschkudla.at)
└── scripts/                # Build utilities (icon generation, local release)
```

## Architecture

### Data Flow
1. Claude Code writes usage data to `~/.claude/projects/**/*.jsonl`
2. UsageReader parses JSONL files and deduplicates entries
3. AnalyticsService calculates stats, costs, burn rates
4. Tray menu displays live stats, updates every 30s
5. Analytics window shows interactive charts

### Hook System

Claudit runs an HTTP server on localhost:3456 that receives events from Claude Code hooks.

**Supported Events:**

| Event | Trigger | Purpose |
|-------|---------|---------|
| `Stop` | Claude finishes main response | Triggers notification |
| `SubagentStop` | Subagent finishes | Triggers notification |
| `PostToolUse` | After tool execution | Granular tracking (Bash only) |

**API Endpoints:**

```bash
# Health check
curl http://localhost:3456/
# Response: {"status":"ok"}

# Send hook event
curl -X POST http://localhost:3456/hook \
  -H "Content-Type: application/json" \
  -d '{"event": "Stop"}'
# Response: {"success":true}
```

**Payload Schema:**

```typescript
interface HookEvent {
  event: "Stop" | "SubagentStop" | "PostToolUse" | "PreToolUse" | "UserPromptSubmit";
  tool?: string;      // Tool name (for PostToolUse)
  context?: string;   // Optional context
  timestamp?: string; // ISO timestamp
}
```

**Claude Code Settings Hook Config:**

```json
{
  "hooks": {
    "Stop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "curl -s -X POST http://localhost:3456/hook -H \"Content-Type: application/json\" -d '{\"event\": \"Stop\"}' > /dev/null 2>&1 &"
      }]
    }],
    "SubagentStop": [{
      "matcher": "*",
      "hooks": [{
        "type": "command",
        "command": "curl -s -X POST http://localhost:3456/hook -H \"Content-Type: application/json\" -d '{\"event\": \"SubagentStop\"}' > /dev/null 2>&1 &"
      }]
    }],
    "PostToolUse": [{
      "matcher": "Bash",
      "hooks": [{
        "type": "command",
        "command": "curl -s -X POST http://localhost:3456/hook -H \"Content-Type: application/json\" -d \"{\"event\": \"PostToolUse\", \"tool\": \"$CLAUDE_TOOL_NAME\"}\" > /dev/null 2>&1 &"
      }]
    }]
  }
}
```

**Port Fallback:** If port 3456 is busy, the server tries ports 3457-3466.

### Token Pricing (per 1M tokens)
- claude-sonnet-4: $3 input, $15 output
- claude-opus-4: $15 input, $75 output
- Other models: $3 input, $15 output (default)

## Features

### Core Analytics
- Real-time token consumption in menu bar
- Cost tracking with burn rate ($/hour)
- 5-hour session block tracking
- Per-project and per-model breakdowns
- Analytics dashboard with interactive charts
- Notifications when Claude finishes responding

### Project Management
- Browse all Claude Code projects with usage stats
- View CLAUDE.md content with markdown rendering
- Browse project-specific commands and MCP servers
- **AI Suggestions**: Get Claude-powered suggestions to improve project workflow
- Set custom project images
- One-click open in Finder or editor

### Configuration Browser
- View and explore Claude config (~/.claude.json)
- Browse installed agents (global and project-specific)
- Browse installed plugins with marketplace info

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [flipace/claudit](https://github.com/flipace/claudit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-22 -->
