---
trigger: always_on
description: Read this file before changing the project. It records the product goal,
---

# RayoMD Agent Guide

Read this file before changing the project. It records the product goal,
architecture, release hygiene, and benchmark guardrails that are easy to lose
between sessions.

## Product Goal

RayoMD is a tiny native Markdown-to-PDF converter. The default value is:

- very small packaged binaries
- fast startup and conversion
- low memory use for normal documents
- no browser, LaTeX, or Pandoc dependency on the native fast path
- a polished Windows Dear ImGui app plus a compact cross-platform CLI

Treat the native renderer as a fast Markdown subset, not a full Pandoc clone.
Pandoc compatibility may exist as an optional Windows mode, but it must not
become a required dependency for the default lightweight package.

## Release Readiness Priorities

- Keep the repository root clean: source entry points, `README.md`, `LICENSE`,
  `CMakeLists.txt`, `AGENTS.md`, `CONTRIBUTING.md`, `VERSION`, and the main
  smoke document belong there.
- Keep source-coupled fixtures, release records, specifications, and decision
  documents under docs/; keep longer user guides, dated reports, and research
  archives in the GitHub wiki.
- Keep generated build trees, generated PDFs, benchmark corpora, local binaries,
  caches, and one-off scratch files out of source.
- Prefer reproducible helper scripts under `scripts/` over ad hoc command blobs.
- Keep README claims accurate, conservative, and easy to reproduce.
- Do not advertise native mode as full CommonMark, Pandoc, LaTeX math, or
  syntax-highlight compatible unless those features are actually implemented.

## Current Feature Surface

Native PDF mode supports the core document features listed in `README.md`,
including headings, paragraphs, lists, block quotes, fenced code blocks, pipe
tables, rule lines, simple math cleanup/boxes, inline emphasis cleanup,
clickable Markdown links, standalone local images, and HTTP/HTTPS images on
Windows or curl-enabled Linux builds with fallback text.
Native exports can opt into the `rayomd-source/1` reversible PDF profile.
Embedding is disabled by default because it exposes the complete source,
including content not visible on rendered pages. Recovery is byte-exact and
format-specific; it must never silently fall back to heuristic PDF conversion.
Keep ordinary non-reversible exports on their unchanged PDF 1.7 fast path.
The profile limits PDFs to 256 MiB and source to 10 MiB. The source cap is
based on the measured maximum exact-recovery case; reject larger inputs before
rendering to avoid excessive output and peak memory.

Important image/link details:

- Local image paths are resolved relative to `TinyPdf::BuildOptions::sourcePath`
  when the caller provides an input file path.
- URL images are controlled by `BuildOptions::enableUrlImages`.
- Windows image support uses WinHTTP/WIC through the Win32 build.
- Linux URL images use libcurl when `RAYOMD_USE_CURL` is defined.
- PNG alpha support can use zlib when `RAYOMD_USE_ZLIB` is defined.
- Failed images should degrade to useful fallback text instead of failing the
  whole conversion.
- Links are emitted as PDF annotations; keep visible text and annotation rects
  aligned when changing wrapping or text layout.

## Architecture Map

- `include/rayomd/tiny_pdf.h`
  Public native exporter API. Keep this small and stable.

- `src/core/tiny_pdf.cpp`
  Native PDF assembly, font handling, image decoding/cache, link annotations, and the
  standard/Unicode renderers. This remains the performance-critical export facade.

- `src/core/markdown_parser.cpp`
  Internal Markdown block model and parser; keep renderer-independent document cleanup here.

- `src/core/rayomd_pdf_source.h` and `src/core/rayomd_pdf_source.cpp`
  Bounded reversible-profile metadata, SHA-256 integrity, hostile-input
  inspection, and byte-exact source recovery. Keep this limited to RayoMD's
  exact classic-xref profile rather than growing a general PDF parser.

- `src/core/inline_markdown.cpp`
  Renderer-neutral inline span parsing for emphasis, code, images, and links. Both renderers
  consume this model so visible text and link annotations cannot drift.

- `src/core/export_options.cpp` and `src/common/text_utils.cpp`
  Shared typed style/margin conversion, CLI option parsing primitives, and non-public text helpers.

- `src/cli/main_cli.cpp`
  Portable CLI entry point for Linux and non-GUI workflows. Supports single
  export, stdin Markdown export, folder batch, stdin batch, warm serve mode,
  and benchmarks.

- `src/win32/main_win32.cpp`
  Windows Dear ImGui + DirectX 11 app, Windows CLI glue, drag/drop, Pandoc mode,
  and native export integration.

- `src/win32/rayomd.rc`
  Windows manifest and app icon resources. Keep resource changes localized here.

- `CMakeLists.txt`
  Cross-platform build. Windows and non-Windows builds produce `rayomd`
  (`rayomd.exe` on Windows).

- `VERSION`
  Single source for the release version. CMake compiles it into both command-line
  entry points; update it only for release/versioning work.

- `CONTRIBUTING.md`
  Short contributor guide with project priorities, build/verify commands,
  performance expectations, and versioning rules.

- `third_party/imgui/`
  Vendored Dear ImGui, currently v1.92.8. Do not edit vendored ImGui files

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Butterski/rayomd](https://github.com/Butterski/rayomd) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-06 -->
