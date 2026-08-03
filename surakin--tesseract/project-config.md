---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Tesseract is a cross-platform desktop Matrix/chat client. The core networking is Rust (using `matrix-sdk`), exposed to C++ via a `cxx` FFI bridge. Platform-specific UIs are written in C++ targeting Win32 (Windows), AppKit (Objective-C++, macOS), Qt6 Widgets, or GTK4 (Linux).

## Build Commands

Per-platform prerequisites, the full preset list, the `-DTESSERACT_UI=` override, and build internals (Corrosion, bundled SQLite, rustls, link strategy) live in [docs/BUILD.md](docs/BUILD.md). The day-to-day loop:

```bash
cmake --preset linux-debug              # full preset list in docs/BUILD.md; builds GTK4 + Qt6
cmake --build build/linux-debug
./build/linux-debug/ui/linux-qt/tesseract    # Qt6 UI
./build/linux-debug/ui/linux-gtk/tesseract   # GTK4 UI

# macOS (pick the preset matching your CPU):
cmake --preset macos-appkit-arm64-debug
cmake --build build/macos-appkit-arm64-debug
open build/macos-appkit-arm64-debug/ui/macos/Tesseract.app
```

## Architecture

```text
sdk/         ← Rust crate: matrix-sdk wrapper + cxx FFI bridge
client/      ← C++ static library: high-level C++ API over the Rust FFI
ui/
  shared/    ← tesseract_tk: cross-platform widget toolkit + shared views
    tk/        ← Canvas / Widget / Layout / Host abstractions (+ genuinely shared
                  GStreamer video/capture); per-platform backend impls
                  (canvas_*/host_*/audio_*/video_*/screen_capture_*) live in each
                  ui/<platform>/tk/, compiled into that platform's own target
    views/     ← MainAppWidget (root widget tree); LoginView, BrandView;
                  RoomListView, RoomView, MessageListView, ComposeBar;
                  ThreadView, ThreadListView (right-side panel inside RoomView);
                  EmojiPicker, StickerPicker, AccountPicker;
                  ImageViewerOverlay, VideoViewerOverlay, ShortcodePopup;
                  SettingsView, JoinRoomView, UserInfo;
                  RecoveryBanner, VerificationBanner;
                  html_spans (HTML→TextSpan), map_tiles, media_utils
                  (Markdown→HTML lives in client/src/markdown.cpp, Rust-backed)
                  shared bases: ListPopupBase, MediaOverlayBase, TabbedGridPicker;
                  MessageListView collaborators (per-concern, behavior-preserving
                  split): TimelineMediaController, TimelineVideoPlaylist,
                  LocationMapPanner, RoomSwitchGateKeeper, UrlPreviewCardDisplay,
                  LinkLayoutCache, SpoilerRevealer, ReadReceiptTracker
  windows/   ← Win32 executable (thin shell) + its tk/ backend (+ third_party/bettertext text control)
  macos/     ← AppKit executable (.app bundle, thin shell) + its tk/ backend
  linux-qt/  ← Qt6 Widgets executable (thin shell) + its tk/ backend
  linux-gtk/ ← GTK4 executable (thin shell) + its tk/ backend
```

### Layer responsibilities

**`sdk/` (Rust)** — All async I/O lives here. A `tokio` runtime runs in background threads. The `cxx` bridge (`sdk/src/lib.rs`) exposes a synchronous C API. `client.rs` wraps `matrix-sdk` for login, sync, and messaging; `oauth.rs` implements the RFC 8252 loopback redirect OAuth flow.

**`client/` (C++)** — `tesseract::Client` (Pimpl) wraps the Rust FFI. `tesseract::IEventHandler` is the interface UIs implement to receive async callbacks (room updates, sync events, session saves). `tesseract::SessionStore` handles platform-specific persistence of the session JSON and the per-account matrix-sdk store. Account data lives under `data_dir()` (`%APPDATA%/Tesseract/` on Windows, `~/Library/Application Support/Tesseract/` on macOS, `~/.local/share/tesseract/` on Linux — XDG state, not config); only `app_settings.json` lives in `config_dir()` (`~/.config/tesseract/` on Linux). `data_dir()` equals `config_dir()` on Windows/macOS, which have no such split.

**`ui/shared/` (C++)** — `tesseract_tk` is the cross-platform UI toolkit. It owns drawing, layout, hit-test, focus, and keyboard. `tk::Canvas` is the abstract 2D backend (D2D on Win32, QPainter on Qt6, Cairo+Pango on GTK4, CoreGraphics+CoreText on macOS). `tk::Host` is the per-platform integration surface (repaint scheduling, post-to-UI, native edit overlays). Shared widget classes live under `tk/`; shared views live under `views/` (see architecture diagram above for the full list). Text input stays native via `tk::NativeTextField` / `tk::NativeTextArea` overlays so IME and selection behave correctly per-OS.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [surakin/tesseract](https://github.com/surakin/tesseract) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
