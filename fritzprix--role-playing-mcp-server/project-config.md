---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

RPG MCP Server is a Model Context Protocol (MCP) server that enables AI assistants to run interactive role-playing games with users. The server provides 7 core tools that form a complete game loop: creating games, progressing narratives, prompting user choices, processing selections, updating game state, handling game over scenarios, and restarting games.

## Development Commands

### Build and Run

```bash
npm run build        # Compile TypeScript to JavaScript
npm start           # Run the compiled server
npm run dev         # Build and run in one step
```

### Code Quality

```bash
npm run type-check   # Run TypeScript type checking without compilation
npm run lint         # Check code for linting errors
npm run lint:fix     # Auto-fix linting errors
npm run format       # Format code with Prettier
npm run format:check # Check code formatting
npm run check        # Run all checks: type-check, lint, and format:check
```

### Testing & Debugging

```bash
npm run inspector     # Launch MCP Inspector for interactive testing
npm run inspector:dev # Build and launch inspector in one step
npm run clean        # Remove dist directory
```

**Important**: Always run `npm run build` before testing with MCP Inspector or publishing.

## Architecture

### Core Components

**`src/index.ts` - Main Server & Tool Handler**

- `RPGMCPServer` class orchestrates all MCP protocol interactions
- Registers 7 tools via `ListToolsRequestSchema`: createGame, updateGame, getGame, progressStory, promptUserActions, selectAction, selectRestart
- Handles tool invocation via `CallToolRequestSchema`
- Generates interactive HTML UIs for game progression and game over screens
- Uses `formatToolResponse()` to provide AI-friendly guidance on workflow state and next steps

**`src/gameManager.ts` - Game State Management**

- `GameManager` class maintains in-memory game storage via `Map<string, Game>`
- Implements all core game operations (create, update, get, progress, prompt, select)
- Handles nested field updates using path notation (e.g., `characters[0].hp`, `world.location`)
- Manages Delta system for tracking state changes between UI prompts
- Maintains game history (last 10 situation-action pairs)

**`src/types.ts` - Type Definitions**

- Defines all game state interfaces (`GameState`, `Character`, `WorldState`, etc.)
- Tool parameter interfaces for each of the 7 tools
- Delta tracking types (`DeltaInfo`)
- Game history types (`GameHistoryEntry`)

### Key Architectural Patterns

**Game Loop Flow**

```
createGame → progressStory → promptUserActions → [USER WAITS] → selectAction → updateGame → progressStory → ...
```

**Game Over & Restart Flow**

```
updateGame(isGameOver=true) → Game Over UI → [USER CLICKS RESTART] → selectRestart → createGame → ...
```

**Delta System**

- Changes accumulate in `_pendingDeltas` between `promptUserActions` calls
- Displayed visually in the next UI rendering
- Cleared after being shown to avoid duplication
- Handles multiple updates to the same field by preserving `initialValue` and updating `finalValue`

**Path-Based State Updates**

- Uses dot notation and bracket notation for nested updates
- Example: `characters[0].level` or `world.location`
- Parsed by `parsePath()` into key arrays for traversal
- Creates intermediate objects/arrays as needed

### MCP UI Integration

The server uses `@mcp-ui/server` for rendering interactive HTML interfaces:

**Game UI (`generateGameUI()`)**

- Shows story progress
- Displays pending deltas (recent changes)
- Renders action buttons (2-4 options)
- Clicking a button triggers `selectAction` tool via `postMessage`

**Game Over UI (`generateGameOverUI()`)**

- Empathetic game over explanation
- Shows what went wrong and how to prevent it
- Displays final game state
- Provides Restart button that triggers `selectRestart` tool

Both UIs use `text/html` MIME type and `preferredRenderContext: 'main'` for proper rendering.

### AI Agent Guidance System

The `formatToolResponse()` method provides structured guidance to AI agents:

- **Game Context**: Current state snapshot (ID, title, key stats)
- **What Happened**: Plain-language description of the tool's effect
- **Next Step**: Recommended tool to call with example parameters
- **Workflow Position**: Shows current step in the game loop
- **Additional Notes**: Implementation tips and warnings

This ensures AI agents understand the game flow and call tools in the correct sequence.

## Important Implementation Details

### Dynamic Choices Requirement

The `promptUserActions` tool requires 2-4 options that **mix positive and negative outcomes**:

- Include both favorable and risky possibilities
- Offer cautious vs daring approaches
- Provide different character alignments
- Ensure varied consequences for engaging gameplay

**Bad Example**: All safe options or all dangerous options
**Good Example**: Mix of "talk cautiously", "attack preemptively", "hide and observe", "search alternate route"

### Game Over Handling

When `updateGame` receives `isGameOver=true`:

1. A special Game Over UI is generated with empathetic messaging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fritzprix/role-playing-mcp-server](https://github.com/fritzprix/role-playing-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
