---
trigger: always_on
description: Handles:
---

# AGENTS.md - AI Agent Guidelines for codecompanion-gitcommit.nvim

> Guidelines for AI agents (Claude, GPT, Copilot, etc.) working with this codebase.

## Project Overview

**codecompanion-gitcommit.nvim** is a Neovim plugin extension for [CodeCompanion](https://github.com/olimorris/codecompanion.nvim) that provides:
- AI-powered Git commit message generation following Conventional Commits
- Comprehensive Git workflow tools (`@{git_read}`, `@{git_edit}`, `@{git_bot}`)
- AI-powered release notes generation
- Multi-language support for commit messages
- Smart buffer integration for gitcommit filetype

### Compatibility
- Requires CodeCompanion **v18.0+**
- Lua 5.1+ / LuaJIT
- Neovim 0.9+
- Multi-platform (Linux, macOS, WSL, Native Windows)

---

## Architecture

```
codecompanion-gitcommit.nvim/
├── lua/codecompanion/_extensions/gitcommit/
│   ├── init.lua           # Main entry point, extension setup, exports
│   ├── config.lua         # Default configuration options
│   ├── git.lua            # Core Git operations (diff, commit, repository detection)
│   ├── generator.lua      # LLM-based commit message generation
│   ├── buffer.lua         # Gitcommit buffer integration & auto-generation
│   ├── ui.lua             # Floating window UI for commit message display
│   ├── langs.lua          # Multi-language support for commit messages
│   ├── types.lua          # Type definitions (LuaLS annotations)
│   ├── prompts/
│   │   └── release_notes.lua  # Prompts for AI release notes generation
│   └── tools/
│       ├── git.lua            # GitTool class - low-level git operations
│       ├── git_read.lua       # Read-only git operations tool schema
│       ├── git_edit.lua       # Write-access git operations tool schema
│       ├── ai_release_notes.lua # AI-powered release notes tool
│       └── validation.lua     # Parameter validation utilities
├── doc/
│   └── codecompanion-gitcommit.txt  # Vim help documentation
├── scripts/
│   └── download_codecompanion.ps1   # Development script
└── .github/workflows/
    └── stylua-check.yml     # CI for code formatting
```

### Module Dependency Graph

```
init.lua (entry point)
    ├── config.lua
    ├── git.lua ──────────────────┐
    ├── generator.lua             │
    ├── buffer.lua ───────────────┼── git.lua (core)
    ├── ui.lua                    │
    ├── langs.lua                 │
    └── tools/                    │
        ├── git.lua ──────────────┘
        ├── git_read.lua ─────────┬── tools/git.lua
        ├── git_edit.lua ─────────┤
        ├── ai_release_notes.lua ─┴── prompts/release_notes.lua
        └── validation.lua
```

---

## Key Components

### 1. Extension Entry Point (`init.lua`)

The main module that:
- Sets up the extension with CodeCompanion
- Registers tools (`git_read`, `git_edit`, `ai_release_notes`) and tool groups (`git_bot`)
- Creates Vim commands (`:CodeCompanionGitCommit`, `:CCGitCommit`)
- Adds slash commands (`/gitcommit`)
- Exposes programmatic API via `exports`

**Chat Config Access:**
```lua
-- v18+ uses interactions
if codecompanion_config.interactions and codecompanion_config.interactions.chat then
  return codecompanion_config.interactions.chat
end
```

### 2. Git Core Module (`git.lua`)

Handles:
- Repository detection (`is_repository()`)
- Diff retrieval with file filtering (`get_staged_diff()`, `get_contextual_diff()`)
- Amend detection (`is_amending()`)
- Commit history retrieval (`get_commit_history()`)
- Glob pattern matching for file exclusion

**Important:** This module maintains its own `config` state set via `Git.setup()`.

### 3. Generator (`generator.lua`)

LLM integration for commit message generation:
- Supports both HTTP and ACP (Anthropic Claude Protocol) adapters
- Handles streaming responses
- Cleans markdown code blocks from LLM output
- Creates structured prompts with commit history context

### 4. Tools System (`tools/`)

CodeCompanion tool implementations following the tool schema pattern:

| Tool | File | Purpose |
|------|------|---------|
| `git_read` | `git_read.lua` | 16 read-only operations (status, log, diff, etc.) |
| `git_edit` | `git_edit.lua` | 20 write operations (stage, commit, push, etc.) |

| `ai_release_notes` | `ai_release_notes.lua` | AI-powered release notes from commit history |

**Tool Schema Structure:**
```lua
Tool.schema = {
  type = "function",
  ["function"] = {
    name = "tool_name",
    parameters = { ... },
    strict = true,  -- Enforce strict parameter validation
  },
}
Tool.system_prompt = [[...]]  -- LLM context
Tool.cmds = { function(self, args) ... end }  -- Execution
Tool.handlers = { setup, on_exit }
Tool.output = { prompt, success, error, rejected }
Tool.opts = { require_approval_before }  -- Tool options
```

### 5. Validation (`tools/validation.lua`)

Centralized parameter validation with consistent error formatting:
- `require_string()`, `optional_string()`
- `require_array()`, `optional_integer()`, `optional_boolean()`
- `require_enum()`, `first_error()`

---

## Coding Conventions

### Style
- **Formatter:** StyLua (config in `stylua.toml`)
- **Line width:** 120 characters
- **Indentation:** 2 spaces
- **Quotes:** Double quotes for strings

### Naming
- Modules: `PascalCase` for classes (`GitTool`, `Generator`)
- Functions: `snake_case` (`get_staged_diff`, `format_git_response`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jinzhongjia/codecompanion-gitcommit.nvim](https://github.com/jinzhongjia/codecompanion-gitcommit.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
