---
trigger: always_on
description: generates named vterm-based terminal commands.
---

# Agent Instructions

This project uses **bd** (beads) for issue tracking. Run `bd prime` for full workflow context.

## Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work atomically
bd close <id>         # Complete work
bd dolt push          # Push beads data to remote
```

## Non-Interactive Shell Commands

**ALWAYS use non-interactive flags** with file operations to avoid hanging on confirmation prompts.

Shell commands like `cp`, `mv`, and `rm` may be aliased to include `-i` (interactive) mode on some systems, causing the agent to hang indefinitely waiting for y/n input.

**Use these forms instead:**
```bash
# Force overwrite without prompting
cp -f source dest           # NOT: cp source dest
mv -f source dest           # NOT: mv source dest
rm -f file                  # NOT: rm file

# For recursive operations
rm -rf directory            # NOT: rm -r directory
cp -rf source dest          # NOT: cp -r source dest
```

**Other commands that may prompt:**
- `scp` - use `-o BatchMode=yes` for non-interactive
- `ssh` - use `-o BatchMode=yes` to fail instead of prompting
- `apt-get` - use `-y` flag
- `brew` - use `HOMEBREW_NO_AUTO_UPDATE=1` env var

## Project: vtermux.el

Single-file Emacs package providing `vtermux-define`, a macro that
generates named vterm-based terminal commands.

### File Architecture

- `vtermux.el` (415 lines) — the entire package
- `vtermux-test.el` (558 lines) — ERT test suite (44 tests)

### Core Macro: `vtermux-define`

**(vtermux-define NAME &key :program :buffer-name :args :directory :bind :dispatch)**

Generates five interactive commands per definition:

| Generated command | Behavior |
|---|---|
| `NAME` | Launch instance. If none exist, creates one. If any exist, prompts for label. |
| `NAME-new` | Always create new instance, always prompts for label. |
| `NAME-select` | Pick live instance via `completing-read`. |
| `NAME-next` | Cycle forward through instances in current scope. |
| `NAME-prev` | Cycle backward through instances in current scope. |

Also creates per-definition customization vars:
- `NAME-program`, `NAME-buffer-name`, `NAME-args`, `NAME-command-directory`
- `NAME-buffer-list` (defvar, holds live buffer objects)

### Private Helper Functions

| Function | Key signature | Purpose |
|---|---|---|
| `vtermux--command-directory` | (&optional method prompt) | Resolves working dir via `:project`/`:buffer`/`:prompt`. Falls back to prompt on error. |
| `vtermux--next-label` | (buffers) | Auto-numbers labels: finds smallest missing positive integer from existing buffer names matching ` (N)*` suffix. |
| `vtermux--format-buffer-name` | (bufname directory &optional label) | Formats `*<base> - <dir>*` or `*<base> - <dir> (<label>)*`. |
| `vtermux--buffers` | (bufname buf-list &optional directory) | Filters live buffers matching prefix. When directory is nil, returns all live. |
| `vtermux--create-buffer` | (prog bufname args buf-list-sym directory &optional label) | Creates vterm buffer in directory. Sets sentinel to kill on process exit. Adds kill-buffer-hook to clean buffer-list. |
| `vtermux--launch` | (prog bufname args buf-list-sym directory) | If buffers exist in scope, prompts for label and creates new. Otherwise creates first instance. |
| `vtermux--launch-new` | (prog bufname args buf-list-sym directory) | Always creates new, always prompts for label. |
| `vtermux--select` | (prog buf-list-sym) | `completing-read` from live buffers. |
| `vtermux--cycle` | (prog bufname args buf-list-sym directory direction offset) | Scoped cycling. If no buffers in scope, delegates to `vtermux--launch`. |

### Buffer Naming Convention

```
*<base> - <dir>*                   — unnamed (first instance)
*<base> - <dir> (<label>)*         — labeled (second+ instance)
```

Example: `*btop - ~/projects/myapp*`, `*btop - ~/projects/myapp (1)*`

### Directory Resolution

Three methods, resolved in `vtermux--command-directory`:

| Method | What it resolves to |
|---|---|
| `:project` | `(project-root (project-current))` |
| `:buffer` | `default-directory` |
| `:prompt` | `read-directory-name` |

Resolution order: `C-u` prefix → per-def `:directory` → global `vtermux-command-directory`.

On error (e.g., no project found), falls back to `read-directory-name`.

### Label Auto-Numbering (`vtermux--next-label`)

- Parses ` (N)*` suffix from existing buffer names in scope.
- Returns smallest missing positive integer as string.
- Example: given labels 1, 2, 4 → returns "3".
- If no labels exist, returns "1".

### Buffer Lifecycle

1. **Creation**: `vtermux--create-buffer` calls `generate-new-buffer`, enters `vterm-mode`, sets `vterm-shell` to program+args.
2. **Exit**: Process sentinel kills buffer when process finishes/exits (if `vtermux-kill-buffer-on-exit` is non-nil).
3. **Cleanup**: `kill-buffer-hook` (buffer-local) removes the buffer from `NAME-buffer-list`.
4. All interactions with the buffer list use `cl-remove-if-not` with `buffer-live-p`, so stale entries are harmless but still cleaned by the hook.

### Edge Cases in Cycling (`vtermux--cycle`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pcmantz/vtermux](https://github.com/pcmantz/vtermux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
