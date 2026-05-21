---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Ableton Device Creator V3.0** is a modern Python library for creating and modifying Ableton Live devices (drum racks, sampler instruments, and Simpler devices) from audio sample libraries.

**Status:** Production-ready, actively maintained
**Version:** 3.0.0 (November 2025)
**Python:** 3.8+
**Dependencies:** Zero (core), Click 8.0+ (CLI optional)

## Architecture

### V3.0 Structure (Current)

```
src/ableton_device_creator/     # Modern Python package
├── core/                        # ADG encoder/decoder
│   ├── decoder.py
│   ├── encoder.py
│   └── __init__.py
├── drum_racks/                  # Drum rack creation
│   ├── creator.py
│   ├── modifier.py
│   ├── sample_utils.py
│   └── __init__.py
├── sampler/                     # Sampler creation
│   ├── creator.py
│   ├── simpler.py
│   └── __init__.py
├── macro_mapping/               # Color, transpose
│   ├── color_mapper.py
│   ├── transpose.py
│   └── __init__.py
├── cli.py                       # Command-line interface
└── __init__.py                  # Package exports
```

### Legacy Code

- `archive-v2-scripts/` - V2 reference scripts (111 scripts, read-only)
- `archive-v1/` - V1 code (preserved, not functional)

## Key Technical Information

### ADG/ADV File Format

- ADG (Ableton Device Group) and ADV (Ableton Device) files are **gzipped XML files**
- Core workflow: Decompress → Modify XML → Recompress
- Compression format: gzip with mtime=0, no filename header

### V3.0 API Usage

#### Python API

```python
from ableton_device_creator.drum_racks import DrumRackCreator
from ableton_device_creator.sampler import SamplerCreator, SimplerCreator
from ableton_device_creator.macro_mapping import DrumPadColorMapper

# Create drum rack
creator = DrumRackCreator(template="templates/input_rack.adg")
rack = creator.from_folder("samples/", output="MyKit.adg")

# Apply colors
colorizer = DrumPadColorMapper("MyKit.adg")
colorizer.apply_colors().save("MyKit_Colored.adg")

# Create chromatic sampler
sampler = SamplerCreator(template="templates/sampler-rack.adg")
sampler.from_folder("samples/", layout="chromatic")
```

#### CLI Usage (requires Click)

```bash
# Create drum rack
adc drum-rack create samples/ -o MyKit.adg

# Apply colors
adc drum-rack color MyKit.adg

# Create sampler
adc sampler create samples/ --layout chromatic

# Show device info
adc util info MyKit.adg
```

## Common Development Tasks

### Running Examples

```bash
# Set PYTHONPATH
export PYTHONPATH=src

# Run examples
python3 examples/drum_rack_example.py
python3 examples/sampler_example.py
python3 examples/cli_demo.py
```

### Testing (Manual)

```bash
# Create test device
python3 examples/test_sampler_simple.py

# Open in Ableton Live and verify:
# - Device loads without errors
# - Samples trigger correctly
# - MIDI mappings are correct
```

### Modifying Core Utilities

**Location:** `src/ableton_device_creator/core/`

**Important:** encoder/decoder must maintain exact Ableton gzip format:
- No timestamp (mtime=0)
- No filename in header
- UTF-8 encoding

**Test after changes:**
```python
from ableton_device_creator.core import decode_adg, encode_adg

# Roundtrip test
xml = decode_adg("templates/input_rack.adg")
encode_adg(xml, "test_output.adg")
xml2 = decode_adg("test_output.adg")
assert xml == xml2  # Must match exactly
```

### Adding New Features

**Follow V3.0 patterns:**
1. Create class in appropriate module (drum_racks, sampler, etc.)
2. Add to module's `__init__.py`
3. Export from main `__init__.py` if public API
4. Add example in `examples/`
5. Add CLI command if appropriate
6. Update README.md and docs/

**Example:**
```python
# src/ableton_device_creator/drum_racks/new_feature.py
class NewFeature:
    def __init__(self, template):
        self.template = Path(template)

    def create(self, input_path, output):
        # Implementation
        pass

# src/ableton_device_creator/drum_racks/__init__.py
from .new_feature import NewFeature
__all__ = [..., "NewFeature"]

# src/ableton_device_creator/__init__.py
from .drum_racks import NewFeature
__all__ = [..., "NewFeature"]
```

## Important Script Behaviors

### Drum Rack Scripts
- Creates 32-pad drum racks (C1 to G3, MIDI notes 36-67)
- Auto-categorizes samples by type (kick, snare, hat, clap, tom, cymbal, perc)
- Multiple layouts: standard, 808, percussion
- Supports velocity layers

### Sampler Scripts
- **Chromatic layout:** Maps 32 samples from C-2 upward (MIDI notes 0-31)
- **Drum layout:** 8 kicks, 8 snares, 8 hats, 8 perc (notes 0-31)
- **Percussion layout:** Maps from C1 upward (note 36+)
- Creates separate instruments for each 32-sample batch

### Simpler Scripts
- Creates one .adv device per sample
- Each Simpler spans full keyboard (notes 0-127)
- Maintains folder structure in batch mode
- Simplest device type for basic sample playback

### Sample Categorization

Detects sample types by filename keywords:
- **Kicks:** "kick", "bd", "bassdrum", "kck"
- **Snares:** "snare", "sd", "snr"
- **Hats:** "hat", "hh", "hihat", "hi-hat"
- **Claps:** "clap", "cp", "handclap"
- **Toms:** "tom", "tm"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ben-juodvalkis/Ableton-Device-Creator](https://github.com/ben-juodvalkis/Ableton-Device-Creator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
