---
trigger: always_on
description: **LAMMPS-GUI** is a graphical user interface for LAMMPS (Large-scale Atomic/Molecular Massively Parallel Simulator). It's a standalone Qt-based C++ application that provides:
---

# LAMMPS-GUI Development Guide for Copilot

## Project Overview

**LAMMPS-GUI** is a graphical user interface for LAMMPS (Large-scale Atomic/Molecular Massively Parallel Simulator). It's a standalone Qt-based C++ application that provides:
- A syntax-highlighting code editor for LAMMPS input files
- Direct execution of LAMMPS simulations
- Real-time visualization and monitoring of simulation output
- Chart plotting capabilities
- Image/slideshow viewing of simulation snapshots

**Repository**: https://github.com/akohlmey/lammps-gui
**Documentation**: https://lammps-gui.lammps.org/
**Version**: 2.0.4 (see CMakeLists.txt line 4)
**License**: GNU GPL v2

### Key Statistics
- **Language**: C++ (C++17 standard)
- **Framework**: Qt (requires Qt 6.2+; Qt 6.10+ for QtGraphs support)
- **Build System**: CMake 3.20+
- **Codebase Size**: ~17.8k lines across 50 source files in `src/`
- **Total Files**: ~11,290 files (includes docs, resources)
- **Disk Size**: ~263 MB

## Architecture & File Layout

### Root Directory Structure
```
├── CMakeLists.txt          # Main build configuration (480 lines)
├── README.md               # Brief overview with CI badges
├── TODO.md                 # Feature roadmap
├── LICENSE                 # GPL v2 license
├── .clang-format           # C++ code formatting rules (LLVM-based, 100 char limit)
├── .gitignore              # Build artifacts, temp files
├── src/                    # C++ source files (50 files)
├── doc/                    # Sphinx documentation (RST format)
├── resources/              # Icons, QRC files, help text
├── plugin/                 # Plugin loader for dynamic LAMMPS library loading
├── packaging/              # Platform-specific packaging scripts
└── .github/                # CI workflows, issue templates
```

### Source Code Organization (`src/`)
**Main application files**:
- `main.cpp` - Application entry point, command-line parsing
- `lammpsgui.{cpp,h}` - Main window and central GUI logic (UI created programmatically in `setupUi()`)
- `lammpswrapper.{cpp,h}` - C++ interface to LAMMPS C library

**Editor components**:
- `codeeditor.{cpp,h}` - Custom editor widget with line numbers
- `highlighter.{cpp,h}` - LAMMPS syntax highlighting
- `linenumberarea.h` - Line number display
- `findandreplace.{cpp,h}` - Search/replace dialog

**Visualization components**:
- `imageviewer.{cpp,h}` - Image display with zoom/pan
- `chartviewer.{cpp,h}` - Chart display with pluggable backend (QtCharts or QtGraphs)
- `chartbackend.h` - Abstract interface for chart backends
- `qtchartsbackend.{cpp,h}` - QtCharts chart backend implementation
- `qtgraphsbackend.{cpp,h}` - QtGraphs chart backend implementation
- `slideshow.{cpp,h}` - Multi-image slideshow
- `rangeslider.{cpp,h}` - Custom slider widget for image sequences

**Dialogs & helpers**:
- `aboutdialog.{cpp,h}` - Custom About dialog with auto-scroll
- `preferences.{cpp,h}` - Settings dialog
- `setvariables.{cpp,h}` - Variable substitution dialog
- `tutorialwizard.{cpp,h}` - Tutorial setup wizard dialog
- `fileviewer.{cpp,h}` - File content viewer
- `logwindow.{cpp,h}` - Log output viewer
- `helpers.{cpp,h}` - Utility functions
- `stdcapture.{cpp,h}` - stdout/stderr capture
- `flagwarnings.{cpp,h}` - LAMMPS flag validation
- `qaddon.{cpp,h}` - Qt helper widgets (QHline, QColorCompleter, QColorValidator, VerticalLabel)
- `constants.h` - Application-wide constants (GuiConstants namespace)

**Networking**:
- `urldownloader.{cpp,h}` - HTTPS file download with SHA-256 checksum verification

**Runner**:
- `lammpsrunner.{cpp,h}` - Thread-based LAMMPS execution

**Note**: All classes are documented in the Architecture section of `doc/introduction.rst` with detailed descriptions organized into:
- Main Window and Application Control (LammpsGui, TutorialWizard)
- Editor Components (CodeEditor, LineNumberArea, Highlighter, FindAndReplace)
- LAMMPS Interface (LammpsWrapper, LammpsRunner)
- Visualization Components (ImageViewer, ChartWindow, ChartViewer, ChartBackend, QtChartsBackend, QtGraphsBackend, SlideShow, RangeSlider)
- Dialog Components (Preferences, SetVariables, FileViewer, LogWindow, AboutDialog)
- Support Components (URLDownloader, StdCapture, FlagWarnings, Qt helper widgets)

### Documentation (`doc/`)
- **Format**: reStructuredText (Sphinx) + Doxygen
- **Build Target**: `html` (creates `build-doc/doc/html/`)
- **Requirements**: `doc/requirements.txt` (Sphinx 6-8.2.3, Breathe, extensions)
- **Doxygen**: `Doxyfile.in` configures API documentation generation
- **Key Files**:
  - `index.rst` - Main documentation index with programmer's guide
  - `api_reference.rst` - Doxygen-generated API documentation
  - `introduction.rst` - Architecture overview of all classes
  - `testing.rst` - Testing infrastructure and test case documentation
  - `installation.rst`, `basic_usage.rst` - User-facing documentation

### Resources (`resources/`)
- `lammpsgui.qrc` - Qt resource collection file
- `icons/` - 80+ PNG icons, ICO/ICNS files
- `lammps_internal_commands.txt` - Command reference for auto-completion
- `help_index.table` - Help system index

### Packaging Scripts (`packaging/`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [akohlmey/lammps-gui](https://github.com/akohlmey/lammps-gui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
