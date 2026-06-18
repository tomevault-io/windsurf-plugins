---
trigger: always_on
description: Convert a YouTube URL into a markdown file that combines a faster-whisper transcript with Claude-vision descriptions of keyframes (on-screen text, chart data, slide contents). Output is written in the source language of the video. Use when the user gives a YouTube link and asks for a transcript, notes, a markdown file, or on-screen visual information captured — especially for talk-style videos with slides or charts.
---


# youtube-to-markdown

Convert a YouTube URL into a single markdown file containing:
1. Video metadata
2. A table of keyframes (thumbnail + one-line caption)
3. Per-keyframe detailed visual description (OCR of on-screen text)
4. The full transcript with timestamps, keyframe markers inlined at their moments

**Output language matches the source language of the video** (detected by whisper).

## Inputs

Positional: `<url>`

Flags (optional):
- `--slug <name>` — filename slug. Default: auto-derived as `<YYYY-MM-DD>-<title-slug>` from metadata (publish date prepended; CJK preserved; unsafe chars stripped; title portion capped at 60 chars).
- `--lang <code>` — force whisper language (e.g. `zh`, `en`, `ja`). Default: auto-detect.
- `--interval <sec>` — keyframe sampling interval. Default: `60`.
- `--out-dir <path>` — root directory for per-video folders. Default: `./raw/youtube` (CWD-relative).
- `--img-dir <path>` — override images subfolder. Default: `<out-dir>/<slug>/images`.

**Default output layout** (with no flags): `./raw/youtube/<slug>/<slug>.md` and `./raw/youtube/<slug>/images/*.jpg`. Each video is a self-contained bundle.

## Pipeline — execute these steps in order

### 1. Prep

Derive a staging dir: `/tmp/ytmd-<video_id>` (extract the `v=` param from the URL; if the URL is `youtu.be/X`, that's the id).

Run:
```bash
~/.claude/skills/youtube-to-markdown/scripts/prep.sh <url> /tmp/ytmd-<video_id> [--interval N] [--lang X]
```

If it exits non-zero, surface the stderr tail and stop. Common failures:
- Missing `~/.local/share/whisper-venv` → tell user to create the venv and `pip install faster-whisper`
- `yt-dlp` failure → URL invalid or network issue
- `ffmpeg` missing → `apt install ffmpeg`

After success the staging dir contains: `metadata.json`, `audio.mp3`, `video.mp4`, `transcript.json`, `frames/t0001.jpg ...`, `frames.json`.

### 2. Read frames

Read `transcript.json` first to learn `info.language` — you will write descriptions in that language.

Then use the `Read` tool on each file in `frames/`. Call Read on **up to 12 frames in a single message** (parallel tool calls) to keep the turn short. If there are more than 12, do it across multiple messages.

For each frame, mentally note:
- What type of content it shows (talking head, chart, slide, code, screenshot, etc.)
- Any on-screen text — transcribe it (prices, axis labels, slide titles, bullet points, table values). This is the high-value part; the transcript already has what the speaker *said*.
- Whether it's a near-duplicate of the previous frame (same slide held across a 60s window)

### 3. Write descriptions.json

Save to `/tmp/ytmd-<video_id>/descriptions.json` a JSON array where each element has:

```json
{
  "index": 1,                          // frame number (matches t0001.jpg)
  "t_sec": 0,                          // (index - 1) * interval
  "keep": true,                        // false for near-duplicates
  "slug": "01-spx-overview",           // short kebab-case label, prefix with index for ordering
  "description": "标普500日K图..."      // 1-3 sentences in the source language
}
```

Rules:
- Mark `keep: false` when the current frame is visually near-identical to the previously kept frame (same slide, same chart with only cursor moved). Aim for roughly 40–70% of frames kept on average for slide-heavy videos.
- `description` must be in the video's source language. Include specific on-screen numbers and text — this is what makes the output useful. Don't just say "a chart is shown" — say *what* the chart shows.
- `slug` should be unique and descriptive. Prefix with a two-digit index so filenames sort chronologically.

### 4. Assemble

Just point `assemble.py` at the staging dir — it derives the slug and all output paths automatically:

```bash
~/.claude/skills/youtube-to-markdown/scripts/assemble.py /tmp/ytmd-<video_id>
```

Only pass flags if the user provided overrides:
```bash
~/.claude/skills/youtube-to-markdown/scripts/assemble.py /tmp/ytmd-<video_id> \
  [--out-dir <path>] [--slug <name>] [--img-dir <path>]
```

It prints a JSON summary on stdout with: `output_md`, `images_dir`, `bundle_dir`, `slug`, detected language, kept/total frame counts, paragraph count. Image references inside the written `.md` are `images/<frame-slug>.jpg` (relative to the `.md`).

### 5. Report

Tell the user in a short message:
- Path to the markdown file
- Path to the images directory
- Detected language
- Kept / total frame count
- Any notable signals (e.g. "transcript is 45 minutes, you may want to skim")

### 6. Cleanup (optional)

Leave `/tmp/ytmd-<video_id>` in place so the user can inspect intermediate artifacts. Only delete if they ask.

## Failure modes and fallbacks

| Symptom | Cause | What to try |
|---|---|---|
| `cuda failed` in transcribe log | GPU busy or no CUDA libs | Script auto-falls back to CPU int8 — warn user this will be slower (10–30 min for a 1h video) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mol2017/youtube-to-markdown](https://github.com/Mol2017/youtube-to-markdown) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
