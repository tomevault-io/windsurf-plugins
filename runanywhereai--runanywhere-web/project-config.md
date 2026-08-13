---
trigger: always_on
description: This is the browser validation app for the Web SDK. It consumes the Swift-aligned public facade from `@runanywhere/web`, backend registration packages from `@runanywhere/web-llamacpp` and `@runanywhere/web-onnx`, and browser helpers from `@runanywhere/web/browser`.
---

# RunAnywhere Web Example — AGENTS.md

## Overview

This is the browser validation app for the Web SDK. It consumes the Swift-aligned public facade from `@runanywhere/web`, backend registration packages from `@runanywhere/web-llamacpp` and `@runanywhere/web-onnx`, and browser helpers from `@runanywhere/web/browser`.

The example may break when the SDK facade changes; update it to the latest API rather than preserving old compatibility imports.

## Architecture and Dependency Rules

**The SDK is consumed remotely, from npm — always.** The four
`@runanywhere/*` packages are ordinary `dependencies` in `package.json`, and
`npm install` is the only mechanism that decides which SDK version this app
runs against. There are no source aliases, `paths` mappings, `--prefix`
scripts, or `fs.allow` whitelists pointing at any SDK checkout, and none may be
reintroduced: to test an unreleased SDK build, `npm install` a packed tarball
or `npm link`, never an alias. Both the TypeScript modules and every WASM
runtime artifact come out of `node_modules/@runanywhere/*`.

The demo consumes exactly three publishable Web packages:

- `@runanywhere/web` for backend-neutral SDK lifecycle and public inference
  facades; `@runanywhere/web/browser` is its browser-helper entrypoint.
- `@runanywhere/web-llamacpp` for LLM/VLM backend registration and CPU/WebGPU
  execution variants.
- `@runanywhere/web-onnx` for ONNX/Sherpa STT, TTS, and VAD registration.

Views may import the public roots and `@runanywhere/web/browser`. They must not
import `@runanywhere/web/internal`, `@runanywhere/web/backend`, deep-import package source, import one
backend from another, or implement SDK model routing/storage/inference rules in
UI code. Put reusable SDK behavior in the lowest applicable SDK package and
keep each view focused on DOM state and user-flow orchestration.

## Types, Inputs, Errors, and Credentials

- Keep strict TypeScript. No `any`, `@ts-ignore`, raw JSON assumptions, or
  hand-written copies of proto DTOs/enums. Use generated
  `@runanywhere/proto-ts` types for models, lifecycle, events, storage,
  modalities, environments, and errors. Use local discriminated unions only
  for browser UI state.
- Treat settings, localStorage, IndexedDB, files, URLs, media, model downloads,
  and network/JSON responses as external input. Validate and narrow before
  calling the SDK; show structured, actionable errors without exposing stack
  traces. Chat history is persistent, origin-scoped IndexedDB data; the current
  Web RAG index is session-only and must not be presented as persistent. Keep
  app-owned chat records in IndexedDB; `RunAnywhere.storage` owns model
  artifacts and storage analysis, not arbitrary application records.
- Never log or persist API keys/tokens. API keys entered in Settings are
  session-only and are sent directly by the browser. Persist only explicitly
  allowlisted non-secret settings, and validate them before use. The configured
  endpoint must explicitly support browser CORS.
- This example is intentionally static and client-only. Do not add `api/`,
  `server/`, serverless functions, proxies, embedded credentials, or secret
  environment variables. A developer who needs secret-bearing control-plane
  calls must build, authenticate, secure, rate-limit, and deploy that backend
  outside this example, then expose an appropriate browser-facing contract.
- UI copy and controls must be truthful. Render distinct typed idle, loading,
  ready, success, unavailable, cancelled, and error states. Never show a fake
  toggle, treat a download as inference success, or silently label a failed
  backend/model as ready.

## Design System

Brand primary is `#FF6900` (the logo orange), with the brand gradient
`linear-gradient(135deg, #FF6900, #FB2C36)`. The canonical palette, typography,
and contrast rules live in `../../DESIGN_GUIDELINE.md`; this app hand-maintains
its mirror of those values as CSS custom properties in
`src/styles/design-system.css` (the single token layer — `commons.css` and
`components.css` consume the variables). Light/dark theming works via
`:root[data-theme="light"|"dark"]` for the explicit toggle plus
`@media (prefers-color-scheme: light)` when no explicit choice was made. Do not
reintroduce the legacy `#FF5500`/`#E65500` orange or hardcode brand hexes in
views — use the tokens.

## Commands

Run from `examples/web/RunAnywhereAI/`.

Vite 8 requires Node `20.19+` or `22.12+`; the example mirrors that constraint
in its `engines` field. Production output is pinned to Chrome 86 syntax
compatibility in `vite.config.ts` so a Vite major upgrade cannot silently raise
the Web SDK's documented browser floor. This build target does not polyfill
missing browser APIs; WebGPU remains optional and falls back to the CPU backend.

```bash
npm install   # pulls the @runanywhere/* SDK packages (JS + WASM) from npm
npm run lint
npm run typecheck
npm run build
npm run dev   # http://localhost:3000 (COOP/COEP enabled)
```

Production Vercel releases use `npm run release:deploy`. No Vercel secrets,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RunanywhereAI/runanywhere-web](https://github.com/RunanywhereAI/runanywhere-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
