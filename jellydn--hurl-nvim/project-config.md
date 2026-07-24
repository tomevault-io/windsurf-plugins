---
trigger: always_on
description: **hurl.nvim** is a Neovim plugin that enables developers to run HTTP requests directly from `.hurl` files within their editor. The plugin provides a seamless API development workflow by executing requests and displaying responses without leaving Neovim.
---

# AGENTS.md

## 📦 Project Overview

**hurl.nvim** is a Neovim plugin that enables developers to run HTTP requests directly from `.hurl` files within their editor. The plugin provides a seamless API development workflow by executing requests and displaying responses without leaving Neovim.

- **Repository:** [jellydn/hurl.nvim](https://github.com/jellydn/hurl.nvim)
- **Primary Language:** Lua
- **Key Dependencies:** `nui.nvim`, `plenary.nvim`, `nvim-treesitter`
- **Test Framework:** `vusted`
- **Build System:** `Makefile`
- **CI:** GitHub Actions
- **Issue Tracking:** GitHub Issues
- **Additional Docs:** `README.md` for user documentation

______________________________________________________________________

## 🗂️ Repository Structure

```
hurl.nvim/
├── lua/hurl/               # Main plugin code
│   ├── init.lua           # Main setup and configuration
│   ├── main.lua           # Core functionality and command registration
│   ├── popup.lua          # Popup window implementation
│   ├── split.lua          # Split view implementation
│   ├── http_utils.lua     # HTTP request utilities
│   ├── utils.lua          # General utility functions
│   ├── git_utils.lua      # Git-related utilities
│   ├── history.lua        # Request history management
│   ├── health.lua         # Health check implementation
│   ├── codelens.lua       # CodeLens integration
│   └── vlog.lua           # Logging utilities
├── test/                  # Vusted-based tests
│   ├── hurl_spec.lua
│   ├── plugin_spec.lua
│   └── hurl_parser_spec.lua
├── example/               # Example .hurl files for testing
├── doc/                   # Vim help documentation
├── .github/               # CI, issue templates, workflows
├── README.md              # Main user documentation
├── CHANGELOG.md           # Version history
├── Makefile               # Build and test commands
└── version.txt            # Version info
```

______________________________________________________________________

## 🚦 Quick Start

1. **Install core dependencies:**

   ```bash
   # For testing and development
   make install
   ```

2. **Run tests:**

   ```bash
   make test                    # Run all tests (requires vusted)
   ```

3. **Lint and format:**

   ```bash
   # Uses stylua for formatting (if available)
   stylua lua/ test/
   ```

4. **Try the plugin:**

   ```lua
   -- Basic setup in init.lua
   require('hurl').setup({
     debug = false,
     mode = 'split',
     show_notification = false,
     formatters = {
       json = { 'jq' },
       html = { 'prettier', '--parser', 'html' },
     },
   })
   ```

______________________________________________________________________

## 🧑‍💻 Development Guidelines

### 1. **Code Style & Quality**

- **Lua 5.1+** (Neovim compatible)
- **Type annotations:** Use EmmyLua format for function documentation
- **Naming:** Use snake_case for functions and variables, PascalCase for modules
- **Error handling:** Use `pcall` for operations that might fail
- **Logging:** Use the built-in logging system (`utils.log_*`)

### 2. **Plugin Architecture**

- **Main entry:** `init.lua` handles setup and configuration
- **Core logic:** `main.lua` registers commands and manages plugin lifecycle
- **UI components:** Separate popup and split view implementations
- **Utilities:** Modular utility functions for HTTP, Git, and general operations
- **Configuration:** Global config stored in `_HURL_GLOBAL_CONFIG`
- **Health checks:** Implement checks in `health.lua`

### 3. **HTTP Request Handling**

- **External dependency:** Uses external `hurl` command for actual HTTP requests
- **Request parsing:** Parse `.hurl` files using treesitter or custom parsers
- **Environment support:** Support for `vars.env` files and variable substitution
- **Response formatting:** Multiple formatters (jq, prettier, tidy)
- **Error handling:** Graceful handling of network errors and timeouts

### 4. **Testing**

- **Test framework:** Use `vusted` for Lua testing
- **Test files:** Place in `test/` directory with `*_spec.lua` naming
- **Coverage:** Test both success and failure scenarios
- **Mocking:** Mock external dependencies (hurl command, file system)
- **CI:** Automated testing via GitHub Actions

### 5. **Documentation**

- **README.md:** Comprehensive user documentation with examples
- **Vim help:** Generate help documentation in `doc/`
- **EmmyLua annotations:** Document function signatures and types
- **Code comments:** Explain complex logic and algorithms
- **Changelog:** Maintain version history in `CHANGELOG.md`

### 6. **Configuration System**

- **Default config:** Provide sensible defaults in `init.lua`
- **User config:** Support deep merging with `vim.tbl_deep_extend`
- **Validation:** Validate user input and provide helpful error messages
- **Backwards compatibility:** Handle deprecated options gracefully

### 7. **UI Implementation**

- **Display modes:** Support both popup and split view modes
- **Key mappings:** Consistent keybindings across UI components
- **Responsive design:** Handle window resizing and repositioning

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jellydn/hurl.nvim](https://github.com/jellydn/hurl.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
