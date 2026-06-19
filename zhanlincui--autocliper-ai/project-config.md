---
trigger: always_on
description: |
---


# AutoCliper.AI — Agent Skill

Convert one long YouTube video into 5–15 short clips with Chinese packaging and hard-burned subtitles.

This document is the **single source of truth** for any AI agent executing the AutoCliper pipeline. Follow every numbered step in order. Do not skip steps. Do not guess file names — inspect the filesystem after each write operation.

---

## 0. Pre-flight Checks

Before doing anything else, verify the environment. Report the **exact blocker** and stop if any check fails.

| Check | Command | Pass Condition |
|-------|---------|----------------|
| yt-dlp installed | `which yt-dlp` | returns a path |
| ffmpeg installed | `which ffmpeg` | returns a path; if missing, try `python3 -c "import imageio_ffmpeg; print(imageio_ffmpeg.get_ffmpeg_exe())"` |
| Python 3.9+ | `python3 --version` | >= 3.9 |
| Skill repo available | check that `corekit/` directory exists relative to the skill root | `corekit/__init__.py` is present |

Set the environment for all subsequent commands:

```bash
export PYTHONPATH="<path-to-AutoCliper.AI-repo>"
```

All `python3 -m corekit.*` commands below assume this `PYTHONPATH` is set.

---

## 1. Create Workspace

Create the output directory structure for this video. Derive `<slug>` from the video title or ID — lowercase, hyphens, no spaces, no special characters.

```bash
mkdir -p "studio/<slug>/intake"
mkdir -p "studio/<slug>/intel"
mkdir -p "studio/<slug>/exports"
```

Expected result:

```
studio/<slug>/
├── intake/       ← raw video + subtitles land here
├── intel/        ← analysis artifacts land here
└── exports/      ← per-clip folders land here
```

---

## 2. Fetch Source Assets

Download the video and subtitles from YouTube:

```bash
python3 -m corekit.fetch_source "<YouTube_URL>" "studio/<slug>/intake"
```

**What happens inside**: the downloader tries English subtitles first (`en`, `en-US`, `en-orig`), falls back to Chinese (`zh-Hans`, `zh-CN`, `zh`). It uses `--cookies-from-browser chrome` for authenticated access.

**After the command finishes**, list the intake folder and identify:

1. The `.mp4` video file (may contain the video title and ID in the filename)
2. The `.srt` subtitle file (may have a language tag like `.en.srt` or `.zh-Hans.srt`)
3. Any sidecar files (`.ytdl`, `.jpg`, etc.) — note them but they are not needed

**If the download fails**:
- Cookie error → tell the user to refresh their Chrome YouTube login, then retry
- No subtitles found → report "no subtitles available for this video" and stop
- Video download error → retry once with a different format; if still failing, report the error

**Record** the exact filenames for the next steps. Do not guess or hardcode names.

---

## 3. Parse Subtitles into JSON

Convert the SRT subtitle file into structured JSON for easier analysis:

```bash
python3 -m corekit.subtitle_to_json \
  "studio/<slug>/intake/<exact-srt-filename>" \
  "studio/<slug>/intel/transcript.json"
```

**Verify** the output file exists and contains an array of cue objects. Each cue has:

```json
{
  "index": 1,
  "start": "00:00:01,234",
  "end": "00:00:03,456",
  "start_seconds": 1.234,
  "end_seconds": 3.456,
  "text": "the spoken content"
}
```

If the transcript is empty or has fewer than 10 cues, report "transcript too short or corrupted" and stop.

---

## 4. Analyze Transcript and Propose Candidates

This is the most important intellectual step. Read the following files **before** starting analysis:

1. **[playbooks/clip-contract.md](playbooks/clip-contract.md)** — defines the exact JSON schema for `selected_clips.json`
2. **[playbooks/content-analysis-playbook.md](playbooks/content-analysis-playbook.md)** — defines the multi-pass analysis method, scoring rubric, and review formatting

### 4a. Multi-Pass Analysis

Do NOT try to pick clips in a single pass. Follow this sequence:

**Pass 1 — Skim and flag**: Read through `transcript.json` looking for stretches that contain memorable, information-dense, opinionated, counterintuitive, or emotionally sharp content. Flag generously — it is better to flag too many than too few.

**Pass 2 — Boundary refinement**: For each flagged stretch, re-read the surrounding cues (5–10 cues before and after). Choose a clean start and end:
- The **start** must land on a line that hooks the viewer within the first 3 seconds of the clip
- The **end** must land *after* the speaker has finished the thought — never mid-sentence
- If a promising moment needs a few seconds of context, extend the start earlier
- If a thought trails off, extend the end until it resolves

**Pass 3 — Completeness check**: For each candidate, verify:
- [ ] It is 20–180 seconds long (can stretch to 3 min if payoff justifies it)
- [ ] It contains exactly one clear idea
- [ ] The opening line is a hook, not filler
- [ ] The closing line resolves the thought
- [ ] It is understandable without watching the rest of the video
- [ ] It is NOT a greeting, sponsor read, Q&A preamble, or long digression

**Pass 4 — Score and rank**: Score each candidate on four dimensions (1–5):
- `hook` — how compelling is the opening in the first 3 seconds?
- `clarity` — is it one clean idea with low ambiguity?
- `standalone` — can it stand alone without prior context?
- `payoff` — is the ending useful, memorable, or share-worthy?


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ZhanlinCui/AutoCliper.AI](https://github.com/ZhanlinCui/AutoCliper.AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
