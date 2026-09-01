---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Origami AI turns PDF slide decks into narrated videos, entirely in the browser: PDF extraction → LLM-written narration script → local TTS (Kokoro.js) → FFmpeg.wasm rendering to MP4, all client-side via WebGPU. It also does screen recording with auto-zoom, MP4 scene analysis, an AI assistant chat, and a "Shorts" video generator (Pollinations image/video + TTS captions). The Express server (`server.ts`) exists almost entirely to proxy calls to cloud LLM/image APIs so secret keys never reach the client bundle — it is not a typical CRUD backend.

## Commands

```bash
npm run dev       # Express + Vite dev server with HMR, http://localhost:3000
npm run build     # Production build -> dist/
npm run preview   # Serve the production build
npm run lint      # ESLint — only lints plain .js files (see eslint.config.js: **/*.ts and **/*.tsx are ignored)
npm run stop      # Kill whatever is on port 3000
npm run pages:dev # Run via Wrangler (Cloudflare Pages) instead of the Express server
```

There is no test suite in this repo. There is no `tsc` script; type-checking happens via editor/IDE or `tsc --build` against the four project references in `tsconfig.json` (app, node, server, functions).

Do not open `index.html` directly or bypass the dev server — FFmpeg.wasm/SharedArrayBuffer require the COOP/COEP headers that `npm run dev` (or the production server) sets. See `TROUBLESHOOTING.md` for header details.

## Architecture

### Dual server runtime — keep in sync manually

The proxy API exists in **two parallel implementations** that must be kept behaviorally identical:

- `server.ts` — Express app, used by `npm run dev`, Docker, and any Node host. Also mounts Vite in middleware mode for dev, and serves `dist/` in production.
- `functions/api/**` — Cloudflare Pages Functions, used when deployed to Cloudflare Pages (`wrangler.toml`, `npm run pages:dev`). Each file exports an `onRequestPost`/`onRequestGet` handler mirroring one Express route.

Routes that exist in both, and must stay matched when you change one:
- `POST /api/llm/chat` ↔ `functions/api/llm/chat.ts`
- `POST /api/llm/analyze-video` ↔ `functions/api/llm/analyze-video.ts`
- `POST /api/llm/analyze-issue` ↔ `functions/api/llm/analyze-issue.ts`
- `POST /api/pollinations/image` ↔ `functions/api/pollinations/image.ts`
- `POST /api/pollinations/video` ↔ `functions/api/pollinations/video.ts`
- `GET /api/music-preview/:filename` ↔ `functions/api/music-preview/[filename].ts`

If you add or change a server API route, update both sides. `functions/utils.ts` holds shared helpers for the Pages Functions side (e.g. `base64ToUint8Array`); the Express side inlines equivalents in `server.ts`.

### API key routing (client key vs. server proxy)

This is the core security model of the app — preserve it when touching LLM/Pollinations code:

- Only `VITE_`-prefixed env vars (`VITE_LLM_API_KEY`, `VITE_LLM_BASE_URL`, `VITE_LLM_MODEL`) are baked into the client bundle by Vite. Never add a new `VITE_`-prefixed secret without understanding it becomes public.
- In production there is normally no client-side key. `src/services/aiService.ts` detects the absence of a client key and routes calls through the server proxy endpoints above instead of calling the provider directly.
- The server reads `LLM_API_KEY` (falls back to `VITE_LLM_API_KEY` for convenience) via `process.env`, never exposing it to responses sent to the browser.
- Pollinations has the same split: `pk_...` publishable keys are safe client-side (stored per-user in `GlobalSettings.pollinationsApiKey` via IndexedDB, see `src/services/storage.ts`), `sk_...` secret keys and `POLLINATIONS_API_KEY` are server-only fallbacks used by `/api/pollinations/image`.
- `src/services/pollinationsAuth.ts` + `src/pages/PollinationsCallbackPage.tsx` implement an OAuth-style flow that exchanges for a `sk_...` token stored client-side; treat this token like any other secret when logging or serializing state.
- **Turnstile & Firebase**: The frontend uses Cloudflare Turnstile for bot protection. Its secret key (`TURNSTILE_SECRET_KEY`) is kept in `.env` (ignored from git) for backend verification. Firebase config keys (`src/config/firebase.ts`) are intentionally public. Be careful when updating CSP headers in `server.ts` to ensure `challenges.cloudflare.com` and `apis.google.com` remain allowed.

### Client-side compute — WebGPU workers

Heavy AI work runs in Web Workers, not the main thread:
- `src/services/webLlmService.ts` + `src/services/webLlm.worker.ts` — local LLM inference via `@mlc-ai/web-llm`, gated by `checkWebGPUSupport()`. WebLLM is **never** eagerly initialized on app start (see comment in `src/App.tsx` around the `WebLLMInitModal`/`useEffect` — eager init caused tab instability on some systems); it's loaded on demand.
- `src/services/tts.worker.ts` + `src/services/ttsService.ts` — Kokoro.js TTS, quantization is user-selectable (`q8` quality vs `q4` speed).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TechMitten/Origami-AI](https://github.com/TechMitten/Origami-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-01 -->
