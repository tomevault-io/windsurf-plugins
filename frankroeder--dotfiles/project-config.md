---
trigger: always_on
description: These dotfiles target macOS (Apple Silicon), Linux x86, Asahi Fedora (aarch64 ARM).
---

# General

These dotfiles target macOS (Apple Silicon), Linux x86, Asahi Fedora (aarch64 ARM).
Share as much as possible across OS while respecting arch diffs.
Makefile defines 5 profiles (micro, minimal, linux, macos, asahi) using symlinks,
brew/dnf installs, services. macOS WM: yabai+skhd or aerospace/flashspace +
sketchybar (lua configs for top/bottom bars in sketchybar/{top,bottom}/).
Asahi: Hyprland (modular lua: asahi/hypr/hyprland.lua + conf.d/*.lua),
quickshell (QML in asahi/quickshell/remix/ for bar/launcher/wallpaper on quicks branch)
+ hyprpaper/hyprlock/hypridle +
ghostty, custom asahi/bin scripts. Shared: nvim (full lua/),
zsh (zim+dots), mpv, ghostty. Always differentiate Linux by arch. Profiles:

- `micro` setup with bash, tmux, and htop where there are almost no rights for the user
- `minimal` setup with nvim, zsh, python, node and more tools installed locally without sudo
- `linux` setup for desktop and server settings with the full suite for both sudo and non-sudo users
- `macos` setup with the full suite of applications, window management and applications for native Apple Silicon
- `asahi` setup with the full suite of applications, window management and applications for Linux ARM

---

We need to always differentiate between the different Linux settings with respect to architecture.

# Executing

- always try to run scripts that do not break the system (have smoketests) and verify that symlinks are present
- always inspect the outputs of scripts and programs yourself to identify bugs and issues

# Documentation

## macOS
- yabai (tiling WM, bspwm-like): https://github.com/asmvik/yabai/wiki
- skhd (hotkey daemon): https://github.com/koekeishiya/skhd
- SketchyBar (lua status bars): https://felixkratz.github.io/SketchyBar/
- AeroSpace (i3/sway-like): https://nikitabobko.github.io/AeroSpace/
- FlashSpace: https://github.com/wojciech-kulik/FlashSpace
- Ghostty: https://ghostty.org/docs

## Asahi Linux Fedora
- Fedora Asahi Remix: https://asahilinux.org/fedora/
- Hyprland wiki: https://wiki.hypr.land/
- hyprpaper (wallpaper daemon): https://wiki.hypr.land/Hypr-Ecosystem/hyprpaper/
- hyprlock (screen locker): https://wiki.hypr.land/Hypr-Ecosystem/hyprlock/
- hypridle (idle daemon): https://wiki.hypr.land/Hypr-Ecosystem/hypridle/
- Quickshell (QML toolkit, bar/launcher + native notifications): https://quickshell.org/docs/
- **QML singletons (Quickshell/Qt6)**: `pragma Singleton` is ignored without `qmldir` registration. `import "File.qml" as X` silently falls back to white/black defaults. Create `qmldir` in module dirs (`singleton Name File.qml`), use module imports (`import "../foo"`), reference by registered name (`Foo.bar`). Prefer Quickshell `Singleton` root type (reloadable) over QtObject. Never `Foo {}` construct singletons.

## Shared
- Neovim: https://neovim.io/doc/
- Zsh: https://zsh.sourceforge.io/Doc/
- Ghostty: https://ghostty.org/docs
- mpv: https://mpv.io/manual/stable/

# SketchyBar layout (macOS)

Three instances: `sketchybar` (bottom), `sketchybar-top` (top), `sketchybar-island` (notch pill).
Config in `sketchybar/{bottom,top,island}/`; shared lua at `sketchybar/*.lua`. Reload each
with `<bin> --reload`. Prefer plain `require` for items (fail loud), not safe_require.

Requirements / decisions:
- Island pill background is notch-black (0xff000000) to blend with the physical notch; border is
  `theme.border`. Foregrounds are the static mocha palette at full alpha (`colors.mocha` in
  island_style) — bright in both modes, since latte fg is unreadable on black.
- Island pills: appswitch, siri, layout (`island_layout` from skhd fn-e/w/s), mic (`island_mic`
  from top mic), bluetooth (`island_bluetooth` from top bt poll on new connect), window
  (`island_window` from skhd fn+shift-w/s float+sticky toggles; re-queries yabai for state).
  No battery/power pills (macOS notifies on low battery), no volume pill (native HUD), no wifi
  pill, no space pill (overlapped appswitch), no now-playing/media pill, no vpn pill.
- Expand priority: lower prio never clobbers higher; sticky siri (duration=0) only yields to higher
  prio or same kind. Dismiss uses cancellable `sbar.delay`. Restore snaps idle geometry outside
  animate then hides.
- `display.refresh()` re-probes notch + arrangement rows on `display_change` (hotplug).
- Every expand grows out of the notch (idle seed in island_core); consecutive expands morph.
- NEVER put constant-valued numeric props inside `sbar.animate` batches when value is unchanged
  (1px jitter). Expand only animates changing geometry. Restore snaps idle geometry un-animated
  then hides (avoids omitted margin/width zeroing to full-display stretch).
- `display.notch_width`: require both auxiliary flanks + n < 40% of screen (else 0). Full-width
  "notch" on externals was a false positive that set idle pill width = display width.
- Smoke: `sketchybar/island/smoke_test.sh [out_dir]`.
- Island tuck equals corner_radius (offsets -16): hides the top rounding above the screen edge so
  the pill sides come out of the notch square; heights include the tucked 16px.
- Island is notch-aware: on the built-in (notched) display the pill straddles the notch — text in

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [frankroeder/dotfiles](https://github.com/frankroeder/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
