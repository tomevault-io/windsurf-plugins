---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Daily podcast generator that summarizes top 10 Hacker News posts using AI and text-to-speech. Audio is generated via OpenAI TTS or ElevenLabs and published to Transistor.fm podcast host. Optionally generates YouTube-ready video with story screenshots and chapter timestamps.

## Terminology

- Story - A Hacker News post (article, link, or discussion)
- Episode - A podcast episode covering the top stories
- Intro - Short AI-generated introduction for the episode mentioning the top 3 headlines
- Summary - AI-generated summary of a story's content and top comments

## Development Commands

```bash
# Run the podcast generator
pnpm start

# Run tests
pnpm test              # Run all tests once
pnpm test:watch        # Run tests in watch mode
pnpm test:screenshots  # Manual screenshot capture verification

# Type checking and linting
pnpm build             # TypeScript type check (no output)
pnpm lint              # ESLint check
pnpm lint:fix          # ESLint with auto-fix
pnpm format            # Format with Prettier

# Cache management (cache/ directory stores intermediate data)
pnpm clean:all         # Remove all cached data and output
pnpm clean:cache       # Remove all cache files
pnpm clean:stories     # Remove cached story data
pnpm clean:summaries   # Remove cached summaries and intros
pnpm clean:audio       # Remove cached MP3 files
pnpm clean:screenshots # Remove cached screenshot images
pnpm clean:output      # Remove output files
pnpm clean:video       # Remove video output files
```

## CLI Arguments

The main script accepts several flags via minimist. **Do not use `--` separator** — pass flags directly to `pnpm start`:

```bash
# Generate podcast with custom story count
pnpm start --count 5

# Preview stories without generating podcast
pnpm start --preview

# Skip audio generation
pnpm start --no-audio

# Publish to podcast host (otherwise only publishes in CI)
pnpm start --publish

# Summarize a specific HN story by ID
pnpm start --storyId 12345

# Parse and summarize arbitrary URL
pnpm start --summarizeLink https://example.com

# Generate audio from arbitrary text
pnpm start --textToAudio "Some text to speak"

# Generate video alongside audio (also runs automatically in CI)
pnpm start --video

# Video generation shorthand
pnpm start:video

# Test intro generation with specific story IDs (requires exactly 3)
pnpm start --testIntro 123,456,789

# Test screenshot capture for a single URL
pnpm start --testScreenshot https://example.com

# Benchmark video pipeline with predefined URLs
pnpm benchmark                    # Uses showcase (default)
pnpm benchmark:stress-test        # Uses challenging URLs
pnpm start --benchmark=data/custom.json  # Custom file
```

### Benchmark Data Files

- **`data/showcase-episode.json`** - Curated URLs with visually appealing screenshots (YouTube, GitHub, major tech blogs)
- **`data/benchmark-stress-test.json`** - Challenging URLs that test edge cases (paywalls, bot detection, varied layouts)

## Architecture

### Data Flow

1. **Fetch** (`src/hn/index.ts`) - Fetches top stories from HN Algolia API, filters previously covered stories, parses content via Readability
2. **Summarize** (`src/ai/index.ts`) - Generates summaries using OpenAI GPT-4.1-nano with structured prompts for story content + comments
3. **Adjust Pronunciation** (`src/audio/adjustPronunciation.ts`) - Pattern-based text transformations for TTS mispronunciations
4. **Generate Audio** (`src/audio/index.ts`) - Converts text to speech via OpenAI or ElevenLabs, concatenates segments with ffmpeg
5. **Generate Video** (`src/video/index.ts`) - Optional: Captures screenshots of story URLs, renders video with Remotion, muxes audio with ffmpeg
6. **Publish** (`src/podcast.ts`) - Uploads to Transistor.fm API with show notes

### Key Modules

- **`src/index.ts`** - Entry point, orchestrates the full pipeline
- **`src/hn/`** - HN API integration, content parsing (Readability + JSDOM), PDF text extraction
- **`src/ai/`** - OpenAI integration for summarization, intro generation, episode title generation
- **`src/audio/`** - TTS integration (OpenAI/ElevenLabs), pronunciation adjustments, audio concatenation
- **`src/browser/`** - Shared Puppeteer browser config with stealth/adblocker plugins for content fetching and screenshots
- **`src/video/`** - Video generation with Remotion, Puppeteer screenshots, YouTube chapter timestamps
- **`src/video/domain-handlers/`** - Domain-specific screenshot handlers (YouTube thumbnails, GitHub repo cards, Twitter embeds)
- **`src/show-notes.ts`** - Generates `show-notes.txt` (episode description with links) and `transcript.txt` for output
- **`src/services.ts`** - TTS service factory (OpenAI vs ElevenLabs based on `VOICE_SERVICE` env)
- **`src/podcast.ts`** - Transistor.fm API client for episode upload/publish
- **`src/utils/cache.ts`** - File-based caching for summaries, audio segments, covered stories

### Caching Strategy

All intermediate data is cached to `cache/` directory to avoid redundant API calls:
- Story summaries: `cache/summary-{storyId}`
- Podcast intro: `cache/intro-{hash}`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [denolfe/hacker-news-highlights](https://github.com/denolfe/hacker-news-highlights) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
