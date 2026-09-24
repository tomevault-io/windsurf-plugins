---
trigger: always_on
description: Research and evaluation harness for offline Quran verse recognition. Everything here is Python: model training (Modal), the experiment framework, the benchmark runner, and the shared inference utilities the experiments share.
---

# Tilawa Lab

Research and evaluation harness for offline Quran verse recognition. Everything here is Python: model training (Modal), the experiment framework, the benchmark runner, and the shared inference utilities the experiments share.

The shipped product is the TypeScript SDK at `../packages/core` (`@tilawa/core`) and the web demo at `../web`. This `lab/` tree is where new models and matching strategies are proven out before their weights/logic graduate into the SDK.

**Everything in this file assumes cwd = `lab/`.** Python modules import each other by top-level name (`from shared.X`, `from benchmark.X`), and benchmark paths like `data/quran.json` are relative to `lab/`. Run everything from inside `lab/`.

## Lab Structure

```
lab/
  shared/       # Common utilities (audio, normalizer, quran_db)
  experiments/  # Each approach gets its own directory with a run.py
  benchmark/    # Runner, test corpus, results
    runner.py   # CLI: python -m benchmark.runner
    test_corpus/  # Audio files + manifest.json (54 samples)
    results/    # Timestamped JSON output
  data/         # quran.json, reference audio, LoRA adapters
  src/offline_tarteel/  # Legacy package (kept for compatibility)
  scripts/      # One-off training/eval scripts (*_modal.py for Modal)
  tests/        # Python tests
  tasks/        # Task queue / autonomy artifacts
  docs/         # Research notes
  EXPERIMENTS.md  # Single source of truth for benchmark numbers + writeups
```

## Experiment Convention

Every experiment lives in `experiments/<name>/` with a `run.py` that exports:

### Required functions

```python
def predict(audio_path: str) -> dict:
    """Run inference on an audio file.

    Returns:
        {"surah": int, "ayah": int, "ayah_end": int|None, "score": float, "transcript": str}

    On failure/no match: surah=0, ayah=0, score=0.0
    transcript="" is fine for non-ASR approaches (embedding search, contrastive).
    """

def model_size() -> int:
    """Total model size in bytes (estimate is fine)."""
```

### Pattern

```python
import sys
from pathlib import Path

PROJECT_ROOT = Path(__file__).parent.parent.parent
sys.path.insert(0, str(PROJECT_ROOT))

from shared.audio import load_audio
from shared.quran_db import QuranDB

# Lazy-load globals
_model = None

def _ensure_loaded():
    global _model
    if _model is not None:
        return
    # ... load model, processor, etc.

def predict(audio_path: str) -> dict:
    _ensure_loaded()
    # ... inference logic
    # For ASR approaches: transcribe -> match against QuranDB
    # For embedding approaches: encode -> find nearest verse

def model_size() -> int:
    return 461 * 1024 * 1024  # estimate in bytes
```

### Multi-model experiments

If one experiment wraps multiple models (like `new-models/`), also export:

```python
def list_models() -> list[str]: ...
def predict(audio_path: str, model_name: str = "default") -> dict: ...
def model_size(model_name: str = "default") -> int: ...
```

The benchmark runner expands these into separate entries automatically.

### Registering a new experiment

Add it to `EXPERIMENT_REGISTRY` in `benchmark/runner.py`:

```python
EXPERIMENT_REGISTRY = {
    "whisper-lora": EXPERIMENTS_DIR / "whisper-lora" / "run.py",
    "your-new-experiment": EXPERIMENTS_DIR / "your-new-experiment" / "run.py",
    # ...
}
```

Directory names use hyphens (e.g. `whisper-lora`). The runner uses `importlib.util.spec_from_file_location` so hyphens work fine.

## Shared Utilities

Use `from shared.X import ...` (not `from offline_tarteel.X`):

- `shared.audio.load_audio(path, sr=16000)` — returns float32 numpy array at 16kHz
- `shared.normalizer.normalize_arabic(text)` — strips diacritics, normalizes alef/taa marbuta
- `shared.quran_db.QuranDB` — loads `data/quran.json` (6,236 verses), provides:
  - `match_verse(text)` — fuzzy match with multi-ayah span support
  - `search(text, top_k=5)` — top-k Levenshtein matches
  - `get_verse(surah, ayah)`, `get_surah(surah)`, `get_next_verse(surah, ayah)`
- `shared.verse_tracker.VerseTracker` — streaming verse detection with continuation bias
- `shared.streaming.StreamingPipeline` — connects ASR backends to verse tracker

## Running Benchmarks

```bash
../.venv/bin/python -m benchmark.runner                          # all experiments
../.venv/bin/python -m benchmark.runner --experiment whisper-lora  # single experiment
../.venv/bin/python -m benchmark.runner --category short         # filter by category
```

Results go to `benchmark/results/<timestamp>.json`.

### Scoring

The benchmark uses **sequence evaluation**. Experiments with `predict()` are called directly; others use `transcribe()` + `VerseTracker`. Metrics:

- **Recall**: fraction of expected verses detected in the correct order
- **Precision**: fraction of predicted verses that are correct
- **Sequence Accuracy**: 1.0 only if the full ordered sequence matches exactly

### Experiment Interface

Each experiment must export:

```python
def transcribe(audio_path: str) -> str:  # raw transcript (required for benchmark)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yazinsai/tilawa](https://github.com/yazinsai/tilawa) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
