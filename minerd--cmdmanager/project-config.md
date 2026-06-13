---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

CmdManager is a WPF (.NET 8, **x64-only**) utility that monitors and remote-controls open `cmd.exe` windows on this machine. It lists cmd windows working on the E: or F: drives, shows a live colored preview of the selected console, indicates busy/idle state, sends commands to consoles without stealing focus, and keeps history/favorites of working directories. It is built around an external `cc` command assumed to be on PATH (the user's Claude Code launcher) — several actions open a cmd and run `cc` in it. The UI is localized (English default, Turkish toggle via the 🌐 button): XAML strings resolve through `DynamicResource` keys rebuilt by `ApplyLanguage()` from the `UiStrings` table, and code-behind strings use `Loc.T(en, tr)`. The chosen language persists in `settings.json` (`Language`).

## Build & Run

- Build: `dotnet build -c Release`
- Run: launch `bin\Release\net8.0-windows\win-x64\CmdManager.exe`. The manifest declares `requireAdministrator` (needed for `ReadProcessMemory`/`AttachConsole` on other processes), so every launch triggers UAC.
- Publish single-file to `publish/`: `dotnet publish -c Release -r win-x64 --self-contained false -p:PublishSingleFile=true -o publish`
- Regenerate the icon: `powershell -File make_icon.ps1` (draws the `>_` glyph programmatically and writes `app.ico`)
- There are no tests and no linter config.
- **Gotcha:** `App.xaml` sets `ShutdownMode="OnExplicitShutdown"` and nothing ever calls `Application.Shutdown()`, so closing the main window leaves the process running. A surviving instance locks the exe and makes the next build fail with a file-in-use error — kill `CmdManager` from Task Manager before rebuilding.

## Architecture

All logic lives in two files:

- `Native.cs` — every Win32 P/Invoke, exposed as static methods on `Native` (enumerate cmd windows, read another process's working directory, read/write another process's console, global hotkey, focus/keyboard helpers).
- `MainWindow.xaml.cs` — all UI state and behavior: the `CmdItem`/`HistoryItem`/`FavoriteItem` models, the polling timers, persistence, and every event handler. `InputDialog` (a code-built prompt window) is at the bottom of the same file.

### Cross-process console access (the core mechanism)

Every operation on another cmd's console (`ReadScreenBuffer`, `ReadScreenCells`, `SendText`) follows the pattern: `FreeConsole()` → `AttachConsole(pid)` → open `CONOUT$`/`CONIN$` → operate → `FreeConsole()`. Console attachment is **process-wide state**, so these calls must not overlap; everything currently runs on the UI thread via `DispatcherTimer`. Do not move `Native` calls to background threads without serializing them.

### Working-directory discovery

`Native.GetWorkingDirectory` reads the target process's PEB via `NtQueryInformationProcess` + `ReadProcessMemory` using **hardcoded x64 offsets** (ProcessParameters at PEB+0x20, CurrentDirectory UNICODE_STRING at ProcessParameters+0x38). This is why the csproj pins `Platforms`/`PlatformTarget` to x64 — the offsets are wrong on x86.

### Drive filter

Only cmds whose current directory starts with a drive in `AllowedDrives` (`E:`, `F:` — `MainWindow.xaml.cs`) appear in the list or get recorded to history. All other cmd windows are deliberately invisible to the app.

### Polling loops

- `_timer` (1.5 s): re-enumerates cmd processes, detects closed ones (their last-known directory is pushed to history), refreshes the preview. `_lastKnownByPid` preserves the previous directory/hwnd when a read transiently fails, so closed-cmd history stays accurate.
- `_busyTimer` (400 ms): hashes each cmd's screen text; unchanged for >700 ms ⇒ `IsIdle = true` (teal dashed border in the list, red while busy). If the command was sent from this app (`MarkSent`), a system beep plays when output settles — a completion notification.

### Sending input — two deliberate paths

- **Silent** (`Native.SendText`): writes KEY_EVENT records directly to the target's `CONIN$` via `WriteConsoleInputW` (default two trailing Enters, callers can pass `enters: 1`). No focus change. Turkish/Unicode works **only because** `KEY_EVENT_RECORD`/`INPUT_RECORD`/`CHAR_INFO` carry `CharSet = CharSet.Unicode` — without it the marshaler silently degrades `char` to one ANSI byte (ı→'1', ş→'_' in the preview; ş→'þ' on send). Don't remove those attributes.
- **Paste** (`SendViaPaste`): saves the clipboard, copies the text, focuses the cmd window, simulates Ctrl+V + Enter with `keybd_event`, then restores the clipboard and refocuses CmdManager. Kept as fallback for very long text; steals focus briefly.

### Preview rendering

`Native.ReadScreenCells` reads the visible console region row-by-row as `CHAR_INFO` (chars + color attributes; wide-char trailing cells are skipped). `RenderColoredPreview` converts rows into `FlowDocument` paragraphs with runs colored by the Campbell palette (`ConsoleColors`). A content hash skips re-rendering when nothing changed, and the scroll position sticks to the bottom only if the user was already there.

### Auto-resume on Claude Code usage limits


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [minerd/CmdManager](https://github.com/minerd/CmdManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
