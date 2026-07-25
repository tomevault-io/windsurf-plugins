---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Commands
- **Install / bootstrap** – Run the shell script that sets up Vim/Neovim and installs plugins.
  ```bash
  bash install.sh
  ```
- **Update plugins** – Refresh all vim‑plug plugins.
  ```bash
  vim +PlugUpdate +qall   # for Vim
  nvim +PlugUpdate +qall  # for Neovim
  ```
- **Open Vim/Neovim** – Start the editor with the current configuration.
  ```bash
  vim      # or nvim
  ```
- **Generate unit tests** – In Vim, the `:GenerateTest` command (provided by `rc/vim-ai.vim`) can create test stubs for a selected code block.
- **Run LSP diagnostics** – Use Neovim's built‑in LSP client (configured via `lua/nvim_lsp.lua`).
- **Install language servers** – Via Mason: `MasonInstall <server>`.
- **Feature management**:
  - `:call te#feat#feat_dyn_enable(1)` - Enable a feature
  - `:call te#feat#feat_dyn_enable(0)` - Disable a feature
  - `<Leader>fe` - Enable feature interactively
  - `<Leader>fd` - Disable feature interactively

## High‑Level Architecture

### Overview
The repository is a **Vim/Neovim configuration** named *vinux* - a highly modular, cross-platform IDE-like setup for Vim.

### Root Layout
```
├── install.sh          # Bootstrap script
├── init.lua            # Neovim entry point (sources vimrc)
├── vimrc               # Main configuration entry point
├── CLAUDE.md           # This file
├── readme.md           # Project documentation
├── bin/                # Helper scripts (e, t, v.sh, v.bat, v.scpt)
├── config/             # Configuration files (ollama.vim)
├── doc/                # Vim documentation (vinux_*.txt)
├── format/             # Formatting tools (clang-format-linux)
├── lua/                # Neovim Lua modules
├── rc/                 # Vim script configuration modules
└── autoload/te/        # Core utility functions
```

### Core Module Structure

#### 1. Entry Points
- **vimrc**: Main Vim configuration - sets up runtime paths, initializes features, loads plugins via vim-plug
- **init.lua**: Neovim-specific entry that sources vimrc for shared configuration

#### 2. Core Utility Library (`autoload/te/`)
| Module | Purpose | Key Functions |
|--------|---------|---------------|
| `feat.vim` | Feature management & plugin loading | `te#feat#init_all()`, `te#feat#feat_enable()`, `te#feat#source_rc()` |
| `utils.vim` | General utilities | `te#utils#confirm()`, `te#utils#EchoWarning()`, `te#utils#quit_win()` |
| `env.vim` | Environment detection | `te#env#IsNvim()`, `te#env#IsMac()`, `te#env#SupportFloatingWindows()` |
| `ai.vim` | AI/LLM integration | `te#ai#get_provider_url()`, `te#ai#get_model_name()` |
| `terminal.vim` | Terminal management | `te#terminal#shell_pop()`, `te#terminal#jump_to_floating_win()` |
| `project.vim` | Project management | `te#project#create_project()`, `te#project#build_project()` |
| `pg.vim` | Tags/cscope management | `te#pg#gen_cs_tags()` |
| `complete.vim` | Code completion helpers | `te#complete#goto_def()` |
| `lsp.vim` | LSP integration | `te#lsp#diagnostics_info()` |
| `git.vim` | Git integration | `te#git#get_cur_br_name()` |
| `tmux.vim` | Tmux integration | `te#tmux#run_command()` |

#### 3. Configuration Modules (`rc/`)
| File | Purpose |
|------|---------|
| `basic.vim` | Basic plugins (file explorer, outline, terminal) |
| `options.vim` | Core Vim options (tabs, indentation, statusline) |
| `mappings.vim` | Key mappings |
| `complete.vim` | Completion plugin configuration |
| `lsp.vim` | LSP configuration |
| `git.vim` | Git plugin setup |
| `c.vim` | C/C++ specific settings |
| `colors.vim` | Color scheme configuration |
| `aerial.vim`, `fzf.vim`, `ctrlp.vim`, etc. | Plugin-specific configs |

#### 4. Neovim Lua Modules (`lua/`)
- `nvim_lsp.lua` - LSP client setup
- `mason_setup.lua` - Mason package manager
- `nvim_cmp.lua` - Completion plugin
- `nvim_dap.lua` - Debug adapter protocol
- `nvim_telescope.lua` - Telescope configuration
- `nvim_tree.lua` - File explorer
- `copilot_chat_setup.lua` - AI chat integration
- `aerial_setup.lua` - Outline plugin

### Plugin Management
- Uses **vim-plug** (`autoload/plug.vim`)
- Plugins are declared lazily with `{ 'on': [...] }` or `{ 'for': [...] }`
- Feature-based loading: `g:feat_enable_*` variables control which modules load

### Modularity & Feature System
The config uses a **feature toggle system**:
- Features are controlled via `g:feat_enable_*` global variables
- Features include: `complete`, `jump`, `tmux`, `git`, `c`, `vim`, `gui`, `tools`, `edit`, `frontend`, `help`, `basic`, `airline`, `writing`, `zsh`, `fun`, `lsp`, `debug`, `ai`
- Each feature corresponds to a `.vim` file in `rc/`

### Key Components

For detailed information about features and plugins, see the official documentation:

```vim
:help vinux_config    " Feature configuration and plugin options
:help vinux_plugins   " Plugin documentation
:help vinux_api       " API reference
```

## Useful Notes

### Compatibility
- **Vim**: 7.3.1058+ recommended
- **Neovim**: 0.5+ recommended (for full features)
- **Platforms**: Linux, macOS, Windows

### Key Mappings


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tracyone/vinux](https://github.com/tracyone/vinux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
