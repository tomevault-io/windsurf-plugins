---
trigger: always_on
description: Provides 8 reusable pattern templates for declarative pattern composition:
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a comprehensive MIDI drum generation system with a modular, plugin-based architecture. It supports multiple genres, styles, drummer imitations, and song structures. The system evolved from a simple single-file metal generator (`generate_metal_drum_track.py`) into a full-featured, extensible platform.

### Key Features
- **Multi-Genre Support**: Metal (7 styles), Rock (7 styles), Jazz (7 styles), Funk (7 styles), expandable to electronic
- **Drummer Imitation**: 7 drummer plugins with authentic styles (Bonham, Porcaro, Weckl, Chambers, Roeder, Dee, Hoglan)
- **Song Structure**: Configurable sections (verse, chorus, bridge, breakdown, intro, outro)
- **Pattern Variations**: Humanization, fills, complexity control, and dynamic variations
- **Multiple Interfaces**: Python API, CLI, and direct module usage
- **Plugin Architecture**: Easily extensible for new genres and drummers
- **Professional Output**: EZDrummer 3 compatible MIDI with realistic velocity and timing

## Development Setup

```bash
# Activate virtual environment
.venv/Scripts/activate  # Windows
source .venv/bin/activate  # Linux/macOS

# Update dependencies from .in files
bin/py_update.bat       # Windows
bin/py_update.sh        # Linux/macOS

# Run linting tools
bin/linting.bat         # Windows
bin/linting.sh          # Linux/macOS

# Quick test - generate using new API
python examples/basic_usage.py

# Generate using CLI interface
python -m midi_drums generate --genre metal --style heavy --tempo 155 --output song.mid

# Run comprehensive architecture tests
python test_new_architecture.py

# Compare old vs new architecture
python migrate_from_original.py

# Run unit tests (when implemented)
pytest
```

## Architecture Overview

The system uses a layered, plugin-based architecture following SOLID principles:

```
┌─────────────────────────────────────────────────────────────────┐
│                        API Layer                                │
│  CLI Interface │ Python API │ Direct Module Usage │ REST (future)│
├─────────────────────────────────────────────────────────────────┤
│                    Application Layer                            │
│  DrumGenerator │ Composition Engine │ Pattern Manager           │
├─────────────────────────────────────────────────────────────────┤
│                    Plugin System                                │
│  Genre Plugins │ Drummer Plugins │ Auto-Discovery │ Registry    │
├─────────────────────────────────────────────────────────────────┤
│                      Core Models                                │
│  Pattern │ Beat │ Song │ Section │ Kit │ GenerationParameters   │
├─────────────────────────────────────────────────────────────────┤
│                 Processing Engines                              │
│  MIDI Engine │ Humanization │ Pattern Builder │ Variations     │
└─────────────────────────────────────────────────────────────────┘
```

### Package Structure
```
midi_drums/
├── __init__.py           # Main exports (DrumGenerator, Pattern, Song)
├── __main__.py           # CLI entry point
├── core/
│   ├── engine.py         # DrumGenerator - main composition engine
│   ├── models/
│   │   ├── pattern.py    # Pattern, Beat
│   │   ├── song.py       # Song, Section, Fill, PatternVariation
│   │   ├── kit.py        # DrumKit configurations (EZDrummer3, Metal, Jazz)
│   │   └── __init__.py
│   ├── value_objects/
│   │   ├── time_signature.py       # TimeSignature
│   │   ├── drum_instrument.py      # DrumInstrument
│   │   ├── generation_parameters.py # GenerationParameters
│   │   └── __init__.py
│   ├── builders/
│   │   ├── pattern_builder.py      # PatternBuilder
│   │   └── __init__.py
│   └── __init__.py
├── export/
│   ├── midi/
│   │   ├── engine.py     # MIDIEngine - MIDI file generation
│   │   ├── exporter.py   # MIDIExporter - high-level MIDI export API
│   │   └── __init__.py
│   ├── reaper/
│   │   ├── engine.py     # ReaperEngine - .RPP file manipulation
│   │   ├── exporter.py   # ReaperExporter - high-level Reaper export API
│   │   ├── models.py     # Marker, ReaperTrack, SectionTemplate, GenreStructurePreset
│   │   └── __init__.py
│   └── __init__.py
├── exporters/
│   └── __init__.py       # Compat shim: re-exports ReaperExporter from export/reaper/
├── plugins/
│   ├── base.py          # GenrePlugin, DrummerPlugin, PluginManager
│   ├── genres/
│   │   ├── metal.py     # MetalGenrePlugin with 7 styles
│   │   ├── rock.py      # RockGenrePlugin with 7 styles
│   │   ├── jazz.py      # JazzGenrePlugin with 7 styles
│   │   └── funk.py      # FunkGenrePlugin with 7 styles
│   ├── drummers/        # 7 drummer style plugins
│   │   ├── bonham.py    # John Bonham - triplets, behind-beat
│   │   ├── porcaro.py   # Jeff Porcaro - shuffle, ghost notes
│   │   ├── weckl.py     # Dave Weckl - linear, fusion
│   │   ├── chambers.py  # Dennis Chambers - funk mastery
│   │   ├── roeder.py    # Jason Roeder - atmospheric sludge
│   │   ├── dee.py       # Mikkey Dee - speed/precision
│   │   └── hoglan.py    # Gene Hoglan - blast beats
│   └── __init__.py
├── api/
│   ├── python_api.py    # DrumGeneratorAPI - high-level interface

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fsecada01/midi-drums](https://github.com/fsecada01/midi-drums) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
