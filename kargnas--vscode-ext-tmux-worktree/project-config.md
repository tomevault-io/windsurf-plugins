---
trigger: always_on
description: This document serves as the primary rule file for AI Agents working on this project.
---

# AI Agent Guidelines

This document serves as the primary rule file for AI Agents working on this project.
**ALWAYS** update this file when you discover new patterns or finish significant tasks.

## Instruction
- After every task and changes, install the compiled extension to `code`, `code-insiders`, `antigravity`. DO NOT INCREASE version. (It will be automatically increased when publish, and we don't want to increase it for testing)

## Tested VS Code
- `code` (VS Code)
- `antigravity` (Google Antigraity)
- `cursor` (Cursor)

## 1. Codebase Understanding

### Project Structure
```
.
├── src/                    # VS Code Extension Source (TypeScript)
│   ├── extension.ts        # Extension Entry Point
│   ├── commands/           # Command Implementations
│   ├── providers/          # Tree Data Providers (Sidebar)
│   └── utils/              # Utilities (tmux, git, execution)
├── cli/                    # CLI Tool Source (Go)
│   ├── main.go             # CLI Entry Point
│   ├── internal/ui/        # TUI Implementation (Bubble Tea)
│   └── pkg/                # Shared Packages
├── out/                    # Compiled Extension Output
├── .vscode/                # Editor Configuration
└── resources/              # Icons and Assets
```

### Key Components
- **VS Code Extension**: Manages the "TMUX Worktrees" view in the Activity Bar. It interacts with the `tmux` CLI and `git worktree` commands.
- **CLI (`twt`)**: A terminal user interface (TUI) for managing sessions/worktrees outside of VS Code, built with Bubble Tea.

## 2. Coding Patterns & Best Practices

- **Polymorphism**: Commands must handle `TmuxItem` base class and variants (`TmuxSessionItem`, `InactiveWorktreeItem`, etc.).
- **Path Handling**: Use `getWorktreePath(item)` helper.
- **Canonical Path Matching**: For path equality/deduplication/current-workspace checks, normalize to absolute paths with `~` expansion before comparison (do not collapse symlink aliases via `realpath`).
- **Managed Worktree Location**: Create extension-managed worktrees under `~/.tmux-worktrees/<repo-name-hash>/` by default. Reuse shared helpers for path checks and orphan cleanup instead of hardcoding repo-local `.worktrees` path fragments.
- **Session Namespace**: Build tmux session prefixes from repo-root identity (basename + short path hash), not display repo name alone, to avoid collisions across same-name repositories in different directories.
- **Legacy Session Compatibility Isolation**: Keep legacy session-prefix compatibility logic centralized in `src/utils/sessionCompatibility.ts`; call helpers from commands/providers instead of duplicating fallback checks.
- **Root Detection**: Determine the primary worktree by comparing worktree path to the primary worktree path derived from `git rev-parse --git-common-dir`, not by branch naming, folder basename, or the current workspace folder.
- **Current Workspace Indicator**: Highlight the active VS Code workspace by comparing worktree/session paths against the current workspace folder (not the primary worktree path). Current items should sort to the top and display a `👆` marker in the label.
- **External Worktrees**: If the worktree folder name matches the repo name, derive a unique slug/label from the parent directory.
- **Slug Collision Handling**: Worktree session slugs must be unique by sanitized tmux name. Start with basename-based slug, then disambiguate with parent directory, and finally append a short path hash when collisions remain.
- **Free-Form Task Branches**: `TMUX: New Task` accepts arbitrary valid git branch names, including `/`. Sanitize `/` to `-` for tmux/worktree slugs, and never infer main-vs-task state from a `task/` prefix.
- **Unpublished Task Branches**: When creating a new task branch, do not preconfigure `branch.<name>.remote` / `branch.<name>.merge` before the first push. VS Code SCM treats that as an upstream and may try to sync against a remote branch that does not exist yet. Store only `branch.<name>.vscode-merge-base` with the chosen base branch so SCM diffs stay anchored while **Publish Branch** remains available.
- **Primary `main` Slug Reservation**: Reserve `main` for the primary worktree. If a non-primary worktree or branch would sanitize to `main`, suffix it during slug collision resolution instead of reusing the primary session name.
- **Tree Context Menu**: Use a single TreeItem `contextValue` (`tmuxItem`) for levels 2/3/4 so the same context menu always appears.
- **Error Handling**: Use `try-catch` in TS and check `err != nil` in Go. Fail gracefully and notify the user.
- **Async/Await**: Use `async/await` for all I/O operations in TypeScript.
- **Terminal Creation**: Use `/bin/sh -c 'exec tmux attach ...'` instead of `shellPath: 'tmux'`. Direct `shellPath: 'tmux'` causes VS Code to treat it as a non-standard shell, breaking mouse drag events (pane resize). The `exec` replaces sh with tmux (no extra process), and `-c` avoids sendText race conditions with other extensions.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kargnas/vscode-ext-tmux-worktree](https://github.com/kargnas/vscode-ext-tmux-worktree) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
