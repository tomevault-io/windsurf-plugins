---
trigger: always_on
description: An Omarchy shell plugin: one window for the settings that otherwise live
---

# OmaSettings

An Omarchy shell plugin: one window for the settings that otherwise live
scattered across `~/.config/omarchy/shell.json`, the Hyprland config, the
per-application configs, and `~/.XCompose`.

It runs inside `omarchy-shell` — a single long-lived Quickshell (QML) process
that also draws the bar. There is no separate app: the bar widget loads the
window on first use.

## Layout, and what each part may know

```
manifest.json          Plugin manifest — id, kinds, entry point
Panel.qml              The bar gear; loads SettingsWindow.qml on first use
SettingsWindow.qml     Window chrome, sidebar, page routing, and the state
                       every page reads. Pages reach it as `app`.
ui/                    Presentational components + the Palette singleton
sections/              One file per page
bin/omasettings        Subcommand routing
lib/                   One module per thing being configured
```

Three rules keep the split honest:

- A `ui/` component knows how a control looks and reports what happened
  through a **signal**. It never knows which setting it is editing, and never
  reaches back into the window.
- A `sections/` page knows what its settings mean and calls `app` to read and
  write them. It never shells out.
- `lib/` shells out. It never knows what any of it looks like.

When a page needs something new, the value goes into `lib/state.sh`'s single
JSON document and a getter goes on the window; the page reads `app.something`.

## The one rule about other people's files

**Never parse and rewrite a hand-written config.** That is how a settings app
eats someone's setup. Every file falls into one of three patterns:

1. **Generated wholly by us** — `~/.config/hypr/omasettings.lua`, rendered
   from our store (`~/.config/omarchy/omasettings.json`). Hyprland settings are
   applied live with `hyprctl eval` — not `keyword`, which does nothing on a Lua
   config — and recorded there; the generated file is loaded last so the user's
   own files still say what they wrote. It is created on the first write, not at
   install, and the `require` line is appended to `hyprland.lua` at the same
   moment.
2. **A marked block inside their file** — keybindings, appended to
   `bindings.lua` between `-- >>> omasettings bindings` markers. Everything
   outside the markers is copied through untouched, and removing the last
   binding removes the block.
3. **Edited in place, one value at a time** — `herdr/config.toml`, `tmux.conf`,
   Neovim's `options.lua`. These are hand-written *and* commented, so a value is
   replaced exactly where it stands (keeping its comment attached) and a new one
   is appended to its table or file.

Two invariants on top:

- **First-touch backup.** `backup_once` copies any hand-written file to
  `<file>.omasettings.bak` before the first write, and never again — later
  writes must not overwrite the pristine copy. Files we generate get no backup.
- **Validate, then roll back.** Each format is checked in its own terms:
  Herdr with `herdr config check`, Neovim by compiling the Lua, tmux by applying
  the option to the running server. On failure, restore the previous content and
  `die` with the tool's own message.

## Pages for applications you may not have

Tmux, Neovim and Herdr get a page each, and an application you do not have is
a page of settings with nowhere to go — so `pageAvailable()` drops it from the
menu, from the search counts and from Alt navigation, and steps off it if the
application goes away while its page is open.

**The binary is the test, not the config file.** An application can be
installed and never yet configured, and a config file can outlive the thing it
configured; only `command -v` answers the question actually being asked. Each
of the three reports it in its own state as `installed`.

## Where each page writes

This app stores almost nothing. Each page writes into whatever owns the
setting:

| Page | Lands in |
| --- | --- |
| Windows, Layout, Effects, Groups, Keyboard, Mouse | `~/.config/hypr/omasettings.lua`, loaded last |
| Appearance | Omarchy's own config, via `omarchy-theme-set`, `omarchy font set`, `omarchy display text size` |
| Bar, Idle & Lock, Plugins | `~/.config/omarchy/shell.json` |
| Keybindings | a marked block in `~/.config/hypr/bindings.lua` |
| Compose Keys | `~/.XCompose` |
| Herdr, Tmux, Neovim | their own configs, edited in place |
| Displays | `~/.config/hypr/omasettings.lua`, applied with `hyprctl eval` — plus `monitors.lua` for the laptop panel's scale |
| Audio, Network, Bluetooth, Power | nowhere — live system state, owned by PipeWire, NetworkManager, BlueZ and power-profiles-daemon |

A display is the one thing here worth writing down as well as applying: it is
unplugged and plugged back in, and Hyprland matches its monitor rules again on
every connect. So a resolution or a scale set on the Displays page becomes an
`hl.monitor` call in the managed file, and a display that is **not** connected
can be set up by name for the next time it arrives. `hyprctl keyword monitor`
was the original bug there — it answers "keyword can't work with non-legacy
parsers" on stderr while exiting 0, so both settings looked applied and never
were.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [twiking/omasettings](https://github.com/twiking/omasettings) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
