---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ABBA (Aligning Big Brains & Atlases) is an ImageJ/Fiji plugin for aligning 2D brain slice images with 3D brain atlases (Allen Brain Atlas). The core Java package provides registration, visualization, and state management capabilities.

**Associated Repositories:**
- Documentation: https://github.com/BIOP/abba-documentation
- QuPath extension: https://github.com/BIOP/qupath-extension-abba
- Python wrapper: https://github.com/BIOP/abba_python
- Windows installer: https://github.com/BIOP/abba-installer

## Build and Test Commands

### Building
```bash
# Standard Maven build
mvn clean install

# Build without tests
mvn clean install -DskipTests

# Build for a specific Fiji installation (uncomment scijava.app.directory in pom.xml)
mvn clean install -Dscijava.app.directory=C:/Fiji
```

### Testing
```bash
# Run all tests
mvn test

# Run a specific test class
mvn test -Dtest=ABBALaunchMouse

# Run benchmark (requires OMERO setup)
mvn test -Dtest=ABBABenchMarkCommand
```

The `src/test/java` directory contains demo launchers and plugin examples rather than traditional unit tests. Key test files:
- `ABBALaunchMouse.java`, `ABBALaunchRat.java` - Launch ABBA with demo atlases
- `pluginexample/` - Examples for extending ABBA with custom registration methods
- `ScijavaCommandToPython.java` - Generates Python API bindings

### CI Build
The project uses SciJava's CI scripts:
- `.github/build.sh` downloads and executes `ci-build.sh` from scijava-scripts
- CI builds on Java 8 (Zulu distribution)
- Deploys to SciJava Maven repository on tagged releases

## Architecture Overview

### Core Architectural Pattern

ABBA uses a three-layer architecture with an action-based state management system:

1. **Model Layer** - Core domain logic
   - `MultiSlicePositioner` - Central orchestrator managing the entire registration workflow
   - `SliceSources` - Represents individual slice images with transformation states
   - `ReslicedAtlas` - Manages dynamic 3D atlas reslicing based on user orientation
   - `CancelableAction` - Base class for all state-changing operations (undo/redo support)

2. **Command/View Layer** - User interface
   - 64 SciJava Command plugins exposing functionality
   - BigDataViewer (BDV) integration for 3D visualization
   - Card-based UI panels (AtlasInfoPanel, EditPanel, NavigationPanel)

3. **Adapter/Serialization Layer** - Persistence
   - Gson-based JSON serialization with custom type adapters
   - `AlignerState` captures complete state snapshots
   - Legacy format compatibility handling

### Key Packages

```
ch.epfl.biop.atlas.aligner/
├── [root]          - Core model classes (MultiSlicePositioner, SliceSources, ReslicedAtlas, *Action)
├── adapter/        - GSON JSON serialization adapters (AlignerState, RegisterSliceAdapter, etc.)
├── action/         - Concrete action implementations (export, raster, key frames)
├── command/        - SciJava commands (64 total) exposing functionality to UI
├── gui/            - UI components and interaction handlers
│   └── bdv/        - BigDataViewer window management, behaviors, overlays
│       └── card/   - UI panels (navigation, editing, info)
├── processor/      - SciJava module processing hooks
└── plugin/         - Extension points for third-party commands

ch.epfl.biop.quicknii/ - QuickNII format import/export
ch.epfl.biop/          - Utility classes (ResourcesMonitor)
```

### Registration Workflow

The system supports multiple registration backends through a plugin architecture:

1. **Elastix-based** - Affine and B-spline deformable registration
2. **DeepSlice** - Deep learning-based (local or web service)
3. **BigWarp** - Manual/interactive landmark-based
4. **QuickNII** - Import pre-existing registrations

Registration flow:
```
User selects slices → RegisterSlices*Command.run()
  → MultiSlicePositioner.registerSelectedSlices()
  → RegisterSliceAction (one per slice)
  → Preprocessing pipeline (channel selection, resampling, filtering)
  → Registration plugin execution
  → Transform stored in SliceSources
  → BDV updates visualization
```

Each `RegisterSliceAction` contains:
- Reference to registration plugin (e.g., Elastix2DAffineRegistration)
- Preprocessing pipeline (SourcesProcessor chain)
- Registration result (AffineTransform3D or deformation field)
- State tracking (pending, running, done, locked, invalid)

### State Persistence

State is saved as ZIP archives containing:
- `metadata.json` - AlignerState with atlas orientation, slice list, action history
- Source references and preprocessing pipeline configurations

**Action filtering during serialization:**
- **Saved**: CreateSlice, MoveSlice, RegisterSlice, KeySliceOn/Off, UnMirror, SetBackground
- **Omitted**: Export actions, temporary edits, failed registrations

Custom GSON adapters handle serialization:
- `RegisterSliceAdapter` - Serializes registration transforms only, not preprocessing artifacts
- `CreateSliceAdapter` - Preserves slice source references
- `SliceSourcesStateDeserializer` - Reconstructs action history from JSON
- Legacy class name conversion for backward compatibility


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BIOP/ijp-imagetoatlas](https://github.com/BIOP/ijp-imagetoatlas) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
