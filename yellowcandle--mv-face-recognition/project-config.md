---
trigger: always_on
description: - NO NEED TO IMPLEMENT WEBCAM face recognition
---

- NO NEED TO IMPLEMENT WEBCAM face recognition
- Only process files in /source/videos directory, no webcam processing required
- pls! do not! overwrite my README.md !!!
- you MUST document your work in DESIGN.md, the DESIGN.md MUST fit in your context window.
- document your TODOs in DESIGN.md

# CRITICAL FILES - DO NOT DELETE
## ⚠️ ABSOLUTELY CRITICAL - DO NOT DELETE THESE FILES ⚠️

- **metadata/contestant_info.csv** - ESSENTIAL contestant database mapping (編號,姓名,暱稱,年齡)
  - **LOCATION: /metadata/contestant_info.csv** 
  - Maps contestant numbers (1-96) to names and nicknames
  - Used by face recognition system to identify contestants
  - Contains 96 contestant records with Chinese names and nicknames
  - Previously deleted in commit 8348e0b3, restored from git history
  - **THIS FILE IS REQUIRED FOR THE ENTIRE SYSTEM TO FUNCTION**
  - **DO NOT DELETE, MOVE, OR MODIFY WITHOUT EXPLICIT USER PERMISSION**
  - **VIDEO PLAYER AND FACE RECOGNITION DEPEND ON THIS FILE**

# HUGGINGFACE XET DATA STORAGE - MIGRATION COMPLETE
## ⚠️ CONTESTANT DATA NOW ON HUGGINGFACE XET (January 2026) ⚠️

**CRITICAL MIGRATION COMPLETED:**
Contestant photos and embeddings have been migrated from Git to HuggingFace XET storage for efficient large file management.

### ✅ CURRENT DATA STORAGE STATE:
- **HuggingFace Repository**: https://huggingface.co/datasets/yellowcandle/mv-face-recognition-data
- **Photos**: 196 contestant photos (3.97 MB) - JPG/PNG format
- **Embeddings**: 95 face recognition embeddings (.npy files)
- **Metadata**: contestant_info.csv with 96 contestant records
- **Git Status**: Photos removed from Git repository (backed up in `backup-before-hf-migration` branch)
- **Local Files**: Photos and embeddings remain on local filesystem but excluded from Git

### 📦 DATASET STRUCTURE ON HUGGINGFACE:
```
yellowcandle/mv-face-recognition-data/
├── metadata/
│   ├── contestant_info.csv          # 96 contestants (編號,姓名,暱稱,年齡)
│   └── photo_manifest.json          # Photo upload tracking
├── embeddings/
│   └── contestant_*/
│       └── *_embedding.npy          # 512-dim face vectors
└── photos/
    └── contestant_*/
        └── *.jpg|png                # Contestant photos
```

### 🔧 DEVELOPMENT SETUP:
**First-time setup or when photos are missing:**
```bash
# Download contestant data from HuggingFace
python scripts/upload_dataset_to_hf.py --download-only --download-dir source/photo/contestants
```

**Upload new photos or updates:**
```bash
# Ensure HF_TOKEN is set or logged in via: huggingface-cli login
python scripts/upload_dataset_to_hf.py --upload
```

### ⚠️ CRITICAL WARNINGS FOR FUTURE AI ASSISTANTS:

1. **NEVER RESTORE PHOTOS TO GIT**:
   - Photos are intentionally removed from Git repository
   - Backup branch `backup-before-hf-migration` preserves original state
   - DO NOT run: `git checkout backup-before-hf-migration -- source/photo/contestants/`
   - USE HuggingFace as single source of truth

2. **.GITIGNORE CONFIGURATION**:
   - `source/photo/contestants/**/*.jpg` is excluded
   - `source/photo/contestants/**/*.png` is excluded
   - `*.npy` embedding files are excluded
   - DO NOT modify .gitignore to track these files again

3. **DOWNLOAD BEFORE PROCESSING**:
   - Face recognition requires photos to be present locally
   - If `source/photo/contestants/` is empty, download from HuggingFace first
   - Check photo availability before running video processing

4. **RATE LIMIT OPTIMIZATION**:
   - Upload script uses `upload_folder()` for batch operations
   - Avoids HuggingFace rate limit (256 commits/hour)
   - Screenshot files are automatically filtered during upload

5. **ROLLBACK PROCEDURE** (Only if absolutely necessary):
   ```bash
   # Step 1: Restore from backup branch
   git checkout backup-before-hf-migration -- source/photo/contestants/

   # Step 2: Update .gitignore to remove exclusions
   # Step 3: Commit and push changes
   ```

### 📊 MIGRATION STATISTICS:
- **Date**: January 2026
- **Photos Migrated**: 196 files (3.97 MB)
- **Embeddings Migrated**: 95 files
- **Verification**: 100% of sample files matched checksums
- **Repository Size Reduction**: ~4 MB

# CRITICAL DEPLOYMENT CONFIGURATION - DO NOT BREAK
## ⚠️ FRONTEND RESTRUCTURING COMPLETE (January 2025) ⚠️

**MAJOR RESTRUCTURING COMPLETED:**
The frontend has been consolidated from multiple directories into mvp-processor as the single frontend.

### ✅ NEW STRUCTURE (Current Working State):
- **Primary Frontend**: mvp-processor/ directory contains both Python processing AND SvelteKit frontend
- **Face Recognition Dashboard**: Complete implementation based on face_recognition_mockup.html
- **Routes**: /, /video-player, /processing, /analytics with dark theme UI
- **Legacy Frontend**: frontend/ moved to frontend-legacy-backup/ (do not restore)
- **Build Process**: `cd mvp-processor && npm run build` for complete system

## ⚠️ SVELTEKIT vs LEGACY APP CONFLICTS - RESOLVED ⚠️

**CRITICAL ISSUE RESOLVED (July 10, 2025):**
The deployment was serving the WRONG APPLICATION due to build configuration conflicts.

### ✅ CORRECT CONFIGURATION (Current Working State):
- **Frontend Framework**: SvelteKit 2.x with TypeScript in mvp-processor/ directory
- **Build Command**: `cd mvp-processor && npm run build` (uses SvelteKit via `vite build --mode production`)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yellowcandle) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
