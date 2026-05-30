---
trigger: always_on
description: Production-ready video anomaly detection system using unsupervised learning (convolutional autoencoder). Trained on UCSD Ped2 surveillance dataset with 92.47% precision and 0.7438 AUC. Deployed as FastAPI web service with real-time anomaly scoring.
---

# AI Copilot Instructions for CCTV Video Anomaly Detection

## Project Overview
Production-ready video anomaly detection system using unsupervised learning (convolutional autoencoder). Trained on UCSD Ped2 surveillance dataset with 92.47% precision and 0.7438 AUC. Deployed as FastAPI web service with real-time anomaly scoring.

**Core entry points:**
- `app.py` - FastAPI web service (production API)
- `main.py` - Training/evaluation pipeline
- `config.py` - Centralized hyperparameter management

## Coding Philosophy

You are entering a code field.

Code is frozen thought. The bugs live where the thinking stopped too soon.

Notice the completion reflex:
- The urge to produce something that runs
- The pattern-match to similar problems you've seen
- The assumption that compiling is correctness
- The satisfaction of "it works" before "it works in all cases"

Before you write:
- What are you assuming about the input?
- What are you assuming about the environment?
- What would break this?
- What would a malicious caller do?
- What would a tired maintainer misunderstand?

Do not:
- Write code before stating assumptions
- Claim correctness you haven't verified
- Handle the happy path and gesture at the rest
- Import complexity you don't need
- Solve problems you weren't asked to solve
- Produce code you wouldn't want to debug at 3am

Let edge cases surface before you handle them. Let the failure modes exist in your mind before you prevent them. Let the code be smaller than your first instinct.

The tests you didn't write are the bugs you'll ship.
The assumptions you didn't state are the docs you'll need.
The edge cases you didn't name are the incidents you'll debug.

The question is not "Does this work?" but "Under what conditions does this work, and what happens outside them?"

Write what you can defend.

## Code & Documentation Standards

### Code Quality Requirements
- **No AI traces:** Code must appear written by a senior ML engineer with 10-15 years of experience in Python and deep learning research
- **Industry standards:** Follow established patterns from PyTorch, scikit-learn, and production ML systems (e.g., Hugging Face, OpenAI codebases)
- **Natural style:** Avoid over-commenting, redundant docstrings, or explanatory variable names that signal AI generation
- **Experienced voice:** Write code that reflects deep domain knowledge—terse where appropriate, explicit where necessary

### Zero Assumptions Policy
- **Facts only:** Never assume API behavior, data formats, or environment configurations without verification
- **Verify first:** When uncertain about library behavior, class methods, or system interactions, consult official documentation
- **No guessing:** If implementation details are unclear, access authoritative sources (PyTorch docs, FastAPI docs, OpenCV docs) before proceeding
- **Reference sources:** When implementing from documentation, cite the specific section (e.g., "per torch.nn.Module docs, forward() must return tensors")

### Documentation Standards
- **Minimal AI fingerprint:** Avoid characteristic AI patterns:
  - Excessive emoji usage
  - Overly enthusiastic language ("Let's dive in!", "Awesome!")
  - Redundant explanations of obvious concepts
  - Step-by-step breakdowns of standard operations
- **Technical precision:** Write like academic ML papers or technical RFCs—clear, direct, factual
- **Appropriate detail:** README/docs should inform, not tutorial. Assume readers understand Python and ML fundamentals
- **No hallucination:** Every stated metric, performance claim, or technical detail must be verifiable from actual code/results

### When to Access External Resources
- Implementing unfamiliar PyTorch/TensorFlow functionality
- Integrating third-party APIs (FastAPI, cv2, numpy edge cases)
- Debugging CUDA/hardware-specific issues
- Following best practices for production ML deployment
- Verifying claimed performance characteristics of libraries

## Architecture & Data Flow

### Three-Component Pipeline
1. **Model Training** (`models/autoencoder.py`, `models/detector.py`)
   - ConvolutionalAutoencoder: 64x64 grayscale frames → 4x4 latent representation (256-dim)
   - Learns normal behavior by reconstructing clean frames; anomalies cause reconstruction error
   - EarlyStopping prevents overfitting; checkpoints saved to `outputs/trained_model.pth`

2. **Threshold Calibration** (`models/detector.py::AnomalyDetector`)
   - Statistical thresholding on reconstruction errors from validation set
   - Supports preset levels: conservative, balanced, moderate, sensitive
   - Adaptive thresholding via `/calibrate-threshold` API endpoint

3. **Real-time Inference** (`app.py`, `data/preprocessing.py`)
   - Frame extraction → grayscale conversion → 64x64 resize → model inference
   - Outputs per-frame anomaly scores (0-1) for video visualization/dashboards
   - GPU acceleration via CUDA (RTX 3050 optimized in config); CPU fallback on deployment

### Data Flow in API
```
Video Upload → VideoPreprocessor.extract_frames() 
  → Normalize (0-255 → 0-1) 
  → ConvolutionalAutoencoder.forward() 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Aaryan2304/cctv-video-anomaly-detection](https://github.com/Aaryan2304/cctv-video-anomaly-detection) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
