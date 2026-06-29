---
trigger: always_on
description: DocTo is a Windows command-line utility written in **Delphi (Object Pascal)** that
---

# DocTo — Agent Guide

## Project Overview

DocTo is a Windows command-line utility written in **Delphi (Object Pascal)** that 
converts Microsoft Office documents (Word `.doc`/`.docx`, Excel `.xls`/`.xlsx`, 
PowerPoint `.ppt`/`.pptx`) to other formats (PDF, CSV, TXT, RTF, etc.) via COM 
Automation. Microsoft Word, Excel, or PowerPoint must be installed on the host machine.


- Repository: https://github.com/tobya/DocTo
- Website: https://tobya.github.io/DocTo/
## Tech Stack

| Layer | Technology |
|-------|-----------|
| Language | Delphi (tested with 10.3; compatible with XE4+) |
| Office integration | Windows COM / Office Interop (Word, Excel, PowerPoint, Visio) |
| Build system | Delphi IDE / `.dproj` project file |
| Tests | Batch scripts (`.bat`) in `/test/` | PHP Pest Tests in `/companion`
| Docs / companion / Test site | Markdown + PHP (`/pages/`, `/companion/`) |

## Repository Layout

```
docTo/
├── src/                    # All Delphi source files (.pas, .dpr, .dproj)
│   ├── docto.dpr           # Project file (entry point)
│   ├── MainUtils.pas       # Core TDocumentConverter class and shared utilities
│   ├── baseConfig.pas      # Abstract TParamLoader base class for CLI params
│   ├── configInput.pas     # -F / --inputfile parameter handler
│   ├── configOutput.pas    # -OX / --outputextension parameter handler
│   ├── WordUtils.pas       # Word COM interop helpers
│   ├── ExcelUtils.pas      # Excel COM interop helpers
│   ├── PowerPointUtils.pas # PowerPoint COM interop helpers
│   ├── PathUtils.pas       # Path/directory utilities
│   ├── ResourceUtils.pas   # String resource helpers
│   ├── datamodSSL.*        # Data module for SSL/webhook support
│   ├── shared/             # Shared/common units
│   ├── Exceptions/         # Custom exception types
│   └── res/                # Resource files
├── test/                   # Manual test scripts and fixture files
│   ├── testDocTo.bat       # Main test runner batch script
│   ├── InputFiles/         # Sample Word/RTF/CSV/XLS input files
│   ├── inputfilesxl/       # Excel-specific input fixtures
│   ├── inputfilespp/       # PowerPoint-specific input fixtures
│   ├── GeneratedFiles/     # Output directory for test conversions
│   └── GeneratedTestputFiles/
├── .github/
│   ├── workflows/          # GitHub Actions (greetings bot)
│   └── ISSUE_TEMPLATE/
├── pages/                  # GitHub Pages / documentation content
├── companion/              # Companion tooling
├── exe/                    # Pre-built binaries
├── readme.md
└── changes.md
```

## Architecture

### Core Pattern: TParamLoader

CLI parameters follow a **registration/dispatch** pattern:

- `TParamLoader` (`baseConfig.pas`) — abstract base class with three responsibilities:
  - `RegisterParams(List)` — adds the parameter key(s) it handles (e.g. `-F`, `--INPUTFILE`) to a lookup list
  - `Load(Converter, Param, Value)` — applies the parsed value to the `TDocumentConverter` instance
  - `ShouldDec` — whether parsing should decrement the argument index after processing
- Each parameter has a dedicated subclass (e.g. `TParamInput`, `TParamOutputExtension`)
- `TDocumentConverter` (`MainUtils.pas`) is the central domain object passed through all param loaders

### Converters

Three COM-based converter paths:
- **Word** (default): use `-WD` flag or omit; format constants from `wdSaveFormat`
- **Excel**: use `-XL` flag; format constants from `xlFileFormat`
- **PowerPoint**: use `-PP` flag

### Logging

Log levels are integers: `1` ERRORS, `2` STANDARD (default), `5` CHATTY, `9` DEBUG, `10` VERBOSE.
Use `Converter.logdebug(msg, LEVEL)` for diagnostic output.

## Building

- **Compiler**: Embarcadero Delphi (tested with 10.3+; XE4 and XE7 also supported)
- **Platform**: Windows only — relies on COM, Word/Excel/PowerPoint interop
- Open `src/docto.dproj` in the Delphi IDE and build, or use the Delphi command-line compiler (`dcc32`)
- Output is a single `docto.exe` binary

No external package manager or build script is present. The project has no Linux/macOS build path.

## Code Structure

- Ensure that If blocks always have a begin end section for all branches even if not strictly neccessary.

## Testing

Tests are `.bat` scripts in `/test/`. They call the compiled `docto.exe` and verify output files are produced. Run them directly from a Windows command prompt with Office installed:

Tests are manual batch scripts in `test/`:

```bat
# Run the main test suite (requires Word/Excel/PowerPoint installed)
.\test\testDocTo.bat
```

- Input fixtures live in `test/InputFiles/`, `test/inputfilesxl/`, `test/inputfilespp/`
- Outputs are written to `test/GeneratedFiles/` and `test/GeneratedTestputFiles/`
- There is no automated unit-test framework; correctness is verified by inspecting generated files

There is no automated test runner — tests must be run manually on a machine with Microsoft Office installed.

Additional Tests are written as Pest Tests in companion Laravel PHP site in the `/companion/` dir

## Key Concepts for Agents

- **Application flags**: `-WD` (Word), `-XL` (Excel), `-PP` (PowerPoint), `-VS` (Visio). Word is the default.
- **Three required parameters**: `-F` (input file/dir), `-O` (output file/dir), `-T` (format type, e.g. `wdFormatPDF`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tobya/DocTo](https://github.com/tobya/DocTo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
