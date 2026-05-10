---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**livepaper** is a Linux desktop app (C# + Avalonia UI) that fetches live wallpapers from online sources and applies them using [mpvpaper](https://github.com/GhostNaN/mpvpaper). mpvpaper renders video wallpapers on Wayland by playing them with MPV behind all windows.

## System Dependencies

- `mpvpaper` — must be installed and on `$PATH`
- `mpv` — underlying player used by mpvpaper
- Wayland compositor (e.g. Hyprland, Sway, GNOME on Wayland)
- `.NET SDK` — for building
- `pactl` / `parec` (from `libpulse` on Arch) — PulseAudio/PipeWire CLI tools; required for Auto-Mute stream detection and audio level measurement
- `ffmpeg` — required for thumbnail extraction in the **Import Wallpaper** flow
- `wl-clipboard` — `wl-copy` is invoked by the Settings-tab keybind Copy buttons so snippets persist after livepaper exits. Falls back to Avalonia's clipboard if missing, but the clipboard releases the selection on app close (snippet only pasteable while livepaper is open).

## Common Commands

```bash
dotnet run --project src/livepaper                            # run the app
dotnet run --project src/livepaper -- --restore               # restore last session without opening UI
dotnet run --project src/livepaper -- --random                # apply a random library wallpaper without opening UI
dotnet build src/livepaper                                    # build (no solution file at repo root)
dotnet publish src/livepaper -r linux-x64 --self-contained    # single binary release
```

## CLI Flags

- `--restore` — re-applies the last session (single video, playlist, or random) without opening the UI. Useful for compositor autostart (e.g. `exec-once = livepaper --restore` in Hyprland). Always one-shot: for timed playlists it `setsid`-spawns `--timer-daemon` and returns immediately.
- `--random` — picks a random video from the library and applies it, then exits. Saves the picked video so `--restore` replays the same one.
- `--kill` — stops playback (kills mpvpaper and signals the timed playlist timer to stop), then exits.
- `--monitor` *(internal)* — starts `AudioMonitor` using saved settings and blocks indefinitely. Spawned automatically as a detached process when the app closes with AutoMute enabled; killed when the app reopens.
- `--timer-daemon` *(internal)* — owns the timed-playlist tick loop and blocks indefinitely. Spawned automatically as a detached process by `--restore` (timed-playlist case) and by the GUI on close; killed when the app reopens.
- `--action=<action>` — sends a command to the running session and exits. Intended for compositor keybinds. The Settings tab also exposes these as copy-paste keybind snippets, so the action set is duplicated with `--kill`/`--restore` for parity. Actions:
  - `toggle-mute` — toggle mpv mute
  - `toggle-pause` — pause/resume mpv playback AND the timed playlist timer (timer resumes with remaining time preserved)
  - `stop` — stop playback (alias of `--kill`)
  - `play` — relaunch the last session (alias of `--restore`)
  - `toggle-play` — stop if playing, otherwise relaunch the last session
  - `next-wallpaper` — advance to next wallpaper in history/playlist
  - `previous-wallpaper` — go back in wallpaper history
  - `random` — alias of `--random` (picks from active playlist if running, else from library)
  - `volume-up` / `volume-down` — adjust volume by 5 (clamped 0-100); persists to settings.json so the slider and next launch reflect the new value

## UI Structure

The app has three tabs:

### Browse Tab
- Source selector (pill-style) to switch between motionbgs.com, moewalls.com, Desktophut, and Wallpaper Engine local
- Grid of wallpaper cards (thumbnail + title); clicking a thumbnail opens a fullscreen preview modal
- Search box (enabled only for sources that support it)
- Refresh button and loading bar (thin strip below the top bar, no layout shift)
- Per-card "Download & Apply" downloads + applies that card only
- **Selection toolbar** docks at bottom when ≥1 card is selected (Shift-click / Ctrl-click / Ctrl+A): "N selected", `Download` (downloads all selected, no apply), `Cancel`

### Library Tab
- Grid of all downloaded wallpapers with circular badge in top-right: `+` to add to playlist, `−` to remove. Always visible.
- Per-card buttons act on that card only (no multi-select fan-out)
- "Import" button opens a file picker (`.mp4`/`.webm`/`.mov`/`.mkv`/`.avi`/`.gif`); a title-input modal then copies the video to the library and runs `ffmpeg` to extract a 320px-wide thumbnail at the 1-second mark. The `.id` sidecar holds `import:<source-path>` for re-import dedupe.
- "Play All" button with a "Shuffle" toggle — plays the entire library; rotation behaviour follows the global Settings → PLAYLIST panel (timer or advance-on-video-end)
- Per-card: Apply (sets as wallpaper) and Delete (removes from disk and library)
- **Selection toolbar** docks above the playlist strip when ≥1 card is selected: "N selected", `Add to Playlist`, `Remove from Playlist`, `Delete`, `Cancel`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunwoo101/livepaper](https://github.com/sunwoo101/livepaper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
