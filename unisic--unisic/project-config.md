---
trigger: always_on
description: Contributor guide for **any** AI coding agent working on Unisic — Cursor, Aider, Zed, Codex, Continue, Cline, Windsurf, GitHub Copilot Agent, Claude Code, and humans reading over their shoulder. This is the canonical, tool-agnostic contract. If your tool also reads `CLAUDE.md`, that file carries the same rules plus Claude-specific per-subsystem deep notes — read both; where they overlap they agree, and if they ever disagree, **this file and the actual code win.**
---

# AGENTS.md

Contributor guide for **any** AI coding agent working on Unisic — Cursor, Aider, Zed, Codex, Continue, Cline, Windsurf, GitHub Copilot Agent, Claude Code, and humans reading over their shoulder. This is the canonical, tool-agnostic contract. If your tool also reads `CLAUDE.md`, that file carries the same rules plus Claude-specific per-subsystem deep notes — read both; where they overlap they agree, and if they ever disagree, **this file and the actual code win.**

> **Read this whole file before your first edit.** Unisic is small on purpose. Most of the hard problems here are *invisible* — Wayland capture authorization, D-Bus signal ownership, QSettings persistence quirks, Qt object lifetimes. A change that "looks obviously correct" has repeatedly been the wrong one. The gotchas below were each paid for in hours of debugging; treat them as landmines, not trivia.

---

## 1. What Unisic is (and is not)

Unisic is a **ShareX-like screenshot + screen-recording tool for Linux Wayland**, prioritizing KDE Plasma/KWin but portable via `xdg-desktop-portal`. Stack: **C++20, Qt 6 (6.5+), Qt Quick / QML**, fully custom UI. GPLv3. Zero telemetry.

Core workflow it owns end-to-end: press hotkey → annotate *on the selection overlay before the shot is taken* → post-capture editor (arrows, shapes, text, blur/pixelate, crop, numbered steps, object cutout) → route the result to clipboard / disk / a custom upload destination with the link auto-copied → or record the same region as GIF/MP4/WebM.

**It is NOT:** a general image editor, a cloud service, a cross-platform app, an X11 tool, or a kitchen-sink utility. Every feature request is measured against "does a screenshot/record/share workflow genuinely need this?" The answer is usually no.

### Non-negotiable product constraints

- **Wayland-legit capture paths ONLY.** `xdg-desktop-portal` Screenshot/ScreenCast, KWin `org.kde.KWin.ScreenShot2` D-Bus (KDE enhancement), `wlr-screencopy` via `grim` (wlroots), `org.gnome.Shell.Screenshot` (niri/GNOME direct), PipeWire for video, KGlobalAccel / portal GlobalShortcuts for hotkeys. **No X11-only capture hacks. No screen-scraping. No compositor-specific hacks that bypass the security model.**
- **Mandatory UI palette** — do not introduce off-palette colors:
  - Primary `#17153B` (window/panel backgrounds)
  - Secondary `#2E236C`, Tertiary `#433D8B` (secondary elements, hover/active)
  - Accent `#C8ACD6` (action buttons, attention)
  - All UI colors flow from `qml/Theme.qml` tokens — never hardcode a hex in a component.
- **Works without KDE.** KDE gets the fully silent native path; everything must degrade gracefully to portals on GNOME/wlroots/niri. Never assume KWin, `kglobalacceld`, or Breeze is present.
- **Zero telemetry, no network calls except user-configured uploads.** No analytics, no auto-update phone-home, no crash reporters that transmit.

---

## 2. Prime directives

These are the reasons this file exists. Every change is judged against them, in order:

1. **Lightweight.** Small binary, small dependency set, fast startup, low idle RAM/CPU. Unisic lives in the tray all day; it must be invisible when not in use.
2. **Correct.** No regressions in capture/hotkey/settings persistence — the load-bearing, hard-to-test subsystems. When in doubt, verify on a real Wayland session (see §11).
3. **No leaks.** Qt makes ownership easy to get wrong. Every `new` needs an owner; every temp file, D-Bus handle, and PipeWire/ffmpeg resource needs a teardown path.
4. **No feature creep.** Adding code is a cost. The best PR is often a smaller diff, or a deletion.

If a change trades any of these away, it needs an explicit, written justification in the PR — not a silent assumption.

---

## 3. Build, run, and dependencies

```sh
cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release
cmake --build build
./build/unisic
```

- **Toolchain:** CMake ≥ 3.21, a C++20 compiler, Ninja.
- **Required:** `qt6-qtbase-devel qt6-qtdeclarative-devel qt6-qtsvg-devel` (+ `Widgets DBus Network Concurrent QuickControls2` Qt modules). Runtime: `ffmpeg`, `wl-clipboard`, `xdg-desktop-portal` (+ a portal backend), `qt6-qtwayland`.
- **Optional, compile-time guarded** — the build *succeeds without them* and prints a warning; the feature is disabled at runtime:
  - `pipewire-devel` → `HAVE_PIPEWIRE` → GIF/screen recording. Without it, recording is off.
  - `tesseract-devel leptonica-devel` + a langpack (e.g. `tesseract-langpack-eng`) → `HAVE_TESSERACT` → OCR ("copy text from capture"). Gates `App.ocrAvailable` in QML.
- **Runtime helpers shelled out, not linked:** `ffmpeg` (GIF/video encode), `curl` (FTP/SFTP uploads), `grim` (wlroots/niri capture), `wl-copy` (clipboard mirror), `kbuildsycoca6` (KDE service-cache rebuild). Treat all as optional-at-runtime: detect with `QStandardPaths::findExecutable`, degrade gracefully, never crash if absent.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [unisic/unisic](https://github.com/unisic/unisic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
