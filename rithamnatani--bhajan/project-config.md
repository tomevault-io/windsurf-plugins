---
trigger: always_on
description: > Context and findings for future AI agents working on this codebase.
---

# AI Agent Notes for bhajan

> Context and findings for future AI agents working on this codebase.

## Project Overview

`bhajan` is a cross-platform CLI tool that generates karaoke videos from YouTube URLs. It downloads audio, separates vocals/instrumental, transcribes lyrics, and either renders a video or launches a GUI player with synchronized lyrics.

## Key Architecture Decisions

### Pipeline Stages
1. **Download** (`stages/download.py`) - yt-dlp, auto-cleans YouTube tracking params
2. **Normalize** (`stages/normalize.py`) - ffmpeg EBU R128 loudnorm
3. **Separate** (`stages/separator.py`) - pluggable: Demucs or audio-separator
4. **Transcribe** (`stages/transcription.py`) - pluggable: faster-whisper or Parakeet
5. **Subtitles** (`stages/subtitles.py`) - ASS (karaoke style) + LRC formats
6. **Render/GUI** - Either ffmpeg video render OR tkinter GUI player

### Critical Files

| File | Purpose | Key Details |
|------|---------|-------------|
| `src/bhajan/pipeline.py` | Main orchestration | `KaraokePipeline.run()` executes all stages |
| `src/bhajan/cli.py` | CLI entry point | Uses `click`, defines all flags |
| `src/bhajan/config.py` | Constants & defaults | Font sizes, colors, model defaults |
| `src/bhajan/utils.py` | Utilities | `safe_filename()`, `clean_youtube_url()`, `parse_artist_and_title()` |
| `src/bhajan/gui_player.py` | GUI karaoke player | tkinter + pygame, word-level highlighting |

## Important Implementation Details

### YouTube URL Cleaning
**Function:** `utils.clean_youtube_url()`

YouTube URLs often have tracking parameters that cause issues:
- Input: `https://youtube.com/watch?v=ABC&list=...&start_radio=...`
- Output: `https://youtube.com/watch?v=ABC`

The pipeline calls this automatically at the start of `run()`.

### Safe Filename Handling
**Function:** `utils.safe_filename()`

Windows has strict filename rules:
- Cannot end with space or period
- Cannot contain `<>?:"/\|*`
- Max path length issues

The function:
1. Replaces invalid chars with `_`
2. Collapses multiple `_` or spaces
3. Strips leading/trailing `_` and spaces
4. Truncates to max_len (default 80)
5. Strips again after truncation (critical!)

**Bug history:** Previously left trailing spaces when truncating at word boundary, causing `[WinError 3] The system cannot find the path specified`.

### Metadata Extraction
**Functions:** `utils.clean_track_name()`, `utils.parse_artist_and_title()`

YouTube titles often have prefixes that break lyrics search:
- "Mix - Artist - Track" → extract "Artist - Track"
- "Artist - Topic" → handle specially
- "Artist 'Track' Official Video" → parse artist from quotes

These are used when `fetch_lyrics=True` to improve LRCLib API results.

### Pluggable Backends

Both separation and transcription use a registry pattern:

```python
# stages/separator.py
from bhajan.stages.separator_base import SeparatorBackend, SeparationResult

class MySeparator(SeparatorBackend):
    def name(self) -> str: return "my_backend"
    def available(self) -> bool: return True  # Check deps
    def separate(self, audio_path: Path, output_dir: Path) -> SeparationResult: ...

register(MySeparator, "my_backend")
```

Same pattern for transcription in `stages/transcription.py`.

### CUDA Handling

The codebase has multiple CUDA fallback strategies:

1. **Demucs separator** (`stages/separator_demucs.py`):
   - Tries `python -m demucs --device cuda`
   - On `CalledProcessError`, retries with `--device cpu`
   - Logs: "CUDA separation failed, falling back to CPU..."

2. **Audio-separator backend** (`stages/separator_audio_separator.py`):
   - Uses `audio_separator.Separator()` with `device="cuda"` or `"cpu"`
   - GPU acceleration requires `audio-separator[gpu]` install

3. **Whisper transcription** (`stages/transcription_whisper.py`):
   - Uses `faster-whisper` with `device="cuda"` or `"cpu"`
   - Auto-detects CUDA availability

4. **Parakeet transcription** (`stages/transcription_parakeet.py`):
   - NVIDIA NeMo toolkit, CUDA only
   - Requires: `pip install nemo_toolkit['asr']`

### GUI Player

**File:** `src/bhajan/gui_player.py`

- Uses **tkinter** for UI (built into Python)
- Uses **pygame** for audio playback (added to deps)
- Features:
  - Word-by-word highlighting synchronized to audio
  - Progress bar with seek
  - Space = play/pause, arrows = seek
  - Large 32px font for lyrics

**Launch:** `bhajan "URL" --gui`

The GUI skips video rendering entirely - much faster than ffmpeg encoding.

### Subtitle Rendering

**ASS Style Configuration** (`config.py`, `stages/subtitles.py`):

Key settings for readable karaoke:
```python
DEFAULT_FONT_SIZE = 96  # Large!
DEFAULT_OUTLINE_WIDTH = 4
DEFAULT_OUTLINE_COLOR = "&H00FFFFFF"  # White outline
```

**Alignment:** 5 = center-middle (was 2 = bottom-center)
**Margins:** Reduced from 250 to 50 pixels

The ffmpeg filter in `stages/render.py` forces these styles:
```python
force_style='FontSize=96,Alignment=5,...'
```

### Intermediate File Cleanup

Default changed from `keep_intermediate=True` to `False`:
- Without `--keep-intermediate`: cleans up source/, stems/, transcript/, subtitles/
- With `--keep-intermediate`: retains all for debugging

Cleanup happens at end of `pipeline.run()` if not skipped.

## Common Issues & Solutions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rithamnatani/bhajan](https://github.com/rithamnatani/bhajan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
