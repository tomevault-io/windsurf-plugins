---
trigger: always_on
description: This file provides guidance to Claude Code when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code when working with code in this repository.

## Project Overview

OneNote Backup Exporter (C# WPF Edition) – a pure C# WPF desktop application for exporting OneNote notebooks via the COM API to multiple formats (.onepkg, .xps, .pdf). This is a migration of the original Go/Wails project; the JSON-RPC subprocess layer has been eliminated – the WPF app calls `OneNoteService` directly in-process.

## Architecture

Single-project WPF app, no subprocesses:

```
MainWindow.xaml / .xaml.cs   (UI + code-behind)
    ↕ async calls
Services/OneNoteComWorker.cs (serialized STA thread + message pump)
    ↕ synchronous calls on the owning STA thread
Services/OneNoteService.cs   (low-level OneNote COM API wrapper)
    ↕ COM API
OneNote Desktop
```

### File Structure

```
OneNoteExporterC#/
├── OneNoteExporter.csproj   Target: net10.0-windows, UseWPF, PlatformTarget=x64
├── OneNoteExporter.sln
├── App.xaml                  Global styles (colors, buttons, inputs)
├── App.xaml.cs               Global exception handler
├── AboutWindow.xaml/.cs      About dialog with runtime version/build metadata
├── MainWindow.xaml           Full UI layout
├── MainWindow.xaml.cs        All UI logic, async export flow
├── Models/
│   └── Models.cs             NotebookInfo, ExportResult, VersionInfo,
│                             BackupAvailability, NotebookViewModel
├── Services/
│   ├── OneNoteComWorker.cs   Dedicated STA owner, serialized async facade
│   └── OneNoteService.cs     Low-level hierarchy and publish COM calls
└── Helpers/
    ├── FileHelper.cs         Local backup path, CopyLocalBackup, OpenFolder,
    │                         GetDefaultDownloadsPath
    └── UserFacingError.cs    Stable English error text and language-neutral codes
```

## Development Commands

```powershell
# Build
dotnet build

# Run (debug)
dotnet run

# Publish self-contained x64
dotnet publish -c Release -r win-x64 --self-contained true
```

## Release Asset Naming

Public release assets must use stable, version-independent names so `releases/latest`
download URLs remain permanent:

- `OneNoteBackupExporter_Setup.exe`
- `OneNoteBackupExporter_Setup.exe.sha256`

Do not add the version number to public release asset filenames. Keep the semantic version
in the project, installer metadata, Git tag, GitHub release title, and About dialog instead.

## Critical Technical Details

### COM Compatibility
- **`PlatformTarget=x64` is mandatory** – this build targets current 64-bit OneNote Desktop installations. Changing the platform can cause "Class not registered" COM errors.
- **NuGet package:** `Interop.Microsoft.Office.Interop.OneNote` v1.1.0.2 (NOT `Microsoft.Office.Interop.OneNote`)
- **Do NOT add `<UseWindowsForms>true</UseWindowsForms>`** – causes namespace conflict with the COM `Application` class

### Threading Model
- `OneNoteComWorker` owns `OneNoteService` and its COM object on one dedicated STA thread
- All OneNote COM calls are dispatched through that thread's WPF `Dispatcher` message pump
- Public worker methods are async and serialized; do not wrap them in `Task.Run(...)`
- Export file polling runs asynchronously outside the STA thread while the serialization gate remains held
- Use `IProgress<string>` for progress reporting (marshals back to UI thread automatically via `Progress<T>`)
- Use `CancellationToken` for cooperative cancellation and `Task.Delay(..., ct)` in polling loops

### Export Process
1. `OneNoteComWorker.CreateAsync()` starts the STA thread and creates `OneNoteService` there
2. `GetNotebooksAsync()` retrieves all notebooks via `GetHierarchy()` XML parse on the worker
3. `ExportNotebookAsync()` publishes to a unique same-directory staging file on the STA worker
4. Recoverable COM/RPC failures get one bounded retry; disconnected COM objects are recreated and validated on the STA thread
5. Async file polling waits until size is stable for 10 seconds while the STA message pump stays responsive
6. A successful staging file atomically replaces the final export; previous good backups remain untouched on failure
7. Timeout: 20 min for local notebooks, 30 min for cloud (SharePoint/OneDrive); polling timeouts are not retried
8. OneNote does not expose notebook byte size through COM. Whole-notebook PDF/XPS
   jobs therefore show a preflight choice: automatically split into individual
   section exports (recommended), switch to `.onepkg`, or explicitly try the whole
   rendered export anyway

### Export Formats
| UI value | PublishFormat | Extension |
|---|---|---|
| `"onepkg"` | `pfOneNotePackage` | `.onepkg` |
| `"xps"` | `pfXPS` | `.xps` |
| `"pdf"` | `pfPDF` | `.pdf` |
| `"localbackup"` | n/a (file copy) | n/a |

### Local Backup Mode
- Copies `%LOCALAPPDATA%\Microsoft\OneNote\16.0\Sicherung` to destination
- Implemented in `FileHelper.CopyLocalBackup()`
- When selected in dropdown: all notebooks auto-checked, checkboxes disabled, warning shown

### OneNote COM Cleanup
- `OneNoteComWorker.Dispose()` is called in `MainWindow.Window_Closing`
- The worker releases the COM object on its owning STA thread and does NOT kill OneNote
- OneNote continues running in background after app closes (by design, avoids "could not be started on last attempt" error)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jli-software/OneNoteBackupExporter](https://github.com/jli-software/OneNoteBackupExporter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
