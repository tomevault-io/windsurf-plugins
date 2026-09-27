---
trigger: always_on
description: **Metixel Photoframe** is a custom, open-source Digital Photo Frame operating system and application suite targeting:
---

# CLAUDE.md — Metixel Photoframe Project Instructions for AI Assistants

## Project Summary

**Metixel Photoframe** is a custom, open-source Digital Photo Frame operating system and application suite targeting:
- **Phase 1:** Raspberry Pi 2, 3, 4, 5, and Zero 2 W (Mesa/DRM on Trixie). Pi 5 (2GB+) is the recommended platform. Pi 4 is supported but untested. Pi 2 and Pi Zero 2 W are 32‑bit manual‑install only — no pre‑built image available. Pi Zero 2 W (512MB) is untested and image‑only (no video, optimisation, or transcoding).
- **Phase 2:** Non-Pi SBCs like the Radxa Zero 3W (Mesa/DRM/Wayland)

The application is written in **Python 3** with a display backend abstraction that isolates the rendering layer (pi3d on Phase 1, PyOpenGL on Phase 2) from the presentation logic. The `Pi3dBackend` works on any Pi with pi3d installed — it runs under cage (minimal Wayland compositor) + XWayland on Trixie/Bookworm.

### Media Pipeline (4-Phase)

```
Phase 1: WATCH   → FolderWatcher gathers metadata only (type, dims, codec)
Phase 2: OPTIMISE → OptimisationQueue thresholds + processes (images first, then videos)
Phase 3: QUEUE   → Slideshow playlist (ready-to-play items only)
Phase 4: SYNC    → Immich downloads to media/sync/immich/ (picked up by Phase 1)
```

**Thumbnails and video frames** are generated ONLY by the backend processors (`image.py` / `video.py`). The frontend (`engine.py`) does NOT generate thumbnails or extract video frames — it never runs ffmpeg/ffprobe.

## Core Rules (ALWAYS follow these)

1. **Consult the architecture first.** Before proposing ANY code change, read `ARCHITECTURE.md` in the repository root. If you've lost context, run:
   ```bash
   cat ARCHITECTURE.md
   ```
   This file contains the complete system design, component relationships, and implementation roadmap.

2. **Respect the display backend abstraction.** Never import `pi3d` directly in presentation or widget code. Always use `metixel.display.backend.DisplayBackend` — the factory in `metixel.display.__init__` auto-detects the hardware and returns the correct backend. The only file that may import pi3d is `src/metixel/display/dispmanx_backend.py`.

3. **Respect the 4-phase media pipeline.** Media flows through four distinct phases:
   - **Phase 1 (Watch):** `FolderWatcher` gathers metadata only — file type, dimensions, video codec. Does NOT process, resize, or transcode. Pushes `MediaItem` stubs to the `OptimisationQueue`.
   - **Phase 2 (Optimise):** `OptimisationQueue` (background thread in `BackendDaemon`) routes ALL media through the appropriate processor. Images exceeding display resolution are resized. **All videos** go through `VideoProcessor.process()` — even H.264 videos within resolution limits — because first-frame (``.1.frame``) and last-frame (``.2.frame``) JPEG extraction is always required. The processor skips the expensive transcode step for videos already in optimal format. Image optimisation runs before video optimisation.
   - **Phase 3 (Queue):** `StateManager` maintains the slideshow playlist. `PresentationEngine` reads from it. Only ready-to-play items are in the playlist.
   - **Phase 4 (Sync):** `ImmichSyncer` downloads to `media/sync/immich/` (default). This directory is a watch path — Phase 1 picks up new files on the next poll.

4. **Thumbnails and video frames belong to the backend.** `ImageProcessor` and `VideoProcessor` generate thumbnails, first frames, and last frames during optimisation. The frontend (`engine.py`) does NOT generate thumbnails, extract video frames, or run ffmpeg/ffprobe. All frame generation was removed from the presentation engine — it only loads pre-generated cache files referenced by `MediaItem.first_frame_path` and `MediaItem.last_frame_path`.

5. **Phase 1 hardware varies in capability.** The Pi Zero 2 W has 512MB RAM; Pi 2/3 have 1GB. Always:
   - Use `Texture(free_after_load=True)` to release CPU memory after GPU upload
   - Prefer `GL_RGB565` (2 bytes/pixel) over `GL_RGBA` (4 bytes/pixel) for GPU textures
   - Limit GPU-resident textures to 3 maximum at any time (current, next, blend)
   - Process media one file at a time in a subprocess that exits to release memory
   - Call `gc.collect()` after large image processing
   - Cap the render loop at 30 FPS (not 60)
   - Pi Zero 2 W (512MB) is untested — not a current deployment target

6. **Display protection is mandatory.** All screens can burn in. Implement:
   - Configurable pixel shifting (±2px cyclic offset every N minutes)
   - Screen dimming during "sleep" hours
   - A screensaver fallback if no media is queued
   - Use `vcgencmd display_power 0` for deep sleep (legacy) or DRM DPMS via sysfs on KMS

7. **Graceful degradation is required.** If:
   - The GPU memory is too low (<64MB) → log warning, fallback to static images only
   - Hardware video decode unavailable → skip video files, log warning
   - Immich server unreachable → continue slideshow with cached media, retry with exponential backoff
   - Network down → continue with local media only
   - Never crash. Never show a traceback on screen. Log errors and continue.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dennisadvani/metixel-photoframe](https://github.com/dennisadvani/metixel-photoframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
