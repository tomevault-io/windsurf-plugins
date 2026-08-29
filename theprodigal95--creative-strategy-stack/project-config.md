---
trigger: always_on
description: This is a complete creative strategy system for DTC brands. It takes you from zero customer knowledge to a full pipeline of static ads, UGC briefs, listicles, and ongoing test batches — all grounded in real customer language.
---

# Creative Strategy Stack — Setup & Usage Guide

This is a complete creative strategy system for DTC brands. It takes you from zero customer knowledge to a full pipeline of static ads, UGC briefs, listicles, and ongoing test batches — all grounded in real customer language.

---

## Quick Setup

Run the setup script from the repo root — it handles everything:

```bash
./setup.sh
```

This will check for Node.js (install via Homebrew if missing), install npm dependencies, copy skills and commands into your Claude Code config, and prompt you for API keys.

### Manual setup (if you prefer)

<details>
<summary>Click to expand step-by-step instructions</summary>

#### 1. Prerequisites

You need **Node.js** (v18+). Check with `node --version`. If not installed:
- **macOS:** `brew install node` (install [Homebrew](https://brew.sh) first if needed)
- **Linux:** `sudo apt install nodejs npm` or use [nvm](https://github.com/nvm-sh/nvm)
- **Windows:** Download from [nodejs.org](https://nodejs.org/)

#### 2. Install skills and commands

```bash
mkdir -p ~/.claude/skills ~/.claude/commands
cp -r skills/* ~/.claude/skills/
cp -r commands/* ~/.claude/commands/
```

#### 3. Set up API keys

```bash
cp .env.example tools/ad-library/.env
cp .env.example tools/gemini-api/.env
```

Then edit both `.env` files with your actual keys:
- **GEMINI_API_KEY** — Get from Google AI Studio (https://aistudio.google.com/apikey)
- **APIFY_TOKEN** — Get from Apify (https://apify.com/) — needed for Meta Ad Library scraping

#### 4. Install tool dependencies

```bash
cd tools/ad-library && npm install && cd ../gemini-api && npm install && cd ../..
```

#### 5. (Optional) Local Transcription (MLX)

Local transcription via `tools/mlx-transcribe.py` is free and fast but requires **macOS with Apple Silicon** (M1/M2/M3/M4).

**Option A — Pinokio (easiest):** Install [Pinokio](https://pinokio.computer/) and add the "MLX Video Transcription" app. Everything is bundled.

**Option B — Manual:**
```bash
brew install ffmpeg
pip install mlx mlx-whisper numpy
```

</details>

**Don't have Apple Silicon?** No problem — use Gemini transcription instead. The `/transcribe` command will guide you to the right tool. Gemini transcription works on any platform with just an API key.

---

## What's in the Stack

### Skills (invoke with `/skill-name`)

| Skill | What it does | When to use it |
|-------|-------------|----------------|
| `/statics-briefer` | 4-gate workflow producing static ad briefs using TEEP stages, Three Selves theory, and Emotional Zones | Writing creative briefs for static ads |
| `/native-ad-creative` | Generates native ad headlines + image direction using direct response psychology | Native advertising creative (headlines + concepts) |
| `/listicle-writer` | 9-gate system where each numbered point is a sales argument | Landing page listicles |
| `/editorial-image-prompts` | Generates editorial-style image prompts that look like real content, not ads | Image generation for native placements |
| `/story-selling` | Framework for Meta ad scripts where the story earns the sale | UGC scripts, testimonial ads, video creative |
| `/critique` | Meta-skill that evaluates work against any loaded skill/framework | Quality review of any creative output |
| `/gemini-api` | Interface to Google Gemini for text, image analysis, video analysis, and image generation | Image/video analysis, AI image generation |

### Commands (invoke with `/command-name`)

| Command | What it does |
|---------|-------------|
| `/ad-library` | Batch scrape Meta Ad Library for competitor creative, download media, optional visual analysis |
| `/transcribe` | Route video/audio to local MLX (free) or Gemini API (detailed visual context) |

### Tools (backend scripts the commands call)

| Tool | Location | Purpose |
|------|----------|---------|
| Ad Library suite | `tools/ad-library/` | Scrape, download, analyze, batch process, cleanup |
| Gemini API wrapper | `tools/gemini-api/` | Universal Gemini interface (text, images, video, generation) |
| MLX Transcribe | `tools/mlx-transcribe.py` | Local video transcription (Apple Silicon only, optional) |
| Reddit Scraper | `tools/reddit-scraper.js` | Lightweight Reddit scraping — fast, low-token alternative to the full Research Engine |

### Research Engine (MCP server — available automatically in Claude Code)

The Research Engine is a 12-step Python pipeline that scrapes Reddit, extracts evidence, discovers themes, scores brand fit, mines language patterns, and produces structured insights. It runs as an MCP server, so you use it directly from Claude Code — no separate commands needed.

**No API key required.** It authenticates through your Claude Code session automatically.

| Tool | What it does |
|------|-------------|
| `create_brand` | Create a new brand from a raw product/brand info dump |
| `run_research_sprint` | Start a Reddit research sprint (7-25 min per sprint) |
| `check_sprint_status` | Monitor a running sprint |
| `list_brands` | See all configured brands |
| `list_sprints` | See all sprints for a brand |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TheProdigal95/creative-strategy-stack](https://github.com/TheProdigal95/creative-strategy-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
