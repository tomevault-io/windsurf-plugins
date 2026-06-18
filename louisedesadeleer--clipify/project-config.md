---
trigger: always_on
description: Find the funniest moments in a video, cut them as standalone clips, optionally reformat 16:9 → 9:16 (face-pan or split-screen), and burn opus-style word-by-word captions. Use when the user mentions "clipify," "cut clips from this video," "make shorts from this," "find funny moments," "reframe to 9:16," "vertical clips," or pastes a video file path and wants social-ready cuts.
---


# Clipify

Find the funniest moments in a video, cut them as standalone clips, optionally reformat 16:9 → 9:16 (face-pan or split-screen), and burn opus-style word-by-word captions.

## Inputs

- A video file path (the user will provide it; otherwise ask)
- Optional: requested format (9:16, 16:9, 1:1) — if not given, ask after candidates are picked
- Optional: subtitle style preference — if not given, ask before captioning

## Tooling (use only the fastest path)

- **Whisper:** `whisper --model tiny.en --word_timestamps True --output_format json` (≈10× faster than `small.en`; quality fine for English). For non-English: `--model base` (drop `--language`).
- **ffmpeg:** add `-hwaccel videotoolbox` for decode and `-preset ultrafast` for renders. Use `-c:v libx264 -crf 20` for the final master.
- **Numpy** for audio alignment (FFT cross-correlation). No scipy/cv2 needed.
- **Scripts:** `<skill-dir>/scripts/` (where `<skill-dir>` is the directory containing this SKILL.md — typically `~/.claude/skills/clipify/`)
  - `analyze.py` — speaker timeline from two ROI motion files
  - `build_pan.py` — ffmpeg crop x-expression with hard cuts
  - `build_ass.py` — opus-style ASS captions from whisper JSON
  - `audio_align.py` — find offset of a sub-clip in a longer source

Working dir: `/tmp/clipify/` (mkdir at start, leave artifacts for debugging).

---

## Workflow

### Step 1 — Find the funniest parts

```bash
mkdir -p /tmp/clipify
ffmpeg -y -hwaccel videotoolbox -i "$VIDEO" -vn -ac 1 -ar 16000 /tmp/clipify/audio.wav
whisper /tmp/clipify/audio.wav --model tiny.en --word_timestamps True --output_format json --output_dir /tmp/clipify --language en
```

Read the resulting JSON (or `.txt`) and pick 3–5 candidate clips. Funny signals to scan for:

- **Punchlines and reactions:** words like "what", "wait", "no way", laughter, "haha", swearing
- **Reversal moments:** setup question → unexpected answer
- **Awkward pauses:** Whisper segment with long gap, or filler ("uh", "um")
- **Self-roast / quotable one-liners:** short declarative sentences that stand alone
- **Audio peaks:** detect via `ffmpeg -af volumedetect` or look for rapid back-and-forth (alternating short Whisper segments)

For each candidate, propose: `[start, end, why-it's-funny, suggested title]`. Aim for 10–25s clips. Show the list and let the user confirm/pick.

### Step 2 — Trim each chosen clip

```bash
ffmpeg -y -ss "$START" -t "$DURATION" -i "$VIDEO" -c copy /tmp/clipify/clip_$N.mp4
```

(Use `-c copy` for instant trim. Re-encode only if cuts must be frame-accurate.)

### Step 3 — Decide the output format

Ask the user (skip if they already specified): "9:16 (TikTok / Reels), 16:9 (YouTube), or 1:1 (Insta feed)?"

### Step 4 — If 16:9 → 9:16: pan-between-faces vs split-screen

Detect source aspect with `ffprobe`. If source is 16:9 and target is 9:16, ask:

> "Two options: **(a) hard-cut pan** that follows whoever is speaking (single face on screen at a time), or **(b) split-screen** stack with both faces visible. Which do you want?"

Skip the question if there's only one face (single-talker clip). For single-talker, just center-crop.

#### Step 4a — Pan-between-faces (recommended for fast-cut talking-head dialogue)

1. **Locate the two face ROIs.** Sample one frame: `ffmpeg -ss <middle> -i <clip> -frames:v 1 /tmp/clipify/probe.jpg`. Read it. Eyeball each face's mouth+chin area as `x,y,w,h` in the source's pixel space. (No cv2 needed — camera is static within a clip; one frame is enough.) Verify by drawing boxes:

   ```bash
   ffmpeg -i probe.jpg -vf "drawbox=x=$LX:y=$LY:w=$LW:h=$LH:color=cyan@0.9:t=4,drawbox=x=$RX:y=$RY:w=$RW:h=$RH:color=magenta@0.9:t=4" verify.jpg
   ```

   Iterate **at most twice**. Boxes should cover mouth + chin and avoid hands/mics. Don't over-tune — frame differencing is forgiving.

2. **Extract per-frame motion energy in each ROI:**

   ```bash
   ffmpeg -y -i clip.mp4 -filter_complex "
   [0:v]split=2[a][b];
   [a]crop=$LW:$LH:$LX:$LY,format=gray,tblend=all_mode=difference,signalstats,metadata=mode=print:key=lavfi.signalstats.YAVG:file=/tmp/clipify/L.txt[la];
   [b]crop=$RW:$RH:$RX:$RY,format=gray,tblend=all_mode=difference,signalstats,metadata=mode=print:key=lavfi.signalstats.YAVG:file=/tmp/clipify/R.txt[ra]
   " -map "[la]" -f null - -map "[ra]" -f null -
   ```

3. **Build speaker timeline** (min dwell 1.0s — short interjections merge into the prior speaker):

   ```bash
   python3 <skill-dir>/scripts/analyze.py /tmp/clipify/L.txt /tmp/clipify/R.txt 1.0 > /tmp/clipify/segments.json
   ```

4. **Pick pan x-coordinates** for a 9:16 vertical strip from the source. With source W=1920 and target W=1080, crop strip width = 608.
   - LEFT_X = `face_left_center_x - 304` (clamp ≥ 0)
   - RIGHT_X = `face_right_center_x - 304` (clamp ≤ source_W - 608)

5. **Generate the hard-cut x expression and render:**

   ```bash
   EXPR=$(python3 <skill-dir>/scripts/build_pan.py /tmp/clipify/segments.json $LEFT_X $RIGHT_X)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [louisedesadeleer/clipify](https://github.com/louisedesadeleer/clipify) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
