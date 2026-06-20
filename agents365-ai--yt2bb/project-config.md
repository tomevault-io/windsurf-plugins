---
trigger: always_on
description: Use when the user wants to repurpose a YouTube video for Bilibili, add bilingual (English-Chinese) subtitles to a video, or create hardcoded subtitle versions for Chinese platforms.
---


# yt2bb — YouTube to Bilibili Video Repurposing

## Overview

Six-step pipeline: download → transcribe → translate → merge → burn subtitles → generate publish info. Produces a video with hardcoded bilingual (EN/ZH) subtitles and a `publish_info.md` with Bilibili upload metadata.

## When to Use

- User provides a YouTube URL (single video or playlist) and wants a Bilibili-ready version
- User needs bilingual EN-ZH subtitles burned into video
- User wants to repurpose English video content for Chinese audience

## Quick Reference

| Step | Tool | Command | Output |
|------|------|---------|--------|
| 0. Update | `git` | Auto-check for skill updates | — |
| 1. Download | `yt-dlp` | `yt-dlp --cookies-from-browser chrome -f ... -o ...` | `{slug}.mp4` |
| 2. Transcribe | `whisper`* | `srt_utils.py check-whisper` then transcribe | `{slug}_{lang}.srt` |
| 2.5 Validate | `srt_utils.py` | `srt_utils.py validate / fix` | `{slug}_{lang}.srt` (fixed) |
| 3. Translate | AI | SRT-aware batch translation | `{slug}_zh.srt` |
| 4. Merge | `srt_utils.py` | `srt_utils.py merge ...` | `{slug}_bilingual.srt` |
| 4.5 Style | `srt_utils.py` | `srt_utils.py to_ass --preset netflix\|clean\|glow` | `{slug}_bilingual.ass` |
| 5. Burn | `ffmpeg` | `ffmpeg -c:v libx264 -vf ass=...` | `{slug}_bilingual.mp4` |
| 6. Publish | AI | Analyze content, generate metadata | `publish_info.md` |

## Update check

Throttle to one check per 24 hours per installation; never mutate the skill directory without explicit user consent. `SKILL_DIR` resolved here is reused by later pipeline steps for script paths.

1. If `<this-skill-dir>/.last_update` exists and is less than 24 hours old, skip this step entirely.

2. Otherwise, fetch the latest tag from upstream:

   ```bash
   git -C <this-skill-dir> ls-remote --tags origin 'v*' 2>/dev/null \
     | awk '{print $2}' | sed 's|refs/tags/||' \
     | sort -V | tail -1
   ```

3. Compare with this skill's `metadata.version` from the frontmatter. If the upstream tag is strictly newer (semver), tell the user one line and ask:

   > "A newer version of this skill is available: vX.Y.Z → vA.B.C. Want me to `git pull`?"

   If they say yes, run `git -C <this-skill-dir> pull --ff-only`. Refresh `.last_update` either way so the prompt doesn't repeat for 24 hours.

4. If upstream is the same or older, refresh `.last_update` silently and continue.

5. On any failure (offline, not a git checkout — e.g. ClawHub-installed copy, read-only path, no permission), swallow the error silently and continue with the user's task. Do not mention the failure.

Resolve `SKILL_DIR` for use by later pipeline steps:

```bash
# Find skill directory (works across Claude Code, OpenClaw, Hermes, Pi)
SKILL_DIR="$(find ~/.claude/skills ~/.openclaw/skills ~/.hermes/skills ~/.pi/agent/skills ~/.agents/skills ~/myagents/myskills -maxdepth 2 -name 'yt2bb' -type d 2>/dev/null | head -1)"
```

## Pipeline Details

### Step 1: Download

**Single video:**

```bash
slug="video-name"  # or: slug=$(python3 "$SKILL_DIR/srt_utils.py" slugify "Video Title")
mkdir -p "${slug}"
yt-dlp --cookies-from-browser chrome \
  -f "bv*[ext=mp4]+ba[ext=m4a]/b[ext=mp4]" \
  -o "${slug}/${slug}.mp4" "https://www.youtube.com/watch?v=VIDEO_ID"
```

**Playlist / series:**

```bash
yt-dlp --cookies-from-browser chrome \
  -f "bv*[ext=mp4]+ba[ext=m4a]/b[ext=mp4]" \
  -o "%(playlist_index)03d-%(title)s/%(playlist_index)03d-%(title)s.mp4" \
  "https://www.youtube.com/playlist?list=PLAYLIST_ID"
```

After downloading, rename each folder to a clean slug and run Steps 2–6 for each video sequentially.

- `-f "bv*[ext=mp4]+ba[ext=m4a]/b[ext=mp4]"`: ensure mp4 output, avoid webm
- `%(playlist_index)03d`: zero-padded index to preserve playlist order
- If `--cookies-from-browser` fails, export cookies first — see Troubleshooting

### Step 2: Transcribe

**First run the environment check** to detect your platform and get a tailored whisper command:

```bash
python3 "$SKILL_DIR/srt_utils.py" check-whisper
```

This auto-detects OS, GPU (CUDA/Metal/CPU), memory, and installed backends, then recommends the best backend + model for your hardware. If memory detection is unavailable, it falls back conservatively instead of assuming a low-memory machine. Use the command it prints.

**Manual fallback** (openai-whisper, works everywhere):

```bash
src_lang="en"      # Change to ja/ko/es/etc. based on source video
whisper_model="medium"  # check-whisper recommends the best model for your hardware
whisper "${slug}/${slug}.mp4" \
  --model "$whisper_model" \
  --language "$src_lang" \
  --word_timestamps True \
  --condition_on_previous_text False \
  --output_format srt \
  --max_line_width 40 --max_line_count 1 \
  --output_dir "${slug}"
mv "${slug}/${slug}.srt" "${slug}/${slug}_${src_lang}.srt"
```

**Supported backends:**

| Backend | Best for | Install |
|---------|----------|---------|
| `mlx-whisper` | macOS Apple Silicon (fastest) | `pip install mlx-whisper` |
| `whisper-ctranslate2` | Windows/Linux CUDA, or CPU (~4x faster) | `pip install whisper-ctranslate2` |
| `openai-whisper` | Universal fallback | `pip install openai-whisper` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Agents365-ai/yt2bb](https://github.com/Agents365-ai/yt2bb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
