---
trigger: always_on
description: > **Note**: These instructions apply to both GitHub Copilot and AI agents. All contributors (human and AI) must follow these guidelines to maintain code quality and consistency.
---

# RustFrame - GitHub Copilot Instructions

> **Note**: These instructions apply to both GitHub Copilot and AI agents. All contributors (human and AI) must follow these guidelines to maintain code quality and consistency.

## Project Overview

**RustFrame** is a high-performance, cross-platform screen capture application built with Rust + Tauri 2 and React. The primary feature is creating a **Preview Window** that displays only a user-selected region of the screen, enabling users with wide-screen monitors to share specific areas in video conferencing apps (Discord, Google Meets, Zoom, Teams) instead of their entire screen.

### Core Purpose
- Capture and display a user-selected screen region in a preview window
- Enable selective screen sharing in video conferencing applications
- Support multiple capture methods optimized for each platform
- Maintain high performance with minimal memory footprint

### Non-Negotiable Core Features (Do Not Break)
- **Hollow Border Window**: A dedicated border window for region selection/moving/resizing.
- **Click-Through Capture Mode**: In capture mode, the border’s interior must remain click-through; only the border edges/corners should be interactive.
- **Preview Window**: A destination/preview window that displays the captured region and can be selected in screen sharing pickers (Meet/Zoom/Discord/Teams).
- **Settings as Source of Truth**: Settings must be persisted in the user config directory and loaded reliably on next startup; changes must not silently reset to defaults.
- **Last Region Memory**: When enabled, the last capture region must be saved and restored on startup.

If you make changes in this repo, you MUST verify that these core features still work on the affected platform(s). If preserving these invariants conflicts with a change, prefer the invariant and adjust the change.

---

## Cross-Platform Architecture

### Platform Structure
```
src/
├── platform/          # Platform-specific abstractions
│   ├── mod.rs        # Common traits and interfaces
│   ├── windows.rs    # Windows-specific implementations
│   ├── macos.rs      # macOS-specific implementations
│   └── linux.rs      # Linux-specific implementations
├── capture/          # Capture engine implementations
│   ├── windows/      # Windows Graphics Capture, GDI
│   ├── macos/        # ScreenCaptureKit, CGDisplayStream
│   └── linux/        # X11, Wayland
└── main.rs           # Platform-agnostic entry point
```

### Critical Rules

1. **Platform-Specific Code Must Be Isolated**
   - Windows-only code: Use `#[cfg(target_os = "windows")]`
   - macOS-only code: Use `#[cfg(target_os = "macos")]`
   - Linux-only code: Use `#[cfg(target_os = "linux")]`
   - Never mix platform-specific APIs in shared code

2. **Conditional Compilation**
   ```rust
   #[cfg(target_os = "windows")]
   use windows::Win32::Graphics::Gdi::*;
   
   #[cfg(target_os = "macos")]
   use core_graphics::display::*;
   ```

3. **Shared Abstractions**
   - When functionality is common across platforms, create traits in `platform/mod.rs`
   - Implement traits separately for each platform
   - Use enums or trait objects for runtime polymorphism

4. **Cargo Dependencies**
   ```toml
   [target.'cfg(windows)'.dependencies]
   windows = { version = "0.58", features = [...] }
   
   [target.'cfg(target_os = "macos")'.dependencies]
   cocoa = "0.26"
   core-graphics = "0.24"
   ```

5. **⚠️ CRITICAL: macOS Main Thread Requirement**
   - **ALL Cocoa/AppKit APIs MUST run on the main thread**
   - Calling NSWindow, NSView, NSColor, etc. from background threads causes NSException
   - Rust cannot catch NSException → immediate crash with "foreign exception" error
   - **Solution**: Use `dispatch_sync_f` with `_dispatch_main_q` to dispatch to main thread
   ```rust
   extern "C" {
       static _dispatch_main_q: std::ffi::c_void;
       fn dispatch_sync_f(
           queue: *const std::ffi::c_void,
           context: *mut std::ffi::c_void,
           work: extern "C" fn(*mut std::ffi::c_void),
       );
       fn pthread_main_np() -> i32; // Returns non-zero if on main thread
   }
   
   // Check thread and dispatch if needed
   let is_main = unsafe { pthread_main_np() } != 0;
   if !is_main {
       unsafe {
           dispatch_sync_f(&_dispatch_main_q, context_ptr, callback);
       }
   }
   ```
   - This applies to:
     - Window creation (HollowBorder, DestinationWindow)
     - Screen capture APIs (CGWindowListCreateImage, CGDisplayStream)
     - Any UI manipulation
   - Common error: `fatal runtime error: Rust cannot catch foreign exceptions, aborting`

---

## Rust Coding Standards

### Performance & Memory Optimization

1. **Zero-Copy When Possible**
   - Use references (`&T`) instead of cloning
   - Prefer `&str` over `String` for read-only text
   - Use `Cow<'_, T>` when conditional ownership is needed

2. **Memory Management**
   - Minimize heap allocations in hot paths
   - Use `Vec::with_capacity()` when size is known
   - Prefer stack allocation for small, fixed-size data
   - Use `Box<T>` for large structs to avoid stack overflow

3. **Concurrency**
   - Use `tokio` for async I/O operations

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [salihcantekin/RustFrame](https://github.com/salihcantekin/RustFrame) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
