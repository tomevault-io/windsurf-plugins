---
trigger: always_on
description: **dicom_toolkit** is a Flutter plugin for workstation-grade DICOM medical imaging. It pairs a **Rust native core** (`dicom` crate v0.9) with **GPU fragment shaders** (GLSL) to deliver 16-bit precision, real-time windowing, and zero-UI-thread-blocking rendering. The Dart↔Rust bridge is powered by `flutter_rust_bridge` v2.12.0.
---

# AGENTS.md — dicom_toolkit

## Project Identity

**dicom_toolkit** is a Flutter plugin for workstation-grade DICOM medical imaging. It pairs a **Rust native core** (`dicom` crate v0.9) with **GPU fragment shaders** (GLSL) to deliver 16-bit precision, real-time windowing, and zero-UI-thread-blocking rendering. The Dart↔Rust bridge is powered by `flutter_rust_bridge` v2.12.0.

- **License**: GPL v3 — forked from [MostafaSensei106/Flutter-Dicom](https://github.com/MostafaSensei106/Flutter-Dicom)
- **Platforms**: Android, iOS, Linux, macOS, Windows, Web (WASM)
- **Version**: `0.2.10`

---

## Architecture

```
lib/
  dicom_toolkit.dart                 ← barrel: exports + DicomToolkit class
  src/
    backend/
      dicom_decoder.dart             ← DicomDecoder (abstract) + RustDecoder (FFI impl)
    core/
      dicom_tag_id.dart              ← DicomTagId (group, element) + 39 constants
      dicom_metadata.dart            ← DicomMetadata wrapper (typed getters + tag lookup)
      dicom_pixel_data.dart          ← sealed DicomPixelData + DicomInt16PixelData
      dicom_parse_result.dart        ← DicomParseResult (metadata + pixels + frame API)
      constants/
        color_maps.dart              ← DicomColorMap enum + ColorMapLut generator
        lib_shaders.dart             ← shader asset path constant
      exceptions/
        dicom_exceptions.dart        ← DicomException + DicomProcessingException
      services/
        dicom_reader.dart            ← readDicomInfo() + DicomFileInfo (metadata-only)
      shader/
        dicom_shader_painter.dart    ← CustomPainter binding shader + texture
      widgets/
        dicom_viewer.dart            ← interactive viewer widget (GPU + gestures)
    tools/
      dicom_export.dart              ← PNG export (web-compatible, no dart:io)
      dicom_parser.dart              ← parsing entry point (DI DicomDecoder)
      dicom_renderer.dart            ← GPU rendering (shader compile, texture pack, render)
      dicom_roi.dart                 ← rectangular ROI + RoiStatistics
      dicom_ruler.dart               ← mm distance from pixel spacing
      dicom_window_preset.dart       ← presets (CT constants + forImage() factory)
    debug_log.dart                   ← conditional debug-only logger (kDebugMode)
    viewer/
      dicom_viewer_controller.dart   ← ChangeNotifier state manager
    rust/                            ← AUTO-GENERATED — never hand-edit
      frb_generated.dart
      frb_generated.io.dart / .web.dart
      api/init.dart, api/core/...

rust/
  Cargo.toml                        ← deps: dicom=0.9, flutter_rust_bridge=2.12.0
  src/
    lib.rs                          ← pub mod api; mod frb_generated;
    frb_generated.rs                ← AUTO-GENERATED
    api/
      init.rs                       ← load_dicom / load_dicom_from_bytes FFI entry
      core/
        config/dicom_config.rs      ← auto_normalize, skip_pixels
        models/dicom_metadata.rs    ← 37-field struct + Default impl
        models/dicom_frame_result.rs← metadata + Vec<i16>
        constants/lib_constants.rs  ← DefaultConfigs consts
        utils/process_dicom_file.rs ← THE CORE: parses .dcm, extracts tags+pixels

assets/shaders/dicom_window.frag    ← GLSL: 16-bit unpack + HU + windowing + LUT

web/pkg/                            ← WASM + JS bindings (wasm-pack output, committed to git)

test/
  dicom_tag_id_test.dart            ← 31 constants, equality, hex
  dicom_pixel_data_test.dart        ← sealed hierarchy, buffer
  dicom_window_preset_test.dart     ← 7 presets, forImage(), equality
  dicom_window_preset_for_image_test.dart ← forImage() edge cases
  dicom_metadata_test.dart          ← typed getters, tag(), pixelSpacing
  dicom_parse_result_test.dart      ← fromFrame, frame(), hasPixels
  dicom_roi_test.dart               ← compute(), copyWith, clamping
  dicom_ruler_test.dart             ← measure(), spacing fallback
  dicom_parser_test.dart            ← mocked decoder delegation
  dicom_renderer_test.dart          ← pack16Bit + applyWindowingRgba helpers
  dicom_color_map_lut_test.dart     ← ColorMapLut.generate for every map
  dicom_export_test.dart            ← PNG export + failure-path disposal
  dicom_viewer_controller_test.dart ← state lifecycle, error paths
  dicom_viewer_test.dart            ← widget states (loading, error, empty)
  dicom_exceptions_test.dart        ← DicomException hierarchy
  features_test.dart                ← cross-tool integration
  dicom_integration_test.dart       ← real .dcm files via parser
  dicom_reader_test.dart            ← readDicomInfo() with real files
```

---

## Data Flow

1. `DicomToolkit.init()` — loads native library / WASM
2. `DicomParser.parse(bytes)` → `DicomDecoder.decode()` → `loadDicomFromBytes()` FFI
3. Rust `process_dicom_file.rs`: opens DICOM, extracts 37 tags + pixel spacing (with Imager Pixel Spacing fallback for X-ray), extracts `Vec<i16>` pixels (first frame only via dual-path: raw bytes for uncompressed, decoder for compressed)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elselawi/dicom_toolkit](https://github.com/elselawi/dicom_toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
