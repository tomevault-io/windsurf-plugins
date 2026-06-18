---
trigger: always_on
description: ffmpeg recipes and best practices: convert, concatenate, merge, resize, compress, GIF creation, audio extraction, subtitles, optimize for social platforms.
---


# ffmpeg Usage

## Overview

This Skill provides comprehensive video and audio processing capabilities using ffmpeg. It includes battle-tested commands and workflows for common multimedia tasks, platform-specific optimizations, and best practices for quality and file size management.

**Version:** 1.0.0
**Requirements:** ffmpeg >= 4.0, ffprobe (optional but recommended)

Claude should use this Skill whenever users mention video or audio processing tasks, format conversions, social media optimization, or multimedia editing.

## When to Apply

Use this Skill when the user requests:
- Video format conversion (MP4, WebM, MOV, etc.)
- Resolution scaling or aspect ratio changes
- GIF creation from videos
- Audio extraction or format conversion
- Video editing (trim, merge, speed adjustment, rotation)
- Subtitle processing (burn-in, soft subs, extraction)
- Video compression or optimization
- Platform-specific formatting (YouTube, Instagram, TikTok, Twitter)
- Thumbnail or frame extraction
- Batch processing of video/audio files

## Prerequisites

Before using this skill, ensure ffmpeg is installed:

```bash
# macOS
brew install ffmpeg

# Ubuntu/Debian
sudo apt-get install ffmpeg

# Windows (with Chocolatey)
choco install ffmpeg
```

Verify installation:
```bash
ffmpeg -version
```

## Supported Operations

### 1. Format Conversion

Convert between video formats with optimized settings.

**MP4 to WebM:**
```bash
ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 30 -b:v 0 -c:a libopus output.webm
```

**MOV to MP4:**
```bash
ffmpeg -i input.mov -c:v libx264 -c:a aac -strict experimental output.mp4
```

**Any to MP4 (universal compatibility):**
```bash
ffmpeg -i input.* -c:v libx264 -preset medium -crf 23 -c:a aac -b:a 128k output.mp4
```

### 2. Resolution Adjustment

Resize videos while maintaining aspect ratio.

**Scale to 720p:**
```bash
ffmpeg -i input.mp4 -vf scale=-1:720 -c:a copy output_720p.mp4
```

**Scale to 1080p:**
```bash
ffmpeg -i input.mp4 -vf scale=-1:1080 -c:a copy output_1080p.mp4
```

**Scale to specific width (auto height):**
```bash
ffmpeg -i input.mp4 -vf scale=1280:-1 -c:a copy output.mp4
```

**Scale with padding (letterbox):**
```bash
ffmpeg -i input.mp4 -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2" output.mp4
```

### 3. GIF Creation

Create high-quality GIFs from videos with optimized file size.

**Basic GIF (10 fps):**
```bash
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos" output.gif
```

**High-quality GIF with palette:**
```bash
# Generate palette
ffmpeg -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos,palettegen" palette.png

# Create GIF using palette
ffmpeg -i input.mp4 -i palette.png -filter_complex "fps=10,scale=480:-1:flags=lanczos[x];[x][1:v]paletteuse" output.gif
```

**GIF from specific time range:**
```bash
ffmpeg -ss 00:00:10 -t 5 -i input.mp4 -vf "fps=10,scale=480:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" output.gif
```

### 4. Audio Operations

Extract, convert, and manipulate audio streams.

**Extract audio to MP3:**
```bash
ffmpeg -i input.mp4 -vn -acodec libmp3lame -q:a 2 output.mp3
```

**Extract audio to WAV:**
```bash
ffmpeg -i input.mp4 -vn -acodec pcm_s16le -ar 44100 -ac 2 output.wav
```

**Convert audio format:**
```bash
ffmpeg -i input.wav -c:a aac -b:a 192k output.m4a
```

**Add background music:**
```bash
ffmpeg -i video.mp4 -i music.mp3 -c:v copy -c:a aac -map 0:v:0 -map 1:a:0 -shortest output.mp4
```

**Mix audio (overlay):**
```bash
ffmpeg -i video.mp4 -i music.mp3 -filter_complex "[0:a][1:a]amix=inputs=2:duration=first" -c:v copy output.mp4
```

### 5. Video Editing

Trim, concatenate, and modify videos.

**Trim video:**
```bash
# From 10s to 30s
ffmpeg -i input.mp4 -ss 00:00:10 -to 00:00:30 -c copy output.mp4

# Duration-based (10s starting from 5s)
ffmpeg -i input.mp4 -ss 00:00:05 -t 10 -c copy output.mp4
```

**Concatenate videos:**

Choose method based on format and compatibility:

**Method 1: Concat Protocol (Preferred - No temporary files needed)**
```bash
# For MPEG formats: .ts, .mpg, .mpeg, .mp3, .aac, etc.
# Direct concatenation without creating list file
ffmpeg -i "concat:file1.mp3|file2.mp3|file3.mp3" -c copy output.mp3
ffmpeg -i "concat:video1.ts|video2.ts|video3.ts" -c copy output.ts

# Works with: TS, MPEG-1, MPEG-2, MP3, AAC
# Does NOT work with: MP4, MOV, MKV (use Method 2 instead)
```

**Method 2: Concat Demuxer (For MP4, MOV, MKV)**
```bash
# Use process substitution to avoid temporary files
ffmpeg -f concat -safe 0 -i <(printf "file '%s'\n" video1.mp4 video2.mp4 video3.mp4) -c copy output.mp4

# If shell doesn't support process substitution:
printf "file '%s'\n" video1.mp4 video2.mp4 video3.mp4 > list.txt
ffmpeg -f concat -safe 0 -i list.txt -c copy output.mp4
rm list.txt
```

**Method 3: Concat Filter (When re-encoding is acceptable)**
```bash
# Use when videos have different codecs/resolutions
ffmpeg -i video1.mp4 -i video2.mp4 -i video3.mp4 \
  -filter_complex "[0:v][0:a][1:v][1:a][2:v][2:a]concat=n=3:v=1:a=1[v][a]" \
  -map "[v]" -map "[a]" output.mp4
```

**Format Decision Guide:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ychoi-kr/claude-ffmpeg-skill](https://github.com/ychoi-kr/claude-ffmpeg-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
