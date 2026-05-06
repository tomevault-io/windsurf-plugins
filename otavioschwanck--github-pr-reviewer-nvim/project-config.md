---
trigger: always_on
description: This document contains essential information about the architecture, patterns, and important details of the `neovim-pr-reviewer` plugin.
---

# Claude Code - Guide for AIs and Developers

This document contains essential information about the architecture, patterns, and important details of the `neovim-pr-reviewer` plugin.

## Overview

Neovim/Lua plugin for reviewing GitHub Pull Requests directly in the editor, with support for:
- Navigation between modified files
- Diff visualization (unified and split/side-by-side)
- Comment system (inline and global)
- PR approval/rejection
- Marking files as "viewed"

## Main Architecture

### Main File
- **`lua/github-pr-reviewer/init.lua`** (~4400 lines): Contains all plugin logic

### Auxiliary Modules
- **`lua/github-pr-reviewer/github.lua`**: GitHub GraphQL API interaction
- **`lua/github-pr-reviewer/ui.lua`**: UI components (pickers, menus)

## Important Concepts

### 1. Diff Visualization Modes

#### Unified Mode (Default)
- Single file with inline highlights
- **Visual indicators**: `│` (vertical bar) in the signs column for modified lines
- **Important namespaces**:
  - `changes_ns_id`: Change indicators (│)
  - `diff_ns_id`: Inline diff highlights (green/red)
  - `hunk_hints_ns_id`: Navigation hints (←/→)

#### Split Mode (Side-by-Side)
- **Two buffers side by side**:
  - Left: `[BEFORE] filename` (git base version)
  - Right: Current file (PR version)
- **Uses Vim's native diff mode**: `diffthis`, `diffoff`, `diffupdate`
- **CRITICAL**: Do NOT add extmarks/visual indicators when in split mode!
- **State stored in**: `M._split_view_state`

**Control variable**: `M._diff_view_mode` ("unified" | "split")

### 2. Buffer System

#### Special Buffers
- **`[BEFORE] <path>`**: Read-only buffer with git base content
- **Review Buffer**: Special buffer with list of modified files

#### Buffer State Tracking
```lua
M._buffer_changes[bufnr] = lines      -- Modified lines
M._buffer_hunks[bufnr] = hunks        -- Change hunks (for navigation)
M._buffer_comments[bufnr] = comments  -- Comments in buffer
M._buffer_stats[bufnr] = stats        -- Statistics (+X/-Y)
M._buffer_jumped[bufnr] = bool        -- Whether already jumped to first change
M._buffer_keymaps_saved[bufnr] = bool -- Whether keymaps already configured
```

### 3. Floating Windows (Popups)

Three floating windows displayed on the right:
1. **General Info**: File X/Total, viewed status
2. **Buffer Info**: Changes count, stats, comments
3. **Keymaps**: Available shortcuts

**IMPORTANT**: The floats depend on `M._buffer_hunks[bufnr]` being populated!

### 4. Navigation System

#### Automatic Navigation (C-h/C-l)
```lua
-- Flow when navigating with C-h or C-l:
1. restore_unified_view()           -- Closes previous split
2. open_file_safe(file)             -- Opens file with `edit`
3. defer_fn (50ms):
   - M._diff_view_mode = "unified"  -- Reset mode
   - M.toggle_diff_view()           -- Creates split if necessary
```

**Important flag**: `M._opening_file` (prevents autocmd interference)

#### Auto-Fix Split (Manual Buffer Change)

When user manually changes buffer (`:b` or fzf), `BufEnter` detects and runs `M.fix_vsplit()` automatically:

```lua
-- Conditions for auto-fix:
- M._diff_view_mode == "split"
- NOT M._opening_file (not navigating)
- Buffer is not [BEFORE]
- Split state doesn't match current buffer
- File is part of the PR
```

## Common Problems and Solutions

### Problem: Duplicate Highlights in Split Mode

**Cause**: Extmarks being added to buffers that are in diff mode, causing duplicate highlights (vim diff + our indicators).

**Solution**:
1. Clear ALL namespaces before creating split: `vim.api.nvim_buf_clear_namespace(bufnr, -1, 0, -1)`
2. Don't add extmarks when `M._diff_view_mode == "split"`
3. `BufEnter` must skip `load_inline_diff_for_buffer` in split mode

### Problem: Floats Don't Appear in Split Mode

**Cause**: `load_changes_for_buffer` wasn't being called in split mode, so `M._buffer_hunks[bufnr]` was empty.

**Solution**:
- **ALWAYS** call `load_changes_for_buffer` (to populate hunks)
- **CONDITIONALLY** add extmarks based on `M._diff_view_mode`

```lua
-- In load_changes_for_buffer:
if M._diff_view_mode ~= "split" then
  -- Only add visual indicators (│) if NOT in split mode
  vim.api.nvim_buf_set_extmark(...)
end
```

### Problem: Split with Wrong Highlights After Manual Change

**Cause**: Vim's internal state (diff cache) not being cleaned when recreating split.

**Solution** (`M.fix_vsplit()`):
```lua
1. restore_unified_view()                    -- Close split
2. vim.cmd("diffoff!")                       -- Turn off diff completely
3. vim.api.nvim_buf_clear_namespace(bufnr, -1, 0, -1)  -- Clear EVERYTHING
4. vim.cmd("edit " .. file_path)             -- Reload file
5. defer_fn(50ms):
   - M._diff_view_mode = "unified"           -- Reset mode
   - M.toggle_diff_view()                    -- Recreate clean split
```

**CRITICAL**: The 50ms delay is essential! Vim needs to process the `edit` before creating the split.

## Patterns and Conventions

### 1. Namespace IDs
```lua
local ns_id = vim.api.nvim_create_namespace("pr_review_comments")
local changes_ns_id = vim.api.nvim_create_namespace("pr_review_changes")
local diff_ns_id = vim.api.nvim_create_namespace("pr_review_diff")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [otavioschwanck/github-pr-reviewer.nvim](https://github.com/otavioschwanck/github-pr-reviewer.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
