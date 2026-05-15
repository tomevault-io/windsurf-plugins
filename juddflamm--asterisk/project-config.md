---
trigger: always_on
description: This project creates a terminal command called `asterisk` that allows users to manage multiple Anthropic account profiles for Claude Code CLI. It enables running different Claude sessions simultaneously, each with different account configurations, without setting global environment variables.
---

# Asterisk - Claude Code Multi-Account Manager Project

## Project Overview

This project creates a terminal command called `asterisk` that allows users to manage multiple Anthropic account profiles for Claude Code CLI. It enables running different Claude sessions simultaneously, each with different account configurations, without setting global environment variables.

**Platform Support**: macOS and Linux
**Repository**: https://github.com/juddflamm/asterisk

## Current Project Status (October 2025)

**Version**: See `version.txt` for current version
**Installation**: One-command curl installer from GitHub
**Interface**: Boxed UI with full-width borders, colored elements, single-key input
**Terminology**: Uses "account profiles" to clarify these are Claude Code configuration folders
**Menu**: Shows configurable default account first, followed by additional profiles
**Updates**: Automatic version checking with one-click updates
**MCP Support**: Built-in MCP tool installation for specific profiles
**Distribution**: Published as open-source project with comprehensive documentation

## Project Purpose

- **Problem**: Users with multiple Anthropic accounts (work, personal, etc.) need to manually switch between them when using Claude Code CLI
- **Solution**: A simple interactive menu that lets users select which account profile to use for each Claude session
- **Terminology**: Uses "account profiles" to refer to Claude Code configuration folders, not actual Anthropic accounts
- **Benefits**: 
  - Multiple terminals can run different accounts simultaneously
  - No global environment variable pollution
  - Clean account switching workflow
  - Lazy directory creation (only creates folders when needed)

## Architecture & Design Decisions

### Command Name Evolution
- Started as `claude-account` (descriptive but long)
- Briefly considered `*` (rejected - shell conflicts)
- Tried `cc` (rejected - conflicts with C compiler)
- Considered `claudex` (good but changed)
- Final choice: `asterisk` (unique, memorable, no conflicts)

### Hidden Directory Structure
- **Location**: `~/.asterisk/`
- **Reasoning**: Hidden folder in user home directory, follows Unix conventions
- **Contents**:
  - `settings.json` - Account configuration
  - Individual account directories (created on-demand)

### Settings File Format
**Current Format** (v1.3.0+):
```json
{
  "defaultAccountName": "Personal",
  "additionalAccounts": [
    "Work",
    "Client"
  ]
}
```

**Legacy Format** (v1.2.x and earlier):
```json
{
  "accounts": [
    "Work",
    "Personal"
  ]
}
```

**Migration**: Asterisk automatically migrates old format to new on startup.

**Key Changes**:
- `defaultAccountName`: Configurable name for the default profile (launches without custom config)
- `additionalAccounts`: Renamed from `accounts` for clarity
- Backward compatible: old settings files are automatically upgraded

**Reasoning**:
- Allows users to customize the default account name instead of hardcoding "Personal"
- Clearer separation between default and additional profiles
- Account name serves as both display name and directory name

### Directory Creation Strategy
**Decision**: Lazy creation
- **What**: Only create account directories when user first selects them
- **Why**: Avoids creating unused folders that clutter the file system
- **Implementation**: Check if directory exists before launching Claude, create if missing

### Environment Variable Management
**Key Requirements**:
1. **Per-session only**: No global environment variables
2. **Clean default**: "Personal (default)" option must explicitly unset `CLAUDE_CONFIG_DIR` 
3. **Isolation**: Each terminal session runs independently

**Implementation**:
- Account selection: `exec env CLAUDE_CONFIG_DIR="$config_dir" claude "$@"`
- Personal (default) selection: `exec env -u CLAUDE_CONFIG_DIR claude "$@"`

### Parameter Pass-through
**Decision**: All parameters passed to `asterisk` are forwarded to `claude`
- **Implementation**: Use `"$@"` to preserve all arguments
- **Benefit**: Makes `asterisk` a complete drop-in wrapper for Claude CLI

## Technical Implementation

### Core Components

1. **Main Script** (`asterisk`):
   - Bash script with color-coded output
   - Interactive menu system with single-key input
   - JSON parsing (with fallback parsing if `jq` not available)
   - Automatic setup on first run

2. **Setup Function** (`setup_accounts_dir()`):
   - Creates `~/.asterisk/` directory
   - Creates default `settings.json` with example accounts
   - Does NOT pre-create account directories

3. **Menu System** (`show_menu()`):
   - Personal (default) option as first menu item (launches without config)
   - Dynamic account list from `settings.json`
   - Edit settings option (E key, opens in VSCode)
   - Single-key input (no enter required)
   - First letter matching for account selection

4. **Account Selection Logic**:
   - Creates directory if missing
   - Sets `CLAUDE_CONFIG_DIR` environment variable
   - Launches Claude with all passed parameters

### Menu Options


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juddflamm/asterisk](https://github.com/juddflamm/asterisk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-08 -->
