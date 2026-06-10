---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Debate Arena — a React web app that orchestrates real-time debates between three AI agents (Advocate/Google Gemini, Critic/OpenAI, Wildcard/Anthropic Claude) across 3 rounds on a user-submitted topic. The Wildcard judges each round. Results are visualized as a D3 force-directed graph.

## First-time setup

1. `brew install gitleaks vercel` — `gitleaks` backs the pre-commit hook; `vercel` is the local dev server.
2. `npm install` — installs deps and activates the gitleaks hook via the `prepare` script.
3. `cp .env.example .env.local` and fill in every required value. `.env.local` is gitignored; never commit real values.
4. `vercel link` — one-time, connects this clone to the Vercel project.
5. `vercel dev` — runs frontend + serverless API on `http://localhost:3000`.

## Commands

```bash
vercel dev         # Local dev: serves frontend + /api/* on one port (http://localhost:3000)
npm run dev        # Vite-only dev server (http://localhost:5173) — /api/* will 404
npm run build      # Production build → dist/
npm run lint       # ESLint
```

Local development uses the Vercel CLI (`vercel dev`) so the Vite frontend and the serverless functions in `api/` run together. `npm run dev` alone won't serve `/api/*` — there's no Vite proxy and no separate Express backend.

No test suite is configured.

## Architecture

**Frontend**: React 19 + Vite 8, no TypeScript, no state management library. All state lives in App.jsx via hooks.

**Backend**: Vercel serverless functions in `api/`. Two parallel per-claim pipelines, plus a verdict and a debate-cache:

- **Streaming** (`POST /api/debate-stream`, used by MSE-capable clients): streams LLM SSE tokens → `api/_chunker.js` sentence chunker → serial ElevenLabs `streamWithTimestamps` with `previousText` for prosody continuity → NDJSON (`chunk_meta` / `audio` / `claim_complete` / `error`) to client. Agents emit a `TEXT:\n<prose>\n---META---\n{...}` format so the chunker can consume raw prose tokens without parsing JSON. Cache namespaces: `getCachedLlm` (shared with legacy path) + `ttsStreamCacheKey` (isolated NDJSON blob).
- **Legacy** (`POST /api/debate` JSON + `POST /api/tts` NDJSON, used by iOS Safari with no MediaSource): two-step LLM→TTS, single-shot per claim. Both endpoints are **permanent** — do not delete; they're also the verdict TTS path.
- `POST /api/verdict` (JSON): wildcard's final judgement. Always non-streaming (~150 tokens, not worth the work).
- `GET/POST /api/debate-cache`: full-debate cache check + persistence.

Shared modules in `api/`: `_shared.js` (provider LLM calls + streaming variants, rate limit, validation, cache helpers), `_prompts.js` (templates + `BEHAVIOR_HASH` invalidation), `_chunker.js` (`SentenceChunker`), `_streaming.js` (TEXT/META state machine + parsers), `_tts.js` (EL client + voice config).

**Data flow**: TopicInput → `runDebate()` in `src/lib/debate.js` → dispatches via `hasMSE()`:
- `liveGenStreaming()`: per claim, opens `/api/debate-stream` and consumes NDJSON via `startClaimStream` in `src/lib/audio.js`. Each claim's network request pipelines behind the previous claim's *audio* playback (gated via `gateBeforePlay`); audio remains strictly serial.
- `liveGenLegacy()`: per claim, `callAgent` (`/api/debate`) then `speakClaim` (`/api/tts`) — identical to pre-refactor.

Claims feed `buildGraphData()` (src/lib/graphUtils.js) → DebateGraph renders via D3.

**Key modules**:
- `src/lib/agents.js` — parser for both TEXT/META prose-trailer and legacy JSON formats. Claim IDs follow `{prefix}_r{round}_{index}` (e.g., `adv_r1_1`).
- `src/lib/debate.js` — orchestrator with `hasMSE()` capability branch. Cache replay path + verdict path shared across both branches.
- `src/lib/audio.js` — `playAudioStream` (legacy single-shot, used by verdict + iOS) and `startClaimStream` (multi-chunk envelope, gated playback for pipelining, cumulative karaoke alignment offset).
- `src/lib/graphUtils.js` — Graph data builder, scoring (`computeWildcardScore`), round winner logic.
- `src/components/DebateGraph.jsx` — D3 SVG graph (800×700 viewBox). Fixed agent anchors: Advocate top-center, Critic bottom-left, Wildcard bottom-right.

**Agent colors**: Green (Advocate), Red (Critic), Purple (Wildcard).

**Wire protocol**:

LLM prompt output format (all three claim agents — see `api/_prompts.js`):
```
TEXT:
<one paragraph of prose>
---META---
{"rebuts": "crt_r1_1" | null, "agrees_with"?: "wld_r1_2"}
```

`/api/debate-stream` NDJSON events (one claim per request):
- `{type:"chunk_meta", seq, chunkText}` — emitted before each TTS chunk
- `{type:"audio", seq, audioBase64, alignment}` — EL frames (alignment timestamps are relative to the chunk; client accumulates a cumulative offset for karaoke)
- `{type:"claim_complete", fullText, rebuts, agrees_with}` — emitted once, before `res.end()`
- `{type:"error", recoverable:false, message}` — on mid-stream failure

Client mirror lives in `src/lib/audio.js` `startClaimStream`. Both sides of the protocol use literal strings (Vite can't import from `api/`); rename one, rename both.

## Environment Variables


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [franalli/debate-arena](https://github.com/franalli/debate-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-10 -->
