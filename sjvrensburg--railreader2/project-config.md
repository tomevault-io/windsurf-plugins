---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**railreader2** is a desktop PDF viewer with AI-guided "rail reading" for high magnification viewing. Built in C# with .NET 10, Avalonia 12 (UI framework), PDFtoImage/PDFium (PDF rasterisation), SkiaSharp 3 (GPU rendering via Avalonia's Skia backend), and ONNX layout detection (Docling Heron INT8 bundled by default, PP-DocLayoutV3 or PP-S available as alternatives).

## Build and Development Commands

**Prerequisites**: .NET 10 SDK (all projects target `net10.0`).

```bash
# Build app + CLI + tests (default solution)
dotnet build RailReader2.slnx

# Run the application (-- separates dotnet args from app args)
dotnet run -c Release --project src/RailReader2 -- <path-to-pdf>

# Run without arguments (shows welcome screen)
dotnet run -c Release --project src/RailReader2

# Run the CLI headless tool
dotnet run -c Release --project src/RailReader2.Cli -- render <pdf> --output-dir ./out
dotnet run -c Release --project src/RailReader2.Cli -- structure <pdf> --output out.json
dotnet run -c Release --project src/RailReader2.Cli -- annotations <pdf> --include-text --output ann.json
dotnet run -c Release --project src/RailReader2.Cli -- vlm <pdf> --output vlm.json
dotnet run -c Release --project src/RailReader2.Cli -- export <pdf> --no-vlm --output doc.md

# Run tests (Export is the test project in this repo; Core tests live upstream in RailReaderCore)
dotnet test tests/RailReader.Export.Tests

# Run specific test class
dotnet test tests/RailReader.Export.Tests --filter "ClassName=RailReader.Export.Tests.HeadingLevelResolverTests"

# Run specific test method
dotnet test tests/RailReader.Export.Tests --filter "FullyQualifiedName~TestMethodName"

# Regenerate README/website screenshots from the real UI (headless, writes into docs/img/)
dotnet run --project src/Tools/RenderHarness.Headless -c Release            # all shots
dotnet run --project src/Tools/RenderHarness.Headless -c Release --only rail_mode

# Publish self-contained release
dotnet publish src/RailReader2 -c Release -r linux-x64 --self-contained   # Linux
dotnet publish src/RailReader2 -c Release -r win-x64 --self-contained     # Windows

# Download ONNX model (required for AI layout)
./scripts/download-model.sh
```

**Always use `-c Release`** — debug builds are significantly slower.

## Architecture

```
RailReader2.slnx                  # Default: app + CLI + screenshot tool + tests
├── src/RailReader2/              # Thin Avalonia UI shell
├── src/RailReader2.Cli/          # Headless CLI tool (zero Avalonia)
├── src/Tools/RenderHarness.Headless/ # Headless doc-screenshot generator (references the GUI project)
└── tests/RailReader.Export.Tests/ # xUnit tests for the upstream Export package (Core tests live upstream)
```

The portable core — `RailReader.Core`, `RailReader.Core.Pdfium`, `RailReader.Core.Analysis`, `RailReader.Renderer.Skia`, `RailReader.Core.Vlm.OpenAI`, `RailReader.Core.Ocr.RapidOcr`, `RailReader.Export` — lives in the separate [RailReaderCore](https://github.com/sjvrensburg/RailReaderCore) repository and is consumed here as NuGet packages. All references in this document to types like `DocumentController`, `DocumentModel`, `Viewport`, `AppConfig`, `AnnotationService`, `LayoutAnalyzer`, `SkiaPdfService`, `OverlayRenderer`, `RailReaderLogging`, etc. resolve through those packages. (Core's per-document model is `DocumentModel` — older code/notes may call it `DocumentState`.) Logger bootstrap goes via `RailReaderLogging.Logger = new ConsoleLogger();` once at startup.

### RailReader2 (Avalonia UI shell)

Thin wrapper delegating all logic to `DocumentController`/`DocumentModel` in Core.

- `ViewModels/MainWindowViewModel.cs` (+ `.Annotations.cs` / `.Documents.cs` / `.Navigation.cs` / `.Search.cs` / `.Vlm.cs` / `.Portals.cs` / `.FreezePanes.cs` / `.TabReset.cs` partials) — thin wrapper handling Avalonia-specific concerns (file dialogs, clipboard, invalidation). Owns the **surface registry** (`Surfaces`/`RegisterSurface`/`FocusSurface`) driving the multi-viewport frame loop (see below).
- `ViewModels/TabViewModel.cs` — wraps **one `Viewport` + a shared `DocumentModel`**: per-view members (Camera/Rail/CurrentPage/dims/images) delegate to the `Viewport`; document-level members (Title, display prefs, caches, annotations, outline) to the `DocumentModel`. Two tabs of the same file share one `DocumentModel` and add a `Viewport` each (shared PDF/caches/annotations, independent camera/page/rail).
- `ViewModels/IViewportSurface.cs` / `ViewportImages.cs` — a renderable, tickable, focusable surface (implemented by `DocumentView`) and its per-viewport `SKImage` lifecycle.
- `Views/MainWindow.axaml.cs` (+ `MainWindow.Panes.cs` / `MainWindow.DocumentWindows.cs`) — window chrome + keyboard shortcuts; wires `InvalidationCallbacks` to each `DocumentView`. `Panes.cs` builds the split-pane `PaneGrid` (N side-by-side `DocumentView`s + `GridSplitter`s); `DocumentWindows.cs` + `DocumentWindow.axaml(.cs)` host tear-off floating windows.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sjvrensburg/railreader2](https://github.com/sjvrensburg/railreader2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
