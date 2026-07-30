---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with the TeraSim-World package.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with the TeraSim-World package.

## Project Overview

TeraSim-World is a bridge package that converts TeraSim traffic simulation data into world model inputs compatible with NVIDIA Cosmos Transfer1 models. This package enables the generation of photorealistic driving scenarios from TeraSim simulations for advanced autonomous vehicle testing and validation.

The package is adapted from [Cosmos-Drive-Dreams toolkits](https://github.com/nv-tlabs/Cosmos-Drive-Dreams/tree/main/cosmos-drive-dreams-toolkits) and extends NVIDIA's original capabilities with TeraSim-specific functionality for seamless integration with TeraSim's simulation outputs.

## Common Development Commands

### Environment Setup
```bash
# Create and activate conda environment
conda env create -f environment.yaml
conda activate cosmos-av-toolkits

# For Waymo Open Dataset support (optional)
pip install waymo-open-dataset-tf-2-11-0==1.6.1
```

### Running Conversions
```bash
# Convert TeraSim data to Cosmos input format
python terasim_to_cosmos_input.py

# Convert TeraSim to RDS-HQ format directly
python convert_terasim_to_rds_hq.py -i <terasim_output_dir> -o <output_wds_path>

# Convert Waymo data to RDS-HQ format
python convert_waymo_to_rds_hq.py -i <waymo_tfrecords_dir> -o <output_dir>/videos -n 16

# Render HD maps and LiDAR from RDS-HQ format
python render_from_rds_hq.py -d waymo -i <input_dir> -o <output_dir> -c pinhole -p True -n 8
```

### Visualization Tools
```bash
# Interactive 3D visualization of RDS-HQ dataset
python visualize_rds_hq.py -i <RDS_HQ_FOLDER> -c <CLIP_ID>

# TeraSim-specific visualization
python terasim_vis.py
```

### Text Embedding Generation
```bash
# Create T5 embeddings for multiview captions
python create_t5_embed_mv.py --text_file ./assets/waymo_multiview_texts.json --data_root <data_root>

# Create T5 embeddings for single view captions
python create_t5_embed.py --caption_file ./assets/waymo_caption.csv --data_root <data_root>
```

## Architecture Overview

### Core Data Flow Pipeline

**TeraSim → RDS-HQ → Cosmos Transfer1**

1. **Input Processing**: TeraSim outputs (FCD, map, monitor files)
2. **Data Conversion**: Transform to RDS-HQ format via WebDataset (WDS)
3. **Rendering**: Generate HD maps, LiDAR visualizations, and camera views
4. **Street View Integration**: Enhance realism with real-world imagery
5. **Output Generation**: Cosmos-compatible inputs for world model training

### Core Components

**Main Entry Point (`terasim_to_cosmos_input.py`)**
- Orchestrates the entire conversion pipeline
- Processes TeraSim FCD (Floating Car Data) and map files
- Extracts vehicle trajectories and collision information
- Converts data to WebDataset (WDS) format
- Renders HD maps and sensor data for world model input

**Data Conversion Layer**
- `convert_terasim_to_rds_hq.py`: Core TeraSim to RDS-HQ converter
  - `TeraSim_Dataset` class: Handles TeraSim XML parsing and iteration
  - Coordinate system conversion (SUMO → Waymo/FLU conventions)
  - Vehicle pose and bounding box extraction
  - HD map feature extraction using SUMO network data
- `convert_waymo_to_rds_hq.py`: Waymo Open Dataset converter for comparison/testing

**Rendering System (`render_from_rds_hq.py`)**
- HD map rendering with depth-based visualization
- LiDAR point cloud simulation
- Multi-camera view generation (f-theta and pinhole models)
- Bounding box overlay and trajectory visualization

**Street View Integration (`street_view_analysis.py`)**
- `StreetViewRetrievalAndAnalysis` class for real-world image enhancement
- Google Street View API integration
- GPT-4 Vision-based environment description generation
- Coordinate transformation from SUMO to geographic coordinates

**Visualization Tools**
- `visualize_rds_hq.py`: Interactive 3D visualization using Viser
- `terasim_vis.py`: TeraSim-specific visualization utilities
- Support for dynamic bounding boxes and trajectory playback

### Camera System Architecture

**Abstract Camera Base (`utils/camera/base.py`)**
- `CameraBase` abstract class defining camera projection interface
- Ray-pixel conversion methods for both PyTorch and NumPy
- 3D point transformation utilities
- Rendering methods for points, lines, and hulls with depth-based coloring

**Camera Implementations**
- `utils/camera/ftheta.py`: F-Theta (fisheye) camera model
- `utils/camera/pinhole.py`: Standard pinhole camera model
- Support for multiple camera viewpoints (front, left, right, rear)

**Camera Configuration**
- Waymo-style multi-camera setup (5 cameras: front, front_left, front_right, side_left, side_right)
- Default RDS-HQ camera configuration for TeraSim data
- Intrinsic parameter handling and pose interpolation

### Data Format Specifications

**Input Formats (TeraSim)**
- **FCD Files** (`fcd_all.xml`): Vehicle trajectory and state information in SUMO format
- **Map Files** (`map.net.xml`): Road network topology from SUMO
- **Monitor Files** (`monitor.json`): Collision and event records with vehicle IDs

**Intermediate Format (RDS-HQ/WebDataset)**
- **Poses**: Camera-to-world and vehicle-to-world transformation matrices
- **Intrinsics**: Camera calibration parameters (pinhole/f-theta)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mcity/TeraSim](https://github.com/mcity/TeraSim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
