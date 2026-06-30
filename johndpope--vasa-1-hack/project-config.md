---
trigger: always_on
description: **Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
---

# Claude Code Instructions

## Task Master AI Instructions
**Import Task Master's development workflow commands and guidelines, treat as if import is in the main CLAUDE.md file.**
@./.taskmaster/CLAUDE.md

## Documentation

All project documentation is stored in the `/docs` folder. Key documents include:
- `docs/AUDIT_TOOL_README.md` - Expression audit tool documentation
- `docs/LOSS_AUDIT.md` - Loss function audit and configuration
- `docs/LOSS_CLEANUP_SUMMARY.md` - Loss cleanup history
- `docs/vasa_model_documentation.md` - VASA model architecture docs
- `docs/vasa-prd.md` - Product requirements document

When creating new documentation, place `.md` files in `/docs` folder (except README.md and CLAUDE.md which stay in root).

## VASA-1 Project Status and Findings

### Current Training Status
- **Overfitting Training**: Successfully running with wandb logging
  - Loss decreased from 40+ to ~1.2-1.3 showing good convergence
  - Using WindowSequenceSampler for temporal context preservation
  - Caching optimization working (source embeddings computed once per batch)
  - Wandb project: https://wandb.ai/snoozie/vasa-overfitting

### Key Fixes Applied

#### 1. DataLoader Issues (FIXED)
- **Problem**: VASAIntegratedDataset.__len__() returned video count (6) instead of window count (316)
- **Solution**: Fixed to return len(self.windows) and implemented WindowSequenceSampler
- **Files**: vasa_dataset.py, vasa_sampler.py, train_overfit.py, vasa_trainer.py

#### 2. Temporal Context Handling (FIXED)
- **Problem**: Breaking prev_context mechanism when changing __getitem__
- **Solution**: Custom WindowSequenceSampler maintains sequences of 4 consecutive windows
- **Implementation**: create_window_sequence_collate_fn adds prev_context to batches

#### 3. CUDA Multiprocessing (FIXED)
- **Problem**: "Cannot re-initialize CUDA in forked subprocess"
- **Solution**: Set num_workers=0 in DataLoader

#### 4. Redundant Computations (FIXED)
- **Problem**: Source embeddings computed repeatedly for same identity image
- **Solution**: Improved caching with stable tensor-based keys instead of id()

### Audio Context Implementation

#### JoyVASA Approach (from paper):
- Uses frozen wav2vec2 encoder for audio features
- Includes both past audio features A_{-w_prev, w_prev} and current motion
- Concatenates past speech with current noisy motion in diffusion

#### Our Implementation:
- ✅ Using wav2vec2 for audio features (768 dimensions)
- ✅ Including prev_context with previous motion parameters
- ✅ Including previous audio features in context
- ⚠️ May need to adjust concatenation strategy to match JoyVASA

### Expression Audit Tool

#### Purpose
Diagnose training issues by comparing ground truth expressions vs model predictions frame-by-frame.

#### Quick Start
```bash
./audit.sh  # Interactive menu
```

Or direct:
```bash
python audit_expressions.py \
    --video junk/videovideoeI2V8Bd5X9s-scene6_scene1.mp4 \
    --identity ./data/IMG_1.png \
    --config overfit_config.yaml \
    --checkpoint checkpoints_overfit/best_checkpoint.pt \
    --output-dir expression_audit
```

#### What It Analyzes
1. **Expression L2 Distance** - Per-frame difference in expression embeddings
   - Target: < 1.0 for good overfitting
   - Current: ~5.5 (model NOT overfitting yet)

2. **Theta L2 Distance** - Per-frame head pose difference
   - Target: < 0.3 for good overfitting
   - Current: ~1.5 (poor pose matching)

3. **Audio Alignment** - Verifies identical audio features used
   - Should be: 0.000000 ✅
   - Confirms audio preprocessing is correct

#### Current Findings (Epoch 226)
- ❌ **Model is NOT overfitting** despite 226 epochs
- ❌ Expression L2: 5.54 (should be < 1.0)
- ❌ Theta L2: 1.50 (should be < 0.3)
- ✅ Audio features identical (preprocessing correct)

**Root causes**:
1. Model capacity too small (12.5M vs 29M target)
2. Loss weights may need tuning
3. Learning rate may be too high
4. Need more training epochs

#### Output Files
- `expression_comparison.png` - Visualization with 3 subplots showing L2 distances over time
- `expression_metrics.csv` - Per-frame metrics for detailed analysis

See `docs/AUDIT_TOOL_README.md` for full documentation.

### Known Issues

#### 1. Wandb Visualization
- `disentangle/frame_j` visualization broken due to shape mismatches
- Occurs when generated_frames has different shape than expected
- Non-critical - doesn't affect training

#### 2. Batch Size Optimization
- Currently using 4 windows per batch (7GB/32GB VRAM)
- Could increase to 16 windows but shape mismatch in collate function needs debugging

### Training Scripts

#### For Overfitting Test:
```bash
./train.sh  # Select option 1 for overfitting
# OR directly:
python train_overfit.py
```

#### For Full Training:
```bash
./train.sh  # Select option 2 for full training
# OR directly:
python vasa_trainer.py --config vasa_config.yaml
```

### Important Configuration

#### overfit_config.yaml:
- learning_rate: 5e-3
- gradient_accumulation_steps: 2
- num_epochs: 1000
- resume_from: "checkpoints_overfit/best_checkpoint.pt"

#### vasa_config.yaml:
- resume_from: "" (set to checkpoint path to resume)
- Similar settings but for full dataset

### JoyVASA Reference
- Location: /media/12TB/JoyVASA

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [johndpope/VASA-1-hack](https://github.com/johndpope/VASA-1-hack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
