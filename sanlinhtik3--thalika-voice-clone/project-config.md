---
trigger: always_on
description: These instructions are for coding agents working on Thalika Voice Clone Studio.
---

# Thalika Agent Instructions

These instructions are for coding agents working on Thalika Voice Clone Studio.

Read `SKILL.md` first for the full project brain. Use this file as the day-to-day working contract.

## Project Identity

Thalika is a local-first Burmese voice-over and voice-clone studio.

Core principles:

- Keep the app local-first.
- Keep generated audio as real PCM WAV.
- Keep user data on local disk.
- Use the managed local VoxCPM2 server as the production inference path.
- Do not add databases, auth, cloud storage, Docker, Runpod, payments, or video features.

## Current Product Surface

Main pages:

- `Script`: paste an original script, rewrite it for narration using Gemini, then send it to Voice Over.
- `Voice Over`: approve script normalization, select/upload voice reference, and generate audio.
- `History`: play, download, delete, and review generated voice-over files.
- `Folders`: inspect local storage, open managed folders, and migrate legacy audio into real PCM WAV.

Current provider:

- `VoxCPM2` (single engine). Burmese pronunciation QA (normalization + lexicon + approval gate) applies automatically when the script is detected as Burmese; multilingual scripts run raw VoxCPM2.

Do not bring back Mock Provider, GPT-SoVITS, or CosyVoice unless the user explicitly asks.

## Architecture Rules

Use:

- Next.js App Router
- React
- TypeScript
- TailwindCSS
- Next.js Route Handlers for the app backend
- Zod for validation
- local Markdown/JSON/files under `data/`

For the local VoxCPM2 server:

- Python 3.10–3.12 with pinned `voxcpm==2.0.3`, served by its own Gradio app
- The Next.js app launches and manages this server as a separate process and talks to it over HTTP — it does NOT embed Python into the Next.js runtime. This is the supported path for local inference.
- Keep generation serialized, seed Python/NumPy/PyTorch consistently, and keep the client/server Gradio argument contract in sync.

Do not use:

- Docker
- Runpod
- Supabase
- PostgreSQL
- MongoDB
- Firebase
- authentication
- payments

All filesystem API routes must include:

```ts
export const runtime = "nodejs";
```

## Local Storage Contract

Store data only in app-managed folders:

- `data/scripts/`
- `data/jobs/`
- `data/outputs/`
- `data/outputs/legacy-backup/`
- `data/memory/`
- `data/profiles/`
- `data/reviews/`
- `data/logs/`

Never expose arbitrary filesystem paths through API responses. Never accept user-supplied paths. Sanitize filenames and prevent path traversal.

Do not store raw reference audio bytes or sensitive transcript contents inside job Markdown. Profile lists must not return raw audio bytes.

## Audio Pipeline Rules

Final output must stay:

- WAV only
- real `RIFF/WAVE`
- `48kHz`
- mono
- `24-bit PCM`

The local server returns WAV. Validate and convert every chunk to `48kHz` mono `24-bit PCM WAV` before downstream processing.

Never concatenate MP3 chunks. Never write mislabeled `.wav` files that contain MPEG audio.

Long script chunk pauses:

- `။`, `.`, `!`, `?`: `260ms`
- `၊`, `,`, `;`, `:`: `160ms`
- no explicit punctuation: `120ms`

Key files:

- `src/lib/script-chunker.ts`
- `src/lib/audio-utils.ts`
- `src/lib/providers/voxcpm2-provider.ts`
- `src/lib/storage/output-wav-migration.ts`

## Burmese Production Quality Rules

Protect these layers:

- reference audio or saved local profile
- browser-side reference quality report
- Burmese normalization
- local Burmese pronunciation lexicon
- normalization approval before generation
- chunked long-script generation
- listening QA after generation

Do not claim 100% human identity match. Phrase quality goals as improving similarity, stability, repeatability, and human review score.

## UI/UX Rules

The app should feel like a calm Apple-style creator tool:

- light theme
- clean rounded cards
- minimal copy
- clear tabs
- lucide icons
- mobile-responsive layout
- no clutter
- no marketing landing page

Prefer shared components. Do not split identical UI into many near-duplicate components.

Important UI files:

- `src/components/AppHeader.tsx`
- `src/components/StudioPageShell.tsx`
- `src/components/ScriptInput.tsx`
- `src/components/VoiceSettings.tsx`
- `src/components/GenerateButton.tsx`
- `src/components/HistoryPanel.tsx`
- `src/components/NormalizationApprovalPanel.tsx`
- `src/app/page.tsx`
- `src/app/script/page.tsx`
- `src/app/history/page.tsx`
- `src/app/storage/page.tsx`

## Common Debug Paths

Generation validation errors:

- `src/lib/validators.ts`
- `src/app/api/generate/route.ts`
- `src/lib/services/generation-service.ts`

Local VoxCPM2 failures:

- `src/lib/providers/hf-utils.ts`
- `src/lib/providers/voxcpm2-health.ts`
- `src/lib/providers/voxcpm2-provider.ts`

Long-script or merge failures:

- `src/lib/script-chunker.ts`
- `src/lib/audio-utils.ts`
- `src/lib/storage/generation-log.ts`

Profile/reference quality issues:

- `src/lib/browser-reference-audio.ts`
- `src/lib/reference-audio-quality.ts`
- `src/lib/storage/voice-profile-store.ts`
- `src/app/api/voice-profiles/route.ts`

Gemini rewrite issues:

- `src/lib/script-rewrite.ts`
- `src/lib/storage/env-store.ts`
- `src/app/api/rewrite/route.ts`

## Verification Checklist

Before finishing backend/provider/audio changes:

```bash
npm run lint
npm run build
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sanlinhtik3/thalika-voice-clone](https://github.com/sanlinhtik3/thalika-voice-clone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
