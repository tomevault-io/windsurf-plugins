---
trigger: always_on
description: This file provides context for Claude Code when working with this codebase.
---

# CLAUDE.md

This file provides context for Claude Code when working with this codebase.

## Project Overview

claudecode.nvim - A Neovim plugin that implements the same WebSocket-based MCP protocol as Anthropic's official IDE extensions. Built with pure Lua and zero dependencies.

## Common Development Commands

### Testing

- `make test` - Run all tests using busted with coverage
- `busted tests/unit/specific_spec.lua` - Run specific test file
- `busted --coverage -v` - Run tests with coverage

### Code Quality

- `make check` - Check Lua syntax and run luacheck
- `make format` - Format code with stylua (or nix fmt if available)
- `luacheck lua/ tests/ --no-unused-args --no-max-line-length` - Direct linting

### Build Commands

- `make` - **RECOMMENDED**: Run formatting, linting, and testing (complete validation)
- `make all` - Run check and format (default target)
- `make test` - Run all tests using busted with coverage
- `make check` - Check Lua syntax and run luacheck
- `make format` - Format code with stylua (or nix fmt if available)
- `make clean` - Remove generated test files
- `make help` - Show available commands

**Best Practice**: Always use `make` at the end of editing sessions for complete validation.

### Development with Nix

- `nix develop` - Enter development shell with all dependencies
- `nix fmt` - Format all files using nix formatter

### Integration Testing with Fixtures

The `fixtures/` directory contains test Neovim configurations for verifying plugin integrations:

- `vv <config>` - Start Neovim with a specific fixture configuration
- `vve <config>` - Start Neovim with a fixture config in edit mode
- `list-configs` - Show available fixture configurations
- Source `fixtures/nvim-aliases.sh` to enable these commands

**Available Fixtures**:

- `netrw` - Tests with Neovim's built-in file explorer
- `nvim-tree` - Tests with nvim-tree.lua file explorer
- `oil` - Tests with oil.nvim file explorer
- `mini-files` - Tests with mini.files file explorer

**Usage**: `source fixtures/nvim-aliases.sh && vv oil` starts Neovim with oil.nvim configuration

## Architecture Overview

### Core Components

1. **WebSocket Server** (`lua/claudecode/server/`) - Pure Neovim implementation using vim.loop, RFC 6455 compliant
2. **MCP Tool System** (`lua/claudecode/tools/`) - Implements tools that Claude can execute (openFile, getCurrentSelection, etc.)
3. **Lock File System** (`lua/claudecode/lockfile.lua`) - Creates discovery files for Claude CLI at `~/.claude/ide/`
4. **Selection Tracking** (`lua/claudecode/selection.lua`) - Monitors text selections and sends updates to Claude
5. **Diff Integration** (`lua/claudecode/diff.lua`) - Native Neovim diff support for Claude's file comparisons
6. **Terminal Integration** (`lua/claudecode/terminal.lua`) - Manages Claude CLI terminal sessions with support for internal Neovim terminals and external terminal applications

### WebSocket Server Implementation

- **TCP Server**: `server/tcp.lua` handles port binding and connections
- **Handshake**: `server/handshake.lua` processes HTTP upgrade requests with authentication
- **Frame Processing**: `server/frame.lua` implements RFC 6455 WebSocket frames
- **Client Management**: `server/client.lua` manages individual connections
- **Utils**: `server/utils.lua` provides base64, SHA-1, XOR operations in pure Lua

#### Authentication System

The WebSocket server implements secure authentication using:

- **UUID v4 Tokens**: Generated per session with enhanced entropy
- **Header-based Auth**: Uses `x-claude-code-ide-authorization` header
- **Lock File Discovery**: Tokens stored in `~/.claude/ide/[port].lock` for Claude CLI
- **MCP Compliance**: Follows official Claude Code IDE authentication protocol

### MCP Tools Architecture (✅ FULLY COMPLIANT)

**Complete VS Code Extension Compatibility**: All tools now implement identical behavior and output formats as the official VS Code extension.

**MCP-Exposed Tools** (with JSON schemas):

- `openFile` - Opens files with optional line/text selection (startLine/endLine), preview mode, text pattern matching, and makeFrontmost flag
- `getCurrentSelection` - Gets current text selection from active editor
- `getLatestSelection` - Gets most recent text selection (even from inactive editors)
- `getOpenEditors` - Lists currently open files with VS Code-compatible `tabs` structure
- `openDiff` - Opens native Neovim diff views
- `checkDocumentDirty` - Checks if document has unsaved changes
- `saveDocument` - Saves document with detailed success/failure reporting
- `getWorkspaceFolders` - Gets workspace folder information
- `closeAllDiffTabs` - Closes all diff-related tabs and windows
- `getDiagnostics` - Gets language diagnostics (errors, warnings) from the editor

**Internal Tools** (not exposed via MCP):

- `close_tab` - Internal-only tool for tab management (hardcoded in Claude Code)

**Format Compliance**: All tools return MCP-compliant format: `{content: [{type: "text", text: "JSON-stringified-data"}]}`

### Terminal Integration Options

**Internal Terminals** (within Neovim):

- **Snacks.nvim**: `terminal/snacks.lua` - Advanced terminal with floating windows
- **Native**: `terminal/native.lua` - Built-in Neovim terminal as fallback


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rxwen) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
