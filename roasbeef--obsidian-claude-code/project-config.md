---
trigger: always_on
description: Obsidian plugin that embeds Claude AI as an assistant using the Claude Agent SDK. Features built-in tools, skill loading, and custom Obsidian-specific tools via SDK MCP server.
---

# obsidian-claude-code

## Project Overview
Obsidian plugin that embeds Claude AI as an assistant using the Claude Agent SDK. Features built-in tools, skill loading, and custom Obsidian-specific tools via SDK MCP server.

## Development Commands
```bash
# Install dependencies (use bun, not npm)
bun install

# Development mode with watch
bun run dev

# Production build
bun run build

# Type check only
bun run typecheck

# Run all checks (typecheck + lint + test)
make check

# Quick dev check (typecheck + unit tests only)
make quick
```

## Package Manager
**Use `bun` for all package management operations:**
- `bun install` - Install dependencies
- `bun add <package>` - Add dependency
- `bun add -d <package>` - Add dev dependency
- `bun remove <package>` - Remove dependency

We use Bun for package management but esbuild for bundling (Obsidian's standard approach).

## Project Structure
```
src/
├── main.ts                    # Plugin entry point
├── types.ts                   # TypeScript types
├── interfaces/                # DI interfaces for testability
│   ├── IVaultAdapter.ts       # File system abstraction
│   ├── IConversationStorage.ts # Persistence abstraction
│   └── ILogger.ts             # Logging abstraction
├── utils/
│   └── Logger.ts              # File-based debug logging
├── agent/
│   ├── AgentController.ts     # Claude Agent SDK query() orchestration
│   ├── ConversationManager.ts # Session/history persistence
│   └── ObsidianMcpServer.ts   # Custom Obsidian tools via SDK MCP
├── views/
│   ├── ChatView.ts            # Main sidebar (ItemView)
│   ├── MessageList.ts         # Message rendering
│   ├── MessageRenderer.ts     # Individual message
│   ├── ToolCallDisplay.ts     # Tool call UI
│   ├── ChatInput.ts           # Input with autocomplete
│   ├── AutocompletePopup.ts   # Command/file suggestions
│   └── ConversationHistoryModal.ts # History browser
└── settings/
    └── SettingsTab.ts         # Settings UI

tests/
├── unit/                      # Pure function tests
├── integration/               # Tests with mocked Obsidian
├── property/                  # Property-based tests (fast-check)
├── mocks/
│   ├── obsidian/              # Obsidian API mocks
│   └── claude-sdk/            # Claude SDK mocks
├── helpers/                   # Test utilities
└── setup.ts                   # Global test setup
```

## Testing in Obsidian
Symlink plugin to vault for hot-reload testing:
```bash
ln -s /Users/roasbeef/codez/obsidian-claude-code \
      /path/to/vault/.obsidian/plugins/obsidian-claude-code
```

Then in Obsidian: Settings > Community Plugins > Reload

## Key Architecture Decisions
- **Desktop-only** - Requires Node.js for Claude Agent SDK
- Uses `@anthropic-ai/claude-agent-sdk` with `query()` function
- Built-in tools (Read, Write, Bash, Grep, etc.) from SDK presets
- Skills auto-load from `vault/.claude/skills/` via `settingSources: ['project']`
- Custom Obsidian tools via SDK MCP server (`ObsidianMcpServer.ts`)
- Supports Claude Max subscription via `CLAUDE_CODE_OAUTH_TOKEN` env var
- Conversations stored in `.obsidian-claude-code/` in vault
- Right sidebar ItemView (Cursor-style chat)
- Debug logs written to `~/.obsidian-claude-code/debug.log`

## Electron/Obsidian Environment Considerations

### Claude CLI Path Resolution
The Agent SDK internally spawns the Claude Code CLI. In bundled Electron environments (like Obsidian), `import.meta.url` doesn't work, so the SDK can't auto-find the CLI.

**Solution**: Explicitly provide `pathToClaudeCodeExecutable` option:
```typescript
query({
  prompt: content,
  options: {
    pathToClaudeCodeExecutable: "/path/to/claude",
    // ...
  }
})
```

The plugin auto-detects Claude in common locations:
- `~/.nvm/versions/node/*/bin/claude`
- `/usr/local/bin/claude`
- `/opt/homebrew/bin/claude`

### PATH for Node.js
The Claude CLI is a Node.js script (`#!/usr/bin/env node`). Obsidian's Electron doesn't have nvm's PATH. The plugin adds the Claude bin directory to `env.PATH` when calling `query()`.

### Model Selection
Use simplified model names (not full IDs):
- `"sonnet"` - Claude Sonnet 4 (faster)
- `"opus"` - Claude Opus 4.5 (more capable)
- `"haiku"` - Claude Haiku (fastest)

### Obsidian File API
Use `vault.adapter.exists()` and `vault.adapter.write()` instead of `vault.getAbstractFileByPath()` for more reliable file operations. Obsidian's internal caching can cause race conditions with `vault.create()`.

## Dependencies
- `obsidian` - Obsidian API
- `@anthropic-ai/claude-agent-sdk` - Claude Agent SDK
- `zod` - Schema validation for SDK MCP tools

## Obsidian-Specific Tools (SDK MCP Server)
The plugin exposes these Obsidian-specific tools to Claude:
- `mcp__obsidian__open_file` - Open file in Obsidian view
- `mcp__obsidian__execute_command` - Run Obsidian command
- `mcp__obsidian__show_notice` - Display notification
- `mcp__obsidian__get_active_file` - Get current file info
- `mcp__obsidian__rebuild_vault_index` - Trigger vault-search index rebuild
- `mcp__obsidian__list_commands` - Discover available commands
- `mcp__obsidian__create_note` - Create new notes
- `mcp__obsidian__reveal_in_explorer` - Show in file explorer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Roasbeef/obsidian-claude-code](https://github.com/Roasbeef/obsidian-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
