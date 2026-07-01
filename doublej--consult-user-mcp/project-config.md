---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

```bash
bun install          # Install dependencies
bun run build        # Compile TypeScript + Swift CLI
bun run build:ts     # TypeScript only
bun run start        # Run the compiled server
bun run dev          # Watch mode for development
bun test             # Run tests
```

## Architecture

This is an MCP (Model Context Protocol) server that exposes native UI dialogs as tools for LLMs. Supports macOS (Swift/AppKit) and Windows (WPF/.NET).

```
src/
├── index.ts                 # MCP server + tool registration (ask, notify, tweak, propose_layout)
├── compact.ts               # Response transformer (provider → compact output)
├── css-resolver.ts          # CSS selector/property → file location resolver (for tweak)
├── humanize.ts              # Optional human-readable response formatting
├── resolve-utils.ts         # Shared resolution utilities for tweak parameters
├── settings.ts              # User settings reader
├── text-search-resolver.ts  # Text pattern search → file location resolver (for tweak)
├── types.ts                 # Shared types and interfaces
├── update-check.ts          # GitHub release update checker
├── validate-choices.ts      # Choice validation (no "all of above")
├── index.test.ts            # Tests
├── providers/
│   ├── interface.ts         # DialogProvider interface
│   ├── swift.ts             # macOS: Swift CLI + sketch CLI implementation
│   └── windows.ts           # Windows: WPF CLI implementation
```

**Provider pattern**: Platform-specific code is abstracted behind `DialogProvider` interface. `createProvider()` in `index.ts` selects `WindowsDialogProvider` on Windows, `SwiftDialogProvider` on macOS.

**Tool architecture**: Each tool in `index.ts`:
1. Defines Zod schema for inputs
2. Registers via `server.registerTool()` with metadata
3. Delegates to `provider.methodName()` for platform execution

**Key files**:
- `types.ts`: `DialogPosition`, option interfaces, result types
- `providers/interface.ts`: `DialogProvider` interface contract
- `providers/swift.ts`: macOS implementation using Swift CLI
- `providers/windows.ts`: Windows implementation using WPF CLI

## Available Tools

| Tool | Purpose |
|------|---------|
| `ask` | Unified interactive dialog (type: confirm/pick/text/form) |
| `notify` | Notification banner |
| `tweak` | Real-time numeric value adjustment with live file writes |
| `propose_layout` | Interactive grid layout editor (macOS only) |

The `ask` tool routes to provider methods via `type` field:
- `confirm` → `provider.confirm()` — Yes/No
- `pick` → `provider.choose()` — List picker (single or multi-select)
- `text` → `provider.textInput()` — Free-form input (supports password masking)
- `form` → `provider.questions()` — Multi-question (wizard or accordion)

Responses are transformed by `compact.ts` to strip verbose/null fields.

## Checkpoints & Rewind Limitations

**IMPORTANT**: These tools are designed as checkpoints to replace `AskUserQuestion` in MCP workflows, BUT they have a critical limitation with Claude Code's rewind feature:

- ❌ **MCP tool interactions do NOT create rewind points**
- ✅ **Only user text messages create rewind points**

This means:
- When Claude uses `ask`, your response via the dialog won't appear in the rewind timeline
- You cannot rewind to restore code/conversation state at MCP interaction points
- Only your typed messages in the chat create restore points

**Recommended Workaround**: Use pre/post-change hooks to commit to a `claude/*` branch:

```bash
# In your Claude Code settings, configure hooks:
# Pre-change hook (before MCP checkpoint):
git add -A && git commit -m "checkpoint: before $(date +%H:%M:%S)" --allow-empty

# Post-change hook (after MCP response):
git add -A && git commit -m "checkpoint: after user response $(date +%H:%M:%S)"
```

This creates git commits at each checkpoint, giving you:
- ✅ Actual code state snapshots
- ✅ Easy revert via `git checkout <commit>`
- ✅ Timeline of all checkpoints via `git log`
- ✅ Works independently of Claude Code's rewind feature

**Alternative**: Send a brief text message (e.g., "approved") after checkpoints to create a rewind point, but this only restores conversation, not code state.

**Why this matters**: While these tools provide excellent control over Claude's workflow, they're invisible to the rewind feature, making it harder to jump back to key decision points without git-based checkpoints.

## Common Parameters

All dialog tools support:
- `position`: `"left"` | `"right"` | `"center"` (default: `"left"`) - screen position
- Titles are automatically prefixed with the calling client name (e.g., "Claude Desktop - Confirmation")

## Handling Snooze & Feedback Responses

All interactive dialogs can return three types of responses:

### Normal Response
User answered the question. The `answer` field contains: `true/false` (confirm), `string` (pick single/text), `string[]` (pick multi), or `Record<string, string>` (form).

### Snooze Response

**CRITICAL**: When `snoozed: true` is returned, you MUST:
1. **Actually wait** using `sleep` command (NOT just say "waiting")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doublej/consult-user-mcp](https://github.com/doublej/consult-user-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
