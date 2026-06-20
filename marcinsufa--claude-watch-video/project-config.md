---
trigger: always_on
description: Make Claude "watch" a video — local file, public URL (yt-dlp), or Jira attachment (fully automated when an Atlassian API token is configured). Extracts keyframes with ffmpeg and transcribes audio with faster-whisper (multilingual), then Claude reads frames + transcript to answer the user's question. Use when the user wants to analyze a screen recording, bug repro video, demo capture, or any video they point at. Triggers - "watch this video", "analyze this video", "what happens in <file>.mp4", "t
---


# /watch-video

Give Claude eyes and ears for a video. Output: a directory of timestamped JPEG frames + a timestamped transcript (granular `.txt` + prose `.md`). Claude `Read`s them as multimodal input and answers grounded in what was on screen and what was said.

## Common mistakes — don't do these

- **Running the full pipeline + reading the full transcript when the user asked a targeted question.** Use `--highlights-prompt "<their question>"` instead — the pipeline runs once, the agent reads only the picked moments, saves ~15k tokens at the answer step.
- **Using `--whisper local` when a hosted key is set.** ~100× slower for no reason. Provider priority below.
- **Reading every stderr progress line.** Each JSON event line is for observability, not for the agent. Read the final JSON block at the bottom of stdout — that's `meta.json` and tells you exactly which files exist.
- **Forgetting to relabel after `--whisper deepgram` / `--whisper whisperx`.** Output has anonymous `**S0**` / `**S1**` tags. If the user's prompt referenced specific people ("what did Alice say about…"), call `scripts/relabel_speakers.py` with inferred names before answering — otherwise the answer reads as *"S0 said…"* which is useless.
- **Posting to Jira without explicit user consent in THIS turn.** `--post-to-jira` is opt-in only; never pass it unless the user just asked "post this to the ticket" in the current message. Same for `confirm=True` in the MCP `post_to_jira` tool.

## Decide before invoking

Parse the user's prompt FIRST, then pick flags. Don't ask clarifying questions — infer.

| User's intent | Flags |
|---|---|
| "summarize", "transcribe", "what's in this video" | (default — full transcript, Claude reads it) |
| Specific question ("why X", "what's the bug", "when does Y happen") | `--highlights-prompt "<their literal question>"` |
| Video > 10 minutes AND specific question | **Mandatory** `--highlights-prompt` (token economy) |
| Multi-speaker (podcast / interview / meeting recording) | `--whisper deepgram` (hosted, ~$0.0043/min) OR `--whisper whisperx` (local + offline, free, needs HF token) |
| Screen recording with on-screen text (UI bug repro, form fields, error toasts) | `--ocr` |
| Bug repro + user just asked to post analysis back | `--highlights-prompt "what is the bug" --post-to-jira` |
| Polish / non-English / mixed content | `--lang pl` (or appropriate ISO) — auto-detect is unreliable on short clips |

**Heuristic:** if the user's message ends in `?` AND names a specific thing to find out, use `--highlights-prompt`. Don't ask the user — infer.

**Recommended order for the agent to choose `--whisper`** *(the orchestrator does NOT auto-select hosted providers from env vars — `--whisper auto` only picks `captions` for URLs with VTT subtitles, otherwise `local`. The agent inspects env vars and passes an explicit `--whisper <name>` flag.)*

1. Multi-speaker content (podcast / interview / meeting):
   - `DEEPGRAM_API_KEY` set → `--whisper deepgram` (hosted, ~$0.0043/min)
   - Else `HF_TOKEN` set with pyannote terms accepted → `--whisper whisperx` (free, local)
   - Else fall through to the single-speaker order below + warn the user diarization is unavailable
2. Single-speaker, want fastest hosted transcription:
   - `GROQ_API_KEY` set → `--whisper groq` (~5 s on a typical clip)
   - Else `OPENAI_API_KEY` set → `--whisper openai`
3. No hosted keys available → `--whisper auto` (default; picks captions if URL has VTT, otherwise local faster-whisper offline)

## Input modes

| Mode | Input | What happens |
|---|---|---|
| **Local file** | `c:\path\video.mp4` | Used in place — no copy |
| **Public URL** | `https://youtu.be/...`, Loom, Vimeo, TikTok, X, ~1500 others | `yt-dlp` downloads into workdir |
| **Jira (full auto)** | `PROJ-1234` or full Jira issue URL — **requires API token configured** (see [Jira token setup](#jira-token-setup) below) | Skill reads token from `credentials.json`, enumerates video attachments via Atlassian REST API, range-downloads the MP4 directly into the workdir. Zero manual steps. |
| **Jira (semi-auto fallback)** | Same input, no token configured | Skill enumerates attachments via Atlassian MCP → asks user to click-download → auto-picks the file from `~/Downloads/` |
| **"I just downloaded it"** | (no path given) | Auto-picks the most recently modified video in `~/Downloads/` from the last 5 minutes |

## When to use

- User shares a video path, URL, or Jira key with words like "watch", "analyze", "transcribe", "summarize", "what's the bug"
- Screen recordings, demo videos, narrated walk-throughs, bug repros
- Use early in a debugging session — frames + narration usually pin down the bug faster than reading the ticket text

## When NOT to use

- Audio-only files — just transcribe directly with `transcribe.py`, no frame extraction needed

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarcinSufa/claude-watch-video](https://github.com/MarcinSufa/claude-watch-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
