---
trigger: always_on
description: vim-quickui is a pure VimScript TUI component library for Vim 8.2+ and Neovim 0.4+. It provides Borland/Turbo C++ flavored popup-based UI widgets: **menu**, **listbox**, **inputbox**, **context menu**, **textbox**, **preview window**, **terminal**, **confirm dialog**, and a data-driven **dialog** system. No `+python` or external dependencies required.
---

# vim-quickui — Coding Agent Guide

## Project Overview

vim-quickui is a pure VimScript TUI component library for Vim 8.2+ and Neovim 0.4+. It provides Borland/Turbo C++ flavored popup-based UI widgets: **menu**, **listbox**, **inputbox**, **context menu**, **textbox**, **preview window**, **terminal**, **confirm dialog**, and a data-driven **dialog** system. No `+python` or external dependencies required.

## Repository Layout

```
plugin/quickui.vim          — Entry point: version, :QuickUI command, theme setup
autoload/quickui/
  core.vim                  — Platform abstraction, buffer pool, utility functions
  utils.vim                 — Item parsing (&hotkey), border drawing, text helpers
  window.vim                — Window abstraction (Vim popup / Neovim floating win)
  readline.vim              — Single-line text editing engine (Unicode-aware)
  highlight.vim             — Highlight group introspection & manipulation
  style.vim                 — Border style global setting
  menu.vim                  — Top menubar widget
  listbox.vim               — Scrollable listbox widget
  context.vim               — Context menu widget
  input.vim                 — Single-line input box (uses readline + window)
  textbox.vim               — Multi-line text display popup
  preview.vim               — Preview window (auto-close on CursorMoved)
  terminal.vim              — Terminal in popup window
  confirm.vim               — Simple confirm dialog
  dialog.vim                — Data-driven dialog with 7 control types
  tools.vim                 — High-level tools (buffer switcher, function list, etc.)
  palette.vim / palette9.vim — Color palette utilities
  tags.vim                  — Tag-related utilities
  command.vim               — :QuickUI command dispatcher
colors/quickui/*.vim        — Color scheme files (borland, gruvbox, solarized, etc.)
docs/                       — Internal module reference docs (for developer use)
test/                       — Test scripts (interactive + headless)
MANUAL.md                   — User manual (public API reference)
DIALOG.md                   — Dialog system user guide
```

## Architecture & Module Dependencies

```
                    plugin/quickui.vim  (entry, theme, :QuickUI command)
                            |
              +-------------+-------------+
              |             |             |
          core.vim      utils.vim     style.vim
          (platform,    (item parse,  (border
           buffer pool,  border gen,   style)
           win_execute)  text utils)
              |             |
              +------+------+
                     |
                 window.vim   (unified popup/floating window abstraction)
                     |
         +-----------+-----------+
         |           |           |
    readline.vim  highlight.vim  |
    (line editor) (hl groups)    |
         |           |           |
         +-----+-----+          |
               |                 |
           input.vim         textbox.vim  listbox.vim  context.vim  menu.vim
           (input box)      (text view)  (scrollable)  (ctx menu)  (menubar)
               |
           dialog.vim  (data-driven dialog: label/input/radio/check/button/separator/dropdown)
               |
           tools.vim   (buffer switcher, function list, help viewer, etc.)
```

### Key Dependency Rules

- `window.vim` depends on `core.vim` + `utils.vim` — never bypass this layer
- `readline.vim` is **UI-independent** — manages edit state only, no window operations
- `dialog.vim` depends on `window.vim` + `readline.vim` + `highlight.vim`
- All modules use `g:quickui#core#has_nvim` for platform detection (never re-detect)

## Code Style & Conventions

### VimScript Patterns

- **Language**: Pure legacy VimScript (no Vim9script). Target Vim 8.2+ and Neovim 0.4+
- **Modeline**: Every `.vim` file starts with a header block and contains a modeline:
  ```vim
  " vim: set ts=4 sw=4 tw=78 noet :
  ```
  or:
  ```vim
  " vim: set noet fenc=utf-8 ff=unix sts=4 sw=4 ts=4 :
  ```
- **Indentation**: Tabs, not spaces. Tab width = 4
- **Naming**:
  - Public functions: `quickui#module#function_name()` (autoload convention)
  - Script-local functions: `s:function_name()` — use `abort` keyword
  - Script-local variables: `s:var_name`
  - Global flags: `g:quickui_option_name` (user-facing) or `g:quickui#module#var` (internal)
- **OOP pattern**: Objects are implemented as dictionaries with function references. Class templates are script-local dicts (e.g., `s:readline`, `s:window`). Constructor functions return `deepcopy()` of the template:
  ```vim
  let s:myclass = {}
  function! s:myclass.method() dict
      " self refers to the instance
  endfunc
  function! quickui#module#new()
      return deepcopy(s:myclass)
  endfunc
  ```
- **Section dividers**: Use comment blocks with `"------` or `"======` separators


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [skywind3000/vim-quickui](https://github.com/skywind3000/vim-quickui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
