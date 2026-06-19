---
trigger: always_on
description: Tighten a long recording — remove silences, fillers, mistakes, and dead air while preserving laughs and comedic pauses. Use when the user pastes a video and says "cut this", "tighten this video", "remove silences", "strip ums", "clean this up", or any variant of "make this video shorter without losing the good parts". Fast (proxy + hardware decode + trim/concat) — runs in seconds, not minutes, on M-series Macs.
---


# Cut Video

Tighten a long-form recording **aggressively**: remove silences, fillers, hedges, weak transitions, false starts, and repetitions. Preserves laughs and comedic pauses. Outputs a cleaned MP4 ready to drop into CapCut for layouts/zooms/memes.

> **This skill is more accurate than cutting from Whisper alone.** Cuts are driven by **Montreal Forced Aligner (MFA)** word boundaries (~10–20ms precision, with true inter-word silences as explicit intervals), not Whisper timestamps (±100–300ms, with pauses embedded inside word durations). Whisper is used only to produce the transcript *text* that MFA aligns to the audio. The result: tighter cuts, no clipped word onsets/tails, and reliable silence detection.

**Style target** (calibrated from `AI mogging my dad.mp4`):
- ~1 cut every 1.3–1.5 seconds (median cut duration ~1.1s)
- ~40–60% total runtime reduction from raw
- Cuts happen mid-sentence, not just between sentences
- Word-by-word tightening — fillers, hedges, and weak openers get sliced

## Inputs

- Video file path (the user will provide; otherwise ask)
- Optional: tone preference (`aggressive` / `balanced` / `sentimental` / `documentary`) — default: `aggressive`

## Working directory

`/tmp/cut-video/<basename>/` — mkdir at start, leave artifacts for debugging.

---

## Step 0 — Make a proxy (the single biggest speed move)

If the source is HEVC, >500MB, or 4K+, transcode to a 1080p H.264 working copy FIRST. Every subsequent step runs against the proxy, not the source.

```bash
ffmpeg -y -hwaccel videotoolbox -i "$SRC" \
  -vf scale=1920:1080 \
  -c:v libx264 -preset fast -crf 20 -pix_fmt yuv420p \
  -c:a aac -b:a 192k \
  /tmp/cut-video/$NAME/proxy.mp4
```

**Critical flags:**
- `-hwaccel videotoolbox` on the INPUT — hardware-decodes HEVC, ~5–10× faster on Apple Silicon. Skip this and you'll wait minutes instead of seconds.
- `-preset fast` — the libx264 default is `medium`, which is ~3× slower for no useful quality gain on a working copy.

Hardware encode alternative (even faster on M-series, slightly larger file):
```bash
-c:v h264_videotoolbox -b:v 8M
```

## Step 1 — Get the transcript TEXT (whisper, as input for MFA)

MFA is a **forced aligner, not a transcriber** — it needs a transcript to align. Whisper's only job here is to produce that text; **whisper word timestamps are NOT used for cutting** (they're ±100–300ms off and turbo embeds pauses inside word durations). MFA (Step 1.5) supplies all timing.

```bash
ffmpeg -y -hwaccel videotoolbox -i /tmp/cut-video/$NAME/proxy.mp4 \
  -vn -ac 1 -ar 16000 /tmp/cut-video/$NAME/audio.wav
whisper /tmp/cut-video/$NAME/audio.wav \
  --model tiny.en --word_timestamps True --output_format json \
  --output_dir /tmp/cut-video/$NAME --language en
```

Why `tiny.en`: ~10× faster than `small.en`, and since we only need the words (not the timings), accuracy is plenty for English. For non-English use `--model base` and drop `--language`. We keep `--word_timestamps True` only so whisper times survive as a cross-check/fallback (Step 1.5 caveats, 2a-legacy) — never as the primary cut source.

## Step 1.5 — Align the transcript to the audio with MFA (Montreal Forced Aligner) — the timing engine

This is the spine of the skill. **MFA forced-aligns the whisper transcript text to the audio** and returns word boundaries at ~10–20ms precision, with true inter-word silences as explicit empty intervals. **All cut decisions (gaps, fillers, false starts, retakes) use MFA word times.**

**Auto-install MFA (the skill does this itself — don't make the user do it).** Run this idempotent bootstrap at the start of every run; it's a no-op once everything's present (a few seconds), and a one-time ~2–3 min install on a fresh machine. Tell the user "installing MFA (one-time)…" only when it actually installs something.

```bash
# 1. Ensure conda exists (MFA's only supported install path). Install miniforge via brew if missing.
if ! command -v conda >/dev/null 2>&1; then
  echo "conda not found — installing miniforge (one-time)…"
  brew install --cask miniforge || brew install miniforge
  # make conda available in this shell
  eval "$("$(brew --prefix)/bin/conda" shell.bash hook 2>/dev/null || conda shell.bash hook)"
fi
source "$(conda info --base)/etc/profile.d/conda.sh"

# 2. Ensure the mfa env exists with MFA installed
if ! conda env list | grep -q '^mfa\b'; then
  echo "creating mfa conda env (one-time)…"
  conda create -n mfa -c conda-forge montreal-forced-aligner -y
fi

# 3. Ensure the English acoustic model + dictionary are downloaded (idempotent; skips if present)
conda run -n mfa mfa model download acoustic english_mfa  2>/dev/null || true
conda run -n mfa mfa model download dictionary english_mfa 2>/dev/null || true
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [louisedesadeleer/cut-video](https://github.com/louisedesadeleer/cut-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
