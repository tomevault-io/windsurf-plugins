---
trigger: always_on
description: This file is the first source of truth for agents working in this repository.
---

# Kiri repository guide for agents

This file is the first source of truth for agents working in this repository.
Read it before editing, then read `docs/architecture.md`.

## Start here

1. Run `git status -sb` before making changes.
2. Read `README.md`, `ROADMAP.md`, and `docs/architecture.md`.
3. For capture-selection behavior, also read
   `docs/adr/0003-manual-region-selection.md`.
4. Treat every pre-existing modification and untracked file as user-owned.
   Never reset, discard, overwrite, or reformat unrelated work.
5. Completed implementation plans and the former Swift migration specs live in
   Git history, not the working tree. Do not reconstruct an old parallel
   project from them; current source, tests, this file, and accepted ADRs win.

## Product contract

Kiri is a local-first capture utility for macOS, Windows, and
Linux. Preserve these decisions:

- The default global capture shortcut is `⇧⌘A` on macOS and `Shift+Ctrl+A` on
  Windows and Linux X11. These use native global-hotkey registration; the shortcut does
  not require Input Monitoring permission. Settings may replace the binding
  with a modified letter or digit and restore this default (ADR 0046).
  On Wayland, the user configures `kiri --capture` in desktop settings, or opts
  in to GlobalShortcuts Portal setup where the actual interface is available.
  Preserve command guidance and truthful session/binding state (ADR 0075). Never
  install compositor bindings or FIFOs automatically (ADR 0051).
- The initial overlay offers Screenshot, Record, and OCR. The screenshot toolbar
  and saved-image editor offer Recognize QR Codes. QR recognition
  stays local; multiple codes keep their clickable image positions. Copy, open,
  and save are explicit actions after viewing content (ADR 0061).
- Window hover shows exactly one restrained monochrome outline without handles,
  dimensions, stacked borders, or a following tooltip. A click selects that
  window; a drag creates a custom region. Both selections remain movable and
  resizable with eight handles. Linux X11 supplies window bounds; Wayland uses
  region drag and currently rejects multiple connected displays before capture.
- Screenshot completion is clipboard-first and returns focus to the original
  application. Do not open the Kiri library after every capture.
- Escape cancels capture and countdown; Return confirms a screenshot.
- Annotation tools appear immediately after region selection. Existing text
  and shapes remain selectable and editable; size controls update live.
- Text backgrounds default to transparent. Mosaic is a continuous brush with
  adjustable diameter and intensity.
- Recording is Retina/DPI-scale, high-quality MP4. Kiri's recording controls
  and paused time must not appear in the exported video.
  Linux MP4 recording can include system audio and microphone through the local
  PulseAudio/PipeWire audio service and hides the floating control panel; use tray
  actions or `kiri --toggle-recording-pause` / `kiri --stop-recording`. ScreenCast
  consent must select the same display as the frozen screenshot.
- The optional high-contrast red click ripple is visible live and is also captured.
- The 3-2-1 countdown is centered and compact; it must not dim the selected
  recording region.
- User-facing UI supports English, Simplified Chinese, and Japanese and follows
  the OS preferred language.
- Captures stay local. Never add uploads, analytics, accounts, or network
  behavior without an explicit product decision and privacy documentation.
  Recording, merging, thumbnails, and GIF conversion use platform media APIs
  and must not download or launch a third-party media executable. On Linux that
  means system GStreamer plugins, not a downloaded FFmpeg binary.
- macOS and installed Windows application updates are manual and signed. Check, download, install, and the
  macOS relaunch are separate user actions; Windows explicitly offers Install
  and Restart, then exits into its passive NSIS installer and reopens afterward.
  GitHub Releases is an error-recovery link, not the normal updater.
  Linux uses replacement `.deb` packages downloaded and installed by the user;
  it does not expose the signed in-app updater. Ubuntu 24.04 / GNOME is the
  initial Linux target, with X11 compatibility. Do not claim AppImage delivery.
- Linux local OCR uses system Tesseract with `eng`, `chi_sim`, and `jpn` data.
  Linux videos support playback, GIF conversion, normal-speed cuts/reordering and
  MP4 export with source audio. Independently gated speed, effects, masks,
  annotations and stickers remain unavailable; never silently drop saved content.

## Repository map

- `src/` — React frontend: capture overlay, annotation canvas, library,
  editor, countdown/control/ripple windows, i18n (en/zh-Hans/ja), design tokens.
- `src-tauri/src/core/` — platform-independent models: geometry, recording
  policy, shortcut model, asset library (byte-compatible with the Swift
  version's `library.json`).
- `src-tauri/src/capture/` — per-platform capture backends (macOS:
  ScreenCaptureKit via objc2; Windows: xcap WGC + windows-capture + cpal;
  Linux: X11 `xcap`, Wayland `grim` or xdg-desktop-portal Screenshot;
  ScreenCast + PipeWire for recording).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuxino/Kiri](https://github.com/yuxino/Kiri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-05 -->
