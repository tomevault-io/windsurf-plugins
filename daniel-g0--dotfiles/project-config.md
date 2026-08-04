---
trigger: always_on
description: Personal dotfiles. Files in this repo are the source of truth — `~/.config/*` are symlinks pointing here. Theme: Tokyo Night across all tools.
---

# CLAUDE.md

## Repo

Personal dotfiles. Files in this repo are the source of truth — `~/.config/*` are symlinks pointing here. Theme: Tokyo Night across all tools.

## Symlink map

| Repo path | Symlinked to |
|-----------|-------------|
| `shell/nushell/` | `~/.config/nushell` |
| `dev/nvim/` | `~/.config/nvim` |
| `shell/starship/starship.toml` | `~/.config/starship.toml` |
| `apps/kitty/` | `~/.config/kitty` |
| `wm/wallpapers/` | `~/.config/wallpapers` |
| `shell/zoxide/config.nu` | `~/.config/zoxide/config.nu` |
| `apps/yazi/` | `~/.config/yazi` |
| `apps/wallrizz/` | `~/.config/WallRizz` |
| `apps/whosthere/` | `~/.config/whosthere` |
| `wm/wayland/hypr/` | `~/.config/hypr` |
| `wm/wayland/waybar/` | `~/.config/waybar` |
| `wm/wayland/swaync/` | `~/.config/swaync` |
| `wm/cursor/icons-default/index.theme` | `~/.icons/default/index.theme` |
| `wm/applications/rofi-filebrowser.desktop` | `~/.local/share/applications/rofi-filebrowser.desktop` |
| `apps/btop/` | `~/.config/btop` |
| `apps/fastfetch/` | `~/.config/fastfetch` |
| `apps/cava/` | `~/.config/cava` |
| `apps/rofi/` | `~/.config/rofi` |
| `dev/claude/settings.json` | `~/.claude/settings.json` |
| `shell/scripts/cava-vibe` | `~/.local/bin/cava-vibe` |
| `system/nixos/configuration.nix` | `/etc/nixos/configuration.nix` (sudo) |

Run `./doller` to (re)create all symlinks. TUI shows status per link, backs up conflicts. `--dry-run` to preview, `--force` to skip prompt. `install.sh` is a shim that calls it.

---

## Nushell

**Config:** `nushell/config.nu` | **Aliases:** `nushell/aliases/<tool>/<tool>-aliases.nu`

### Shell behavior
- Vi mode — block cursor in normal, line in insert; `󱄅 N` / `󱄅 I` indicators
- History: SQLite, 100k entries, per-session, dedup, timestamps, auto-sync on entry
- Zoxide smart directory jumping (`z`, `zi`)
- `$EDITOR` / `$VISUAL` / `buffer_editor` all set to `nvim`

### Hooks
- **Pre-prompt title:** Updates kitty tab bar every prompt: `~/path 󰊢 branch [+staged ~modified ?untracked] | Nf Nd`
- **PWD change:** Auto-clears screen on `cd` to any non-home directory
- **Greeting:** Fastfetch + centered fortune "Quote of the Day" — only fires when `NU_BANNER=1` is in env (set by waybar distro logo kitty launch)

### FZF keybinds
| Key | Action |
|-----|--------|
| `Ctrl+R` | Fuzzy history search |
| `Ctrl+T` | Fuzzy file picker → insert path (fd-powered) |
| `Alt+C` | Fuzzy cd to directory |

### Alias modules
| Module | Key aliases |
|--------|-------------|
| `bat` | `b`, `bn` (numbered), `bp` (plain), `bl` |
| `docker` | 35+ aliases — build, image, container, network, volume; `dsta` (stop all) |
| `git` | 150+ aliases + helpers (`git_current_branch`, `git_main_branch`, worktree, rebase flows) |
| `nixos` | `nixos-edit`, `nixos-re-sw` (rebuild switch), `nixos-garbage` (7d+ cleanup) |
| `nvim` | `n` |
| `rip` | `rm` → rip (trash), `rd` (restore deleted) |
| `utils` | `grep`→rg, `find`→fd, `du`→dust, `top`→btop, `cb` (copy), `cbp` (paste) |
| `chezmoi` | `ch`, `chad`, `chap`, `chd`, `chda`, `chs` |

### Custom completions
Bat, Docker, GitHub CLI, Git, Less, Make, Man, Nix, SSH, Tar, tldr, uv, Zoxide

---

## Starship

**Config:** `starship/starship.toml`

- Single-line, no newline
- **Left:** Language icons (C, Python, Node, Rust, Go, Java, Docker) — appear only in matching project dirs
- **Right:** Username (purple `#bb9af7`; red `#f7768e` for root)
- No cmd_duration, no directory/git in starship (handled by nushell pre-prompt hook + kitty tab bar)

### Git status symbols
`⇡N` ahead · `⇣N` behind · `⇕` diverged · `~` conflicted · `*` stashed · ` ` modified · `++N` staged · `?` untracked · ` ` deleted

---

## Kitty

**Config:** `kitty/kitty.conf` | **Tab bar:** `kitty/tab_bar.py`

- Font: JetBrainsMono Nerd Font 12pt
- Tokyo Night colors, 75% opacity, frosted blur (via Hyprland window rule)
- Scrollback: 10,000 lines; padding 8px; block cursor, no blink
- Repaint 10ms, input delay 3ms, sync to monitor
- Remote control via Unix socket `/tmp/kitty`

### Custom tab bar (`tab_bar.py`)
Parses nushell pre-prompt title (`~/path 󰊢 branch [+1 ~2] | 5f 2d | 14:23`):
- **Single tab:** Title centered full-width
- **Multi-tab:** Segments — directory (blue), branch (purple), file counts (cyan), time (green)
- Colors: bg `#1a1b26`, blue `#7aa2f7`, purple `#bb9af7`, cyan `#7dcfff`, green `#9ece6a`

---

## Hyprland

**Config:** `wayland/hypr/hyprland.lua` (Lua-based)

### Autostart
`waybar`, `swaync`, `hypridle`, `awww-daemon` (wallpaper: `Nix Tokyo Night.png`, 2s fade), `hyprctl setcursor Layan-cursors 24`

### Appearance
- Gaps: in 5px / out 20px; border 3px; rounding 10px
- Active border: liquid gradient blue→purple→cyan (animated, angle loops)
- Inactive border: animated gradient — same hues as active at 33% opacity (`rgba(...55)`)
- Shadow: `#7aa2f7` 22% / inactive `#1a1b26` 12%
- Blur: 16px, 6 passes, vibrancy 0.3

### Display configuration
**Tool:** `wdisplays` GUI for display layout (Super+M)  
Configure monitor arrangement, resolution, refresh rate graphically.

### Keybindings (Super key)
| Binding | Action |
|---------|--------|
| `Super+Return` | Open kitty |
| `Super+Q` | Close window |
| `Super+F` | File manager (kitty -e yazi) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [daniel-g0/dotfiles](https://github.com/daniel-g0/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-30 -->
