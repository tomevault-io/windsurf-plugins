---
trigger: always_on
description: **Magic Georeferencer** is a QGIS plugin that leverages the MatchAnything AI model to automatically georeference ungeoreferenced images (maps, aerial photos, sketches) by matching them against real-world basemap data.
---

# Magic Georeferencer - QGIS Plugin Specification

## Project Overview

**Magic Georeferencer** is a QGIS plugin that leverages the MatchAnything AI model to automatically georeference ungeoreferenced images (maps, aerial photos, sketches) by matching them against real-world basemap data.

### Core Concept

1. User loads an ungeoreferenced image into QGIS
2. User navigates map canvas to approximate location
3. Plugin captures current map view (OSM or aerial tiles)
4. MatchAnything model finds correspondence points between images
5. Confidence filtering generates high-quality Ground Control Points (GCPs)
6. QGIS georeferencer applies transformation and outputs georeferenced image

### Key Features

- **Zero manual GCP placement** - fully automated matching
- **Cross-modality support** - handles maps→aerial, aerial→maps, sketches→maps, etc.
- **Progressive refinement** - multi-scale matching for accuracy
- **GPU acceleration** with CPU fallback
- **Confidence visualization** - review matches before committing
- **Flexible basemap sources** - OSM standard, aerial imagery, etc.

---

## Technical Architecture

### Component Overview

```
magic_georeferencer/
├── __init__.py                      # QGIS plugin entry point
├── magic_georeferencer.py           # Main plugin class
├── metadata.txt                     # QGIS plugin metadata
├── icon.png                         # Plugin icon
│
├── core/
│   ├── __init__.py
│   ├── model_manager.py             # Weight download, CUDA detection, model loading
│   ├── matcher.py                   # MatchAnything inference wrapper
│   ├── tile_fetcher.py              # Basemap tile capture/fetching
│   ├── gcp_generator.py             # Match → GCP conversion
│   └── georeferencer.py             # QGIS georeferencer integration
│
├── ui/
│   ├── __init__.py
│   ├── main_dialog.py               # Primary UI dialog
│   ├── main_dialog.ui               # Qt Designer UI file
│   ├── progress_dialog.py           # Download/processing progress
│   ├── confidence_viewer.py         # Match confidence visualization
│   └── resources.qrc                # Qt resources
│
├── matchanything/                   # MatchAnything integration
│   ├── __init__.py
│   ├── inference.py                 # Simplified inference wrapper
│   └── requirements.txt             # Model dependencies
│
├── config/
│   ├── tile_sources.json            # Basemap tile configurations
│   └── default_settings.json        # Plugin default settings
│
└── weights/                         # Downloaded on first run
    └── .gitkeep
```

---

## Core Components Specification

### 1. Model Manager (`core/model_manager.py`)

**Responsibilities:**
- Detect CUDA availability
- Download model weights on first run
- Load MatchAnything model into memory
- Manage model inference device (GPU/CPU)

**Key Functions:**

```python
class ModelManager:
    def __init__(self, weights_dir: Path):
        self.weights_dir = weights_dir
        self.device = None
        self.model = None
        
    def is_cuda_available(self) -> bool:
        """Check for CUDA-capable GPU"""
        
    def weights_exist(self) -> bool:
        """Check if model weights are downloaded"""
        
    def download_weights(self, progress_callback: Callable) -> bool:
        """
        Download weights from HuggingFace Hub
        Model: zju-community/matchanything_eloftr
        Uses huggingface_hub library for automatic download and caching
        Returns: True on success, False on failure
        """
        
    def load_model(self) -> bool:
        """
        Load MatchAnything model
        - Auto-detect device (CUDA/CPU)
        - Load appropriate config
        - Return success status
        """
        
    def get_inference_config(self) -> dict:
        """
        Return device-appropriate inference settings
        GPU: {'size': 1024, 'num_keypoints': 2048}
        CPU: {'size': 512, 'num_keypoints': 512}
        """
        
    def unload_model(self):
        """Free memory"""
```

**Download Process (HuggingFace Hub):**
```python
from huggingface_hub import hf_hub_download

def download_weights(self, progress_callback):
    """
    Download model from HuggingFace Hub using transformers/huggingface_hub

    Model Repository: zju-community/matchanything_eloftr

    Approach 1 (Recommended - using transformers):
    from transformers import AutoImageProcessor, AutoModel
    processor = AutoImageProcessor.from_pretrained("zju-community/matchanything_eloftr",
                                                   cache_dir=self.weights_dir)
    model = AutoModel.from_pretrained("zju-community/matchanything_eloftr",
                                      cache_dir=self.weights_dir)

    Approach 2 (Manual with hf_hub_download):
    model_path = hf_hub_download(
        repo_id="zju-community/matchanything_eloftr",
        filename="pytorch_model.bin",
        cache_dir=self.weights_dir,
        resume_download=True
    )

    Benefits:
    - No Google Drive authentication issues
    - Built-in resume support
    - Automatic caching and versioning
    - Progress tracking available

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FungoBungaloid/georefio](https://github.com/FungoBungaloid/georefio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
