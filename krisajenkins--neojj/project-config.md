---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with
code in this repository.

## NeoJJ - Neovim Plugin for Jujutsu VCS

NeoJJ is a Neovim plugin that provides integration with the Jujutsu (jj) version control system.

## Development Setup

The project uses Nix for development environment. Always enter the Nix shell before development:

```bash
nix develop
```

This provides:

- luacheck (static analysis)
- stylua (code formatting)
- lua-language-server (type checking)
- neovim (testing)

## Common Development Commands

```bash
# Run all checks and tests
make

# Run tests only
make test

# Run a specific test file
make test_file FILE=tests/test_components.lua

# Run linting/static analysis
make typecheck

# Format code
make format
```

## Documentation

### Regenerating Help Tags

After modifying documentation in `doc/`, regenerate the help tags file:

```bash
# From within Neovim
:helptags doc/

# Or from command line
nvim --headless -c "helptags doc/" -c "quit"
```

The `doc/tags` file is tracked in git and should be updated when documentation changes.

## Architecture Overview

### Core Components

1. **UI Component System** (`lua/neojj/lib/ui/`)
   - `component.lua`: Defines the component abstraction with support for folding, interactivity, and custom options
   - `renderer.lua`: Renders components to buffer lines with proper highlight tracking
   - Components are immutable data structures with methods for querying properties

2. **Buffer Management** (`lua/neojj/lib/buffer.lua`)
   - Manages Neovim buffers with component rendering support
   - Handles buffer lifecycle, options, and cursor management
   - Integrates with the renderer for displaying UI components

3. **JJ Integration** (`lua/neojj/lib/jj/`)
   - `cli.lua`: Executes jj commands and parses output
   - `repository.lua`: Repository abstraction with caching and state management
   - `status.lua`: Parses jj status output into structured data
   - Uses async execution with plenary for non-blocking operations

   **Repository resolution is per-buffer.** A `:JJ` command targets the repo
   owning the *current buffer*, not Neovim's working directory, so one session
   can drive several jj projects. `M.get_repo(dir)` is the single resolution
   point: a nil `dir` resolves via `M.current_buffer_dir()`, so **every** entry
   is buffer-aware — the `:JJ` dispatcher (which passes an explicit dir), a
   leader mapping wired straight to `neojj.jj_log` (which calls it with no args,
   so `dir` is nil), or any future caller. `current_buffer_dir()` resolves in
   priority order:
   1. `b:neojj_repo_dir` — a buffer-local tag every NeoJJ view sets to its repo
      root (in `Buffer.create`, from the `cwd = repo.dir` each view passes).
      This is why a `:JJ` command run *inside* a view (which is a `nofile`
      buffer with no path) stays on that view's repo instead of falling back.
   2. The directory of a normal file buffer's file.
   3. `getcwd()` for anything else (terminals, `[No Name]`).

   `JjRepo.instance()` then keys its instance cache by the repository **root**
   (resolved with `util.find_jj_dir`), so all buffers/subdirectories of one repo
   share a single `JjRepo` (and watcher).

4. **Status Buffer** (`lua/neojj/buffers/status/`)
   - `ui.lua`: Creates the component tree for displaying jj status
   - `init.lua`: Manages the status buffer lifecycle and updates
   - Provides interactive UI for viewing repository state

### Key Design Patterns

- **Component-Based UI**: All UI elements are components that can be composed hierarchically
- **Immutable Data**: Components and state are immutable for predictable rendering
- **Async Operations**: JJ commands run asynchronously to avoid blocking the editor
- **Caching**: Repository state is cached to minimize jj command executions
- **Interactive Components**: Components can be marked as `interactive = true` to support cursor-based interaction

### View Stack (`lua/neojj/lib/view_stack.lua`)

Drilling down (log → a change's status → a file) builds a stack of **live**
views. Rather than snapshot each view's (cursor, revision, folds) and re-render
on the way back, the stack keeps every view's buffer alive (they use
`bufhidden = "hide"`) and just switches the shared window between them, so Vim
preserves cursor/fold/scroll state for free.

- `q` / `<esc>` on a NeoJJ view pops one frame, revealing the frame beneath;
  popping the last frame closes the view.
- No-arg `:JJ` returns to the stack from anywhere (raises the top frame; from a
  file opened out of a status view it steps back into that status view).
- A view registers itself as a frame from its `show`/`show_split`/`show_tab`
  entry points (`StatusBuffer:_push_frame` / `LogBuffer:_push_frame`); opening a
  file from the status view pushes a file frame.

**Only log, status and file views are stack frames today.** describe, annotate
and the split terminal are transient and are NOT pushed. **When adding a new
buffer type, consider whether it should be a stack frame** — if it is a view the
user navigates *into* and expects to `q`/`:JJ` back out of, push it onto the
view stack; if it is a transient editor/terminal, leave it off.

### Component Position Tracking System


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [krisajenkins/neojj](https://github.com/krisajenkins/neojj) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
