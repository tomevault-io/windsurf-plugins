---
trigger: always_on
description: OpenScrub is a local, GPU-accelerated video redaction tool: it detects and
---

# CLAUDE.md — OpenScrub

OpenScrub is a local, GPU-accelerated video redaction tool: it detects and
blurs PII (faces, names, SSNs, addresses, license plates, full-body person blur, QR/barcodes, screens — 14 default categories) in
videos and screen recordings, with a human-review step before anything is
trusted. Apache-2.0. Python 3.10+. Runs on Windows/Linux; primary dev/deploy
target is Windows 10 + NVIDIA RTX 3060.

## File map

| File | Role |
|---|---|
| `openscrub.py` | The engine. CLI + all detection/render logic. Single file, ~2400 lines. |
| `openscrub_web.py` | Flask web app. The entire UI is one embedded `PAGE` string (HTML/CSS/JS). Serves via cheroot (production WSGI, TLS) with Flask-dev fallback. |
| `zones_ui.py` | The app SHELL (`ZONES_PAGE`): dark theme, header (gear → settings), and the Scan Setup editor — load a video, stack detection windows on a timeline (one lane per window, overlap allowed), per-window categories + zones, copy/paste zones, audio mute lanes, clip bookends, Start scan. Contains `%%MARKER%%` slots that openscrub_web.py fills at import to build the single-page app. |
| `openscrub_gui.py` | Legacy Tk GUI. Frozen; ships but is not actively developed. |
| `openscrub_setup.py` | `openscrub-setup` command: detects/installs Tesseract + FFmpeg (winget/apt), optional spaCy model + plate model, Windows Start Menu shortcuts. Ships in the wheel. |
| `windows/` | Native Windows packaging: `openscrub.spec` (PyInstaller, two branded exes), `installer.iss` (Inno Setup → Program Files), `build_installer.bat` (runs both; attach output exe to the GitHub release). Build on Windows only. |
| `install.py` | Windows-friendly installer (deps, GPU OCR, shortcut, `--with-plates`). |
| `docker/` | `Dockerfile.opencv-cuda` builds the CUDA-OpenCV base image (rare, via opencv-cuda-base.yml); `Dockerfile.cuda` FROMs it (base published to ghcr; bump the FROM tag only when the base is rebuilt). `prefetch_models.py`: standalone (NO openscrub import) model prefetch run in its OWN Docker layer ABOVE the app copy in all three images, so releases don't re-download unchanged models; its (url,sha256,filename) table is pinned to the engine constants by test_docker_prefetch_matches_engine_pins — update BOTH when a model changes. |
| `Dockerfile.intel` | `:intel` image (amd64): Debian non-free iHD media driver + libmfx/libvpl for QSV encode, `onnxruntime-openvino` replacing stock onnxruntime (build asserts the provider registered). Needs `--device /dev/dri`; everything falls back loudly to CPU without it. |
| `plate_models.json` | Curated license-plate model registry (see PLATES.md). |
| `face_models.json` | Curated optional face-model registry (CenterFace/SCRFD); built-in YuNet needs no file. Ships everywhere plate_models.json does (wheel, sdist, Dockerfiles, PyInstaller spec, updater pin-carry). |
| `person_models.json` | Curated person-model registry (YOLOv10 ONNX from onnx-community on HF, AGPL — download-only, never bundled; hashes PRE-pinned, validated at authoring). Ships everywhere plate_models.json does. |
| `fetch_plate_models.py` | Alt path to fetch plate models via the open-image-models pip package. |
| `openscrub_update.py` | `openscrub-update` command + web self-update backend: PyPI version check, sha256-verified sdist download, data-preserving folder update (PRESERVE set), TOFU pin carry-forward. Ships in the wheel. |
| `openscrub_vault.py` | At-rest encryption for the job store: scrypt keystore, chunked AES-256-GCM files (`.osvault`), lock/unlock tree walkers. NO password reset by design. Ships in the wheel. Lock-on-shutdown lives in openscrub_web: a SIGTERM handler (docker stop; locks then os._exit — sys.exit is swallowed by cheroot) + an atexit hook (Ctrl+C; uses the import-time `_HERE` constant because `__file__` is gone during interpreter teardown — both failure modes were real and verified). Encryption must finish inside the container stop grace period (`docker stop -t 120`). |
| `test_openscrub.py` | pytest suite (52 tests). Must stay green. |
| `deploy/` | App-store submission kit: winget/CasaOS/Runtipi/TrueNAS/Umbrel/Portainer/CapRover/Coolify manifests + a novice-friendly submission guide (deploy/README.md). Platforms with their own reverse proxy get `--http` in the command; direct-port platforms keep default TLS. Bump pinned versions at submission time. |
| `tools/make_icons.py` | Regenerates every icon/logo asset from `assets/badge_master.png`. |
| `tools/make_wordmark.py` | Regenerates the typeset Poppins wordmarks (navy + white). |
| `assets/` | Brand assets. `badge_master.png` (canonical, mosaic+brackets style) and `badge_master_blurbox_alt.png` (alternate) are the sources; everything else is generated. |

## Engine architecture (openscrub.py)

Pipeline: `run_pipeline` → `run_scan` (OCR sampling + detectors, builds
`Detection` list) → `merge_detections` → review or render (`blur_region`
with modes blur/box/mosaic/inpaint, per-category via `--mode-map`).
Mode strength hierarchy (README documents it): box > inpaint (Telea
reconstruction via `_inpaint_fill`, works on 8-bit BGR AND 10-bit
planes — synthetic content, so the in-region 8-bit round-trip is
harmless) > mosaic (tile floor 6px — 2px tiles are the depixelation

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [austinmabry/OpenScrub](https://github.com/austinmabry/OpenScrub) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
