---
trigger: always_on
description: **EmotionScope** is an open-source Python toolkit for extracting, probing, and visualizing "functional emotion" vectors from open-weight language models. It replicates and extends Anthropic's April 2026 paper ["Emotion Concepts and their Function in a Large Language Model"](https://transformer-circuits.pub/2026/emotions/index.html) on models anyone can run.
---

# CLAUDE.md — EmotionScope

## Project Identity

**EmotionScope** is an open-source Python toolkit for extracting, probing, and visualizing "functional emotion" vectors from open-weight language models. It replicates and extends Anthropic's April 2026 paper ["Emotion Concepts and their Function in a Large Language Model"](https://transformer-circuits.pub/2026/emotions/index.html) on models anyone can run.

Independent research. Not affiliated with Anthropic or Google.

---

## Quick Orientation

The project has three layers:

1. **`emotion_scope/`** — Python package. Extracts emotion direction vectors from transformer residual streams, probes them in real-time, validates them.
2. **`backend/server.py`** — FastAPI server. Loads a model + vectors, serves chat + emotion probe endpoints.
3. **`frontend/`** — React + Three.js app. Renders animated orbs that visualize emotion state in real-time.

---

## Setup

### Prerequisites
- Python 3.11+ 
- [uv](https://docs.astral.sh/uv/) package manager
- Node.js 18+ (for frontend)
- NVIDIA GPU recommended (8GB+ VRAM for Gemma 2 2B in fp16)

### Detect your hardware
```bash
# GPU
nvidia-smi

# CPU + RAM
# Windows:
systeminfo | findstr /C:"Processor" /C:"Total Physical Memory"
# Linux/Mac:
lscpu && free -h
```

### VRAM requirements by model
| Model | fp16 | 4-bit NF4 |
|-------|------|-----------|
| Gemma 2 2B IT | ~4 GB | ~2 GB |
| Gemma 2 9B IT | ~18 GB | ~5-6 GB |
| Llama 3 8B | ~16 GB | ~5 GB |
| Gemma 2 27B IT | ~54 GB | ~16 GB |

### Install
```bash
git clone https://github.com/AidanZach/EmotionScope.git
cd EmotionScope
uv sync                        # Python dependencies
cd frontend && npm install     # Frontend dependencies
```

---

## Running the Full Stack

### 1. Extract emotion vectors (one-time per model)
```bash
uv run python scripts/extract_all.py \
  --model google/gemma-2-2b-it \
  --sweep-layers
```
Output: `results/vectors/google_gemma-2-2b-it.pt`

### 2. Validate
```bash
uv run python scripts/validate_all.py \
  --vectors results/vectors/google_gemma-2-2b-it.pt
```

### 3. Start backend
```bash
uv run uvicorn backend.server:app --port 8000
```
Or with a different model:
```bash
ES_MODEL=google/gemma-2-9b-it \
ES_VECTORS=results/vectors/google_gemma-2-9b-it.pt \
uv run uvicorn backend.server:app --port 8000
```

### 4. Start frontend
```bash
cd frontend && npm run dev
# Open http://localhost:5173
```

---

## Swapping Models

EmotionScope supports any HuggingFace-compatible transformer. The pipeline is:

```bash
# 1. Extract (always use --sweep-layers for a new model)
uv run python scripts/extract_all.py \
  --model <HF_MODEL_ID> \
  --sweep-layers \
  --use-4bit              # optional, for large models on limited VRAM

# 2. Validate
uv run python scripts/validate_all.py \
  --vectors results/vectors/<model_slug>.pt

# 3. (Optional) Generate model-specific training data
#    For 7B+ models that can generate coherent text:
uv run python scripts/generate_stories.py \
  --model <HF_MODEL_ID> \
  --use-4bit
uv run python scripts/ingest_stories.py   # merge into corpus

# 4. Run server with the new model
ES_MODEL=<HF_MODEL_ID> \
ES_VECTORS=results/vectors/<model_slug>.pt \
uv run uvicorn backend.server:app --port 8000
```

**Supported chat template families:** Gemma, Llama, Mistral, Phi, Qwen, DeepSeek, and a generic fallback. Auto-detected from the tokenizer name.

**Backend selection:** TransformerLens is preferred (faster, cleaner hooks) but only supports ~15 model families. All other models automatically fall back to HuggingFace with manual forward hooks. No code changes needed.

---

## Architecture

```
emotion_scope/
  config.py          20 core emotions, valence/arousal metadata, paths, thresholds
  models.py          load_model() — TransformerLens or HuggingFace backend
  extract.py         EmotionExtractor — contrastive mean difference + PCA denoising
  probe.py           EmotionProbe — real-time cosine similarity scoring
  validate.py        Tylenol intensity, top-3 recall, valence separation, richness
  visualize.py       OKLCH color mapping, scores_to_orb_state()
  speakers.py        Speaker separation (experimental)
  utils.py           Token range detection, cosine matrix, valence separation metric

backend/
  server.py          FastAPI: /health, /chat, /probe endpoints

frontend/src/
  components/
    EmotionOrb.jsx   R3F orb with MeshDistortMaterial, environment map, bloom
    ValenceStrip.jsx  Dual-marker valence gradient
    DetailPanel.jsx   Collapsible per-emotion score bars
    Timeline.jsx      Conversation emotion history
    Legend.jsx         Visual guide explaining each orb property
    ExplainerPanel.jsx "How this works" explainer
    ChatPanel.jsx     Chat input + message display
  utils/
    palette.js        OKLCH emotion palette, emotionToOrbProps()
    oklch.js          OKLCH to RGB conversion, circular hue blending

scripts/
  extract_all.py     CLI: extract vectors for any model

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AidanZach/EmotionScope](https://github.com/AidanZach/EmotionScope) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
