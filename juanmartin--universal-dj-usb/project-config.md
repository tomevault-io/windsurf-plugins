---
trigger: always_on
description: <!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
---

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Universal DJ USB Playlist Converter - Copilot Instructions

## Project Overview

This is a clean, modern Python application that converts Rekordbox USB playlists to multiple formats (NML, M3U, M3U8). The project has been restructured to follow Python best practices with minimal dependencies and a clear architecture.

## Current Architecture (Post-Migration August 2025)

**Clean, minimal structure achieved through project restructuring and uv migration:**

- **Models**: All data structures in `models.py` (Track, Playlist, PlaylistTree, ConversionConfig, ConversionResult)
- **Parser**: Single Kaitai Struct-based parser in `parser.py` (removed duplicate parsers)
- **Generators**: Modular format generators in `generators/` directory
  - `base.py`: Abstract base generator class
  - `nml.py`: Traktor NML format generator
  - `m3u.py`: Basic M3U playlist generator
  - `m3u8.py`: Extended M3U8 playlist generator
- **CLI**: Rich-enhanced command-line interface in `cli.py`
- **Kaitai**: Struct definitions in `kaitai/rekordbox_pdb.py`

**Removed components:** GUI, converter orchestration class, utils, multiple redundant parsers

## Key Technologies

- **Python 3.9+**: Main programming language (updated constraint for PySide6 compatibility)
- **uv**: Modern Python package manager and build system (migrated from Poetry August 2025)
- **PySide6**: Modern Qt6-based GUI framework for cross-platform desktop application
- **Kaitai Struct**: Direct parsing of Rekordbox PDB files (removed rekordcrate dependency)
- **Click**: CLI framework with rich integration
- **Rich**: Beautiful terminal output with progress bars, tables, and colors
- **lxml**: XML processing for NML generation
- **pathlib**: Cross-platform path handling
- **psutil**: System and process utilities for USB drive detection

**Removed dependencies:** Tkinter (GUI), rekordcrate (replaced with direct Kaitai), toml, pydantic, pathlib2 (now built-in), Poetry (replaced with uv)

## Development Guidelines

### Code Style

- Follow PEP 8 style guidelines
- Use type hints for all function parameters and return values
- Use dataclasses with field defaults for data structures
- Prefer pathlib.Path over string paths
- Use logging instead of print statements
- Modular generator pattern for output formats

### Project Structure

```
src/universal_dj_usb/
├── __init__.py          # Package initialization with version info
├── models.py           # All data models (Track, Playlist, etc.)
├── parser.py           # Single Kaitai-based PDB parser
├── cli.py              # Rich-enhanced CLI interface
├── generators/         # Modular format generators
│   ├── __init__.py
│   ├── base.py         # Abstract base generator
│   ├── nml.py          # Traktor NML format
│   ├── m3u.py          # Basic M3U format
│   └── m3u8.py         # Extended M3U8 format
└── kaitai/
    ├── __init__.py
    └── rekordbox_pdb.py # Kaitai struct definition
```

### Cross-Platform Considerations

- Always use pathlib.Path for file paths
- Normalize file paths with forward slashes for NML output
- Support relative paths for USB drive portability
- Handle Windows drive letters, macOS /Volumes, and Linux mount points

### Error Handling

- Use try/catch blocks for file operations
- Provide meaningful error messages to users
- Log errors with appropriate severity levels
- Return ConversionResult objects with success/failure status

### Testing

- Write unit tests for all core functionality
- Mock external dependencies (rekordcrate CLI)
- Use temporary directories for file operations
- Test cross-platform path handling

### Build and Dependency Management

- **uv only**: No Makefile, shell scripts, or setup.py needed (migrated from Poetry August 2025)
- **pyproject.toml**: Single configuration file for everything
- **Minimal dependencies**: Only essential packages included
- **Dev dependencies**: Separated development tools (pytest, black, flake8, mypy)

## External Dependencies

### Kaitai Struct Parser

The application uses Kaitai Struct for direct PDB parsing:

- **Direct parsing**: No external CLI dependencies
- **Kaitai definition**: `kaitai/rekordbox_pdb.py` (generated from .ksy files)
- **Performance**: Fast, in-memory parsing
- **Reliability**: No subprocess calls or external tools needed

### Git Submodule

- **crate-digger**: Kaitai struct definitions for Rekordbox formats
- **Path**: `external/crate-digger` (if using submodule approach)
- **Purpose**: Source of truth for Kaitai .ksy files

### Rekordbox File Structure

```
USB Drive/
├── PIONEER/
│   ├── rekordbox/
│   │   └── export.pdb          # Main database file
│   └── USBANLZ/                # Analysis files (cue points, etc.)
└── Contents/                      # Your music files
    ├── <Artist>
    │   ├── <Album>
    │   │   ├── track1.mp3
    │   │   ├── track2.mp3
    │   │   └── ...
    │   └── ...
    └── ...
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [juanmartin/universal-dj-usb](https://github.com/juanmartin/universal-dj-usb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
