---
trigger: always_on
description: Neovim plugin (Lua) for [neocrush](https://github.com/taigrr/neocrush) LSP integration. Provides flash highlights on AI edits, auto-focus of edited files, a Crush terminal manager, cursor/selection sync, and a Telescope-based AI locations picker.
---

# AGENTS.md - neocrush.nvim

## Project Overview

Neovim plugin (Lua) for [neocrush](https://github.com/taigrr/neocrush) LSP integration. Provides flash highlights on AI edits, auto-focus of edited files, a Crush terminal manager, cursor/selection sync, and a Telescope-based AI locations picker.

**Requirements**: Neovim >= 0.10, `neocrush` binary, `crush` CLI, `glaze.nvim` for binary management, optional `telescope.nvim`.

## Commands

```bash
make test       # Run tests (requires nvim + plenary.nvim)
make lint       # Check formatting with stylua
make format     # Auto-format lua/ and tests/ with stylua
make demo       # Generate demo GIF (requires vhs: brew install vhs)
```

### Test Details

Tests use [plenary.nvim](https://github.com/nvim-lua/plenary.nvim) busted-style runner. They run headless inside Neovim:

```bash
nvim --headless --noplugin -u scripts/minimal_init.vim \
  -c "PlenaryBustedDirectory tests/ { minimal_init = './scripts/minimal_init.vim' }"
```

The minimal init at `scripts/minimal_init.vim` adds plenary.nvim from `~/.local/share/nvim/lazy/plenary.nvim/` to runtimepath. Tests use `describe`, `it`, `before_each`, `after_each`, `assert` globals (declared in `.luarc.json`).

**CI**: Only the `lint` job runs in GitHub Actions (`.github/workflows/ci.yml`). The unit test job is commented out. The VHS demo workflow is manual dispatch only.

## Project Structure

```
lua/neocrush/
  init.lua        # Thin orchestrator: types, config, auto-focus API, setup(), delegates to submodules
  terminal.lua    # Terminal state & management: open/close/toggle/focus/restart/paste/cancel/logs
  lsp.lua         # LSP client, workspace/applyEdit handler, flash highlights, cursor/selection sync
  commands.lua    # User command registration and keybinding setup
  health.lua      # :checkhealth neocrush implementation
  install.lua     # Simple utility: is_installed() helper (binary management via glaze.nvim)
  locations.lua   # Telescope picker for AI-annotated code locations (with quickfix fallback)
  cvm/            # Crush Version Manager directory
    init.lua      # CVM entry point: config, setup, version detection, install functions
    releases.lua  # Telescope picker for GitHub releases
    local.lua     # Telescope picker for local repo commits
plugin/
  neocrush.lua    # Plugin loader (guard, version check, binary check)
tests/
  neocrush_spec.lua   # Tests for setup() and auto_focus API
  install_spec.lua    # Tests for install.is_installed()
  cvm_spec.lua        # Tests for CVM setup and version detection
scripts/
  minimal_init.vim    # Test harness init (loads plenary + plugin)
  vhs_init.lua        # VHS demo recording init
doc/
  neocrush.txt        # Vim help documentation
demo/
  demo.tape           # VHS tape for generating demo GIF
```

## Code Conventions

### Formatting (StyLua)

Configured in `.stylua.toml`:

- **Indent**: 2 spaces
- **Column width**: 120
- **Quote style**: `AutoPreferSingle` (single quotes preferred)
- **Call parentheses**: `None` (omit parens on single-string/table args)
- **Line endings**: Unix

This means function calls look like `vim.cmd 'startinsert'` not `vim.cmd('startinsert')`, and `require 'neocrush'` not `require('neocrush')`. Multi-arg calls still use parens normally.

### Lua Style

- **Runtime**: LuaJIT (configured in `.luarc.json`)
- **Type annotations**: EmmyLua-style `---@class`, `---@field`, `---@param`, `---@return`, `---@type`
- **Module pattern**: Each file returns a table `M = {}` with public functions
- **Private functions**: Local functions above `M`, not on the module table
- **Test helpers**: Exposed as `M._name` (underscore prefix) for unit testing — see `M._is_file_window`, `M._find_edit_target_window`, `M._flash_range`, `M._is_handler_installed`
- **Section separators**: `-------------------------------------------------------------------------------` comment blocks with section titles
- **Doc blocks**: `---@brief [[ ... ---@brief ]]` at file top for vimdoc generation

### Naming

- **Functions**: `snake_case` for both public and private
- **Variables**: `snake_case`
- **Constants**: `snake_case` (e.g., `local ns = ...`)
- **User commands**: `CamelCase` with `Crush` prefix (e.g., `:CrushToggle`, `:CrushFocusOn`)
- **Autocommand groups**: `Neocrush` prefix + `CamelCase` (e.g., `NeocrushCursorSync`, `NeocrushLspAttach`)
- **Highlight namespace**: `neocrush-highlight`

### API Pattern

- Public API exposed via `require('neocrush')` module table
- `setup(opts)` merges user config with defaults via `vim.tbl_deep_extend('force', ...)`
- User commands are created in `setup()`, not at load time
- LSP starts automatically on `VimEnter`/`BufEnter` (not via lspconfig/Mason)
- Keybindings are opt-in via `opts.keys` table

## Architecture

### Module Organization

The plugin is split into focused submodules, all orchestrated by `init.lua`:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [taigrr/neocrush.nvim](https://github.com/taigrr/neocrush.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
