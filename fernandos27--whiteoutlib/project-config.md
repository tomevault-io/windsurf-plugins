---
trigger: always_on
description: > This document is the authoritative map for navigating WhiteoutLib.
---

# WhiteoutLib — Architecture Guide for LLM Assistants

> This document is the authoritative map for navigating WhiteoutLib.
> Read it before making changes. It is optimized for LLM context windows.

## 1. Project Identity

WhiteoutLib is a **C++20 library** (public headers are C++11-compatible) for
reading and writing binary 3D model, texture, and data formats used in
Blizzard Entertainment games. It is a **standalone static library** with one
optional dependency (CascLib, fetched via CMake FetchContent).

| Area | Games | Formats |
|------|-------|---------|
| 3D models | Warcraft III | `.mdx` (Classic + Reforged) |
| 3D models | World of Warcraft | `.m2`, `.skin`, `.anim`, `.skel`, `.bone` |
| 3D models | StarCraft II / HotS | `.m3`, `.m3a` |
| Textures | WC3 / WoW | `.blp` (BLP1 + BLP2) |
| Textures | General | `.dds` (legacy + DX10) |
| Textures | General | `.bmp` (24/32-bit uncompressed) |
| Textures | General | `.tga` (uncompressed + RLE) |
| Textures | D3 / D4 | `.tex` (Blizzard proprietary) |
| Data files | Diablo III | SNO files (`.acr`, `.app`, `.ani`, etc.) |
| Data files | Diablo IV | SNO files (same extensions, different layout) |
| Archive access | D3 / D4 / WoW | CASC (via CascLib wrapper) |

**License:** BSD-3-Clause. **Build:** CMake ≥ 3.15, MSVC or GCC/Clang.

---

## 2. Repository Layout

```
WhiteoutLib/
├── CMakeLists.txt              # Single build file: library + all examples
├── include/whiteout/           # PUBLIC HEADERS (C++11-compatible surface)
│   ├── common_types.h          #   u8/u16/u32/i32/f32/f16/snorm/unorm/fixed_point
│   ├── compatibility.h         #   std::optional/std::span shims for C++11/14
│   ├── vector_types.h          #   Vector2f/3f/4f, Quaternion, Matrix44f
│   ├── utils.h                 #   VirtualFileSystem, VertexBuffer, VertexBufferBuilder
│   ├── storages/casc/           #   Pure-C++ CASC archive reader/writer (storage.h, types.h)
│   ├── models/                 #   3D model formats
│   │   ├── m2/                 #     WoW M2 format (parser.h, writer.h, structures.h, types.h)
│   │   ├── m3/                 #     SC2 M3 format (same pattern)
│   │   ├── mdx/                #     WC3 MDX format (same pattern)
│   │   └── wem/                #     WEM intermediate format (wem.h, converters.h, parser.h, writer.h)
│   ├── sno/                    #   Diablo SNO (sno_reader.h, sno_value.h, sno_types.h, core_toc.h)
│   └── textures/               #   Texture class + format-specific public headers
│       ├── texture.h           #     Texture, PixelFormat, TextureKind, TextureType
│       ├── blp/                #     BLP1/BLP2 (blp.h, parser.h, writer.h, types.h)
│       ├── bmp/                #     BMP (bmp.h, parser.h, writer.h)
│       ├── dds/                #     DDS (dds.h, parser.h, writer.h)
│       ├── tex/                #     Blizzard TEX (tex.h, parser.h, writer.h, types.h)
│       └── tga/                #     TGA (tga.h, parser.h, writer.h)
├── src/whiteout/               # PRIVATE IMPLEMENTATION
│   ├── common/                 #   binary_reader.h, binary_writer.h, concepts.h, streams.h
│   ├── models/                 #   3D model format implementations
│   │   ├── m2/                 #     parser.cpp, writer.cpp, file_system, binary_*_visitor/
│   │   ├── m3/                 #     parser.cpp, writer.cpp, binary_*_visitor (*.inl)
│   │   ├── mdx/                #     parser.cpp, writer.cpp
│   │   └── wem/                #     parser.cpp, writer.cpp, converters/{mdx,m2,m3}_converter.cpp
│   ├── sno/                    #   sno_reader.cpp, sno_defs.h/.cpp, sno_value.cpp, core_toc.cpp
│   │   ├── d3/                 #   D3-specific auto-generated type registry
│   │   └── d4/                 #   D4-specific auto-generated type registry
│   ├── textures/               #   texture.cpp, format parsers/writers, codec suite
│   │   ├── bcn/                #     BC1–BC7 encode/decode (bc1.cpp … bc7.cpp, bcn_common.cpp)
│   │   ├── bcn.h / bcn.cpp     #     Format-agnostic BCn dispatcher
│   │   ├── blp/                #     BLP1/BLP2 parser + writer
│   │   ├── bmp/                #     BMP parser + writer
│   │   ├── dds/                #     DDS parser + writer
│   │   ├── jpeg/               #     Custom Huffman + JPEG (no libjpeg dependency)
│   │   ├── mipmap/             #     Mipmap generation pipeline (filters, stages, generator)
│   │   ├── tex/                #     Blizzard TEX parser + writer
│   │   ├── tga/                #     TGA parser + writer (uncompressed + RLE)
│   │   ├── utils/              #     Shared texture utilities
│   │   │   ├── pixel_convert.h #       Per-pixel format converters via RGBA32F intermediate
│   │   │   ├── srgb_linearize.h#       sRGB ↔ linear conversion with LUT
│   │   │   ├── quantize.h/.cpp #       Wu's color quantization for palette generation
│   │   │   ├── blue_noise.h/.cpp#      Void-and-Cluster blue-noise threshold map
│   │   ├── io_helpers.h        #     File I/O utilities (read_file_bytes, write_file_bytes)
│   │   └── issue_sink.h        #     Base class for strict/lenient error reporting
│   └── storages/casc/           #   Pure-C++ CASC implementation (storage.cpp, writer.cpp, etc.)
├── examples/                   # Standalone example programs (one per format)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FernandoS27/WhiteoutLib](https://github.com/FernandoS27/WhiteoutLib) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
