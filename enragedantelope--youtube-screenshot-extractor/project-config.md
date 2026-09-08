---
trigger: always_on
description: Pull clean, high-quality still frames from videos — YouTube, 1000+ other sites via yt-dlp, or local files. Downloads (if needed), finds best frames, filters blurry/low-quality ones, crops black bars, saves the rest. Built for gathering training images for LoRAs/checkpoints. GUI + CLI, Python 3.10+, requires Deno (YouTube) and FFmpeg (keyframes).
---

# AGENTS.md — youtube-screenshot-extractor

Pull clean, high-quality still frames from videos — YouTube, 1000+ other sites via yt-dlp, or local files. Downloads (if needed), finds best frames, filters blurry/low-quality ones, crops black bars, saves the rest. Built for gathering training images for LoRAs/checkpoints. GUI + CLI, Python 3.10+, requires Deno (YouTube) and FFmpeg (keyframes).

**Deep reference: Previous CLAUDE.md contained implementation notes about YouTube PO Tokens, rate limiting, and authentication — that content is now archived in the project history**

## Current state

_Last verified: 2026-08-24_

- **Status:** working and maintained, no version number and no release tags — `git log` is the only version record. The recent history is a run of audits fixing silently-broken options rather than new features.
- **Works:** all four extraction methods (interval, every frame, keyframes, scene detection); automatic blur/quality filtering, black-bar cropping and watermark flagging; YouTube authentication via browser cookies plus request-rate throttling; resume for large extractions; parallel worker-pool processing that streams frames instead of loading the whole video; GUI and CLI at parity.
- **In progress:** nothing — recent history is three audit rounds: dependency/security floors plus the first test suite and CI; a robustness/parity pass (Stop kills the whole process tree, failed downloads clean up their partial files, `--png` honored by keyframes, FFmpeg filter frames piped instead of temp-filed, GUI blur range matched to the CLI); then a correctness pass (scene-frame numbering, GUI keyframes gating, typed-entry validation, CI job timeouts and xvfb GUI coverage).
- **Known gaps / next steps:** tests cover the pure helpers, `process_frame`, and the GUI's behaviour against a real Tk tree (`tests/`, run with `pytest`; CI runs them under xvfb) — **downloading is still verified only by hand**, since nothing in CI touches the network; **YouTube extraction is inherently fragile** — yt-dlp must be kept current (launcher option 2, or `pip install --upgrade "yt-dlp[default]"`), and the working client selection changes over time; Deno is required for YouTube and FFmpeg for keyframes, so a partial install silently limits which methods work; the rate-limit and client-selection notes in *Conventions & gotchas* are the most perishable content in this file — re-verify them before trusting them.
- **Deep docs:** none — `README.md` is the user-facing reference. Earlier implementation notes on PO tokens and authentication live only in the git history.

## Architecture in 60 seconds

- **Any source:** YouTube + 1000+ sites via yt-dlp, or local video files
- **Four extraction methods:** interval (every N seconds), every frame, keyframes only, scene-change detection
- **Automatic quality control:** sharpness/blur and quality scoring filter out bad frames before saving
- **Black bar removal:** letterboxing and pillarboxing cropped automatically
- **Watermark detection:** flags likely-watermarked frames in filename
- **YouTube authentication:** browser cookie support (`--cookies-from-browser firefox/chrome/edge/safari`) for age-restricted and private videos, plus rate limiting (`--sleep-requests`)
- **Resume support:** pick large extractions up where they left off
- **Parallel processing:** streams frames through a worker pool instead of loading whole video into memory
- **Dual interface:** GUI (`youtube-screenshot-gui.py`) and CLI (`youtube-screenshot-script.py`)

## Layout

| File | Purpose |
|------|---------|
| `youtube-screenshot-script.py` | CLI for frame extraction |
| `youtube-screenshot-gui.py` | GUI (point-and-click interface) |
| `START.bat` / `start.sh` | Startup menu: setup, update, launch GUI, help |
| `requirements.txt` | Python dependencies (yt-dlp, etc.) |
| `requirements-dev.txt` | Test/lint dependencies (pytest, pyflakes) |
| `tests/` | `test_extractor.py` (CLI helpers, static GUI/CLI parity scans), `test_gui.py` (real Tk widget tree) |
| `.github/workflows/ci.yml` | Lint, byte-compile, unit tests, CLI smoke run, dependency audit, shellcheck |
| `assets/` | Screenshots and documentation images |

## Build / test / run

```bash
# Run the tests (no Deno/FFmpeg needed for the unit tests)
pip install -r requirements-dev.txt
python -m pytest tests -v

# Windows quick start
START.bat

# Or manual setup
python -m venv venv
venv\Scripts\activate          # macOS/Linux: source venv/bin/activate
pip install -r requirements.txt

# Install Deno (required for YouTube)
winget install DenoLand.Deno   # Windows
brew install deno              # macOS
curl -fsSL https://deno.land/install.sh | sh  # Linux

# Install FFmpeg (required for keyframe extraction)
winget install Gyan.FFmpeg     # Windows
brew install ffmpeg            # macOS
sudo apt install ffmpeg        # Linux

# Launch GUI
python youtube-screenshot-gui.py

# CLI usage examples

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [EnragedAntelope/youtube-screenshot-extractor](https://github.com/EnragedAntelope/youtube-screenshot-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-08 -->
