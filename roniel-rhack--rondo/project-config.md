---
trigger: always_on
description: **RonDO** is a modern terminal user interface (TUI) productivity app built with **Go** and the **Charm** ecosystem. It combines task management with a daily journal in a single keyboard-driven interface.
---

# RonDO - Project Guide

## Project Overview

**RonDO** is a modern terminal user interface (TUI) productivity app built with **Go** and the **Charm** ecosystem. It combines task management with a daily journal in a single keyboard-driven interface.

### Tech Stack
- **Language**: Go 1.23+
- **TUI Framework**: Bubbletea v1.3.x (MVU pattern)
- **Components**: Bubbles v1.0.x (list, viewport, help, key, textinput)
- **Styling**: Lip Gloss v1.1.x (including lipgloss/table for styled CLI tables)
- **Forms**: Huh v0.8.x (task add/edit dialogs)
- **CLI Framework**: Cobra v1.10.x (subcommands, flags, completions)
- **Database**: SQLite via modernc.org/sqlite (CGO-free)
- **Terminal Detection**: charmbracelet/x/term (width), go-isatty (TTY detection)

### Key Dependencies (`go.mod`)
```
github.com/charmbracelet/bubbletea v1.3.10
github.com/charmbracelet/bubbles v1.0.0
github.com/charmbracelet/lipgloss v1.1.0
github.com/charmbracelet/huh v0.8.0
github.com/charmbracelet/x/term v0.2.2
github.com/spf13/cobra v1.10.2
modernc.org/sqlite v1.46.1
```

---

## Application Features

### Task Management
- **CRUD**: Create, view, edit, and delete tasks
- **Subtask Support**: Tasks can have subtasks with independent completion state
- **Status Tracking**: Cycle tasks between Pending, In Progress, Done
- **Tab Navigation**: All / Active / Done tabs with counts
- **Task Details**: Right panel shows description, subtasks, progress bar
- **Date Tracking**: Automatic creation date + optional due date
- **Sorting**: Sort by creation date (F1), due date (F2), or priority (F3)
- **Search**: Fuzzy search/filter via built-in bubbles list filtering
- **Priority Levels**: Low, Medium, High, Urgent with color coding
- **Tags**: Comma-separated tag support
- **Recurring Tasks**: Daily, weekly, monthly, or yearly recurrence; auto-spawns next occurrence on completion
- **Task Dependencies**: Block tasks by other task IDs; CLI via `--blocks` flag on `add`/`edit`, `--clear-blocks` on `edit`
- **Delete Guard**: Refuses to delete tasks that block others; CLI returns exit code 1 unless `--cascade`; TUI shows yellow warning dialog requiring double-confirm
- **Self-Block Guard**: Store-level rejection of self-referential dependencies
- **Metadata**: Structured key-value pairs (`--meta key=value`) stored as JSON; filterable with AND logic via `rondo list --meta key=value`
- **Time Logging**: Log time spent on tasks with optional notes

### Journal
- **Daily Notes**: One note per calendar day, auto-created
- **Entries**: Multiple timestamped entries per note
- **Edit/Delete Entries**: Cursor-based entry selection with edit and delete
- **Hide/Restore Notes**: Hide old notes, toggle visibility with `H`
- **Smart Date Labels**: "Today", "Yesterday", weekday names, or full dates
- **Search Notes**: Filter notes by date

### Pomodoro Timer
- **Full Pomodoro cycle**: Work → Short Break → Work → ... → Long Break (4-session sets)
- **Session types**: Work (🍅), Short Break (☕), Long Break (🌿) with distinct colors
- **Cycle indicator**: ●●●○ showing progress through 4-session set
- **Configurable**: Durations, daily goal, auto-start breaks via `P` settings form or config.json
- **Notifications**: Terminal bell on phase completion
- **Stats**: Daily goal progress, weekly summary, streak tracking in `G` overlay
- **Task linkage**: Focus sessions linked to selected task

### General
- Keyboard-driven navigation (vim-style j/k + arrows)
- Two-panel layout with focus switching (1/2 keys), resizable with `<`/`>`
- Status bar with context-sensitive keybinding hints
- Confirmation dialogs for all destructive actions
- Huh forms with validation for all input
- Adaptive color theme (auto-detects light/dark terminal)
- Persistence via SQLite at `~/.todo-app/todo.db`
- Auto backups at `~/.todo-app/backups/`
- Config file at `~/.todo-app/config.json`
- Undo last destructive action (`Ctrl+Z`)

### CLI Mode
Full Cobra-based CLI with all features available as subcommands:
- **Global flags**: `--format table|json`, `--json`, `--quiet`, `--no-color`
- **TTY-aware output**: Styled tables with Unicode borders + ANSI colors when TTY; plain tabwriter when piped
- **Commands**: `add`, `done`, `list`, `show`, `edit`, `delete`, `status`, `subtask`, `timelog`, `note`, `recur`, `journal`, `focus`, `export`, `stats`, `config`, `batch`, `completion`, `skill`
- **Shell completions**: bash, zsh, fish, powershell via `rondo completion`
- **Claude Code skill**: `rondo skill install [--project]` / `rondo skill uninstall [--project]`

### UI Layout
```
┌──────────────────────────────────────────────────────────────────┐
│  RonDO  │  All (7)  │  Active (4)  │  Done (3)  │  Journal (5) │
├────────────────────────┬─────────────────────────────────────────┤
│  1: Panel (list)       │  2: Panel (detail/viewport)             │
│  - Custom delegate     │  - Context-sensitive content             │
│  - Fuzzy search        │  - Cursor selection in both panels       │
│  - Colored items       │  - Subtasks/entries with progress        │
├────────────────────────┴─────────────────────────────────────────┤

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [roniel-rhack/rondo](https://github.com/roniel-rhack/rondo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-23 -->
