---
trigger: always_on
description: handles = function() -> Schema[],          -- List schemas this matcher handles
---

<!-- markdownlint-disable MD013 -->

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## Project Overview

yaml-companion.nvim is a Neovim plugin that enhances YAML editing by
automatically detecting and managing JSON schemas for YAML files. It acts as a
companion to the yaml-language-server (yamlls), providing intelligent schema
detection and selection capabilities.

## Development Commands

```bash
make lint                # Check code style with StyLua and Selene
make test                # Run Plenary test suite (headless Neovim)
make prepare             # Setup dev environment (clone deps, install stylua, yaml-language-server)
make generate-kubernetes # Regenerate Kubernetes schema files
```

To run a single test file:

```bash
nvim --headless --noplugin -u tests/minimal_init.vim \
  -c "PlenaryBustedFile tests/schema_spec.lua"
```

## Architecture

### Core Flow

1. User opens YAML file → LSP client (yamlls) attaches
2. LSP sends `yaml/schema/store/initialized` event
3. `context.autodiscover()` tries schema sources in order:
   - LSP-provided schema
   - User-defined schemas from config
   - Matcher-detected schemas (Kubernetes, cloud-init)
   - SchemaStore schemas from LSP
4. If matched, updates LSP config with schema override
5. User can manually select via `vim.ui.select`

### Module Responsibilities

| Module                                   | Purpose                                                                           |
| ---------------------------------------- | --------------------------------------------------------------------------------- |
| `lua/yaml-companion/init.lua`            | Public API: `setup()`, `get_buf_schema()`, `set_buf_schema()`, `open_ui_select()` |
| `lua/yaml-companion/context/init.lua`    | Buffer state management, autodiscovery, LSP sync                                  |
| `lua/yaml-companion/schema.lua`          | Schema resolution from multiple sources                                           |
| `lua/yaml-companion/lsp/`                | LSP communication (requests, handlers, utils)                                     |
| `lua/yaml-companion/_matchers/init.lua`  | Matcher loading/registration (lazy loading via metatable)                         |
| `lua/yaml-companion/builtin/kubernetes/` | K8s detection (searches for `kind:` field)                                        |
| `lua/yaml-companion/builtin/cloud_init/` | cloud-config detection (searches for `#cloud-config` header)                      |

### Matcher Interface

Custom matchers must implement:

```lua
{
  match = function(bufnr) -> Schema | nil,   -- Return schema if file matches
  handles = function() -> Schema[],          -- List schemas this matcher handles
  health = function() -> nil,                -- Optional: :checkhealth integration
}
```

### Schema Object Structure

```lua
{
  name = "Schema Name",
  uri = "https://example.com/schema.json"
}
```

## Code Style

- Formatter: StyLua (config in `stylua.toml`)
- Linter: Selene (config in `selene.toml`, Neovim globals in `neovim.yaml`)
- 2-space indentation, 100 column width
- The `undefined-global vim` warnings are expected in Neovim plugins

## Testing

Tests use Plenary's busted-style test framework. Test files are in `tests/` directory:

- `yaml-companion_spec.lua` - Integration tests
- `schema_spec.lua` - Schema resolution tests

Tests require `plenary.nvim` cloned as a sibling to this repo (done by `make prepare`). Neovim 0.11+ is required for `vim.lsp.config` support.

## EmmyLua Annotations

**CRITICAL:** Documentation is auto-generated from EmmyLua annotations. Every
public function, configuration option, and data structure MUST be properly
annotated. Missing or incorrect annotations will result in incomplete or wrong
documentation.

### What Goes Where

#### In `lua/yaml-companion/meta.lua` (Shared Types)

Add to `meta.lua` when the type is:

- **Shared across modules** (used in multiple files)
- **Part of the public API** (configuration, return types, callback parameters)
- **Complex data structures** (classes with multiple fields)

```lua
---@class MyNewConfig
---@field enabled boolean Enable the feature
---@field timeout number Timeout in milliseconds

---@class MyReturnType
---@field success boolean Whether operation succeeded
---@field data string|nil Result data if successful
```

#### In-Place (Source Files)

Add annotations directly in source files for:

- **Function signatures** (parameters, return types, descriptions)
- **Local types** only used within that file
- **Module-level variables**

### Function Annotation Format

**Every public function MUST have complete annotations:**

```lua
--- Brief description of what the function does.
--- Additional details can go on subsequent lines.
---@param bufnr number Buffer number to operate on
---@param schema Schema Schema to apply
---@param opts? ApplySchemaOpts Optional configuration
---@return boolean success Whether the operation succeeded
---@return string|nil error Error message if failed
function M.apply_schema(bufnr, schema, opts)
  -- implementation
end
```

### Annotation Requirements

1. **Description line** - First line(s) without `@` describe the function

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mosheavni/yaml-companion.nvim](https://github.com/mosheavni/yaml-companion.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
