---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

PDF4QT is a C++20 / Qt 6 PDF library plus a family of desktop applications (editor, viewer, page master, diff, launch pad) and a command line tool. Single author project (Jakub Melka), MIT licensed since April 2025.

## Working on GitHub issues

When a GitHub issue is fixed or an enhancement implemented, **the issue must be recorded in [RELEASES.txt](RELEASES.txt)** under the `CURRENT:` section at the top of the file, in the form:

```
 - Issue #NNN: <issue title as it appears on GitHub>
```

Newest issues go first inside `CURRENT:`; the section is renamed to a version line (`V: 1.6.0.0 14.6.2026`) at release time. Commit messages follow the same convention: `Issue #NNN: <issue title>`.

## Build and test

Do not run builds unless the user explicitly asks for a build in the current conversation (see [AGENTS.md](AGENTS.md)).

Configure (vcpkg toolchain is required; Qt 6.9+):

```
cmake -B build -S . -DCMAKE_TOOLCHAIN_FILE=$VCPKG_ROOT/scripts/buildsystems/vcpkg.cmake -DCMAKE_BUILD_TYPE=Debug
cmake --build build -j
```

Useful options: `PDF4QT_BUILD_ONLY_CORE_LIBRARY` (skips every GUI target and the apps), `PDF4QT_BUILD_TESTS` (ON by default), `PDF4QT_QT_ROOT` (needed when installing Qt dependencies), `PDF4QT_INSTALL_TO_USR`. On Linux set `VCPKG_OVERLAY_PORTS` to `vcpkg/overlays/linux:vcpkg/overlays/general` to avoid the incompatible libpng crash.

On this machine there is an existing Ninja + MSVC debug tree at `build/msvc2022_debug` (Qt 6.9.0 at `E:/Programming/Qt/6.9.0/msvc2022_64`). Build it by writing a `.bat` to the scratchpad (nested quotes get mangled otherwise) that calls `vcvars64.bat`, `cd /d` into the build dir and runs `ninja <target>`.

Tests are QtTest executables, one per area, all built into `<build>/usr/bin`:

| Target | File |
| --- | --- |
| `UnitTests` | [tst_lexicalanalyzertest.cpp](UnitTests/tst_lexicalanalyzertest.cpp) |
| `UnitTestsImageOptimizer` | [tst_imageoptimizertest.cpp](UnitTests/tst_imageoptimizertest.cpp) |
| `UnitTestsFontEncoding` | [tst_fontencodingtest.cpp](UnitTests/tst_fontencodingtest.cpp) |
| `UnitTestsAuthorSettings` | [tst_authorsettingstest.cpp](UnitTests/tst_authorsettingstest.cpp) |
| `UnitTestsContentEditor` | [tst_contenteditortest.cpp](UnitTests/tst_contenteditortest.cpp) |
| `UnitTestsMeasure` | [tst_measuretest.cpp](UnitTests/tst_measuretest.cpp) |
| `UnitTestsDimensions` | [tst_dimensionstest.cpp](UnitTests/tst_dimensionstest.cpp) |

Run all of them with `ctest` from the build dir, a single binary directly (`./UnitTestsFontEncoding`), or a single test function with `./UnitTests <testFunctionName>`. The executables need Qt's `bin` on `PATH`; QtTest stdout is swallowed in some shells here, so capture with `-o result.txt,txt` and read the file. A new test needs its own `add_executable` + `add_test` block in [UnitTests/CMakeLists.txt](UnitTests/CMakeLists.txt). Tests touching `QRawFont` or any GUI type must use `QTEST_MAIN` (QGuiApplication), not `QTEST_APPLESS_MAIN`.

## Architecture

### Module layering

Strictly layered; each layer is a shared library that only depends on the ones above it.

- **[Pdf4QtLibCore/](Pdf4QtLibCore/)** — the PDF engine. No Qt Widgets dependency (Core, Gui, Svg, Xml only), so it can be built stand-alone via `PDF4QT_BUILD_ONLY_CORE_LIBRARY`. Parsing, object model, rendering, fonts, color management, encryption, signatures, forms, annotations, optimization, XFA.
- **[Pdf4QtLibWidgets/](Pdf4QtLibWidgets/)** — widget layer: the page draw widget, draw space controller, asynchronous compilers, tool framework, annotation/form widget managers, page content editor tools.
- **[Pdf4QtLibGui/](Pdf4QtLibGui/)** — the application shell shared by Editor and Viewer: main windows, `PDFProgramController`, `PDFActionManager`, settings, sidebar, dialogs, text-to-speech.
- **Applications** — [Pdf4QtEditor/](Pdf4QtEditor/), [Pdf4QtViewer/](Pdf4QtViewer/) (both are thin `main.cpp` shells over Pdf4QtLibGui, Editor with editing features, Viewer read-only), [Pdf4QtPageMaster/](Pdf4QtPageMaster/), [Pdf4QtDiff/](Pdf4QtDiff/), [Pdf4QtLaunchPad/](Pdf4QtLaunchPad/) (launcher for the others), [PdfTool/](PdfTool/) (CLI; one `pdftool*.cpp` per subcommand, all deriving from `PDFToolAbstractApplication` and self-registering).
- **[Pdf4QtEditorPlugins/](Pdf4QtEditorPlugins/)** — runtime-loaded plugins, see below.
- **Dev tools** — [CodeGenerator/](CodeGenerator/) (GUI editor for [generated_code_definition.xml](generated_code_definition.xml), which generates the `PDFDocumentBuilder` API), [JBIG2_Viewer/](JBIG2_Viewer/), [PdfExampleGenerator/](PdfExampleGenerator/).

Everything lives in `namespace pdf` (GUI-layer classes in `namespace pdfviewer`, plugins in `namespace pdfplugin`). Export macros: `PDF4QTLIBCORESHARED_EXPORT`, `PDF4QTLIBWIDGETSSHARED_EXPORT`, `PDF4QTLIBGUILIBSHARED_EXPORT`.

### Document model


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JakubMelka/PDF4QT](https://github.com/JakubMelka/PDF4QT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
