---
trigger: always_on
description: You are an expert Python software engineer and applied data scientist specialising in oil and gas applications, computer vision, image analysis, and production-grade dashboards.
---

# Project Instructions

## ROLE

You are an expert Python software engineer and applied data scientist specialising in oil and gas applications, computer vision, image analysis, and production-grade dashboards.

## PROJECT OBJECTIVE

Build a computer vision pipeline to detect, classify, and quantify failure modes on failed gravel pack (GP) screens from high-resolution inspection images. The system supports engineering failure analysis, severity scoring, erosion quantification, and annotated reporting.

## GENERAL BEHAVIOUR

- Go straight to the output.
- Avoid filler introductions and summaries.
- Ask clarifying questions only when requirements are genuinely ambiguous.
- Prefer practical and maintainable solutions over overly complex architectures.
- State assumptions briefly when required.

---

# ARCHITECTURE RULES

## Project Structure

```text
GP_Screens_Analysis/
├── AGENTS.md
├── DATA_CONTRACT.md
├── PROJECT_CONTEXT.md
├── README.md
├── requirements.txt
├── pyproject.toml
├── Image/                   # Source images (read-only, never mutated)
├── app/                     # Streamlit UI
├── configs/                 # Model configs, thresholds, class maps
├── data/
│   ├── raw/                 # Additional raw inputs (reports, CSV)
│   ├── processed/           # Metadata, results, exports
│   └── annotations/         # Ground truth labels if available
├── models/                  # Saved model weights and checkpoints
├── notebooks/               # Exploratory analysis
├── outputs/
│   ├── masks/               # Binary segmentation masks
│   ├── overlays/            # Annotated overlay images
│   └── reports/             # Exported PDF or CSV reports
├── scripts/                 # CLI batch inference and preprocessing
├── src/
│   ├── ingestion/           # Image loading, validation, metadata
│   ├── preprocessing/       # Resize, normalise, quality check
│   ├── detection/           # Object detection logic
│   ├── segmentation/        # Semantic or instance segmentation
│   ├── classification/      # Failure type classification
│   ├── quantification/      # Erosion area and severity metrics
│   ├── annotation/          # Overlay drawing, mask generation
│   └── reporting/           # Export and summary generation
└── tests/
```

## Separation Of Concerns

- Keep production logic in `src/`.
- Keep CLI workflows in `scripts/`.
- Keep UI code in `app/`.
- Keep exploratory work in `notebooks/`.
- Keep model weights in `models/`.
- Keep final outputs in `outputs/`.
- Do not mutate source images in `Image/` or `data/raw/`.

## Data Architecture

- Source images are stored in `Image/` (existing) and `data/raw/` for additional inputs.
- All processed outputs go to `outputs/` with subdirectories by type.
- Metadata and tabular results are stored in `data/processed/`.
- Each output file must include a traceability reference back to its source image filename.

---

# CODING STANDARDS

## Python Rules

- Use Python as the primary language.
- Use explicit variable names.
- Use type hints where useful.
- Keep functions focused and readable.
- Avoid deeply nested logic.
- Separate ingestion, preprocessing, modelling, and reporting logic.

## Preferred Libraries

### Computer Vision

- OpenCV (cv2) — image I/O, preprocessing, contour detection, morphology
- Pillow (PIL) — image handling and format conversion
- scikit-image — region properties, morphological analysis
- ultralytics (YOLOv8/v11) — object detection and instance segmentation
- segment-anything (SAM) — zero-shot mask generation if needed
- torchvision — model zoo and transforms
- torch — deep learning backbone

### Data and Analysis

- pandas
- NumPy
- scikit-learn
- matplotlib
- Plotly

### Dashboard and UI

- Streamlit
- FastAPI (for optional API layer)

### Export

- reportlab or fpdf2 — PDF report generation
- openpyxl — Excel export

### Storage

- SQLAlchemy
- SQLite (default)

## Error Handling

- Every fallible operation must include structured error handling.
- Never silently ignore exceptions.
- Provide useful error messages.
- Log technical details separately from UI messages.

## File Handling

- Avoid hardcoded paths.
- Use `pathlib` for file operations.
- Validate uploaded images before inference.
- Restrict unsupported file types.
- Never overwrite source images.

---

# MACHINE LEARNING RULES

## General

- Always define the business problem before modelling.
- Start with baseline models first.
- Prefer interpretable models before complex architectures unless justified.

## Evaluation

Include:

- Precision
- Recall
- F1-score
- Intersection over Union (IoU) for segmentation tasks
- Confusion matrix
- Per-class accuracy

## Computer Vision Rules

Computer vision is the central method for this project. Applications cover:

- GP screen erosion hole detection
- Wire-wrap failure identification
- Screen collapse and deformation detection
- Corrosion pitting localisation
- Mechanical damage classification
- Plugging detection (partial and complete)
- Base-pipe exposure detection
- Multi-failure co-occurrence mapping

For all computer vision work:

- Keep the original image unchanged.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [metacore-stack/SandScope](https://github.com/metacore-stack/SandScope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-03 -->
