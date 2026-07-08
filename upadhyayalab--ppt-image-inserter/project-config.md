---
trigger: always_on
description: This is a Python toolkit for programmatically inserting images into PowerPoint presentations at scale. The main use case is batch-generating presentation slides from large collections of images (e.g., scientific plots, data visualizations) using a template-based approach.
---

# Codex Instructions for PPT Batch Image Inserter

## Project Overview

This is a Python toolkit for programmatically inserting images into PowerPoint presentations at scale. The main use case is batch-generating presentation slides from large collections of images (e.g., scientific plots, data visualizations) using a template-based approach.

**Core value proposition**: Instead of manually inserting hundreds of images into PowerPoint, users define a YAML config and run a script.

## Platform & Environment

- **Platform**: Cross-platform (Windows, macOS, Linux)
  - **Note**: Primarily tested on Windows. Most users are on Windows.
  - Avoid Unix-specific commands in user-facing code
  - **Use forward slashes in paths** (works on all platforms including Windows)
- **Python**: 3.9+ (recommended)
- **Main dependencies**: `python-pptx`, `PyYAML`
- **Package manager**: conda (recommended) or pip

## Setup & Running

### Installation

```bash
# Clone and navigate
git clone https://github.com/UpadhyayaLab/ppt-image-inserter.git
cd ppt-image-inserter

# Create environment
conda create -n ppt_inserter python=3.9
conda activate ppt_inserter

# Install dependencies
pip install -r requirements.txt
```

### Running the Batch Script

```bash
python examples_and_configs/batch_insert_images.py examples_and_configs/configs/example_config.yaml
```

**CRITICAL**: The file must be closed in PowerPoint before running the script. The script needs exclusive file access to modify the presentation.

## Architecture

### Package Structure: `ppt_image_inserter/`

The package is organized into modular components. All functions can be imported directly from the package:

```python
from ppt_image_inserter import insert_image, copy_slide_replace_image
```

**Core Principles:**
- **General-purpose**: No experiment-specific logic
- **Well-documented**: Clear docstrings for all public functions
- **Backwards-compatible**: Don't break existing APIs without good reason
- **Standalone**: Minimal external dependencies (just python-pptx, PyYAML)

**Module Organization:**

- **`core.py`** - Basic image insertion functions
  - `insert_image()` - Insert image at exact position on existing slide
  - `insert_image_preserve_aspect()` - Insert image with aspect ratio preserved
  - `list_slides()` - Get slide information

- **`position.py`** - Position and unit conversion utilities
  - `get_image_position()` - Extract position from template image (auto-detection)
  - `cm_to_inches()` - Unit conversion utility

- **`slide_utils.py`** - Slide manipulation utilities
  - `duplicate_slide()` - Copy a slide
  - `remove_pictures_from_slide()` - Remove all pictures from slide
  - `remove_all_text_from_slide()` - Remove all text elements
  - `delete_slide()` - Delete slide with automatic backup

- **`workflow.py`** - High-level workflow functions
  - `copy_slide_replace_image()` - Main workflow for batch processing (duplicate template, replace image)
  - `replace_image_on_existing_slide()` - Update existing slide with new image

- **`backup.py`** - Backup system
  - `backup_presentation()` - Create timestamped backups with multiple time intervals

- **`metadata.py`** - Metadata extraction
  - `extract_image_metadata()` - Extract image source info from all slides

### User-Facing Components

1. **Config files (YAML)**: Define presentation path, template slide, image list
2. **Scripts**: User-written Python scripts that use the core module
3. **Examples**: Template configs and scripts in `examples_and_configs/`

**Config location rule**: Reusable YAML configs should live inside this
repo under `examples_and_configs/configs/`, grouped by project/domain
(for example, `examples_and_configs/configs/live/nuc_rotation/`). Do not
leave generated or reusable configs beside output `.pptx` files in
`K:/FF/PPT/PPT_autogeneration/...`; those output folders should contain
decks, backups, and other generated presentation artifacts only.

### YAML Configuration Structure

Since you'll frequently help users create and modify configs, here's the structure:

**Minimal working config:**
```yaml
presentation: "path/to/presentation.pptx"
template_slide: 1                    # 0-based index (slide 2 in PowerPoint UI)
preserve_slides: [0, 1]              # Keep title slide (0) and template slide (1)
base_dir: "path/to/images"
images:
  - image1.png                       # Single image = one slide
  - [image2.png, image3.png]         # Two images = one slide (multi-image feature)
  - image4.png                       # Single image = one slide
```

**All available fields:**
```yaml
# Required fields
presentation: "presentations/my_presentation.pptx"
template_slide: 1
base_dir: "data/images"
images:
  - plot1.png
  - plot2.png

# Optional fields
preserve_slides: [0, 1]              # Default: [0, template_slide]
backup_dir: "custom/backup/location" # Default: backups/ folder next to the presentation file
output_path: "output/new.pptx"       # Template preservation mode (see below)
auto_position: true                   # Default: true (auto-detect from template image)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [UpadhyayaLab/ppt-image-inserter](https://github.com/UpadhyayaLab/ppt-image-inserter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
