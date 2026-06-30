---
trigger: always_on
description: YOLO-Label is a GUI image-labeling tool for creating YOLO object detection training datasets. Built with **C++17 and Qt 6**, it runs on Windows, Linux, and macOS.
---

# Project Overview

YOLO-Label is a GUI image-labeling tool for creating YOLO object detection training datasets. Built with **C++17 and Qt 6**, it runs on Windows, Linux, and macOS.

Key design: uses a **two-click method** (not drag-and-drop) to define bounding boxes, reducing wrist strain during long labeling sessions. Keyboard shortcuts (A/D for prev/next image, W/S for prev/next class) enable rapid workflow.

## Architecture

- **MainWindow** (`mainwindow.cpp/h`): Application logic, UI controls, file I/O, keyboard shortcuts
- **label_img** (`label_img.cpp/h`): Custom QLabel widget for image display, mouse-based bounding box drawing, contrast adjustment
- **YoloDetector** (`yolo_detector.cpp/h`): ONNX Runtime-based YOLO inference engine for pseudo labeling (see below)
- **Build**: qmake (`YoloLabel.pro`), CI via GitHub Actions (`.github/workflows/ci-release.yml`)
- **Output format**: YOLO annotation `.txt` files with normalized bounding box coordinates

## Pseudo Labeling (Auto-Label)

Local YOLO object detection that auto-generates bounding boxes from an Ultralytics `.onnx` model. Loading a single `.onnx` file is sufficient — class names, input size, and model configuration are all read from the ONNX metadata embedded by Ultralytics.

### Design Principle

This feature targets full compatibility with the [Ultralytics](https://github.com/ultralytics/ultralytics) project's ONNX export. When modifying or extending the pseudo labeling feature:

1. **Always check the upstream Ultralytics source** for the latest supported models, ONNX export format, and metadata schema. Key references:
   - Model export: `ultralytics/engine/exporter.py` — how metadata is embedded in ONNX
   - ONNX predict: `ultralytics/nn/autobackend.py` — how metadata is read back and used for inference
   - Postprocessing: `ultralytics/utils/ops.py` (`non_max_suppression`) and `ultralytics/models/yolo/detect/predict.py`
   - Supported models list: https://docs.ultralytics.com/models/
2. **Ultralytics may add new model versions, change ONNX metadata keys, or alter output tensor layouts.** Before making changes, search the Ultralytics repo and docs for the current state rather than relying solely on what is documented here.
3. **The goal is: a user exports any Ultralytics object detection model to `.onnx` and loads it in YOLO-Label — it just works.** No separate class names file, no manual configuration.

### ONNX Metadata

Ultralytics embeds metadata as string key-value pairs in ONNX `metadata_props` during export. The detector reads these via `Ort::Session::GetModelMetadata()` to configure itself automatically. Key fields (check `ultralytics/engine/exporter.py` for the latest schema):

| Key | Example | Usage |
|---|---|---|
| `names` | `"{0: 'person', 1: 'car', ...}"` | Auto-populate class list (Python dict format, parsed in C++) |
| `task` | `"detect"` | Validate model type; reject non-detection models |
| `stride` | `"32"` | Model stride |
| `imgsz` | `"[640, 640]"` | Input resolution for dynamic-shape models |
| `end2end` | `"True"` | Skip NMS when model has NMS baked in |
| `description` | `"Ultralytics YOLOv8n model"` | Detect specific YOLO version |

### Inference Pipeline (`yolo_detector.cpp`)

The C++ inference code references the Ultralytics Python implementation but maximizes use of ONNX metadata to make a single `.onnx` file self-sufficient for detection:

1. **Load**: `Ort::Session` loads the model. Metadata is read via `GetModelMetadata()` → `GetCustomMetadataMapKeysAllocated()` / `LookupCustomMetadataMapAllocated()`.
2. **Version detection**: First checks metadata `description` for specific version, then falls back to output shape heuristic (`dim1 > dim2` → V5-style, else V8-style). End-to-end is a boolean flag (`endToEnd`), not a version. The `YoloVersion` enum and `detectVersionFromMetadata()` may need new entries when Ultralytics releases new model architectures.
3. **Preprocess**: Letterbox resize via `QImage::scaled()`, gray padding (114/255), HWC→CHW, normalize to [0,1].
4. **Postprocess**: Three paths depending on model output tensor layout:
   - **V5-style**: `[B, N, C+5]` — has objectness score, row-major layout
   - **V8-style**: `[B, C+4, N]` — no objectness, transposed (column-major per detection). Used by V8, V11, V12, V26 and likely future versions that share this layout.
   - **End-to-end**: `[1, maxDet, 6]` — `[x1, y1, x2, y2, score, class_id]`, NMS already applied
   - **When adding support for a new model**, check whether its ONNX output layout matches an existing path or requires a new `postprocess*()` function.
5. **NMS**: Pure C++ greedy NMS (class-aware, sorted by confidence). Skipped for end-to-end models.
6. **Output**: `DetectionResult` with normalized [0,1] coordinates matching `ObjectLabelingBox.box` format.

### Conditional Build

ONNX Runtime is optional. When `ONNXRUNTIME_DIR` points to a valid installation, `YoloLabel.pro` defines `ONNXRUNTIME_AVAILABLE` and compiles the detector. Without it, the app builds normally without the auto-label UI. See `scripts/download_onnxruntime.sh` for downloading pre-built ONNX Runtime binaries.

# Project Guidelines

## Git Configuration

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [developer0hye/Yolo_Label](https://github.com/developer0hye/Yolo_Label) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
