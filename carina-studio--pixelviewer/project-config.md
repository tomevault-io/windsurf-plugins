---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.

## Code Conventions

### General
- Nullable reference types are enabled (`#nullable enable`) everywhere.
- Unsafe blocks are allowed globally.
- To pin a managed array or buffer, prefer the `fixed` statement for scope-bound pinning; reach for `GCHandle.Alloc(…, GCHandleType.Pinned)` only when the pin must outlive the current scope.
- To read fields of a native interop struct from a raw byte buffer, mark the method `unsafe` and access the struct in place through a `fixed` pointer (`fixed (byte* p = buffer) { var header = (SomeHeader*)p; … header->field … }`) rather than copying the whole struct out (`*(SomeHeader*)p`) or `MemoryMarshal.Read<T>()`; use `sizeof(SomeHeader)` for the length check and offsets rather than a hardcoded byte count. If the method also needs to be awaitable, keep it non-`async` and return the `Task` directly (an `await` cannot sit in the `unsafe` context, and a pointer cannot be held across it).
- Compare native handles against `IntPtr.Zero` explicitly (`handle == IntPtr.Zero`), not `default`.
- All public async methods return `Task`/`ValueTask`; UI-thread operations use `Dispatcher.UIThread`.
- `[ThreadSafe]` attributes mark thread-safe members explicitly.

### File and Type Organization
- One type per file; file name matches the type name exactly. **A class which exists only to hold extension members for another type is the exception.** When the extended type `T` is defined in this project, its `TExtensions` class is **preferred** to live at the bottom of `T.cs`, below `T` itself, rather than in a file of its own — as `BitmapFormatExtensions` does in `BitmapFormat.cs`, `ImageRendererExtensions` in `IImageRenderer.cs`, and `ColorTableExtensions` in `ColorTable.cs`. An extension class for a type this project does not define (a framework or library type) has no such file to join, so it keeps its own. `BayerPatternExtensions.cs` predates the rule and has not been moved yet.
- Namespace matches the folder path: `Carina.PixelViewer.<Subfolder>`.
- Inner types within a class/file are ordered **alphabetically** by name.
- `extension` blocks (C# 14 extension members) are placed **first** in the containing class, before all other members; they are not sorted with the members listed below. Members inside an `extension` block are ordered alphabetically.
- Enum values are ordered **alphabetically** — except in native-interop declarations (e.g. the P/Invoke types in `Native/Win32.cs`), where an enum mirrors the OS definition and keeps its native member names, ordering, and value grouping (for a native constant set used as a single field's value, such as `BITMAPV5HEADER.bV5CSType`, prefer a `[uint]`-backed enum over flattened constants).

Members inside a class are ordered as follows:

1. **Public constants** — no section comment; each member has its own `///` XML doc.
2. **Public static fields** — no section comment; each member has its own `///` XML doc.
3. **Inner types** — alphabetically ordered; each type has its own dedicated comment.
4. **Constants** (private/internal) — under a `// Constants.` section comment.
5. **Static fields** (private/internal) — under a `// Static fields.` section comment.
6. **Private fields** — under a `// Fields.` section comment.
7. **Static initializer** — under a `// Static initializer.` section comment.
8. **Constructors** — under a `// Constructor(s).` section comment.
9. **Non-private fields, properties, and methods** — ordered **alphabetically** by member name. Each member is preceded by:
   - a `///` XML doc comment for public members, OR
   - a single-line `//` comment describing the member, for private/internal members.

Exception: struct fields declared with `[StructLayout(LayoutKind.Sequential)]` must preserve their memory-layout order and cannot be reordered alphabetically.

### Naming

| Element | Convention | Example |
|---|---|---|
| Public properties | PascalCase | `ImageWidth`, `IsActivated` |
| Private fields (instance and static) | camelCase; instance fields always qualified with `this.` | `this.imageSource`, `this.isRendering`, `defaultRenderingOptions` |
| Internal / protected / public fields | PascalCase | `DefaultColorSpace` |
| Platform-specific fields/members | Prefix with the platform (`MacOS` / `Windows` / `Linux`) | `MacOSDataForTypeSelector`, `MacOSNSPasteboardClass` |
| Static `ObservableProperty` fields | PascalCase + `Prop` suffix | `ColorSpaceProp`, `BrightnessAdjustmentProp` |
| Private/helper methods | PascalCase | `GenerateHistogramImage()`, `UpdateSourceImageEffectiveBits()` |
| Public methods | PascalCase | `Render()`, `ExportImage()` |
| Async methods | Must end with `Async` | `RenderAsync()` |
| Constants | PascalCase | `RenderImageDelay` |
| Parameters & local variables | camelCase | `cancellationToken`, `pixelStride` |
| Interfaces | `I` prefix + PascalCase | `IImageDataSource`, `IImageRenderer` |
| Event handlers | `On` prefix | `OnRendererPropertyChanged` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [carina-studio/PixelViewer](https://github.com/carina-studio/PixelViewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
