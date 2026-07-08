---
trigger: always_on
description: **Ouroboros** — AI-native vertical video engine that researches a topic, writes a script, sources b-roll, generates voiceover + captions, mixes music, assembles a 9:16 Short, and uploads to YouTube (and optionally Facebook).
---

# CLAUDE.md — Ouroboros

**Ouroboros** — AI-native vertical video engine that researches a topic, writes a script, sources b-roll, generates voiceover + captions, mixes music, assembles a 9:16 Short, and uploads to YouTube (and optionally Facebook).

---

## Tech Stack & Key Dependencies

| Layer | Tech |
|---|---|
| Language | Python 3.10+ |
| LLM | Anthropic Claude (default `claude-sonnet-5`), Gemini, OpenAI, Ollama |
| TTS | Edge TTS (free default, 300+ voices), ElevenLabs (premium) |
| Captions | OpenAI Whisper (`base` model) → ASS burn-in + SRT for upload |
| Video | FFmpeg + ffprobe (must be on PATH) |
| B-roll / thumbnails | Pexels API (free tier: 200/hr, 20k/mo) |
| Upload | YouTube Data API v3 + YouTube Analytics API v2 (OAuth2) |
| Social | Facebook Graph API (optional, non-fatal if unconfigured) |
| UI | Gradio (`pip install ouroboros[ui]`) |
| Config | `~/.ouroboros/config.json` — keys live here or in env vars |

---

## Package Structure

```
ouroboros/                  # main package (import as `ouroboros`)
  __main__.py               # CLI entry point: python -m ouroboros <cmd>
  config.py                 # ALL paths, constants, key resolution, setup wizard
  state.py                  # PipelineState — stage completion tracking in draft JSON
  pipeline/
    runner.py               # Pipeline class — single source of truth for stage ordering
  draft.py                  # LLM script generation with hook competition
  broll.py                  # Pexels b-roll fetch + crop to 9:16
  tts.py                    # Edge TTS / ElevenLabs voiceover generation
  captions.py               # Whisper timestamps → ASS (burn-in) + SRT (upload)
  music.py                  # Track selection + speech-aware volume ducking
  assemble.py               # FFmpeg final video assembly
  thumbnail.py              # Pillow thumbnail generation
  upload.py                 # YouTube OAuth upload
  facebook.py               # Facebook Graph API upload (optional)
  analytics.py              # YouTube Analytics fetch + scorer weight updates
  research.py               # Topic research (web scrape / news API)
  niche.py                  # YAML niche profile loader + stage-specific config getters
  llm.py                    # Multi-provider LLM abstraction + Claude CLI/API backends
  retry.py                  # @with_retry exponential backoff decorator
  scheduler.py              # Windows Task Scheduler runner (twice-daily automation)
  topic_memory.py           # Tracks seen topics in ~/.ouroboros/topic_memory.json
  topic_scorer.py           # Virality scorer (recency/engagement/relevance/emotion/novelty)
  sfx.py                    # Sound effects layer (whoosh/impact via FFmpeg lavfi)
  upload_timing.py          # Peak engagement window waiter (niche-specific UTC hours)
  topics/                   # Multi-source topic discovery
    base.py                 # TopicCandidate dataclass + TopicSource ABC
    engine.py               # TopicEngine — orchestrates all sources
    reddit.py / rss.py / google_trends.py / newsapi.py / youtube_trending.py
  reddit_pipeline.py        # Reddit storytime pipeline (separate from main pipeline)
  reddit_fetch.py / reddit_assemble.py
  ui.py                     # Gradio web UI
  utils/
    ffmpeg.py               # ALL ffmpeg/ffprobe subprocess calls live here
    http.py                 # Shared requests.Session with retry (get_session())
    pexels.py               # Pexels API client + quota tracker
    text.py                 # extract_keywords, text helpers
niches/                     # 16 YAML niche profiles (tech, gaming, finance, ...)
backgrounds/                # bg1-5.mp4 loop videos for Reddit pipeline
music/                      # *.mp3 background tracks (EMPTY by default — add your own)
scripts/
  setup_youtube_oauth.py    # First-run YouTube OAuth flow
  refresh_facebook_token.py # Refresh long-lived Facebook page token
tests/                      # pytest suite (188/188 pass)
```

---

## Architectural Rules

### Stage ordering
Defined in `pipeline/runner.py:_ALL_STAGES`. Never reorder or skip in production:

```
draft → broll → voiceover → captions → music → assemble → thumbnail → upload
```

`state.py:STAGES` is the authoritative list including the internal `research` stage.

### How `Pipeline` works (`pipeline/runner.py`)
- `Pipeline(niche, provider, voice, lang, privacy, platform, channel_context, ab_test, require_research, visuals_mode)` — construct once, call `run(topic)`.
- `run()` returns `(success: bool, youtube_url: str, draft_path: Path)`.
- Partial execution: `draft_only(topic)`, `produce(draft_path, force=, script_override=)`, `upload(draft_path, force=, privacy=)`. The CLI commands and the Gradio UI are thin wrappers over these — NEVER re-implement stage logic outside this class (it used to exist in four copies and they diverged).
- Each stage checks `PipelineState.is_done(stage)` before executing — safe to call multiple times.
- On failure, `draft_path` is returned so the caller can invoke `resume_from(draft_path)`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SaazNeupane/Ouroboros-Shorts-Pipeline](https://github.com/SaazNeupane/Ouroboros-Shorts-Pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-08 -->
