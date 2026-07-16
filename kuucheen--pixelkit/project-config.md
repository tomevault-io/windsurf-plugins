---
trigger: always_on
description: This file applies to the entire PixelKit repository. Read it before changing
---

# PixelKit agent guide

This file applies to the entire PixelKit repository. Read it before changing
code, packaging, desktop metadata, CI, or release configuration. The adjacent
`../PowerToys` tree is an MIT-licensed behavioral reference; its `AGENTS.md`
applies only inside that tree.

## Project intent and invariants

PixelKit is a lightweight, native Linux implementation of the PowerToys Color
Picker and Screen Ruler workflows. Preserve these properties:

- One Rust binary with short-lived picker/ruler/editor processes and a small
  event-blocked shortcut daemon.
- No privileged helper, `/dev/input`, uinput, key logger, telemetry, or runtime
  network service.
- Direct X11 integration where the protocol permits it; compositor-controlled
  `xdg-desktop-portal` APIs on Wayland and in Flatpak.
- Full-resolution pixel sampling and rendering. Never trade correctness for a
  downscaled preview.
- Local, version-tolerant JSON settings/history in XDG directories.
- The stable publication/application ID is
  `io.github.Kuucheen.PixelKit`. Do not rename it after publication.

The code uses Rust edition 2024, MSRV 1.88, and eframe/egui 0.31 with the Glow
renderer. The upstream repository is `git@github.com:Kuucheen/PixelKit.git`.

## Source map

- `src/capture.rs`: PNG loading, direct X11 root capture, Wayland Screenshot
  portal capture, RGBA normalization, and DPI.
- `src/color.rs`: color conversions, names, PowerToys-compatible format tokens.
- `src/measurement.rs`: inclusive measurement rectangles, physical units, and
  same-color edge scans.
- `src/config.rs`: XDG paths, defaults, atomic saves, and color history.
- `src/daemon.rs`: X11 hotkeys and Wayland GlobalShortcuts portal sessions.
- `src/ui/mod.rs`: shared egui style, process launching, wheel normalization,
  and lossless tiled capture textures.
- `src/ui/picker.rs`: full-screen picker and magnifier/loupe.
- `src/ui/ruler.rs`: full-screen ruler, measurements, and async recapture.
- `src/ui/editor.rs`: saved-color editor/history/export UI.
- `src/ui/hub.rs`: settings and launcher window.
- `docs/ARCHITECTURE.md`: high-level design.
- `docs/PACKAGING.md`: release and repository submission guidance.

## Non-obvious implementation findings

### Capture and GPU rendering

- `CaptureFrame` always retains the original RGBA pixels. Sampling and
  measurements operate on this full-resolution buffer.
- Some systems report an egui/OpenGL maximum texture side of 2048 even when the
  desktop capture is larger (the Fedora/KDE reference system produced
  2240x1400 captures). Uploading one oversized texture panics.
- `TiledCaptureTexture` in `src/ui/mod.rs` splits captures into lossless GPU-safe
  regions and maps each tile back into the full image rectangle. Do not replace
  this with proportional downscaling: that visibly loses screen pixels and was
  a previously reported regression.
- Keep nearest-neighbor texture sampling. Pixel/source coordinate mapping must
  continue using the original capture dimensions.
- Direct X11 capture must honor the advertised visual channel masks and scanline
  padding; do not assume one fixed BGRX byte layout.

### Wayland portals and recapture

- Wayland deliberately forbids silent global capture and key grabs. Do not add
  compositor-specific bypasses. Screenshot permission/selection dialogs are
  expected behavior.
- Each screenshot request creates its own `ashpd::zbus::Connection::session()`
  and best-effort registers `APP_ID`. Do not use ashpd's process-global cached
  connection from a short-lived Tokio runtime: a later recapture can inherit a
  connection owned by a dead runtime and stop making progress.
- Never call `capture_screen()` synchronously from an egui `update()` method.
  The old synchronous ruler refresh blocked the Wayland event loop and KDE
  terminated the app as unresponsive.
- Ruler **Recapture** uses a worker thread, waits briefly for one transparent
  overlay frame, polls through a channel, and has a 90-second recovery timeout.
  Preserve the transparent frame so PixelKit does not photograph itself.
- Wayland ruler content is a snapshot. Manual Recapture / `R` is intentional;
  continuous capture is limited to X11.

### Input and UI behavior

- Use the shared raw-event `wheel_steps()` helper for discrete behavior. Egui's
  smoothed scroll delta can turn one notched mouse-wheel event into many zoom or
  tolerance changes. Raw Line/Page or large Point events are one step; small
  touchpad Point events accumulate gradually.
- The picker loupe has no fixed minimum content width. Its box width is derived
  from the 13x13 grid, formatted value, and optional color name. The color
  swatch width is exactly the magnified grid width.
- Picker Escape/Backspace closes without picking. Escape also closes the color
  editor opened after a pick.
- Ruler mode icons are painter-drawn vectors. Do not replace them with Unicode
  glyphs; font coverage was unreliable.
- Egui's selected `Button` state defaults to a zero corner radius. Ruler mode
  buttons explicitly set their corner radius so selected controls stay rounded.
- The ruler tolerance toolbar order is `Tolerance`, numeric input, minus, plus.
  The numeric input must remain visible and directly typeable from 0 to 255.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Kuucheen/PixelKit](https://github.com/Kuucheen/PixelKit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
