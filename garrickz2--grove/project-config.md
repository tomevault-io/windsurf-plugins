---
trigger: always_on
description: Grove is a Rust TUI application for managing Git Worktrees + tmux sessions, designed for parallel AI coding agent workflows.
---

# Grove - Project Guide

## Overview

Grove is a Rust TUI application for managing Git Worktrees + tmux sessions, designed for parallel AI coding agent workflows.

## Tech Stack

- **Language**: Rust 2021 Edition
- **TUI Framework**: ratatui 0.29
- **Terminal Backend**: crossterm 0.28
- **Config**: toml + serde
- **Time**: chrono

## Project Structure

```
src/
├── main.rs              # Entry point, terminal initialization
├── app.rs               # App state management, core logic
├── event.rs             # Keyboard event handling
├── cli/
│   ├── mod.rs           # CLI subcommand definitions
│   ├── agent.rs         # `grove agent` commands (status/summary/todo/notes)
│   ├── hooks.rs         # `grove hooks` notification commands
│   └── init.rs          # Worktree AI integration setup 
├── git/
│   └── mod.rs           # Git command wrappers
├── tmux/
│   └── mod.rs           # tmux session management
├── storage/
│   ├── mod.rs
│   ├── config.rs        # Global config read/write
│   ├── tasks.rs         # Task data persistence
│   ├── workspace.rs     # Project registration
│   ├── ai_data.rs       # AI summary & TODO persistence
│   └── notes.rs         # Task notes persistence
├── model/
│   ├── mod.rs
│   ├── worktree.rs      # Worktree/Task data structures
│   ├── workspace.rs     # Workspace state (grid navigation, filtering)
│   └── loader.rs        # Data loading logic
├── theme/
│   ├── mod.rs           # Theme enum, ThemeColors struct
│   ├── colors.rs        # 8 theme color definitions (including accent palettes)
│   └── detect.rs        # System dark/light mode detection
└── ui/
    ├── mod.rs
    ├── workspace.rs     # Workspace view
    ├── project.rs       # Project view
    └── components/      # Reusable UI components
        ├── workspace_list.rs  # Card grid with gradient color blocks
        ├── worktree_list.rs
        ├── preview_panel.rs   # Side panel (Git/AI/Notes tabs)
        ├── header.rs
        ├── footer.rs
        ├── tabs.rs
        ├── toast.rs
        ├── theme_selector.rs
        ├── help_panel.rs
        ├── new_task_dialog.rs
        ├── add_project_dialog.rs
        ├── delete_project_dialog.rs
        ├── confirm_dialog.rs
        ├── input_confirm_dialog.rs
        ├── branch_selector.rs
        ├── merge_dialog.rs
        └── ...
```

### Web Frontend Structure

```
grove-web/src/
├── main.tsx                # Entry point
├── App.tsx                 # Root component
├── api/                    # Backend API clients
│   ├── client.ts
│   ├── config.ts
│   └── index.ts
├── components/             # React components
│   ├── Blitz/              # Blitz mode (cross-project view)
│   │   └── BlitzPage.tsx   # ~675 lines (refactored)
│   ├── Tasks/              # Zen mode (single-project view)
│   │   └── TasksPage.tsx   # ~610 lines (refactored)
│   ├── Config/             # Settings page
│   ├── Terminal/           # Terminal integration
│   ├── Editor/             # Code editor integration
│   └── ui/                 # Reusable UI components
│       ├── ContextMenu.tsx
│       ├── Dialog.tsx
│       └── ...
├── hooks/                  # Custom React hooks
│   ├── index.ts
│   ├── useHotkeys.ts       # Keyboard shortcuts
│   ├── useTaskPageState.ts # Page state management (~250 lines)
│   ├── useTaskNavigation.ts # j/k navigation (~70 lines)
│   ├── usePostMergeArchive.ts # Post-merge workflow (~160 lines)
│   └── useTaskOperations.ts # All task operations (~450 lines)
├── utils/                  # Utility functions
│   ├── archiveHelpers.tsx  # Archive confirmation builder
│   └── taskOperationUtils.ts # Context menu builder
└── data/
    └── types.ts            # TypeScript type definitions
```

## Core Concepts

### Hierarchy

```
Workspace (multiple projects)
└── Project (single git repo)
    └── Task (worktree + tmux session)
```

### Entry Logic

- Run `grove` outside git repo → Workspace view
- Run `grove` inside git repo → Project view

### Task States

- `Live (●)` — tmux session running
- `Idle (○)` — no active session
- `Merged (✓)` — merged to target branch

## Commands

```bash
cargo build            # Build
cargo run              # Run
cargo check            # Check
cargo build --release  # Release build
```

## Data Storage

All data stored in `~/.grove/`:

```
~/.grove/
├── config.toml           # Theme config
└── projects/
    └── <path-hash>/      # Hash of project path
        ├── project.toml  # Project metadata
        ├── tasks.toml    # Active tasks
        ├── archived.toml # Archived tasks
        ├── ai/
        │   └── <task-id>/
        │       ├── summary.md   # AI agent summary
        │       └── todo.json    # AI agent TODO list
        └── notes/
            └── <task-id>.md     # User-provided task notes
```

## Development Guidelines

### Completion Summary (IMPORTANT)

**After completing any code modifications, ALWAYS provide a clear summary to the user:**

1. **Frontend Changes** — If `grove-web/` was modified:
   - Explicitly state: "✅ `npm run build` executed successfully" (if you ran it)
   - OR state: "⚠️ You need to run `npm run build` in `grove-web/`" (if you didn't run it)

2. **Backend Changes** — If Rust code (`src/`) was modified:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [GarrickZ2/grove](https://github.com/GarrickZ2/grove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
