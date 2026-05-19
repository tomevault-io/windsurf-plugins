---
trigger: always_on
description: Dooing is a minimalist todo list manager for Neovim. It provides a floating window UI for managing tasks with tags, priorities, due dates, nested subtasks, and per-project todo lists. Target: Neovim users who want lightweight task tracking without leaving the editor.
---

# CLAUDE.md

Dooing is a minimalist todo list manager for Neovim. It provides a floating window UI for managing tasks with tags, priorities, due dates, nested subtasks, and per-project todo lists. Target: Neovim users who want lightweight task tracking without leaving the editor.

## Tech Stack & Constraints

- **Language:** Lua only (no Vimscript except the 4-line bootstrap in `plugin/dooing.vim`)
- **Runtime:** Neovim ≥ 0.10.0 plugin, managed by [lazy.nvim](https://github.com/folke/lazy.nvim)
- **Dependencies:** None (no luarocks, no build step, no external tools)
- **Testing:** No test framework or CI — all testing is manual (check `:messages` for errors, visual inspection)
- **Linting/Formatting:** No `.luarc.json`, `.stylua.toml`, or `.editorconfig` — follow existing code style

## Architecture

```
plugin/dooing.vim          ← Bootstrap: calls require('dooing').setup()
lua/dooing/
├── init.lua               ← Entry point: setup(), user commands (:Dooing, :DooingLocal, :DooingDue), keymaps
├── config.lua             ← M.defaults + M.setup(opts) merges user config via vim.tbl_deep_extend
├── state.lua              ← Data layer: todo CRUD, persistence (JSON), sorting, filtering, undo, git detection
├── server.lua             ← QR code share server (raw TCP via vim.loop) — self-contained, rarely touched
└── ui/
    ├── init.lua            ← UI coordinator: public API that delegates to sub-modules
    ├── constants.lua       ← Shared mutable state: win/buf IDs, namespace, highlight cache
    ├── highlights.lua      ← Highlight group setup and priority-based coloring
    ├── utils.lua           ← Utility functions: time formatting, time parsing, todo text rendering
    ├── window.lua          ← Main floating window creation, positioning, quick-keys panel
    ├── rendering.lua       ← Todo list rendering and highlight application
    ├── actions.lua         ← Todo CRUD UI operations (new, edit, toggle, delete, import/export, etc.)
    ├── components.lua      ← Sub-windows: help, tags, search, scratchpad
    ├── keymaps.lua         ← Keymap registration for the todo buffer
    ├── calendar.lua        ← Calendar picker for due dates (multi-language)
    └── due_notification.lua ← Due/overdue item notification window
```

### Module Dependency Flow

```
init.lua → config.lua, state.lua, ui/init.lua
ui/init.lua → ui/constants, ui/window, ui/rendering, ui/actions, ui/keymaps, ui/utils
ui/actions.lua → ui/constants, ui/utils, state, config, ui/calendar, server
ui/rendering.lua → ui/constants, ui/utils, ui/highlights, state, config
state.lua → config (for save_path, priorities, nested_tasks settings)
```

All modules are singletons accessed via `require()`. No events or callback systems between modules.

## Data Model

Todos are stored as a **flat JSON array** in a single file (default: `vim.fn.stdpath("data") .. "/dooing_todos.json"`). Nesting is simulated via `parent_id`/`depth` fields — **not** nested JSON.

### Todo Object Fields

| Field              | Type           | Description                                       |
|--------------------|----------------|---------------------------------------------------|
| `id`               | `string`       | Unique ID: `os.time() .. "_" .. math.random()`    |
| `text`             | `string`       | Todo text, may contain `#tags` inline              |
| `done`             | `boolean`      | Completion status                                  |
| `in_progress`      | `boolean`      | In-progress status (3-state cycle: pending → in_progress → done) |
| `category`         | `string`       | First `#tag` extracted from text                   |
| `created_at`       | `number`       | Unix timestamp                                     |
| `completed_at`     | `number\|nil`  | Unix timestamp when marked done                    |
| `priorities`       | `string[]\|nil`| List of priority names (e.g. `{"important","urgent"}`) |
| `estimated_hours`  | `number\|nil`  | Estimated completion time in hours                 |
| `due_at`           | `number\|nil`  | Due date as Unix timestamp (end of day)            |
| `notes`            | `string`       | Scratchpad notes for this todo                     |
| `parent_id`        | `string\|nil`  | ID of parent todo (nil = top-level)                |
| `depth`            | `number`       | Nesting level (0 = top-level)                      |

**Critical rule:** `state.lua` owns all data mutations. Always call `state.save_todos()` after modifying `state.todos`.

## Configuration Pattern

- `config.lua` defines `M.defaults` with all default values
- `M.setup(opts)` merges user config: `vim.tbl_deep_extend("force", M.defaults, opts or {})`
- All runtime access goes through `config.options.*`
- Keymaps can be disabled by setting them to `false` (checked in `init.lua` before `vim.keymap.set`)
- When adding a new config option: add default to `M.defaults`, access via `config.options.your_option`

## Code Conventions

- Use `vim.api.*` for all buffer/window operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [atiladefreitas/dooing](https://github.com/atiladefreitas/dooing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
