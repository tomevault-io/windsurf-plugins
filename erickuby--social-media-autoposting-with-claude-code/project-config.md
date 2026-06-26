---
trigger: always_on
description: The complete AI-powered content creation and distribution system. Create, edit, and publish content across 13+ platforms using Claude Code skills.
---

# IX AI Agent Social Media Manager

The complete AI-powered content creation and distribution system. Create, edit, and publish content across 13+ platforms using Claude Code skills.

## Skills (17)

| Category | Skill | Triggers |
|----------|-------|----------|
| **Distribution** | `late-social-media` | "post to", "schedule post" |
| | `short-form-posting` | "post short", "post reel" |
| | `youtube-content-package` | "youtube package", "publish video" |
| **Visual Creation** | `thumbnail-creator` | "create thumbnail", "youtube thumbnail" |
| | `carousel-generator` | "create carousel", "carousel images" |
| | `document-carousel` | "document carousel", "LinkedIn PDF" |
| **Video Pipeline** | `clip-extractor` | "extract clips", "reframe video", "face tracking" |
| | `clip-selection` | "select clips", "find best clips" |
| | `edit` | "edit video", "edit clip" |
| | `video-editing` | (invoked by /edit -- router) |
| | `short-form-editing` | (invoked by router -- <90s) |
| | `long-form-editing` | (invoked by router -- 5+ min) |
| | `extracting-transcripts` | "transcribe", "extract transcript" |
| | `visual-overlay-creation` | "create illustration", "new visual" |
| **Voice** | `voice-dna` | "write post", "create caption", "write description", "draft post" |
| **Utility** | `video-upload-helper` | "compress video", "upload video" |
| | `content-analytics` | "check analytics" |

## Rules

1. **Never post without user approval.** Always show the content package and get explicit confirmation.
2. **Each platform gets unique content.** Same message, different wording. Never copy-paste across platforms.
3. **Ask for thumbnails** before posting video content to YouTube.
4. **Confirm titles** before posting to YouTube.
5. **Use the Late REST API** (curl) for posts that need platform-specific features.
6. **Use Late MCP tools** for simple single-platform posts.
7. **Voice DNA is mandatory.** Before writing ANY social media post, caption, description, script, or written content, load the `voice-dna` skill. All written output must match Eric's voice — direct, warm, practical, British English, never corporate or salesy. See `.claude/skills/voice-dna/SKILL.md` for the full profile.

## Session Commands

| Command | Purpose | Triggers |
|---------|---------|----------|
| `/continue` | Load context, check system, review state, suggest next action | "continue", "resume", "what's next", "where are we" |
| `/done` | Validate, sync docs, report, commit and push | "done", "wrap up", "commit this", "push it" |

## Pipeline Flow

```
/clip-selection > /clip-extractor > /transcribe > /edit > /post-short
```

1. **Clip Selection** -- Analyze transcript, score clips (5 categories, 0-100), select best moments
2. **Clip Extraction** -- Face-tracking reframe (16:9 to 9:16) via Python tool at `tools/clip_extractor/`
3. **Transcription** -- WhisperX GPU or AssemblyAI for word-level timestamps
4. **Editing** -- `/edit` routes to `video-editing` (router) then `short-form-editing` or `long-form-editing`
5. **Publishing** -- `/short-form-posting` or `/youtube-content-package` via Late/Zernio

## Clip Extractor — How It Works (IMPORTANT)

The clip extractor does **intelligent face-tracking reframe**, NOT a static center crop. Never use a raw ffmpeg `crop=` filter as a substitute.

**6-stage pipeline:**
1. **Face Detection** (MediaPipe BlazeFace) -- finds the face every N frames
2. **Signal Fusion** -- combines face + pose + saliency for robust tracking
3. **Temporal Smoothing** (Kalman/EMA) -- smooth, natural camera motion
4. **Deadzone Filtering** -- suppresses micro-jitter so the crop doesn't twitch
5. **Crop Calculation** -- centers the 9:16 window ON the face (mouth at center)
6. **Frame-by-frame rendering** -- interpolated crop positions piped through ffmpeg

**Dependencies:** `mediapipe`, `opencv-contrib-python`, `numpy`, `filterpy`, `pyyaml`, `rapidfuzz`

**All extracted clips go to:** `output/clips/YYYY-MM-DD-slug/` with a `clips-metadata.json`

## Editing Architecture (3-Skill System)

```
/edit > video-editing (ROUTER) > short-form-editing (<90s)
                                > long-form-editing (5+ min)
```

### Format Detection

| Duration | Format | Primary Component | Pop-Outs |
|----------|--------|------------------|----------|
| <90s (pipeline) | Short-form | ConceptOverlay | 8-15 |
| <90s (standalone) | Short-form | AppleStylePopup + FloatingCard | 5-7 |
| 5+ min | Long-form | ConceptOverlay | 30-40+ |

## Essential Commands

```bash
# Remotion
npm run studio                    # Preview in browser
npm run render -- <Id> out/x.mp4  # Render composition

# Clip Extractor
cd tools
python -m clip_extractor reframe --video input.mp4 --output clips/ --format 9x16
python -m clip_extractor batch --video source.mp4 --clips defs.json --output clips/
```

## Critical Editing Rules

1. **ALWAYS use WORDS data** (`.ts` files) for frame timing
2. **Pop-out at EXACT frame** the keyword is spoken
3. **ILLUSTRATION-FIRST** -- NO caption on most pop-outs
4. **illustrationSize:** 800 (no text), 700 (with text), 620 (CTA)
5. **NEVER use template illustrations** -- every pop-out needs UNIQUE visual metaphor

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Erickuby/Social-Media-Autoposting-With-Claude-Code](https://github.com/Erickuby/Social-Media-Autoposting-With-Claude-Code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-26 -->
