---
trigger: always_on
description: AI UGC video production from the terminal using the `agent-media` CLI.
---


npm release: https://www.npmjs.com/package/agent-media-cli
agent-media cli github: https://github.com/gitroomhq/agent-media
official website: https://agent-media.ai

# agent-media — AI UGC Video Production & Media Generation

Produce complete UGC videos, Product Acting UGC, and SaaS review videos from the terminal using the `agent-media` CLI.

---

## MANDATORY RULES — READ BEFORE EVERY COMMAND

**You MUST follow ALL of these rules. Violating any rule produces a broken, unwatchable video.**

### Rule 1: ALWAYS use `--actor` — PICK A RANDOM ONE
Every UGC video MUST include `--actor <slug>`. Without an actor, there is no talking head and no lip sync — the video will just be a static image with voiceover.

**If the user hasn't specified an actor:**
1. Run `agent-media actor list` to get the full list of available actors
2. Pick a **random** actor from the list — do NOT always use the same one (e.g., don't always default to `sofia` or `naomi`)
3. Tell the user which actor you picked and suggest they can browse with `agent-media actor list` or pick a specific one with `--actor <slug>`

**NEVER hardcode a default actor.** Every video should feel different — variety in actors is key to quality UGC content.

### Rule 2: ALWAYS count words — 2.5 words per second
Natural speech is 2.5 words/second. Scripts MUST match the target duration **exactly** — too many words sounds robotic, too few words creates awkward pauses and silence:
- **5s video** → 10–12 words (not fewer!)
- **10s video** → 22–25 words (not fewer!)
- **15s video** → 33–37 words (not fewer!)

**CRITICAL**: Count the words before submitting. If the script is too short, ADD more content. If too long, CUT words. A 15-word script on a 10s video = 5 seconds of dead silence. A 50-word script on a 10s video = rushed robotic speech. Both are broken.

### Rule 3: SaaS reviews MUST have screenshots
For any SaaS/product review video, you MUST provide 1-3 product screenshots via `--broll --broll-images`. Without screenshots, the video has no product context — viewers see only a talking head with no evidence of the product.

`--broll-images` accepts both **HTTP/HTTPS URLs** and **local file paths** (local files are auto-uploaded). Images are **semantically matched** to the most relevant broll scene based on filename — so use descriptive filenames! Examples:
- `--broll-images ./dashboard.png,./calendar-view.png` (local files — descriptive names!)
- `--broll-images https://example.com/pricing-page.png,https://example.com/editor.png` (URLs)
- Mix of both works too

If the user provides a product URL but no screenshots, visit the site yourself and extract image URLs from the page.

### Rule 4: SaaS reviews MUST have the product name
Ask the user: "What SaaS product should I review?" Do NOT proceed without it. The product name must appear in the script so it shows up in subtitles.

### Rule 5: Always use `--sync`
Always append `--sync` to wait for the result and get the output URL.

### Rule 6: Name screenshot files descriptively
Broll images are **semantically matched** to scenes by filename. Use descriptive names so the AI assigns the right image to the right scene:
- **GOOD**: `dashboard.png`, `calendar-view.png`, `post-editor.png`
- **BAD**: `screenshot1.png`, `IMG_0042.png`, `image.png`

When saving screenshots for `--broll-images`, rename them to describe what they show.

---

## Prerequisites

The `agent-media` CLI must be installed and authenticated:

```bash
npm install -g agent-media-cli
agent-media login
```

Verify with `agent-media whoami`. If not logged in, run `agent-media login` and follow the OTP flow.

## UGC Pipeline (Flagship Feature)

The UGC pipeline turns a script into a complete video with AI talking heads, B-roll, voiceover, and animated subtitles — one command.

### Flow

Script → Scene splitting → TTS voiceover → AI talking heads + B-roll → Crossfade assembly → Animated subtitles → Background music → End screen CTA

### Basic UGC

```bash
# ALWAYS include --actor for talking heads + lip sync
agent-media ugc "Ever wonder why some videos go viral? Here's the secret..." --actor sofia --sync

# From file
agent-media ugc ./script.txt --actor naomi --sync

# AI-generated script from a product description
agent-media ugc -g "A fitness tracker that monitors sleep quality" --actor marcus --sync
```

### UGC with B-roll

```bash
# With B-roll cutaway scenes mixed in
agent-media ugc "Your script here..." --actor marcus --broll --sync

# With product screenshots as B-roll (REQUIRED for SaaS reviews)
agent-media ugc "Your script here..." --actor sofia --broll --broll-images https://example.com/screenshot1.png,https://example.com/screenshot2.png --sync
```

### UGC Flags

| Flag | Description | Example |
|------|-------------|---------|
| `--actor <slug>` | Library actor for talking heads | `--actor sofia` |
| `--persona <slug>` | Custom persona (cloned voice + face) | `--persona brand-voice` |
| `--face-url <url>` | Direct face photo URL or local file | `--face-url ./photo.png` |
| `--voice <name>` | TTS voice | `--voice nova` |
| `--tone <name>` | Voice tone: energetic, calm, confident, dramatic | `--tone energetic` |
| `--style <name>` | Subtitle style (17 options — pick random!) | `--style tiktok` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuvalsuede/agent-media-skill](https://github.com/yuvalsuede/agent-media-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
