---
trigger: always_on
description: Dotfiles for a Python/web development Neovim setup with Catppuccin theme.
---

# CLAUDE.md — Neovim Config

Dotfiles for a Python/web development Neovim setup with Catppuccin theme.

## Quick Facts

| Aspect         | Detail                                        |
| -------------- | --------------------------------------------- |
| Plugin manager | lazy.nvim                                     |
| Theme          | Catppuccin (auto: latte/mocha)                |
| Leader key     | `<Space>`                                     |
| Background     | auto (terminal-dependent)                     |
| Indent         | Python: 4 spaces, others: 2                   |
| LSP servers    | pyright (Python), ruff (Python), html, djlint |
| Formatters     | conform.nvim (stylua, prettier, djlint, ruff, sqlfluff) |
| Debug          | nvim-dap + debugpy (Python)                   |
| DB             | vim-dadbod + dadbod-ui                        |
| HTTP Client    | kulala.nvim                                   |
| AI Assistant   | sidekick.nvim                                 |
| Diff Viewer    | diffview.nvim                                 |
| Icon Provider  | mini.icons                                    |
| Commenting     | Comment.nvim (gc/gb)                          |

## Project Structure

```
%LOCALAPPDATA%\nvim\          # Windows config (this repo)
├── init.lua                  # Entry point — bootstraps lazy.nvim, loads vim-options + plugins
├── lua/
│   ├── vim-options.lua       # Core options, leader key, window nav keymaps, filetype overrides
│   ├── plugins.lua           # Empty stub (return {}) — kept for lazy.nvim discovery
│   └── plugins/              # One file per plugin or plugin group
│       ├── comment.lua       # Comment toggle (Comment.nvim, gc/gb)
│       ├── alpha.lua         # Start screen (ASCII art dashboard)
│       ├── aerial.lua        # Code outline tree
│       ├── catppuccin.lua    # Theme setup (auto light/dark)
│       ├── completions.lua   # nvim-cmp + LuaSnip + sources
│       ├── debugging.lua     # nvim-dap + dap-ui + dap-python
│       ├── git-stuff.lua     # vim-fugitive + diffview.nvim + gitsigns
│       ├── lualine.lua       # Status line (with venv display)
│       ├── lsp-config.lua    # Mason + lspconfig (ruff, html, djlint)
│       ├── mini-icons.lua    # Icons (mini.icons)
│       ├── neo-tree.lua      # File explorer + bufferline
│       ├── conform.lua       # Formatters (stylua, prettier, djlint, ruff, sqlfluff)
│       ├── oil.lua           # Directory editing
│       ├── sidekick.lua      # AI assistant (CLI)
│       ├── snacks.lua        # Animations, scroll, indent, LazyGit
│       ├── telescope.lua     # Fuzzy finder (fzf-native + ui-select)
│       ├── treesitter.lua    # Syntax highlighting + indent (uses MSVC 'cl' on Windows)
│       ├── venv-selector.lua # Python venv auto-selection
│       ├── vim-dadbod.lua    # Database UI
│       ├── which-key.lua     # Key binding popup hints
│       ├── kulala.lua        # HTTP client (.http, .rest)
│       └── neotest.lua       # Tests (pytest, nvim-neotest)
└── .gitignore
```

## Plugin Management

Manager: [lazy.nvim](https://github.com/folke/lazy.nvim).

### Adding a New Plugin

1. Create `lua/plugins/<name>.lua`
2. Return a lazy.nvim spec table:

```lua
return {
    "owner/repo",
    lazy = false,
    config = function()
        require("plugin").setup({})
    end,
    keys = { { "<leader>x", "<cmd>PluginCmd<cr>", desc = "Plugin command" } },
    event = "VeryLazy",
}
```

### Style Conventions

- **4-space indentation** everywhere
- Prefer explicit `config = function()` over `opts = {}` when keymaps are involved
- Always include `desc = "..."` on keymaps (shows in which-key)
- Group related keymaps with comments
- One logical plugin (or tightly coupled group) per file

## LSP Configuration

Stack: `mason.nvim` → `mason-lspconfig.nvim` → `nvim-lspconfig`

Current servers (in `lua/plugins/lsp-config.lua`):

- **ruff** — Python linting/formatting
- **html** — HTML language support
- **djlint** — HTML template formatting
- **pyright** — Python type checking

LSP keymaps (active only on LspAttach):

| Key          | Action              |
| ------------ | ------------------- |
| `K`          | Hover documentation |
| `<leader>ld` | Go to definition    |
| `<leader>lr` | Find references     |
| `<leader>la` | Code action         |

## Formatting (conform.nvim)

Defined in `lua/plugins/conform.lua`:

| Formatter              | Language                  |
| ---------------------- | ------------------------- |
| ruff_fix + ruff_format | Python                    |
| stylua                 | Lua                       |
| prettier               | JS/JSON/CSS/Markdown/YAML |
| djlint                 | HTML templates            |
| sqlfluff               | SQL (PostgreSQL)          |

Trigger: `<leader>gf` → `conform.format({ lsp_fallback = true })`

Format on save is enabled automatically.

## Keymaps Overview

### Window Navigation (vim-options.lua)

| Key     | Action       |
| ------- | ------------ |
| `<C-h>` | Window left  |
| `<C-j>` | Window down  |
| `<C-k>` | Window up    |
| `<C-l>` | Window right |

### General

| Key         | Action                 |
| ----------- | ---------------------- |
| `<leader>h` | Clear search highlight |

### Terminal (vim-options.lua)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Petro-lium/p_nvim_config](https://github.com/Petro-lium/p_nvim_config) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
