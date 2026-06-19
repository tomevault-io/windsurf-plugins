---
trigger: always_on
description: Make polished videos with ffmpeg: a production pipeline plus vetted recipes for conversion, scaling, compositing, GIFs, captions, audio/voiceover, motion, and platform-ready export — for demos, ads, tutorials, launches, social clips, explainers, and pitches.
---


# ffmpeg Usage

## Overview

Two things in one skill: a **command library** of vetted ffmpeg recipes for everyday media jobs, and a **production pipeline** for making videos that actually look intentional. Treat ffmpeg as the final compositor and encoder — the renderer — and bring in helper tools for the parts ffmpeg is clumsy at (asset design, animation, captions, 3D). Use it whenever a request touches media: converting, resizing, building GIFs, pulling audio, editing, or producing a finished video for marketing, demos, tutorials, launches, social, explainers, app-store previews, or pitches.

**Version:** 2.0.0
**Needs:** ffmpeg ≥ 4.0; `ffprobe` recommended. Other tools (below) are optional and used only when they earn their place.

## When to apply

Trigger on requests to:

- Convert, rescale, or re-fit aspect ratio
- Build a GIF, extract or convert audio
- Edit clips — trim, merge, speed, rotate
- Work with subtitles or captions — burn-in, soft, extract, generate
- Compress, web-optimise, or format for a platform (YouTube, Instagram, TikTok, X)
- Grab thumbnails or frames; batch-process a folder
- **Plan and produce a finished video** — story beats, scene composition, animated text, framing, voiceover/music, and a repeatable render pipeline

## Prerequisites

```bash
brew install ffmpeg              # macOS
sudo apt-get install ffmpeg      # Debian/Ubuntu
choco install ffmpeg             # Windows
ffmpeg -version                  # confirm
```

---

# Command library

## 1. Format conversion

**MP4 → WebM:**
```bash
ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libopus output.webm
```

**MOV → MP4:**
```bash
ffmpeg -i input.mov -c:v libx264 -c:a aac -strict experimental output.mp4
```

**Anything → MP4 (broad compatibility):**
```bash
ffmpeg -i input.* -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k output.mp4
```

## 2. Resolution

Rescale while preserving aspect ratio.

**To 720p / 1080p:**
```bash
ffmpeg -i input.mp4 -vf scale=-1:720 -c:a copy output_720p.mp4
ffmpeg -i input.mp4 -vf scale=-1:1080 -c:a copy output_1080p.mp4
```

**Fixed width, auto height:**
```bash
ffmpeg -i input.mp4 -vf scale=1280:-1 -c:a copy output.mp4
```

**Fit with letterbox padding:**
```bash
ffmpeg -i input.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" output.mp4
```

## 3. GIFs

Palette generation is what keeps GIFs sharp and small.

**Quick (10 fps):**
```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos" output.gif
```

**Two-pass with palette:**
```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos,palettegen" palette.png
ffmpeg -i input.mp4 -i palette.png -filter_complex "fps=10,scale=480:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif
```

**From a time range (single pass):**
```bash
ffmpeg -ss 00:00:10 -t 5 -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" output.gif
```

## 4. Audio

**Extract to MP3 / WAV:**
```bash
ffmpeg -i input.mp4 -vn -acodec libmp3lame -q:a 2 output.mp3
ffmpeg -i input.mp4 -vn -acodec pcm_s16le -ar 44100 -ac 2 output.wav
```

**Convert format:**
```bash
ffmpeg -i input.wav -c:a aac -b:a 192k output.m4a
```

**Replace audio with a track / mix two together:**
```bash
ffmpeg -i video.mp4 -i music.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 -shortest output.mp4
ffmpeg -i video.mp4 -i music.mp3 -filter_complex "[0:a][1:a]amix=inputs=2:duration=first" -c:v copy output.mp4
```

## 5. Editing

**Trim (stream-copy, no re-encode):**
```bash
ffmpeg -i input.mp4 -ss 00:00:10 -to 00:00:30 -c copy output.mp4   # 10s–30s
ffmpeg -i input.mp4 -ss 00:00:05 -t 10 -c copy output.mp4          # 10s from 5s
```

**Concatenate** — pick by container:

```bash
# Protocol — TS/MPEG/MP3/AAC, no list file needed:
ffmpeg -i "concat:file1.mp3|file2.mp3|file3.mp3" -c copy output.mp3

# Demuxer — MP4/MOV/MKV, via process substitution:
ffmpeg -f concat -safe 0 -i <(printf "file '%s'\n" video1.mp4 video2.mp4 video3.mp4) -c copy output.mp4
# Fallback if the shell lacks process substitution:
printf "file '%s'\n" video1.mp4 video2.mp4 video3.mp4 > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4 && rm list.txt

# Filter — mismatched codecs/resolutions, re-encodes:
ffmpeg -i video1.mp4 -i video2.mp4 -i video3.mp4 \
  -filter_complex "[0:v][0:a][1:v][1:a][2:v][2:a]concat=n=3:v=1:a=1[v][a]" \
  -map "[v]" -map "[a]" output.mp4
```

Decision: MP3/AAC/TS/MPEG → protocol; MP4/MOV/MKV → demuxer; differing codecs or sizes → filter.

**Speed / rotate:**
```bash
ffmpeg -i input.mp4 -filter:v "setpts=0.5*PTS" -an output.mp4   # 2x
ffmpeg -i input.mp4 -filter:v "setpts=2.0*PTS" output.mp4       # 0.5x
ffmpeg -i input.mp4 -vf "transpose=1" output.mp4               # 90° CW
ffmpeg -i input.mp4 -vf "transpose=2,transpose=2" output.mp4   # 180°
```

## 6. Subtitles

```bash
ffmpeg -i input.mp4 -vf subtitles=subtitles.srt output.mp4       # burn in
ffmpeg -i input.mp4 -i subtitles.srt -c copy -c:s mov_text output.mp4   # soft subs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muhammaddadu/ffmpeg-skill](https://github.com/muhammaddadu/ffmpeg-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
