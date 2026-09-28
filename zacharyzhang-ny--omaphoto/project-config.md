---
trigger: always_on
description: OmaPhoto is the Linux port of Compositor, the macOS app at https://github.com/robbietilton/Compositor. That repository's Swift tree (`Compositor/`, `CompositorTests/`) is the specification; this repository mirrors it. Keep a checkout of it beside this one to read the Swift sources; never edit it for OmaPhoto work.
---

# OmaPhoto — agent notes

OmaPhoto is the Linux port of Compositor, the macOS app at https://github.com/robbietilton/Compositor. That repository's Swift tree (`Compositor/`, `CompositorTests/`) is the specification; this repository mirrors it. Keep a checkout of it beside this one to read the Swift sources; never edit it for OmaPhoto work.

## Trees

- The repository root: the app. C++20, Qt 6 Widgets (floor Qt 6.4), CMake, Qt Test. CPU raster only: QPainter plus the C kernels in `kernels/`, copied unchanged from Compositor's `Compositor/Rendering/*.c` and compiled in place.
- `packaging/`: the desktop entry, MIME type, icons (Compositor's app icon) and the Arch `PKGBUILD`; `flake.nix` at the root for NixOS.
- `scripts/install/`: one install script per distribution family, each fetching the latest release's package.
- `.github/workflows/release.yml`: a `v*` tag builds the DEB, RPM and Arch package and publishes them.
- `TASKS.md`: ordered task list and progress truth. Its entries before 11.4b speak of the port's old home, `linux/` inside Compositor's repository.

Names: the app is OmaPhoto (binary `omaphoto`, app id `io.github.ZacharyZhang_NY.OmaPhoto`, logging `omaphoto.*`, build options `OMAPHOTO_*`). Compositor's project format stays whole (`.comp`, `application/x-compositor-project`), so projects move between the two apps; C++ types keep their Swift twins' names (`CompositorApp`, `CompositorMenus`).

## Mirror rule

Each Swift file has one Linux twin under `src/` in the same folder (`Document`, `Rendering`, `UI`, `IO`) with the same type and function names. Tests in `tests/` twin `CompositorTests/`. A Swift file over 500 lines splits as `Name+Part`. A Swift extension of `EditorSession` declares its members in `Name+Session.h` (`EditorSession+Brush.h` and `EditorSession+Projects.h` for the extensions named after the session): C++ declares members inside the class, so `EditorSession.h`, the twin of `EditorSession.swift` with every stored property, includes each fragment inside its body. A fragment holds declarations alone, its extension's private helpers under `private:`; state stays in `EditorSession.h`, as Swift keeps stored properties in the class. Qt types are used directly: `QImage` for `CGImage`, `QRectF`, `QPointF`, `QTransform`, `QUuid`. No wrapper types. Nothing outside `UI/` includes `UI/` headers, except where the Swift source does.

Platform adaptations, each forced by a missing Qt analog:

- `ImportedImage` exposes pixels through `image()`, size through `size()`, identity through `identity()`. CoreGraphics backs a painted layer's `CGImage` with a lazy data provider; Qt has none. `image()` flattens painted tiles on first use, so commits, display and history never flatten a layer. Never cache its result in a model type.
- Image identity (`===` on `CGImage`) is `ImageIdentity`: the `RasterSnapshot` pointer for painted assets, `QImage::cacheKey()` otherwise.
- Layer pixels are `QImage::Format_RGBA8888_Premultiplied`; masks are `QImage::Format_Grayscale8`. `BrushRaster::context` makes both.
- Contexts are top-left already, so the CoreGraphics y-flips vanish. `BrushRaster::draw` has no `mask` flag: both CoreGraphics paths are a plain copy in Qt. It takes an optional source part instead of `CGImage.cropping(to:)`, which Qt cannot do without a copy.
- `BrushRaster::visibleRect` stands in for `boundingBoxOfClipPath`.
- Images are drawn into explicit target rects. The positional `drawImage(x, y, image)` scales by the source's device pixel ratio; CoreGraphics never does.
- `DownsampleCache` halves with its own Lanczos-3 filter (vImage has no Linux twin), threaded with QtConcurrent. Swift's label-overloaded `image(_:drawnAt:)` and `image(_:level:)` are `imageDrawnAt` and `imageAtLevel`. Its constructor takes a pixel budget so eviction can be tested; the app uses `shared()`.
- `LayerRenderer::draw` takes Swift's labelled arguments as an `Options` struct. QPainter has no mask clip, so a masked layer is composed aside in device pixels; the mask is drawn with the same placement into a zero-filled `Alpha8` surface and multiplied in 1:1 (`DestinationIn`), so nothing outside the mask survives; the result is composited with the layer's opacity and blend mode. Work that can throw finishes before `save()`. `LayerRenderer` needs a painter on a `QImage`.
- CoreGraphics multiplies nested mask clips inside the context; QPainter has no soft clip. `FolderMaskClip::apply` multiplies a folder mask into device-sized `Alpha8` coverage, `FolderMaskClip::draw` hands each layer the product of its folders' coverage (memoized per folder), and draw routines take it as `LayerRenderer::Options::clip`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZacharyZhang-NY/OmaPhoto](https://github.com/ZacharyZhang-NY/OmaPhoto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
