---
trigger: always_on
description: This project is managed with **Frame**. AI assistants should follow the rules below to keep documentation up to date.
---

# Frame - Project Instructions

This project is managed with **Frame**. AI assistants should follow the rules below to keep documentation up to date.

> **Note:** This file is named `AGENTS.md` to be AI-tool agnostic. A `CLAUDE.md` symlink is provided for Claude Code compatibility.

---

## Project Overview

Vibe Console - Electron-based IDE for Claude Code. Terminal emulator with file tree, git integration, plugin system.

## Commands

| Command | Description |
|---------|-------------|
| `npm run build` | Bundle renderer and generate macOS DMG |
| `npm run build:renderer` | Bundle renderer with esbuild |
| `npm start` | Build renderer + launch Electron app |
| `npm run dev` | Watch mode + launch app |
| `npm run watch` | esbuild watch (renderer only) |
| `npm run structure` | Update STRUCTURE.json |
| `npm run dist:mac` | Alias of `npm run build` (builds macOS DMG) |

## Architecture

Electron app with a hardened renderer:

- `nodeIntegration: false`
- `contextIsolation: true`
- `sandbox: true`
- `preload: src/main/preload.js`

```
src/
  main/           # Main process (Node.js)
    index.js       # Entry point, window creation, IPC setup
    ptyManager.js  # Multi-terminal PTY management (node-pty)
    fileTree.js    # File tree + file deletion (shell.trashItem)
  renderer/        # Renderer process (bundled by esbuild)
    index.js       # Entry point, module init
    terminalManager.js  # xterm.js terminal instances
    fileTreeUI.js  # File tree with context menu, drag-drop
    githubPanel.js # Git changes, branches, worktrees
    state.js       # Centralized app state
  shared/
    ipcChannels.js # IPC channel constants (single source of truth)
dist/
  renderer.js     # Bundled output
```

## Key Patterns

- IPC channels defined in `src/shared/ipcChannels.js` - always add new channels here
- Circular deps: use `window.*` globals instead of direct imports (see architectureNotes in STRUCTURE.json)
- File deletion uses `shell.trashItem()` (recoverable, moves to trash)
- Drag from file tree to terminal pastes quoted path via custom MIME `application/x-vibeconsole-file`

## Gotchas

- esbuild renames modules on circular dependency - use window globals to avoid
- Electron apps from Finder don't inherit shell PATH - extra paths added in git managers
- With `contextIsolation: true`, expose new capabilities via `src/main/preload.js` (keep IPC allowlist in sync with `src/shared/ipcChannels.js`)

---

## 🧭 Project Navigation

**Read these files at the start of each session:**

1. **STRUCTURE.json** - Module map, which file is where
2. **PROJECT_NOTES.md** - Project vision, past decisions, session notes
3. **tasks.json** - Pending tasks

**Workflow:**
1. Read these files to understand the project and capture context
2. Identify relevant files based on the task
3. Update STRUCTURE.json after making changes (if new modules/files are added)

**Note:** This system doesn't prevent reading code - it just helps you know where to look.

---

## Task Management (tasks.json)

### Task Recognition Rules

**These ARE TASKS - add to tasks.json:**
- When the user requests a feature or change
- Decisions like "Let's do this", "Let's add this", "Improve this"
- Deferred work when we say "We'll do this later", "Let's leave it for now"
- Gaps or improvement opportunities discovered while coding
- Situations requiring bug fixes

**These are NOT TASKS:**
- Error messages and debugging sessions
- Questions, explanations, information exchange
- Temporary experiments and tests
- Work already completed and closed
- Instant fixes (like typo fixes)

### Task Creation Flow

1. Detect task patterns during conversation
2. Ask the user at an appropriate moment: "I identified these tasks from our conversation, should I add them to tasks.json?"
3. If the user approves, add to tasks.json

### Task Structure

```json
{
  "id": "unique-id",
  "title": "Short and clear title (max 60 characters)",
  "description": "AI's detailed explanation - what will be done, how it will be done, which files will be affected",
  "userRequest": "User's original request/prompt - copy exactly",
  "acceptanceCriteria": "When is this task considered complete? List of concrete criteria",
  "notes": "Important notes, decisions, alternatives that came up during discussion",
  "status": "pending | in_progress | completed",
  "priority": "high | medium | low",
  "category": "feature | fix | refactor | docs | test",
  "context": "Session date and context",
  "createdAt": "ISO date",
  "updatedAt": "ISO date",
  "completedAt": "ISO date | null"
}
```

### Task Content Rules

**title:** Short, action-oriented title
- ✅ "Add tasks button to terminal toolbar"
- ❌ "Tasks"

**description:** AI's detailed technical explanation
- What will be done (what)
- How it will be done (how) - brief technical approach
- Which files will be affected
- Minimum 2-3 sentences

**userRequest:** User's original words
- Copy the user's prompt/request exactly
- Important for preserving context
- In "User said: ..." format

**acceptanceCriteria:** Completion criteria
- Concrete, testable items
- "Task is complete when this happens" list

**notes:** Discussion notes (optional)
- Alternatives considered

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nesdesignco/vibeconsole](https://github.com/nesdesignco/vibeconsole) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
