---
trigger: always_on
description: **agentic.nvim** is a Neovim plugin that emulates Cursor AI IDE behavior,
---

# Agents Guide

**agentic.nvim** is a Neovim plugin that emulates Cursor AI IDE behavior,
providing AI-driven code assistance through a chat interface.

## Nested instructions

Read these before touching the matching area:

- `@lua/agentic/acp/AGENTS.md` - ACP client, tool calls, permissions,
  providers
- `@tests/AGENTS.md` - test framework, TDD workflow, assertions, helpers

## Reasoning Preference

Prefer retrieval-led reasoning (reading files, searching the codebase) over
pre-training-led reasoning. Training data may be outdated, always verify
against actual code.

## CRITICAL: No Assumptions - Gather Context First

**NEVER make assumptions. ALWAYS gather context before decisions or
suggestions.** Read relevant files, search for existing patterns, verify
types. If you haven't read the relevant code, you don't have enough context.

Forbidden phrases: "this probably...", "I assume...", "it should...", "you
might need to...", "based on similar projects...". Never suggest partial
implementations expecting the user to fill gaps.

## CRITICAL: Multi-Tabpage Architecture

**EVERY FEATURE MUST BE MULTI-TAB SAFE.** This plugin supports one session
instance per tabpage.

### Architecture overview

- `SessionRegistry` maps `tab_page_id -> SessionManager`
- 1 ACP provider instance (single subprocess, shared across tabpages, managed
  by `AgentInstance`)
- 1 ACP session ID per tabpage (ACP supports multiple but only one is active
  per tab)
- 1 `SessionManager` + 1 `ChatWidget` per tabpage (full UI isolation)

Each tabpage has its own: ACP session ID, chat widget (buffers, windows,
state), status animation, permission manager, file list, code selection.

### Implementation requirements

- **NEVER use module-level shared state** for per-tabpage runtime data
  - WRONG: `local current_session = nil` (single for all tabs)
  - RIGHT: Store in tabpage-scoped instances
  - Module-level constants OK for truly global config: `local CONFIG = {}`

- **Namespaces are GLOBAL, extmarks are BUFFER-SCOPED**
  - Module-level namespace constants are fine. `nvim_create_namespace()` is
    idempotent (same name = same ID globally). Isolation comes from buffer
    separation.
  - Clear with
    `vim.api.nvim_buf_clear_namespace(bufnr, ns_id, start_line, end_line)`

  ```lua
  -- Module level (shared namespace ID is OK)
  local NS_ANIMATION = vim.api.nvim_create_namespace("agentic_animation")

  function Animation:new(bufnr)
      return { bufnr = bufnr }
  end

  vim.api.nvim_buf_set_extmark(self.bufnr, NS_ANIMATION, ...)
  vim.api.nvim_buf_clear_namespace(self.bufnr, NS_ANIMATION, 0, -1)
  ```

- **Highlight groups are GLOBAL** (shared across all tabpages). Defined once
  in `lua/agentic/theme.lua`. Use namespaces to control WHERE highlights
  appear, not to isolate definitions.

- **Scoped storage:** use the correct accessor

  | Scope   | Accessor         | Purpose          | Example                          |
  | ------- | ---------------- | ---------------- | -------------------------------- |
  | Buffer  | `vim.b[bufnr]`   | Custom variables | `vim.b[bufnr].my_state = {}`     |
  | Buffer  | `vim.bo[bufnr]`  | Built-in options | `vim.bo[bufnr].filetype = "lua"` |
  | Window  | `vim.w[winid]`   | Custom variables | `vim.w[winid].my_state = {}`     |
  | Window  | `vim.wo[winid]`  | Built-in options | `vim.wo[winid].number = true`    |
  | Tabpage | `vim.t[tabpage]` | Custom variables | `vim.t[tabpage].my_state = {}`   |

  `vim.b`/`vim.w`/`vim.t` are custom variables (Vimscript `b:`/`w:`/`t:`).
  `vim.bo`/`vim.wo` are built-in options (`:setlocal`). State is auto-cleaned
  when scope is deleted. Invalid option names in `vim.bo`/`vim.wo` throw.

- **Get tabpage ID:** `self.tab_page_id` in instance methods; from buffer:
  `vim.api.nvim_win_get_tabpage(vim.fn.bufwinid(bufnr))`; current:
  `vim.api.nvim_get_current_tabpage()`.

- **Buffers/windows are tabpage-specific.** Never assume global existence.
  Use `vim.api.nvim_tabpage_*` when needed.

- **Window creation and validation must be tab-scoped.** When checking if a
  window exists or creating a new one, scope the lookup to the session's
  tabpage. Never query windows globally (e.g. `vim.api.nvim_list_wins()`) and
  assume a hit belongs to the current session. Use
  `vim.api.nvim_tabpage_list_wins(self.tab_page_id)` and validate that the
  window's tabpage matches before using it.

- **Autocommands must be tabpage-aware.** Prefer buffer-local:
  `vim.api.nvim_create_autocmd(..., { buffer = bufnr })`. Filter by tabpage
  in global autocommands.

- **Keymaps must be buffer-local.** Use
  `BufHelpers.keymap_set(bufnr, "n", "key", fn)`. NEVER use global keymaps.

### Logger

- **NEVER use `vim.notify` directly.** Always use `Logger.notify` to avoid
  fast context errors.
- Logger only has `debug()`, `debug_to_file()`, and `notify()`. No `warn()`,
  `error()`, or `info()`. `debug()`/`debug_to_file()` output depends on
  `Config.debug`.

## Code Style

### LuaCATS annotations

Use a space after `---` for both descriptions and annotations. Use `@private`
or `@protected` for internal details. Do NOT write meaningful parameter/
return descriptions unless requested. Group related annotations together.

```lua
--- Brief description of the class
--- @class MyClass

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carlos-algms/agentic.nvim](https://github.com/carlos-algms/agentic.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
