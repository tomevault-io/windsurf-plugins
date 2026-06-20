---
trigger: always_on
description: Use when the user wants to turn their app, UI, or product into a polished ~50s feature/demo video showing how it works — capturing the real running app (or an HTML mockup, or a terminal), narrated with AI voiceover, synced karaoke captions, a music bed, and brand framing. Triggers on "make a feature video", "demo video of my app", "show how my app works as a video", "UI walkthrough video", "product launch film", "video for landing page", "record my app with narration", "screen recording with voi
---


# /demo-video — turn an app/UI into a ~50s narrated feature film

## What it makes

A single `final-framed.mp4` (~50s, 1920×1080) that shows **how an application works**, in a polished product-film aesthetic:

- **Your scenes, your order** — `scenes.sequence` composes any mix of: `browser_capture` (drive + record the REAL running app), `before_after` (a labeled BEFORE/AFTER comparison of two clips — bug→fix task demos), `html_mockup` (a designed screen for an unbuilt feature), `screen_recording` (an existing clip), `terminal` (CLI via VHS), plus built-in `graph` and `endcards`.
- **AI voiceover** narrating the feature (Edge TTS, free, no API key, many languages/voices)
- **Karaoke word-highlight captions** synced via WordBoundary events, readable on light AND dark UI
- **Music bed** — procedural / your own file / none — auto-ducked under the voice
- **Brand framing** — palette, fonts, logo, end cards; optional styled-window-on-desk composite

Built entirely with **free tools** — ffmpeg, Playwright headless Chromium, VHS (Charm), Edge TTS. Zero paid services.

The bundled default (`memory-product-default`) is a 7-scene terminal narrative, but that's just one example arc — the common case is composing your own scenes around your real app's UI.

## When to use

- User wants a feature/demo/launch video of their **app or UI** — showing how it works, narrated and captioned
- User wants to capture their **real running app** (localhost or live site) as a polished film
- User wants to demo a **feature that isn't built yet** (use an `html_mockup` scene — no throwaway code in the real app)
- User has a CLI/terminal product to showcase
- User wants the aesthetic of Anthropic/Linear-style product films without an After Effects / Synthesia / HeyGen subscription

## When NOT to use

- User wants a raw, unedited live screen recording of a working session → use /qa or /browse instead
- User wants a 5-minute tutorial → this targets ~50s feature films, not long-form content
- User wants pure motion graphics / explainer animation with no real UI, terminal, or mockup → use a design tool

## Workflow (single command per project)

```
1. /demo-video init                  → scaffolds demo-video/ in current project
2. user edits brand.yaml             → palette, voice, VO script, scenes
3. /demo-video plan                  → dry run (~1s): VO + video length estimate, PASS/WARN — no render
4. /demo-video build                 → runs pipeline, produces final-framed.mp4
5. /demo-video preview               → starts local server + tunnel for mobile review
```

Inside Claude Code, the skill orchestrates these by reading config and invoking bundled scripts.

## Pipeline (what `build` runs)

```
apply-brand.py    → compile brand.yaml → .build/ (rendered templates + config.json)
render-mascot.py  → mascot sprite frames (if mascot: configured) — overlaid inside build-scenes.sh per row/step
make-vo.py        → Edge TTS streams vo.mp3 + vo-words.json (word-level timing)
make-captions.py  → captions.ass (karaoke) + captions.srt
make-music.sh     → procedural ambient pad music.mp3 (or mode: file / none)
plan-scenes.py    → resolve the scene sequence → scene-plan.json
make-auth.mjs     → (if `auth:` set) log in once → auth.json (authed scenes reuse it).
                    mode: scripted (creds) | manual (headed, you log in — SSO/OIDC/2FA)
build-scenes.sh   → render each scene by type; pin to `duration:` (normalize-clip.py) if set; cached
autofit.py        → (if `scenes.autofit`) adjust speedup so video holds the voiceover
assemble.sh       → normalize + speedup + crossfade N scenes → final-rough.mp4
mix-final.sh      → ⚠ timing gate (check-timing.py) → voice + music sidechain-ducked → final-with-audio.mp4
burn-captions.sh  → ffmpeg subtitles filter → final-with-captions.mp4
record-frame.mjs  → Playwright snapshot + ffmpeg overlay → final-framed.mp4
```

Each step caches outputs. Re-running only re-processes what changed.

**Timing safety net (P0-1).** `mix-final.sh` cuts the voice track to the video length,
so a voiceover longer than the assembled video would silently lose its closing line.
Before muxing, `check-timing.py` compares the real speech-end (from `vo-words.json`)
against the video and **fails the build** with an actionable message if narration
overruns. Override with `DEMO_ALLOW_TRUNCATE=1`. Catch it earlier with `/demo-video plan`.

## Prerequisites

| Tool | Install | Why |
|---|---|---|
| **ffmpeg** | `winget install Gyan.FFmpeg` / `brew install ffmpeg` | All video/audio operations |
| **Python 3.10+** | system | VO + caption scripts |
| **Node 18+ + pnpm** | system | Playwright recorder |
| **edge-tts** | `pip install --user edge-tts` | Free Microsoft TTS, no API key |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MarcinSufa/claude-demo-video](https://github.com/MarcinSufa/claude-demo-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
