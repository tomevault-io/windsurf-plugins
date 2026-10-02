---
trigger: always_on
description: ExyokiOffice is a C++20 shared library for creating, opening, editing, and
---

# ExyokiOffice repository guide

## Project in one paragraph

ExyokiOffice is a C++20 shared library for creating, opening, editing, and
saving Microsoft Office Open XML packages (`.docx`, `.xlsx`, and `.pptx`). It
writes ZIP/XML packages directly; Microsoft Office and .NET are not build or
runtime dependencies. Hand-written high-level editors cover Word, Excel, and
PowerPoint, while generated typed OpenXML classes and a package layer provide
lower-level access. The repository also builds the `exyoki` command-line tool
and three MCP servers that expose the editors to AI agents.

## Orient yourself before reading code

Read the smallest relevant source of truth first:

1. `README.md` — project overview, three-format quickstarts, build and install.
2. `docs/README.md` — complete documentation index.
3. `docs/Compatibility.md` — supported formats, versions, and feature depth.
4. The task map below — primary API, guide, and runnable example.
5. The relevant public header under `include/ExyokiOffice/`.

Do not begin by enumerating `include/ExyokiOffice/DOM`, `sources/DOM`, or
`data`; these generated/imported trees are very large and usually not the
right implementation layer.

## Task map: goal to API to documentation to example

| Goal | Primary API or tool | Documentation | Runnable example |
| --- | --- | --- | --- |
| Create or edit Word documents | `Word::WordDocumentEditor` | `docs/Word.md`, `docs/word/` | `examples/ExampleWordEditor/main.cpp` |
| Modify an existing Word document | `WordDocumentEditor::Open`, body cursors | `docs/word/documents.md`, `docs/word/text.md` | `examples/ExampleWordEdit/main.cpp` |
| Create or edit Excel workbooks | `Excel::ExcelDocumentEditor` | `docs/Excel.md`, `docs/excel/` | `examples/ExampleExcelEditor/main.cpp` |
| Create or edit PowerPoint presentations | `PowerPoint::PowerPointDocumentEditor` | `docs/PowerPoint.md`, `docs/powerpoint/` | `examples/ExamplePowerPointEditor/main.cpp` |
| Work directly with typed OpenXML elements | `OpenXmlElement`, generated DOM types | `docs/introduction.md` | `examples/ExampleWord/main.cpp` |
| Open/save packages and manipulate parts or relationships | `OpenXmlPackage`, `OpenXmlPackagePart`, `Packaging::*Document` | `docs/introduction.md` | `examples/ExampleWord/main.cpp` |
| Inspect, validate, convert, diff, or query packages in C++ | `ExyokiOffice::Tools`, `ExyokiOffice::Xml` | `docs/tools/exyoki.md`, `docs/tools/conversion-formats.md` | `tools/exyoki/main.cpp` |
| Do the same from a shell | `exyoki` | `docs/tools/exyoki.md` | `tools/exyoki/main.cpp` |
| Expose documents to AI agents over MCP | `exyoki-mcp-word`, `exyoki-mcp-excel`, `exyoki-mcp-power-point` | `docs/tools/mcp-servers.md` | `tools/mcp/word/main.cpp` |
| Handle signatures or linked resources | `ExyokiOffice::Security` | `docs/Signatures.md`, `docs/ExternalResources.md` | focused unit tests under `tests/package/` |
| Design application concurrency | One document graph per worker, or one external mutex per graph | `docs/Threading.md` | locking and cancellation examples in the guide |
| Change generated OpenXML behavior | `gen/` and its metadata readers | `gen/README.md`, `data/README.md` | generator tests under `tests/generator/` |
| Build, test, lint, sanitize, fuzz, or measure coverage | CMake presets and Windows scripts | `README.md`, `docs/ci.md`, `docs/fuzzing.md`, `docs/coverage.md` | `WinBuild.ps1`, `WinLint.ps1`, `WinFuzz.ps1`, `WinCoverage.ps1` |

## Architecture and main API classes

The library has four layers. Prefer the highest layer that models the task:

1. High-level editors under `include/ExyokiOffice/{Word,Excel,PowerPoint}`.
2. Generated typed DOM under `include/ExyokiOffice/DOM`.
3. OPC packaging under `include/ExyokiOffice/Packaging` and the package base
   headers directly under `include/ExyokiOffice`.

Above these sits the tooling and front-end layer: `ExyokiOffice::Tools` and
`ExyokiOffice::Xml` in the shared library, the `exyoki` command line in
`tools/exyoki/`, and the three MCP servers in `tools/mcp/`. A front end never
implements document behavior of its own — it adapts the layers below, so a
missing capability is fixed in the library and surfaced here.

The six main document types are:

- `Word::WordDocumentEditor` in `Word/WordDocument.hpp`: user-facing Word
  authoring through body cursors, paragraphs, runs, tables, images, styles,
  fields, sections, comments, revisions, and related features.
- `Excel::ExcelDocumentEditor` in `Excel/ExcelDocument.hpp`: workbook and
  worksheet editing, cells, ranges, formulas, styles, tables, charts, pivots,
  slicers, validation, and layout.
- `PowerPoint::PowerPointDocumentEditor` in
  `PowerPoint/PowerPointDocument.hpp`: slides, masters, layouts, shapes, text,
  media, tables, charts, transitions, animations, notes, and comments.
- `Packaging::WordDocument`, `Packaging::ExcelDocument`, and
  `Packaging::PowerPointDocument`: lower-level package lifecycle and parts.
  Each is re-exported as an alias in its format namespace.

Editors own a `std::shared_ptr` to their document and expose `GetDocument()` so
lower layers remain reachable. Packages are held in memory and saved explicitly.

Additional public subsystems:

- `ExyokiOffice::Tools`: validation, inspection, archiving, Flat OPC,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JakubMelka/ExyokiOffice](https://github.com/JakubMelka/ExyokiOffice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
