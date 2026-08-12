---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

review.nvim is a Neovim plugin for reviewing AI-generated code changes. `:Review` opens a dedicated tab of floating windows: a sidebar of stacked floats (Branch, Files, Branches, Commits, Comments — configurable via `ui.panels`, though Files cannot be disabled) on the left and a diff pane on the right. It browses git diffs, attaches typed comments to lines, and exports review feedback to the clipboard or tmux (designed for Claude Code workflows).

## Commands

```bash
# Lint
luacheck lua/

# Run all tests
make test

# Run a single test file
make test-file FILE=tests/test_diff.lua

# Format code
stylua lua/
```

## Workflow

After every code change, run `luacheck lua/` and `make test`. Fix ALL luacheck warnings (not just errors) before considering the task done — zero warnings is the target. When adding new features or logic to pure modules (non-UI, non-git-shelling), always add corresponding tests in `tests/`. If a module is testable (pure logic, no vim.api dependencies), it should have tests.

CI (`.github/workflows/ci.yml`) runs three jobs: `make test` on Neovim stable and nightly, `luacheck lua/`, and `stylua --check lua/` (pinned to stylua 2.5.2). Formatting is enforced, so run `stylua lua/` before committing or CI fails.

Behavior changes are documented in three places that are expected to stay in sync: `README.md`, `doc/review.txt` (vimdoc), and this file.

The plugin targets Neovim 0.10+; `plugin/review.lua` hard-refuses to load below that.

## Testing

Uses **mini.test** (from mini.nvim). Dependencies are auto-cloned into `.deps/` (gitignored) by the Makefile.

Test files live in `tests/` and follow the naming convention `test_<module>.lua`. Each test file requires only the modules it needs — the plugin is not loaded globally.

Shared fixtures and factories are in `tests/helpers.lua`.

Tested modules: `comment_types`, `config`, `core/diff`, `core/format`, `core/json_persistence`, `core/paths`, `export/markdown`, `quick_comments/markdown`, `quick_comments/state`, `state`, `ui/panel_keymaps`. `core/git` is tested only for its pure parsers (`tests/test_git_parse.lua` covers name-status and commit-line parsing); everything in it that shells out to git is not.

Not tested (integration-heavy): `core/async`, `core/log`, `core/persistence`, `core/watcher`, `commands`, `health`, `quick_comments/init`, `quick_comments/panel`, `quick_comments/persistence`, `quick_comments/signs`, `ui/*` (except `ui/panel_keymaps`).

## Architecture

```
lua/review/
├── init.lua                    # Public API: setup(), toggle(), open(), close(), export()
├── config.lua                  # Default config merged with user options
├── state.lua                   # Centralized state: comments, files, review status
├── comment_types.lua           # Static comment type definitions (note, fix, question)
├── commands.lua                # :Review command routing (registered from plugin/review.lua)
├── health.lua                  # :checkhealth review
├── core/
│   ├── git.lua                 # Git operations (diffs, status, staging) + pure parsers
│   ├── diff.lua                # Unified diff parsing into structured hunks
│   ├── format.lua              # Date shortening, author initials, UTF-8 truncation
│   ├── paths.lua               # Path helpers (relative paths, fence language, test files)
│   ├── async.lua               # Coroutine-based async utilities
│   ├── log.lua                 # File-based logger (DEBUG/INFO/WARN/ERROR), rotates at 1 MB
│   ├── json_persistence.lua    # JSON file read/write, resolves paths inside the git dir
│   ├── persistence.lua         # Session persistence (wraps json_persistence + state)
│   └── watcher.lua             # Per-directory fs watchers for auto-refresh
├── ui/
│   ├── init.lua                # UI orchestration (open/close/toggle), wires panel callbacks
│   ├── layout.lua              # Floating-window tab layout (sidebar floats + diff pane)
│   ├── file_tree.lua           # Files panel: file list with status icons
│   ├── diff_view.lua           # Diff pane: diff, inline comments, comment input
│   ├── comment_list.lua        # Comments panel
│   ├── commit_list.lua         # Commits panel
│   ├── branch_list.lua         # Branches panel
│   ├── panel_keymaps.lua       # Keymaps shared by the sidebar panels
│   ├── push.lua                # Shared push action
│   ├── highlights.lua          # Highlight groups, re-applied on ColorScheme
│   ├── palette.lua             # Semantic color names the highlight groups are built from
│   ├── help.lua                # Help overlay built from tracked keymaps
│   └── util.lua                # UI utilities (buffer mapper, scrolling, type cycling)
├── quick_comments/
│   ├── init.lua                # Quick comments public API
│   ├── state.lua               # Quick comments state management
│   ├── panel.lua               # Side panel UI for quick comments
│   ├── markdown.lua            # Quick comments markdown export
│   ├── persistence.lua         # Quick comments persistence
│   └── signs.lua               # Gutter signs for quick comments
└── export/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vuki656/review.nvim](https://github.com/vuki656/review.nvim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
