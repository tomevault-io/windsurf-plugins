---
trigger: always_on
description: Read this file first. It tells you what this project is, how it is shaped, and exactly where to write different kinds of code.
---

# de-omarchy — Project Brief for AI Assistants

Read this file first. It tells you what this project is, how it is shaped, and exactly where to write different kinds of code.

## What this project is

`de-omarchy` is a **pure Arch / CachyOS port of [Omarchy](https://github.com/omarchy/omarchy)** — DHH's opinionated Hyprland desktop config — with every macOS assumption removed and Arch tooling (pacman/AUR, systemd, uwsm) used instead. It is a desktop environment layer, not an application: it configures Hyprland, a Quickshell QML status bar, keybindings, themes, and ~435 small CLI helper commands.

The repo (this checkout, e.g. `~/de-omarchy`) is the **source**. It is deployed (rsync'd) to the runtime at `/usr/share/de-omarchy`. The running desktop reads from the runtime, never from the repo. `OMARCHY_PATH` (set in `/etc/environment.d/10-de-omarchy.conf`) always points at the runtime dir.

## Architecture in one breath

- **Hyprland** (Wayland compositor) configured in Lua: `default/hypr/` is the source tree, merged into `~/.config/hypr/` on deploy/refresh.
- **Quickshell** (a QML Wayland shell) renders the bar, panels, OSD, notifications, lock screen, etc. Source: `shell/`.
- **~435 bash commands** in `bin/`, all prefixed `omarchy-`, routed by `bin/omarchy`. They are the stable CLI surface for scripts, keybinds, and the menu.
- **Themes**: `themes/*/colors.toml` + `default/themed/*.tpl` templates rendered into user config.
- **Plugin system**: first-party shell plugins live in `shell/plugins/`; installable plugins are listed in `registry.json` and surfaced in a Plugin Library window (`plugin-library/`).
- **Keybindings**: `default/hypr/keymap.lua` (Lua table of binds) — the live copy is `~/.config/hypr/keymap.lua`.

## Directory map (where things live)

```
de-omarchy/
  bin/                     # ALL CLI commands (omarchy-*). Router = bin/omarchy
  shell/                   # Quickshell QML desktop
    shell.qml              #   entry point; loads plugins via pluginRegistry
    Commons/  Ui/          #   shared QML imports (use `import qs.Commons`, `import qs.Ui`)
    plugins/               #   first-party plugins, one dir each
      bar/                 #     the main bar + indicators/ widgets (+ BarStyles.js presets)
      panels/              #     popup panels (audio, bluetooth, network, power, ...)
      services/            #     background services (idle, media, nightlight, battery)
      notification-center/ #     bell widget + history panel (SUPER+ALT+N)
      plugin-library/  plugin-manager/  docker-status/  sysmon/  ...  # feature plugins
  plugin-library/          # STANDALONE Plugin Library window (ShellRoot + Window)
    shell.qml              #   launched by `omarchy-plugin-library` (SUPER+SHIFT+I)
  default/                 # system configs copied to runtime (the "defaults")
    hypr/                  #   Hyprland Lua config (keymap.lua, bootstrap.lua, apps/, bindings/)
    themed/                #   theme templates (*.tpl) with {{ variable }} placeholders
    agents/skills/         #   AI agent skills symlinked into ~/.claude, ~/.codex, ...
    chromium/  xcompose  voxtype/  tensaku/  audio/  alacritty/  foot/  ghostty/
    bash/  uwsm/  applications/  systemd/  fonts/
  config/                  # user-dotfile defaults copied to ~/.config (hypr, kitty, git, ...)
  themes/                  # 22 themes, each themes/<name>/colors.toml
  docs/                    # vendored upstream dev docs + FORK-NOTES.md (read deltas first)
  manual/                  # vendored 51-chapter upstream user manual (paths rewritten)
  agents/                  # vendored agent guides (command-metadata.md etc.)
  audit/PARITY-AUDIT.md    # fork-vs-upstream feature audit
  registry.json            # plugin catalog shown in the Plugin Library
  keybindings/KEYMAP.md    # human-readable keybind reference
  install.sh  uninstall.sh # deploy / remove runtime
  packages/  applications/  scripts/  display/  monitor-manager/
```

## Where to write each kind of change

### 1. A new CLI command  →  `bin/omarchy-<group>-<name>`
- Create `bin/omarchy-<group>-<name>`, `#!/bin/bash` shebang, executable bit.
- Add the metadata header block the router parses (`bin/omarchy` scans for these):
  ```bash
  # omarchy:summary=One-line description.
  # omarchy:group=<group>          # optional; inferred from filename prefix otherwise
  # omarchy:args=[--flag] <arg>
  # omarchy:examples=omarchy <group> <name> | omarchy <group> <name> foo
  # omarchy:hidden=true            # optional, hide from listings
  ```
- If `<group>` is new, register it in `GROUP_DESCRIPTIONS` inside `bin/omarchy` so it shows in the top-level help.
- Use bash 5 style: `[[ ]]` for strings/paths, `(( ))` for numbers. Quote string literals in comparisons. Avoid `exec`/`exit` to skip unreachable code; prefer full `if/else`.
- After writing, deploy the single file to the runtime: `sudo cp bin/omarchy-x /usr/share/de-omarchy/bin/` (or re-run `install.sh`).

### 2. A new shell plugin (bar widget / panel / service)  →  `shell/plugins/<name>/`
- Make `shell/plugins/<name>/` with:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [swadhinbiswas/de-omarchy](https://github.com/swadhinbiswas/de-omarchy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-24 -->
