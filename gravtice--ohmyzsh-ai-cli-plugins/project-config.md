---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a collection of Oh My Zsh plugins that provide intelligent auto-completion for AI CLI tools:
- **claude** - Anthropic Claude Code CLI completion
- **codex** - OpenAI Codex CLI completion
- **gemini** - Google Gemini CLI completion

Each plugin is self-contained in its own directory with a single `.plugin.zsh` file.

## Language Guidelines

**IMPORTANT:** All written content in this repository must be in English, including:
- Code comments
- Documentation files (README, CLAUDE.md, etc.)
- Commit messages
- Pull request descriptions
- Issue reports
- Code variable names and function names

**EXCEPTION:** When communicating with the repository owner in conversation/chat, use Chinese (中文).

This ensures the codebase remains accessible to the international open-source community while maintaining effective communication with the primary maintainer.

## Core Architecture

### Plugin Structure Pattern

All plugins follow a consistent Zsh completion architecture:

1. **Main completion function** (`_<tool_name>`) - Entry point that handles:
   - Global options via `_arguments`
   - Top-level command dispatching via state machine
   - Delegates to subcommand-specific completion functions

2. **Subcommand completion functions** (`_<tool_name>_<subcommand>`) - Nested handlers for:
   - Multi-level command hierarchies (e.g., `claude mcp add`, `gemini extensions install`)
   - Context-aware parameter completion
   - Dynamic completion (e.g., listing configured MCP servers)

3. **Registration and aliases** - At file end:
   - `compdef` directives to register completion functions
   - Command aliases for convenience

### Key Patterns

**State machine pattern for command routing:**
```zsh
_arguments -C \
    '1: :->command' \
    '*::arg:->args'

case $state in
    command)
        _describe -t commands 'commands' commands
        ;;
    args)
        case $line[1] in
            subcommand)
                _tool_subcommand
                ;;
        esac
        ;;
esac
```

**Dynamic completion with command substitution:**
```zsh
'*::server:( $(claude mcp list 2>/dev/null | grep -E "^\s+\w+" | awk "{print \$1}") )'
```

**Multi-level nested completions:**
- Level 0: `claude` → `_claude_code`
- Level 1: `claude mcp` → `_claude_code_mcp`
- Level 2: `claude plugin marketplace` → `_claude_code_plugin_marketplace`

## Development Commands

### Testing Completions

```bash
# Clear completion cache (required after changes)
rm -f ~/.zcompdump*

# Reload shell to test changes
exec zsh

# Test specific completion manually
claude [Tab]
claude mcp [Tab]
```

### Installation & Uninstallation

```bash
# Install plugins for all detected CLI tools
bash install.sh

# Uninstall all plugins
bash uninstall.sh

# Check synchronization with actual CLI commands
bash check_sync.sh
```

### Validation Workflow

The `check_sync.sh` script is critical for maintenance:
- Recursively executes each command with `--help`
- Extracts subcommands and options from help output
- Compares against plugin definitions
- Reports missing completions

Run this after CLI tool updates to identify missing completions.

## Plugin File Conventions

### File Naming
- Plugin file: `<tool>/<tool>.plugin.zsh`
- Function naming: `_<tool>_<subcommand_path>` (underscores replace spaces and hyphens)
  - Example: `claude plugin marketplace add` → `_claude_code_plugin_marketplace`

### Command Definitions

Commands are defined as arrays with format `'command:description'`:
```zsh
commands=(
    'mcp:Configure and manage MCP servers'
    'plugin:Manage Claude Code plugins'
)
```

For commands with aliases, use pipe separator:
```zsh
'install|i:Install plugin'
'remove|rm:Remove marketplace'
```

### Option Arguments Syntax

Common `_arguments` patterns used across plugins:

```zsh
# Boolean flag
{-h,--help}'[Show help]'

# Option with value
'--model[Specify model]'

# Option with enumerated choices
'--scope[Configuration scope]:scope:(user project local)'

# Option with file completion
'--mcp-config[Load MCP servers]:config:_files'

# Option with directory completion
'--add-dir[Additional directories]:directory:_files -/'

# Option with dynamic completion
'*::server:( $(claude mcp list 2>/dev/null | grep -E "^\s+\w+" | awk "{print \$1}") )'
```

## Installation Script Architecture

The `install.sh` script handles:

1. **Environment detection** - Checks for Oh My Zsh and Zsh installation
2. **CLI tool detection** - Identifies which AI tools are installed via `command -v`
3. **Selective installation** - Only installs plugins for detected tools
4. **Smart .zshrc updates** - Supports both single-line and multi-line `plugins=()` format
5. **Backup creation** - Creates timestamped backups before modifications
6. **Symbolic link handling** - Properly resolves and updates symlinked `.zshrc` files

### .zshrc Update Logic

The script handles two formats:

**Single-line:**
```bash
plugins=(git brew claude)
```

**Multi-line:**
```bash
plugins=(
  git
  brew
  claude
)
```

Uses `sed` for single-line and `awk` for multi-line insertion.

## Maintenance Guidelines


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gravtice/ohmyzsh-ai-cli-plugins](https://github.com/gravtice/ohmyzsh-ai-cli-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
