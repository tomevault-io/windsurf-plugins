---
trigger: always_on
description: Phantty is a Windows terminal emulator written in Zig. It uses [libghostty-vt](https://github.com/ghostty-org/ghostty) (Ghostty's VT parser and terminal state machine) for terminal emulation, with its own rendering pipeline (OpenGL + FreeType + DirectWrite on Windows).
---

# AGENTS.md

## Overview

Phantty is a Windows terminal emulator written in Zig. It uses [libghostty-vt](https://github.com/ghostty-org/ghostty) (Ghostty's VT parser and terminal state machine) for terminal emulation, with its own rendering pipeline (OpenGL + FreeType + DirectWrite on Windows).

This is a **Windows-only** project. Development is expected to happen on Windows in PowerShell, targeting `x86_64-windows-gnu`.

## Hard Rules

When changing application **keyboard shortcuts** (bindings in `src/input.zig` and related input paths), **update `README.md`** so the [Keyboard shortcuts](README.md#keyboard-shortcuts) section stays accurate. Also update user-visible shortcut text in `src/renderer/overlays.zig` (startup overlay, command palette entries) when those strings describe the same bindings.

When working on implementing a plan from the plans directory:
 * never deviate from the plan without asking for clear consent
 * never deem something too big and choosing not to do it in the name of pragmatism
 * always ask if you have trouble because something is too big, we will break it down together and work on it step by step

## Planning

When planning, always compare what we are planning to do with https://github.com/ghostty-org/ghostty.
This is the gold standard, we want to be as close to their implementation as possible.

Use the github cli gh to browse https://github.com/ghostty-org/ghostty and always add descriptions on how ghostty does things. 

Exception: work under `remote/` is Phantty's own web remote console and relay implementation. Ghostty does not have an equivalent feature, so `remote/` planning and implementation **does not need to compare against or reference Ghostty**. For `remote/`, follow the existing `remote/` architecture, browser platform constraints, and the user-approved design/plan for that feature.

## Build Commands

```powershell
zig build                         # Default Debug build; use this for development.
zig build -Doptimize=ReleaseFast  # Optimized ReleaseFast build with Windows GUI subsystem (no console window).
Remove-Item -Recurse -Force .\zig-out, .\.zig-cache -ErrorAction SilentlyContinue
```

**Always use `zig build`** for builds during PowerShell development. Only use `zig build -Doptimize=ReleaseFast` for final/shipping builds.

The Makefile may exist as a convenience wrapper, but normal development instructions must use PowerShell plus direct `zig` commands. Do not assume non-PowerShell shell tooling.

### Zig Toolchain

Use Zig 0.15.2 on Windows and make sure `zig.exe` is available on `PATH`.

Check the active Zig version from PowerShell:

```powershell
zig version
```

`build.zig` already defaults to `x86_64-windows-gnu`, so a normal development build should not need an explicit `-Dtarget`.

After a successful debug build, the expected artifact is:

```powershell
Test-Path .\zig-out\bin\phantty.exe
Get-Item .\zig-out\bin\phantty.exe
```

`Remove-Item -Recurse -Force .\zig-out, .\.zig-cache -ErrorAction SilentlyContinue` removes build outputs and Zig caches.

## Windows UI Automation

When debugging UI behavior, automate Phantty as a real visible Windows app from PowerShell. Prefer Win32-driven automation over shell-only assumptions.

Use the checked-in automation script for File Explorer regressions:

```powershell
zig build
powershell -NoProfile -ExecutionPolicy Bypass -File .\debug\test-file-explorer-ui.ps1
```

The script launches a real Phantty window, sets DPI awareness, fixes the window position and size, captures before/after screenshots, crops the right panel, sends `Ctrl+Shift+E`, performs a region-based pixel check, and writes screenshots plus JSON metrics under `zig-out\ui-test\`.

When adding more UI automation, follow the same pattern:
- Wait until `MainWindowHandle` is non-zero, call `ShowWindow` and `SetForegroundWindow`, then click inside the client area before sending keys.
- Prefer Win32 `keybd_event` or `SendInput` for shortcuts; `System.Windows.Forms.SendKeys` can silently miss GLFW/terminal windows when focus is not exactly right.
- Capture both full-window and cropped target-region screenshots, and inspect the crop when a pixel check fails.
- Always clean up test windows with `CloseMainWindow()`, then `Stop-Process -Force` if the process remains.

## Windows SSH/SCP Compatibility

When changing SSH/SCP code paths (`src/scp.zig`, SSH clipboard image paste, remote file explorer listing/upload/download, or SSH session metadata), test against the existing real SSH profile in `%APPDATA%\phantty\ssh_hosts` whenever it is available. The profile fields are hex encoded as `name, host, user, password, port`; decode them locally for the test, but never print or commit the password. At minimum, verify:

```powershell
ssh.exe ... user@host pwd
scp.exe ... local-file user@host:/tmp/test-file
ssh.exe -T ... user@host "cat > '/tmp/test-file'"  # only if testing the stream fallback
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xuzhougeng/phantty](https://github.com/xuzhougeng/phantty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
