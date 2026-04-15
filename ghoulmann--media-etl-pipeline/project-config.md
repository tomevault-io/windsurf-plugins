---
trigger: always_on
description: **Media-ETL** is a five-stage Docker-based ETL pipeline for automated music & video triage. Core principle: **Confidence-Gate automation** routes high-confidence matches directly to NAS, while low-confidence files await manual curation.
---

# Media-ETL Copilot Instructions

## Project Overview

**Media-ETL** is a five-stage Docker-based ETL pipeline for automated music & video triage. Core principle: **Confidence-Gate automation** routes high-confidence matches directly to NAS, while low-confidence files await manual curation.

**Architecture Stages:**
1. **Ingest** (`slskd`) - Soulseek P2P download client
2. **Sifter** (Beets + inotify) - Real-time file detection & metadata matching
3. **Workbench** (Picard/TMM) - Manual curation for rejected files
4. **Syncer** (Rclone + Ofelia) - Scheduled atomic migration to NAS
5. **Monitor** (Dozzle/Portainer) - Live container & log visibility

## Critical Workflows

### Pre-Deployment Setup (Required)
```bash
cp .env.example .env                          # Copy template
# Edit .env: set DATA_DIR, NAS_MUSIC_MOUNT, GITHUB_USERNAME
cp rclone/rclone.conf.example rclone/rclone.conf
# Edit rclone.conf with remote credentials (SMB/S3/cloud storage)
docker-compose up -d                          # Deploy stack
```

**Key files that must exist before `docker-compose up`:**
- `.env` - Contains all volume mounts and environment variables
- `rclone/rclone.conf` - Remote storage authentication
- NAS mount must be active on host system

### Common Development Tasks

**View pipeline logs:**
```bash
docker-compose logs -f sifter              # Real-time sifter matching
docker-compose logs -f rclone-sync         # Sync operations
docker-compose logs dozzle                 # Container dashboards
```

**Test Beets matching locally (before deployment):**
```bash
cd sifter && beet import -v /test/audio/file.mp3
```

**Rebuild sifter container after config changes:**
```bash
docker-compose down sifter
docker build -t ghcr.io/USERNAME/media-etl/sifter:latest ./sifter
docker-compose up -d sifter
```

## Key Architecture Patterns

### Volume Mounts & Data Flow
- **`/watch`** (sifter input) ← `${DATA_DIR}/downloads/complete` mounted from slskd output
- **`/manual_staging`** ← `${DATA_DIR}/manual_staging` - split into `audio_picard/` and `video_staging/`
- **`/mnt/nas/music`** ← `${NAS_MUSIC_MOUNT}` - final destination after rclone sync
- **Volumes are CRITICAL:** Verify mounts in `docker-compose.yml` before troubleshooting file movement issues

### The Confidence Gate (Beets Logic)
Located in [sifter/config.yaml](../sifter/config.yaml) — see [docs/architecture/confidence-gate.md](../docs/architecture/confidence-gate.md) for detailed scoring explanation:
- `strong_rec_only: yes` - Only moves files with confidence ≥ `BEETS_CONFIDENCE` threshold
- `BEETS_CONFIDENCE` env var (default 0.90) controls strictness
  - **0.90+**: Strict, most files need review
  - **0.75-0.85**: Moderate balance (recommended)
  - **<0.50**: Loose, many false positives
- Files below threshold → moved to `manual_staging/audio_picard/` for Picard review
- Files above threshold → renamed with metadata tags + moved directly to NAS

### File Detection & Routing
Located in [sifter/sifter.sh](../sifter/sifter.sh):
- **Download completion**: Uses `inotifywait -e close_write` event (fires when file finishes writing). slskd separates `/incomplete` (active) from `/complete` (finished) folders; Sifter only watches `/complete`
- **File filtering**: Whitelisted extensions for both audio and video; slskd default config already filters dangerous files (.exe, .bin, etc.), Sifter adds defense-in-depth filtering
- **Audio vs Video determination**: By file extension regex matching (whitelisted extensions)
  - **Audio**: `mp3|flac|m4a|ogg|wma` → routed to Confidence Gate (Beets)
  - **Video**: `mkv|mp4|avi` → **entire parent folder** moved to `manual_staging/video_staging/` (preserves hierarchy & companion files)
- **Folder hierarchy preservation**: When a video file is detected, the entire parent folder is moved to `video_staging`, ensuring:
  - Show/Season/Episode structure is maintained for TV series
  - Companion files (.nfo, .srt, .jpg, .png) move with the video
  - All related metadata stays associated with the content
- **Shows vs Movies**: NOT differentiated by Sifter. All video files treated identically. Show/movie classification is handled downstream by tinyMediaManager (TMM) during manual Workbench curation
- **Mixed audio/video folders**: If a folder contains both audio and video, the video detection triggers folder move first; audio files within that folder are not processed separately (prevention of duplicate processing)

### Video File Handling (Folder Structure & Companion Files)
**Implementation status**: Now fully implemented with folder hierarchy preservation
- **Behavior**: When a video file is detected, the **entire parent folder** is moved to `manual_staging/video_staging/` (not just individual files)
- **Folder structure**: Complete hierarchy is preserved during the move (important for TV shows with Show/Season/Episode structure)
- **Companion files**: All files in the folder are moved along with the video, including:
  - `.nfo` metadata files
  - `.srt` subtitle files
  - `.jpg`, `.png` artwork/poster files
  - Any other files in the folder (documents, notes, etc.)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ghoulmann) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
