---
trigger: always_on
description: **NEVER CREATE DUPLICATE CODE IN experiments/ AND src/**
---

# .cursorrules - Brain-Go-Brrr Project (FIXED ARCHITECTURE)

## 🔥🔥🔥 CRITICAL: ARCHITECTURE RULES TO PREVENT DISASTERS 🔥🔥🔥

### RULE #1: NO PARALLEL IMPLEMENTATIONS EVER
**NEVER CREATE DUPLICATE CODE IN experiments/ AND src/**

#### What Went Wrong (The Disaster):
- Created TWO PARALLEL UNIVERSES that don't communicate
- experiments/ reimplemented everything from scratch
- src/ had working components that were ignored
- Result: AUROC=0.50 (complete training failure), wasted compute, confusion

#### The ONLY Correct Architecture:
```python
# experiments/train_anything.py - MUST BE THIN
from brain_go_brrr.infra.data import Dataset  # ALWAYS USE SRC
from brain_go_brrr.infra.ml_models import Model  # NEVER REIMPLEMENT
from brain_go_brrr.domain.preprocessing import preprocess  # REUSE!

# FORBIDDEN: Creating new datasets, models, preprocessing in experiments/
```

### RULE #2: CHECK BEFORE BUILDING
1. **ALWAYS** search src/ for existing implementations
2. **NEVER** build without checking what exists
3. **ALWAYS** reuse components from src/
4. **NEVER** create "isolated" implementations

### RULE #3: NORMALIZATION IS CRITICAL
- MNE outputs: 1e-5 scale (Volts)
- EEGPT expects: N(0,1) normalized
- ALWAYS normalize before model input
- NEVER trust raw sensor data

## 🚨 Current Architecture Status (Aug 28, 2025)

**PROBLEM DISCOVERED**: Parallel implementations in experiments/ and src/
- Status: BOTH FIXED with normalization
- TODO: Migrate experiments/ to use src/ components

## 🚨 CRITICAL WARNING: PyTorch Lightning 2.5.2 Bug

**DO NOT USE PYTORCH LIGHTNING FOR TRAINING!** Lightning 2.5.2 has a critical bug that causes training to hang indefinitely at:
```
Loading `train_dataloader` to estimate number of stepping batches
```

This occurs with large cached datasets (>100k samples) and CANNOT be fixed with any settings. We tried:
- `deterministic=False` ❌
- `limit_train_batches` as integer ❌
- `max_steps=10000` ❌
- `num_sanity_val_steps=0` ❌
- `fast_dev_run=True` ❌

**SOLUTION**: Use `experiments/eegpt_linear_probe/train_tuab.py` (pure PyTorch)

## 🧠 Critical Context

This is a medical-adjacent EEG analysis system using the EEGPT foundation model. While not FDA-approved, code quality matters - bugs could impact clinical decisions. Always prioritize safety and accuracy over speed.

## ✅ What's Currently Working
- **YASA Sleep Analysis**: 100% functional, 87% accuracy
- **Autoreject QC**: Bad channel detection, artifact rejection
- **EEGPT Features**: 2,048-dim features (4×512 summary tokens, flattened)
- **FastAPI Server**: REST API with Redis caching
- **CI/CD Pipeline**: All branches green, pre-commit hooks fixed
- **Unit Tests**: All tests pass locally (run `make test`)
- **Architecture**: Unified - experiments/ uses src/ components
- **Normalization**: SSOT in wrapper, datasets emit Volts (SI units)
- **Channel Validation**: Enforces correct order per dataset

## 🟡 In Progress
- **TUAB Abnormality Detection**: Training linear probe
- **experiments/ cleanup**: Removing remaining sys.path.insert hacks

## ❌ Not Implemented
- **Event Detection**: Architecture docs only, no code
- **Authentication**: No OAuth2/JWT
- **Production Deployment**: No Kubernetes/PostgreSQL
- **Message Queue**: No Celery implementation

## 📚 Project Overview

- **Purpose**: Production-ready Python wrapper around EEGPT for EEG analysis (QC, abnormality detection, sleep staging, event detection)
- **Model**: EEGPT 10M parameters at `/data/models/pretrained/eegpt_mcae_58chs_4s_large4E.ckpt`
- **Architecture**: Service-oriented (not microservices yet), FastAPI + PyTorch + MNE
- **License**: Apache 2.0 (NOT MIT)
- **Python**: 3.11-3.12 (3.13 not supported due to scipy/OpenBLAS issues)

## 📖 Essential Documentation

```bash
# Consolidated, accurate documentation (6 files total)
/docs/README.md           # Documentation hub - START HERE
/docs/QUICK_START.md     # Get running in 5 minutes
/docs/ARCHITECTURE.md    # System design (actual implementation)
/docs/API.md            # REST endpoints (working only)
/docs/TRAINING.md       # Model training guide
/docs/TESTING.md        # Testing guidelines

# Root documentation
/README.md              # Project overview
/CLAUDE.md             # This file - AI assistant context
```

## 🔧 Development Commands

### 🚨 CRITICAL: Pre-Push Validation (MUST PASS OR CI WILL FAIL)

```bash
# ALWAYS RUN THESE BEFORE PUSHING (uses project Makefile targets)
make format      # Auto-format code
make lint        # Check for lint errors
make typecheck   # Run type checking
make validate    # Full pre-push validation suite

# Or run all checks at once:
make check-all   # Runs all CI checks locally
```

### 🔒 SECURITY: torch.load Requirements

**CI/CD WILL FAIL if torch.load is used unsafely!** All `torch.load` calls MUST either:
1. Use `weights_only=True` for pure tensor data
2. Use `weights_only=False` WITH `# nosec:weights_only` comment explaining why unsafe loading is needed
3. Use the `safe_load` wrapper from `brain_go_brrr.infra.safe_load`

```python
# ✅ GOOD - Safe loading for tensors
checkpoint = torch.load(path, map_location='cpu', weights_only=True)

# ✅ GOOD - Unsafe with explicit justification
data = torch.load(cache_file, weights_only=False)  # nosec:weights_only - cache contains numpy arrays


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Clarity-Digital-Twin/brain-go-brrr](https://github.com/Clarity-Digital-Twin/brain-go-brrr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
