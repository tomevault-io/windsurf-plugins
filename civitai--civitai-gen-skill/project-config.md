---
trigger: always_on
description: Handles template expansion (product, zip, random modes), wildcard resolution, and meaningful file naming.
---

# civitai-gen Developer Guide

This file is for agents working on the skill itself. If you're just using the skill, see SKILL.md.

**Keep this file updated** when you add, move, or change files.

## Architecture

```
civitai-gen/
├── lib/
│   ├── api.mjs             # Shared API layer (auth, workflows, downloads)
│   ├── image.mjs            # Image step builder, ecosystem configs, AIR URN parsing
│   ├── video.mjs            # Video step builder, engine registry
│   └── audio.mjs            # TTS, music (ACE Step), transcription step builders
├── docs/
│   ├── engines.md           # Engine catalog + model selection (two-path model)
│   ├── tts.md               # TTS parameter reference (for agents)
│   ├── music.md             # Music generation reference (for agents)
│   └── transcription.md     # Transcription reference (for agents)
├── test/
│   └── smoke-test.mjs       # Smoke tests (--readonly for 0 buzz, full ~16 buzz)
├── generate.mjs              # Unified CLI — thin dispatcher over lib/ modules
├── experiment.mjs            # Wildcard expansion, wraps generate.mjs via child_process
├── wildcards/                # Pre-built wildcard files (.txt, .json)
├── SKILL.md                  # User-facing docs — lean router to domain docs
├── CLAUDE.md                 # This file (developer guide)
└── .env                      # API key (CIVITAI_API_KEY)
```

## Module Responsibilities

### lib/api.mjs — Shared API Layer

Domain-agnostic infrastructure shared by all generation types.

| Export | Description |
|--------|-------------|
| `loadEnv()` | Reads `.env` from skill root into `process.env` |
| `BASE_URL` | Orchestrator base URL |
| `WORKFLOWS_URL` | Workflow submission endpoint |
| `CIVITAI_API_URL` | Civitai tRPC API base URL |
| `getApiKey()` | Returns `CIVITAI_API_KEY` or exits |
| `authHeaders(apiKey)` | Returns `{ Authorization, Content-Type }` headers |
| `apiSubmitWorkflow(apiKey, body)` | POST workflow, returns workflow object |
| `apiWhatIf(apiKey, body)` | POST workflow with `?whatif=true`, returns cost estimate |
| `apiGetWorkflow(apiKey, workflowId)` | GET workflow status |
| `downloadFile(url, destPath)` | Download a single file |
| `downloadAll(items, opts)` | Download multiple files with concurrency limit |
| `pollWorkflow(apiKey, workflowId, opts)` | Poll until terminal state or timeout |
| `collectDownloads(workflow, manifest, opts)` | Extract downloadable media (images, videos, audio) from workflow response |

### lib/image.mjs — Image Generation

| Export | Description |
|--------|-------------|
| `ECOSYSTEM_CONFIGS` | Aspect ratio presets per base model (SD1.5, SDXL, Flux, etc.) |
| `DEFAULT_ECOSYSTEM` | Default ecosystem (`flux1`) |
| `RESOLUTION_MULTIPLIERS` | Scale factors: small (0.75x), medium (1.0x), large (1.5x) |
| `buildImageStep(job, index)` | Builds `$type: 'textToImage'` workflow step |
| `detectEcosystem(modelUrn)` | Detect ecosystem from AIR URN |
| `parseResources(resourceStr)` | Parse comma-separated LoRA/embedding AIR URNs |
| `resolveDimensions(opts)` | Resolve width/height from aspect + ecosystem + resolution |
| `parseAirUrn(urn)` | Parse AIR URN into components |
| `IMAGE_ARG_HANDLERS` | CLI arg handler map for image-specific flags |
| `IMAGE_HELP` | Help text for image generation flags |

### lib/video.mjs — Video Generation

| Export | Description |
|--------|-------------|
| `VIDEO_ENGINE_REGISTRY` | Static capabilities for 11 video engines |
| `buildVideoStep(job, index)` | Builds `$type: 'videoGen'` workflow step |
| `VIDEO_ARG_HANDLERS` | CLI arg handler map for video-specific flags |
| `VIDEO_HELP` | Help text for video generation flags |

### lib/audio.mjs — Audio: TTS, Music, Transcription

| Export | Description |
|--------|-------------|
| `buildTTSStep(job, index)` | Builds `$type: 'textToSpeech'` workflow step |
| `buildMusicStep(job, index)` | Builds `$type: 'aceStepAudio'` workflow step |
| `buildTranscriptionStep(job, index)` | Builds `$type: 'transcription'` workflow step |
| `AUDIO_ARG_HANDLERS` | CLI arg handler map for audio-specific flags |
| `AUDIO_HELP` | Help text for audio flags |

### generate.mjs — Unified CLI Dispatcher

**Subcommands:** wait, submit, status, download, cost, engines, tts, music, transcribe (alias: stt)

The CLI is a thin orchestrator. Domain logic lives in `lib/` modules:

1. `parseArgs()` merges arg handlers from all domains (`IMAGE_ARG_HANDLERS`, `VIDEO_ARG_HANDLERS`, `AUDIO_ARG_HANDLERS`)
2. Audio subcommands (`tts`, `music`, `transcribe`) set `opts.jobType` and remap to the `wait` lifecycle
3. `buildStep()` dispatches to the right builder based on `job.jobType` or `job.engine`
4. `detectMediaType()` determines output type from step `$type` for status/download logic
5. `collectDownloads()` (in api.mjs) handles images, videos, and audio blob outputs

### experiment.mjs — Wildcard Expansion

Wraps generate.mjs via `child_process.execFile`. Does not import from it directly.
Handles template expansion (product, zip, random modes), wildcard resolution, and meaningful file naming.

## Canonical Documentation

Civitai now ships official developer docs — prefer these over reverse-engineering:

- **Docs site:** <https://developer.civitai.com>

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [civitai/civitai-gen-skill](https://github.com/civitai/civitai-gen-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
