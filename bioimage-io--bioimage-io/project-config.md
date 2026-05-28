---
trigger: always_on
description: This skill enables any AI agent (Claude Code, Gemini CLI, etc.) to guide a researcher through the full model contribution pipeline — no BioImage.IO expertise required:
---

# BioImage.IO — Claude Code Guidelines

## Project Goal

**BioImage Model Zoo** (<https://bioimage.io>) is a community-driven, fully open platform for sharing, discovering, testing, and deploying deep learning models for bioimage analysis. The platform makes models truly **FAIR** — Findable, Accessible, Interoperable, and Reproducible — across frameworks, operating systems, and software ecosystems.

### Core Mission

1. **Model repository** — Host pre-trained DL models (segmentation, restoration, classification, etc.) with standardized metadata (RDF/YAML), DOIs, memorable nicknames, and full provenance (training data, notebooks, authors).
2. **Cross-tool interoperability** — A single model format (bioimageio spec) runs across ilastik, deepImageJ, QuPath, StarDist, ImJoy, ZeroCostDL4Mic, CSBDeep, Icy, and more — without per-tool re-integration.
3. **In-browser testing via BioEngine** — Users can evaluate any model on their own images directly on the website; the BioEngine serves GPU inference from cloud infrastructure (de.NBI / Kubernetes + Triton).
4. **Community contribution pipeline** — Model submission through Zenodo or the web upload form; automatic CI quality assurance; manual curator review; community partner collections via GitHub.
5. **FAIR developer tooling** — Python (`bioimageio.core`) and Java libraries let developers programmatically load, run, export, and re-upload models in only a few lines of code.

### Active Feature: Collaborative Annotation & Fine-Tuning

The current in-browser application layer extends the platform with:

- **Collaborative annotation** — mount local folders or upload images to cloud, share annotation sessions with teammates via URL, collect segmentation masks in real time.
- **AI-assisted annotation** — Cellpose / Cellpose-SAM auto-segmentation with interactive correction tools.
- **Fine-tune Cellpose-SAM** — trigger training from the UI using annotated data stored in Hypha Artifacts, then deploy trained models back into the annotation workflow.

This layer runs **entirely in the browser** (Python via Pyodide/WebAssembly, image visualization via OpenLayers) backed by **Hypha Cloud** for service registration, artifact storage, and RPC coordination.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend framework | React 18 + TypeScript |
| Package manager | pnpm |
| Styling | Tailwind CSS + Material-UI (MUI) |
| State management | Zustand |
| Image visualization | OpenLayers |
| In-browser Python | web-python-kernel (Pyodide/WebAssembly) |
| Backend / services | Hypha RPC (`hypha-rpc` npm package) |
| Artifact storage | Hypha Artifact Manager |
| AI segmentation | Cellpose / Cellpose-SAM (via Hypha service) |

---

## What Has Already Been Built

### `src/components/colab/` — Collaborative Session Management

- **ColabPage.tsx** — Main orchestrator: file system state, session lifecycle, 3-step workflow (Start → Collaborate → Train).
- **KernelContext.tsx** — React Context that shares a single web-python-kernel instance across routes; exposes `executeCode`, `mountDirectory`, `syncFileSystem`, `writeFilesToPyodide`.
- **useColabKernel.ts** — Hook that initializes kernel manager (`KernelMode.WORKER` + `KernelLanguage.PYTHON`), handles streaming output and errors.
- **SessionModal.tsx** — Creates sessions: mounts local folder (File System Access API), uploads to cloud, or resumes an existing artifact. Installs Python packages and loads `colab_service.py`.
- **ShareModal.tsx** — Displays annotation URL + QR code, Cellpose model selector, session resume URL.
- **ImageViewer.tsx** — Session dashboard: image list with annotation status, colorized mask preview, progress tracker, ZIP download, cloud upload/delete.
- **TrainingModal.tsx** — Triggers Cellpose-SAM fine-tuning via `cellpose-finetuning` Hypha service; browses existing trained models.
- **DeleteArtifactModal.tsx** — Safe artifact deletion with stats and ID confirmation.
- **ColabGuide.tsx** — In-app help tutorial.

### `src/components/annotate/` — Interactive Annotation Interface

- **AnnotationViewer.tsx** — OpenLayers map with pixel-space coordinate system; exposes refs for reset view, vector source, image layer.
- **ToolBar.tsx** — 6 tools (Move M, Select S, Draw D, Cut C, Eraser E, Expand A) + AI (Cellpose), Undo, Clear, CLAHE, Filter by Area, Upload GeoJSON, Save, Help.
- **LabelPanel.tsx** — Color-coded label selector (Cell, Nucleus, Background defaults); backed by `useAnnotationStore`.
- **useHyphaService.ts** — Connects to image provider service; wraps `getImage`, `getSaveUrls`, `saveAnnotation`, `runCellpose`; converts masks ↔ polygons.
- **useAnnotationMap.ts** — Initializes OpenLayers map, image layer, vector layer with styled features; exports GeoJSON.
- **useDrawInteraction.ts** — Draw/modify/snap interactions for polygon editing.
- **CellposeConfigDialog.tsx** — Model selection, diameter, thresholds, iteration parameters.
- **SegmentationDialog.tsx** — Review and accept/modify Cellpose output masks.
- **FloatingBanners.tsx** — Toast notifications (info/loading/success/warning/error) with auto-dismiss.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bioimage-io/bioimage.io](https://github.com/bioimage-io/bioimage.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
