---
trigger: always_on
description: Deepfake detection and media safety — detect AI-generated audio, images, and video, trace synthesis sources, and analyze media intelligence using direct Resemble AI API calls
---


# Resemble Detect — Deepfake Detection & Media Safety

Analyze audio, image, and video for synthetic manipulation, AI-generated content, and media intelligence using **direct Resemble AI API calls**.

## Core Principle — THE IRON LAW

**"NEVER DECLARE MEDIA AS REAL OR FAKE WITHOUT A COMPLETED DETECTION RESULT."**

Do not guess, infer, or speculate about media authenticity. Every authenticity claim must be backed by a completed Resemble Detect job with a returned `label`, `score`, and `status: "completed"`. If the detection is still `processing`, wait. If it `failed`, say so — do not substitute your own judgment.

## When to Use

Use this skill whenever the user's request involves any of these:

- Checking if audio, video, or image is AI-generated or manipulated
- Detecting deepfakes in any media format
- Verifying media authenticity or provenance
- Identifying which AI platform synthesized audio (source tracing)
- Analyzing media for speaker info, emotion, transcription, or misinformation
- Asking natural-language questions about detection results
- Any mention of: "deepfake", "fake detection", "synthetic media", "media forensics", "authenticity check", "source tracing", "is this real"

**Do NOT use** for text-to-speech generation, voice cloning, or speech-to-text transcription — those are separate Resemble capabilities.

## Required Setup

- **API key:** Bearer token from the Resemble dashboard: <https://app.resemble.ai/account/api>
- **Environment variable:** prefer `RESEMBLE_API_KEY`
- **Base URL:** `https://app.resemble.ai/api/v2`
- **Auth header:** `Authorization: Bearer $RESEMBLE_API_KEY`
- **Media inputs:** `POST /detect` accepts exactly one of:
  - direct `multipart/form-data` file upload as `file` (up to 150 MB),
  - public HTTPS `url`, or
  - `media_token` from `POST /secure_uploads`.

Never print API keys or paste bearer tokens into chat. Use environment variables in examples and commands.

## Capability Decision Tree

| User wants to...                                      | Use this                  | API endpoint               |
|-------------------------------------------------------|---------------------------|----------------------------|
| Check if media is AI-generated / deepfake             | **Deepfake Detection**    | `POST /detect`, then `GET /detect/{uuid}` |
| Upload a private/local file without public hosting    | **Direct Upload**         | `POST /detect` multipart `file=@...` |
| Analyze a file larger than 150 MB without public URL  | **Secure Upload**         | `POST /secure_uploads`, then `POST /detect` with `media_token` |
| Know *which AI platform* made fake audio              | **Audio Source Tracing**  | `POST /detect` with `audio_source_tracing: true` |
| Get speaker info, emotion, transcription from media   | **Intelligence**          | `POST /intelligence`       |
| Ask questions about a completed detection             | **Detect Intelligence**   | `POST /detects/{uuid}/intelligence`, then poll answer |

When multiple media capabilities apply, combine them in a single `POST /detect` call using flags such as `intelligence: true`, `audio_source_tracing: true`, `visualize: true`, `use_reverse_search: true`, and `zero_retention_mode: true` instead of making separate jobs.

## Direct API Call Rules

1. **Use direct HTTP requests first.** This skill is intentionally written around `curl` and the Resemble REST API, not MCP tool calls.
2. **Use `Prefer: wait` when a synchronous result is acceptable.** Without it, submit the job, capture the returned UUID, and poll.
3. **Poll async jobs until terminal status.** Terminal statuses are `completed` and `failed`.
4. **Use zero retention for sensitive media.** Set `zero_retention_mode: true` for media detection when privacy matters.
5. **Only report completed results.** Pending/processing jobs are not verdicts.

## Reusable Shell Setup

Use this at the start of any command sequence:

```bash
: "${RESEMBLE_API_KEY:?Set RESEMBLE_API_KEY first}"
BASE_URL="https://app.resemble.ai/api/v2"
AUTH_HEADER="Authorization: Bearer ${RESEMBLE_API_KEY}"
```

If you need JSON extraction and `jq` is available, use it. If not, use `python3 -c 'import json,sys; ...'`.

---

## Phase 1: Deepfake Detection

Submit any audio, image, or video for AI-generated content analysis.

### Submit a Detection from a Public URL

Use this when the media is already reachable via HTTPS:

```bash
curl --request POST "${BASE_URL}/detect" \
  -H "$AUTH_HEADER" \
  -H "Prefer: wait" \
  -H "Content-Type: application/json" \
  --data '{
    "url": "https://example.com/media.mp4",
    "visualize": true,
    "intelligence": true,
    "audio_source_tracing": true,
    "use_reverse_search": true,
    "zero_retention_mode": true
  }'
```

For asynchronous mode, omit `Prefer: wait`, capture `.item.uuid`, then poll `GET /detect/{uuid}`.

### Submit a Detection from a Local File

Direct file uploads are supported for files up to 150 MB:

```bash
curl --request POST "${BASE_URL}/detect" \
  -H "$AUTH_HEADER" \
  -H "Prefer: wait" \
  -F "file=@/path/to/media.mp4" \
  -F "intelligence=true" \
  -F "visualize=true" \
  -F "audio_source_tracing=true" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [resemble-ai/detect-skill](https://github.com/resemble-ai/detect-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
