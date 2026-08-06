---
trigger: always_on
description: WPF desktop compression/decompression app (.NET 9, Windows only). Three projects: `MantisZip.Core` (class library) + `MantisZip.ShellExt` (COM component class library) + `MantisZip.UI` (WinExe).
---

# MantisZip — Agent Guide

## Project overview

WPF desktop compression/decompression app (.NET 9, Windows only). Three projects: `MantisZip.Core` (class library) + `MantisZip.ShellExt` (COM component class library) + `MantisZip.UI` (WinExe).

## Quick start

```powershell
# Build everything
dotnet build src\MantisZip.UI\MantisZip.UI.csproj

# Run (requires Windows)
dotnet run --project src\MantisZip.UI\MantisZip.UI.csproj

# Tests are in tests/MantisZip.Tests/ (xUnit, 40+ test cases).
# test_encoding/ is a throwaway CLI tool for debugging ZIP encoding, not a test suite.
```

## Architecture

### Dependency flow

```
MantisZip.UI (WPF) ──reference──▶ MantisZip.Core (net9.0)
                                        │
                        ┌───────────────┼───────────────┐
                   ZipEngine    SevenZipEngine    TarGzEngine
                  (SharpCompress) (SharpSevenZip) (SharpCompress)

MantisZip.ShellExt (COM) ──独立──▶ (Explorer.exe 宿主，无直接项目引用)
```

### Engine pattern (strategy + factory)

- `IArchiveEngine` interface: `ListEntriesAsync`, `ExtractAsync`, `CompressAsync`, `TestArchiveAsync`
- `ArchiveEngineFactory` registers engines in static constructor, dispatches by file extension
- `SevenZipEngine.CompressAsync` uses `SharpSevenZipCompressor` (7z.dll COM binding)
- `ArchiveEntryExtractor` (Core/Utils) handles single-entry extraction for preview; only supports Zip and 7z

### Progress reporting

- `ArchiveProgress` (Core/Abstractions/ArchiveEngine.cs): `PercentComplete` (overall, 0–100), `FilePercentComplete` (nullable double, 0–100 for per-file granularity), `FileName` (current file name), `Message`
- `ZipEngine` reports per-file progress via buffered I/O copy loop with 100ms throttle; reports initial 0% and final 100% for each file
- `SevenZipEngine.ExtractAsync` and `TarGzEngine.ExtractAsync` report progress only at completion (100%)
- `SevenZipEngine.CompressAsync` reports progress via `SharpSevenZipCompressor.Compressing` event
- `ProgressWindow` shows two progress bars: file-level (top) and overall (bottom); `SetProgress(ArchiveProgress)` overload drives both

### ArchiveItem duality

- **Core**: `MantisZip.Core.Abstractions.ArchiveItem` — engines produce these
- **UI**: `MainWindow.xaml.cs` defines a subclass `ArchiveItem : Core.Abstractions.ArchiveItem` adding `DisplayName`, `SizeDisplay`, `NameDisplay`, `SortOrder` — all engine output is mapped into UI instances in `LoadArchiveAsync`

### UI pattern: code-behind, not MVVM

Despite using `CommunityToolkit.Mvvm`, **all logic lives in `MainWindow.xaml.cs`**. No ViewModel classes exist. The `FolderNode` class at the bottom of that file implements `INotifyPropertyChanged` for TreeView binding only.

### Preview subsystem

- Trigger: `FileListGrid_SelectionChanged` → files via `ShowPreviewAsync(item)`, directories via `ShowDirectoryPreview(item)` (system folder icon + directory info panel)
- **`ExtractPreviewFileAsync(item, fallbackName, ct)`** — shared helper (lines ~139) for temp extraction; creates temp dir, extracts, returns file path. Replaces 14 identical 5-line extraction blocks. Callers just `var tempFile = await ExtractPreviewFileAsync(item, "preview" + ext, ct);`
- **`HideAllPreviewControls()`** — collapses all 5 content controls (`PreviewImage`, `PreviewTextBox`, `PreviewFileIcon`, `PreviewUnsupported`, `PreviewWebView2`). Called at the start of every `Show*Preview` method before showing the relevant control. Ensures no orphaned visibility states from previous formats.
- **`SetToolbar(ToolbarButton[] leftButtons, ToolbarButton[] rightButtons)`** — left side is common controls (zoom, font size), right side is format-specific (transparency toggle, ligature toggle, GIF frame nav). Separator auto-inserted between left and right arrays. Callers specify both arrays explicitly.
- **Info panel clearing** is centralized in `ShowPreviewAsync` (line ~165-167) — `PreviewExtraInfoPanel.Children.Clear()` + `.Visibility = Collapsed` before any format's display method runs. Individual format methods no longer need to clean up.
- **`SetFormatSpecificInfo(params (string, string)[] pairs)`** — adds key-value rows to `PreviewExtraInfoPanel`; used by all metadata formats (PE/PDF/Font/Audio/SQLite/ISO/Office/Video) and Image/GIF.
- Display (per format):
  - **Image**: `ShowImagePreviewAsync` — BitmapImage with DecodePixelWidth=1920 downsampling; zoom toolbar + transparency toggle for PNG/ICO/WebP
  - **GIF**: Same `ShowImagePreviewAsync` path with `WpfAnimatedGif` — play/pause/frame-nav toolbar + editable frame input (TextBox + total label)
  - **Text**: `ShowTextPreview` — UTF-8/GBK detection via Ude.NetStandard; font size toolbar
  - **HTML**: `ShowHtmlPreview` — WebView2 rendering (network requests blocked)
  - **Markdown**: `ShowMarkdownPreview` — Markdig → HTML → WebView2 (dark theme support via `prefers-color-scheme`)
  - **PE**: `ShowPePreview` — product name, company, version, architecture, subsystem
  - **PDF**: `ShowPdfPreview` — metadata + WebView2 PDF content rendering (size-gated)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mantis3d/MantisZip](https://github.com/mantis3d/MantisZip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
