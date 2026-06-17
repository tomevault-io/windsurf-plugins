---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## [Aerotri-Web Docs Index]

IMPORTANT: Prefer retrieval-led reasoning over pre-training-led reasoning. When working on this project, consult the actual code files rather than relying on training data.

|Backend API:app/api:{blocks.py,filesystem.py,georef.py,gpu.py,gs.py,gs_tiles.py,images.py,partitions.py,queue.py,recon_versions.py,reconstruction.py,results.py,system.py,tasks.py,tiles.py,unified_tasks.py}
|Backend Services:app/services:{gltf_gaussian_builder.py,gpu_service.py,gs_runner.py,gs_tiles_runner.py,image_service.py,log_parser.py,openmvs_runner.py,partition_service.py,ply_parser.py,queue_scheduler.py,result_reader.py,sfm_merge_service.py,spz_loader.py,spz_loader_helper.py,system_monitor.py,task_notifier.py,task_runner.py,task_view_service.py,tiles_runner.py,tiles_slicer.py,workspace_service.py}
|Backend Services Notification:app/services/notification:{base.py,dingtalk.py,manager.py,scheduler.py,templates.py}
|Backend Models:app/models:{block.py,database.py,partition.py,recon_version.py}
|Backend WebSocket:app/ws:{progress.py,visualization.py}
|Backend Config:backend/config:{defaults.yaml,image_roots.yaml,image_roots.yaml.example,notification.yaml,notification.yaml.example,settings.yaml,settings.yaml.example}
|Backend Core:app/:{main.py,settings.py,config.py,schemas.py}
|Frontend Views:frontend/src/views:{App.vue,HomeView.vue,BlockDetailView.vue,CompareView.vue,ReconCompareView.vue}
|Frontend Components:frontend/src/components:{BlockCard.vue,BlockStats.vue,BrushCompareViewer.vue,CameraDetailPanel.vue,CameraList.vue,CesiumViewer.vue,DenseComparisonTab.vue,GPUSelector.vue,GaussianSplattingPanel.vue,GaussianSplattingViewer.vue,ImagePreview.vue,InstantSfMRealtimeViewer.vue,ParameterForm.vue,ParameterSummary.vue,PartitionConfigPanel.vue,PartitionList.vue,PartitionSelector.vue,ProgressView.vue,ReconParamsConfig.vue,ReconstructionPanel.vue,ReconstructionViewer.vue,SplitCesiumViewer.vue,SplitModelViewer.vue,StatisticsView.vue,ThreeViewer.vue,TilesConversionPanel.vue}
|Frontend Stores:frontend/src/stores:{blocks.ts,cameraSelection.ts,gpu.ts,queue.ts}
|Frontend API:frontend/src/api:{index.ts}
|Frontend Composables:frontend/src/composables:{useInstantsfmVisualization.ts,useThreeScene.ts,useThreeViewer.ts,useWebSocket.ts}
|Frontend Types:frontend/src/types:{index.ts}
|Frontend Core:frontend/src/:{main.ts,router.ts}
|Tests:backend/tests/:{test_algorithm_integration.py,test_config.py,test_core_paths_integration.py,test_output_paths_integration.py}
|Tests:frontend/tests/:{components/BlockCard.test.ts,stores/blocks.test.ts,setup.ts}

---

## MCP Usage Rules

**Context7 MCP**: Always use Context7 MCP when I need library/API documentation, code generation, setup or configuration steps without me having to explicitly ask. This is especially important for:
- Finding up-to-date API documentation for external libraries (FastAPI, Vue.js, SQLAlchemy, Pinia, etc.)
- Getting code examples for specific library versions
- Configuration and setup instructions
- Best practices and patterns

## Project Overview

**AeroTri** is a photogrammetry and 3D reconstruction platform that integrates multiple computer vision libraries with a web interface for aerotriangulation (SfM - Structure from Motion) workflows.

### Core Components
- **aerotri-web/**: Main web application (FastAPI backend + Vue.js frontend)
- **colmap/**: Incremental SfM library source code
- **glomap/**: Global SfM optimization library
- **openMVG/**: CPU-friendly global SfM alternative
- **openMVS/**: Multi-view stereo dense reconstruction
- **instantsfm/**: Fast global SfM implementation
- **gs_workspace/gaussian-splatting/**: 3D Gaussian Splatting framework
- **visionary/**: WebGPU 3DGS viewer
- **ceres-solver/**: Non-linear optimization library

## Development Commands

### Backend (aerotri-web/backend)
```bash
cd aerotri-web/backend

# Install dependencies
pip install -r requirements.txt

# Run tests
pytest

# Start development server
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run specific test
pytest tests/test_specific.py::test_function
```

### Frontend (aerotri-web/frontend)
```bash
cd aerotri-web/frontend

# Install dependencies
npm install

# Run tests
npm run test

# Start development server
npm run dev -- --host 0.0.0.0 --port 5173

# Build for production
npm run build

# Lint code
npm run lint
```

## Architecture Overview

### Backend Architecture (FastAPI)

**Core Modules:**
- `app/api/`: REST API endpoints organized by resource (blocks, queue, reconstruction, tiles, georef, system, unified_tasks, etc.)
- `app/models/`: SQLAlchemy ORM models (Block, ReconVersion, BlockGS, etc.)
- `app/services/`: Business logic layer
  - `task_runner.py`: Core SfM execution service (COLMAP/GLOMAP/InstantSfM/OpenMVG) with georeferencing support
  - `gpu_service.py`: GPU monitoring via PyNVML (lazy loading to avoid crashes)
  - `gs_runner.py`: 3D Gaussian Splatting training orchestration
  - `tiles_runner.py`: 3D Tiles conversion (OBJ/GLB → 3D Tiles) with version support and georef transform injection
  - `openmvs_runner.py`: OpenMVS dense reconstruction pipeline

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ycsyyds/Aerotri-Web](https://github.com/Ycsyyds/Aerotri-Web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
