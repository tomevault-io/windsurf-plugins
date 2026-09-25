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

Kiri is a local-first capture utility for macOS and Windows. Preserve these
decisions:

- The default global capture shortcut is `⇧⌘A` on macOS and `Shift+Ctrl+A` on Windows.
  Both use the platform's native global-hotkey registration; the shortcut does
  not require Input Monitoring permission. Settings may replace the binding
  with a modified letter or digit and restore this default (ADR 0046).
- The initial overlay offers Screenshot, Record, and OCR.
- Window hover shows exactly one restrained monochrome outline without handles,
  dimensions, stacked borders, or a following tooltip. A click selects that
  window; a drag creates a custom region. Both selections remain movable and
  resizable with eight handles.
- Screenshot completion is clipboard-first and returns focus to the original
  application. Do not open the Kiri library after every capture.
- Escape cancels capture and countdown; Return confirms a screenshot.
- Annotation tools appear immediately after region selection. Existing text
  and shapes remain selectable and editable; size controls update live.
- Text backgrounds default to transparent. Mosaic is a continuous brush with
  adjustable diameter and intensity.
- Recording is Retina/DPI-scale, high-quality MP4. Kiri's recording controls
  and paused time must not appear in the exported video.
- The optional monochrome click ripple is visible live and is also captured.
- The 3-2-1 countdown is centered and compact; it must not dim the selected
  recording region.
- User-facing UI supports English, Simplified Chinese, and Japanese and follows
  the OS preferred language.
- Captures stay local. Never add uploads, analytics, accounts, or network
  behavior without an explicit product decision and privacy documentation.
  Recording, merging, thumbnails, and GIF conversion use platform media APIs
  and must not download or launch a third-party media executable.
- Application updates are manual and signed. Check, download, install, and the
  macOS relaunch are separate user actions; Windows explicitly offers Install
  and Restart, then exits into its passive NSIS installer and reopens afterward.
  GitHub Releases is an error-recovery link, not the normal updater.

## Repository map

- `src/` — React frontend: capture overlay, annotation canvas, library,
  editor, countdown/control/ripple windows, i18n (en/zh-Hans/ja), design tokens.
- `src-tauri/src/core/` — platform-independent models: geometry, recording
  policy, shortcut model, asset library (byte-compatible with the Swift
  version's `library.json`).
- `src-tauri/src/capture/` — per-platform capture backends (macOS:
  ScreenCaptureKit via objc2; Windows: xcap WGC + windows-capture + cpal).
- `src-tauri/src/platform/` — per-platform helpers: global shortcut, focus
  restoration, file reveal, click monitoring, capture exclusion.
- `src-tauri/src/record.rs` — platform-native encoding coordination (H.264 +
  AAC → MP4); macOS bridging lives in `src-tauri/src/macos_media.{rs,m}`.
- `src-tauri/src/commands.rs` — the AppModel-equivalent command surface.
- `src-tauri/src/{ocr,gif,thumbnail,protocol,state}.rs` — OCR, GIF export,
  thumbnails, `kiri://` protocol, shared state.
- `scripts/` — packaging, stable development signing, and app-icon validation.
- `docs/architecture.md` — current runtime structure and platform boundaries.
- `docs/README.md` — index of current documentation and accepted decisions.
- `docs/adr/` — accepted architecture/product decisions.

## Architecture boundaries

- `AppState` (state.rs) coordinates capture, library operations, recording
  state, and transient feedback. Synchronous Tauri commands run on the main
  thread (mirroring the Swift @MainActor design); heavy work spawns
  background threads.
- `capture::macos` runs the SCK stream on a dedicated thread; control flows
  through channels. `SCShareableContent` is main-thread-only — resolve it on
  the main thread.
- The recording pipeline is: platform capture (BGRA frames + PCM audio) →
  AVFoundation on macOS or Media Foundation on Windows → H.264/AAC MP4.
  macOS pause/resume segments are merged with AVFoundation.
- `AssetLibrary` is the persistence boundary. It shares the Swift version's
  storage layout (`~/Library/Application Support/kiri` on macOS,
  `%APPDATA%\kiri` on Windows) so existing libraries keep working. Preserve
  recoverable Trash and never manipulate a user's library directly during QA.
- Frontend windows render by `?window=` query param; the frozen capture is

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuxino/kiri](https://github.com/yuxino/kiri) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
