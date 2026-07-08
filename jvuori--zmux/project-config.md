---
trigger: always_on
description: This skill documents all file locations that must be synchronized:
---

# Rules

## Installation Script Completeness

When tmux config files source other config files (e.g., `keybindings.conf` sources `lock-mode-bindings.conf`), ensure **all sourced files are copied during installation and updates**. Missing sourced files will cause "No such file or directory" errors on startup.

**Action**: Always check that both `install.sh` and `update.sh` copy all configuration files that are referenced via `source-file` directives.

## Script File Completeness

All scripts in the `scripts/` directory that are used by keybindings, systemd service, or other components must be copied during installation and updates.

**Critical scripts that must be copied**:

- `systemd-tmux-start.sh` - Used by XDG autostart
- `tmux-start.sh` - Used by WezTerm/terminal emulators
- `swap-pane-left.sh`, `swap-pane-right.sh` - Used by move mode keybindings
- `lock-mode-indicator.sh`, `toggle-lock-mode.sh` - Used by lock mode
- All other helper scripts referenced in keybindings.conf

## CRITICAL: Avoid Hardcoded Paths

**NEVER** use absolute paths like `/home/username/...` in ANY files. This is a critical portability issue.

### Configuration Files and Scripts

Always use:

- `~` or `$HOME` for home directory references
- Relative paths from `~/.config/tmux/` where appropriate

### Desktop Entry Files (.desktop)

For XDG autostart desktop files, paths in `Exec=` must use one of these methods:

1. **Wrap with shell expansion** (PREFERRED):

   ```ini
   Exec=sh -c "$HOME/.config/tmux/scripts/systemd-tmux-start.sh"
   ```

2. **Use tilde expansion** (if supported):
   ```ini
   Exec=~/.config/tmux/scripts/systemd-tmux-start.sh
   ```

**NEVER** use:

```ini
Exec=/home/username/.config/tmux/scripts/systemd-tmux-start.sh  # ❌ WRONG
```

### Test Files

Test scripts must also use portable paths:

- Use `$HOME` instead of `/home/username`
- Use `$PWD` or `$(pwd)` for current directory
- Example: `tmux new-session -d -s "$TEST_SESSION" -c "$HOME"`

### Documentation

When writing documentation or examples:

- Use `~/.config/...` or `$HOME/.config/...` in examples
- Never include actual usernames in path examples
- If showing output, sanitize usernames to `$USER` or generic placeholders

## Never Modify ~/.config Directly

**NEVER** create, modify, or delete files directly in `~/.config`. All configuration files must live in this project repository and be deployed via the installation/update scripts.

**Action**: Make all changes within this project directory, then run `./update.sh` to apply them to `~/.config` and verify the desired behavior.

## Pane Program Save/Restore Architecture

Program state is read directly from the **tmux-resurrect save file** — no separate save step needed.

tmux-resurrect already captures the full command for each pane (via `/proc/<pid>/cmdline`, the same technique as `linux_procfs.sh` strategy). tmux-continuum auto-saves this file every ~15 minutes. A systemd user service (`tmux-shutdown-save.service`) runs a final resurrect save at logout/shutdown to capture any state from the last auto-save cycle.

- **`scripts/restore-pane-apps.sh`** — runs on `client-attached` and after Ctrl+a Ctrl+r. Parses the resurrect `last` symlink for pane program data and re-launches each pane's program. Skips panes already running a non-shell (prevents double-restore on re-attach).

The resurrect file format (tab-separated, 11 fields per `pane` line):
```
pane | session | window | win_active | win_flags | pane_idx | pane_title | :dir | pane_active | cmd | :full_cmd
```
`restore-pane-apps.sh` uses field 10 (`cmd`) and field 11 (`:full_cmd`, strip the leading colon).

### Restore logic per tool type

**Generic programs**: re-launched with the full saved command as-is (`vim notes.txt`, `htop`, `lazygit`, etc.).

**Shells** (`bash`, `zsh`, etc.): skipped — pane is already at a prompt.

**Blocklisted** (`dd`, `mkfs`, `fdisk`, `apt`, etc.): never auto-restarted.

**Claude Code**: add `--continue` unless the command already contains a session flag (`--continue`, `--resume`, `--session-id`, `--from-pr`). Strip one-time flags that must not replay: `--fork-session` (would fork again) and `--worktree`/`-w` (would create another worktree). Skip non-interactive modes (`--print`/`-p`, `--bg`).

**Cursor Agent / Copilot**: use the saved command as-is. Their session UUIDs live in the tool's own state and survive reboots, so `--resume=UUID` remains valid.

### Adding a new tool with special restore logic

Add a `case` entry in `restore-pane-apps.sh` matching the tool's `pane_current_command` value (the process basename). No save-side changes needed — the resurrect file captures all programs generically.

## Key Binding Architecture: tmux vs ZLE

When adding a new Ctrl+key binding that works both inside and outside tmux, there are **two separate layers** that must be kept consistent:

1. **tmux layer** — `bind -n C-x switch-client -T <mode>` in `keybindings.conf`
2. **ZLE layer** — `bindkey '^X' <widget>` generated by `setup-shell.sh` into `~/.config/zmux/shell-config`

### The Core Rule

**Any ZLE widget that handles a key tmux also handles must be guarded with `$TMUX`.**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvuori/zmux](https://github.com/jvuori/zmux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
