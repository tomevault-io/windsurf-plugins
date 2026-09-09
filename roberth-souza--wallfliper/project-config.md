---
trigger: always_on
description: Minimalist, Rofi-inspired desktop wallpaper selector. Targets **Wayland compositors that implement `wlr-layer-shell`**: Hyprland, Sway, river, Wayfire, and niri (Smithay-based, but implements layer-shell). Full image + video on all of them. KDE Plasma, GNOME, Windows, and X11 are out of scope — do not add code for them.
---

# Wallfliper

Minimalist, Rofi-inspired desktop wallpaper selector. Targets **Wayland compositors that implement `wlr-layer-shell`**: Hyprland, Sway, river, Wayfire, and niri (Smithay-based, but implements layer-shell). Full image + video on all of them. KDE Plasma, GNOME, Windows, and X11 are out of scope — do not add code for them.

Why this scope: both `swww` (images) and `mpvpaper` (video) require the `wlr-layer-shell` protocol. The criterion is the protocol, not the wlroots library — which is why niri (not wlroots) is in, and KWin/Mutter (no layer-shell, or partial) are out.

## Stack
- Python 3.11+, PySide6 (LGPL — not PyQt6, to keep distribution unrestricted).
- UI is **QML (QtQuick)**, not QWidgets — required for the `wlr-layer-shell` overlay (QWidgets can't be configured for layer-shell from Python). Python is the engine; QML is the view only. No Electron, no web view.

## Performance (first-class constraint)
- This app must be **lightweight on the system**. Treat performance as a feature, not an afterthought.
- The selector GUI is launched on demand and should exit cleanly — it must not idle in the background consuming RAM/CPU. Wallpaper rendering is delegated to detached daemons, never held by our GUI process.
- No work on the UI thread that can be deferred. Lazy-load everything (thumbnails, network, scraping). Bounded caches only.
- Prefer in-process/stdlib over heavy deps. Justify every dependency by weight. No polling loops; event-driven only.
- Measure before optimizing, but never ship obvious waste (full-res decode for a thumbnail, re-scanning folders, unbounded caches).

## Compatibility (first-class constraint)
- Target the **widest range of setups** within the supported compositors (see scope). A feature that works on the author's machine but not on a stock Hyprland/Sway/river/Wayfire/niri install is a regression, not a feature.
- **No hardware-vendor or distro-exclusive choices.** Nothing that needs a specific GPU/driver (no AMD-only / NVIDIA-only / Intel-only paths), a specific distro, or a niche tool not everyone has. Prefer auto-detecting, portable options with a software fallback (e.g. `--hwdec=auto`, never a fixed vendor decoder like `vaapi`/`nvdec`).
- **Degrade gracefully when an optional tool is missing** — never hard-fail. The seamless video transition falls back to a plain cut without ffmpeg/swww; follow that pattern for anything new.

## Design
- UI must follow `DESIGN.md` (TUI/yazi-style: borderless, dark, flat, keyboard-first, minimal). Read it before touching anything in `ui/`. If a change makes the app look more "consumer-polished" and less like a fast keyboard tool, reject it.

## Architecture
- `ui/qml/` — QML views (the whole UI: overlay window, top bar, grid, search). The overlay is a `wlr-layer-shell` surface via the system `org.kde.layershell` QML module (overlay layer, exclusive keyboard, Esc to close).
- `ui/qml/Theme.qml` — QML singleton (registered in `ui/qml/qmldir`) holding the shared style tokens, matched to the author's rofi theme: fixed near-black fill `rgba(5,5,5,0.95)` (floating bar only — there is no backdrop panel), 2px white frame (floating-bar border **and** card-selection language), the idle-card shear factor (`cardSlant`), greyscale text shades, and the font — JetBrainsMono Nerd Font when installed, else the fontconfig generic `monospace` (never a hard font dependency). Backdrop darkness is **not** a setting (the old `background_opacity` config/`backgroundOpacity` slot were removed; stale keys in an existing `config.json` are ignored by `load_config`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Roberth-Souza/wallfliper](https://github.com/Roberth-Souza/wallfliper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
