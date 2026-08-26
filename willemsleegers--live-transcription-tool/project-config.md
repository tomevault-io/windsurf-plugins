---
trigger: always_on
description: A **privacy-focused live audio transcription web app** built with Next.js, React, TypeScript, and Tailwind CSS. All transcription and summarization happen **on-device in the browser** — no data leaves the user's device.
---

# CLAUDE.md

## Project Overview

A **privacy-focused live audio transcription web app** built with Next.js, React, TypeScript, and Tailwind CSS. All transcription and summarization happen **on-device in the browser** — no data leaves the user's device.

### Core functionality

- **On-device transcription** using Whisper (large-v3-turbo) via Transformers.js + WebGPU
- **Document-based**: the app is a library of saved transcripts; each document is either a rich-text **editor** document or a speaker-segmented **segments** document. A document's type is chosen at creation and never changes.
- **Near-real-time** transcription via AudioWorklet VAD chunking
- **Browser-based AI summarization** (Chrome Summarizer API + WebLLM fallback)
- **Export** to TXT, JSON, and DOCX
- **localStorage persistence** for documents

### Primary language

Dutch (`nl`). UI copy is in Dutch.

## Architecture

### The document model

The app revolves around `TranscriptDocument` (see [lib/types.ts](lib/types.ts)). A document's **`type` (`"editor"` | `"segments"`) is fixed at creation** — there is intentionally **no in-app conversion or mode switching** because the two document types cannot be converted without loss of information.

### App shell and routing

[app/page.tsx](app/page.tsx) is a thin shell that:

- Loads the Whisper model **once** at app level (so switching documents never reloads the ~1.5GB model).
- Auto-loads the model **only if it is already in the browser's Cache Storage** (`isModelCached()`); first-time users start the download themselves. No surprise download.
- Detects WebGPU via `useSyncExternalStore` (SSR-safe, no effect/setState).
- Routes between two views based on `useDocuments().currentDocument`:
  - `null` → [components/DocumentLibrary.tsx](components/DocumentLibrary.tsx) (home: create / open / rename / delete)
  - set → [components/DocumentWorkspace.tsx](components/DocumentWorkspace.tsx) (editing UI for one fixed-type document)

### Data flow for transcription

1. `ControlDock` record toggle → `DocumentWorkspace` `startRecording()` (`useAudioCapture`).
2. `useAudioCapture` runs VAD in the AudioWorklet ([public/audio-processor.js](public/audio-processor.js)); emits 2–10s audio chunks.
3. Chunks are queued serially and sent to the Web Worker via `transcribe()`.
4. [app/whisper-worker.ts](app/whisper-worker.ts) runs Whisper on WebGPU, filters hallucinations, returns text (with a 1-retry-then-reset error policy).
5. `DocumentWorkspace.handleAudioChunk` routes the text by document type: `editor` → `editorRef.appendText()`; `segments` → append a new `TranscriptSegment`.
6. Changes flow back up via `onUpdateDocument(patch)` → `useDocuments.updateDocument` → debounced localStorage write.

## Important notes

### React Compiler enabled (CRITICAL)

`reactCompiler: true` in [next.config.ts](next.config.ts). **Do NOT add `React.memo()`, `useMemo()`, or `useCallback()` for performance** — the compiler handles memoization. (You'll see plain handler functions in `DocumentWorkspace`; that's intentional.) Manual optimization is only appropriate for algorithmic/data-structure changes, debouncing, or direct DOM/Web-API work (AudioWorklet, ProseMirror transactions) — not React-level memoization.

### ⚠️ Webpack, not Turbopack (CRITICAL)

Dev and build use `--webpack`. Transformers.js (v3) has Turbopack compatibility issues in Web Workers (`process.env` undefined). [app/whisper-worker.ts](app/whisper-worker.ts) also includes a `process.env` polyfill. Re-test Turbopack only when Transformers.js supports it (see Transformers.js issue #1026); until then keep `--webpack`.

[next.config.ts](next.config.ts) also aliases `sharp$`/`onnxruntime-node$` to false and strips `node:` prefixes for browser builds.

### Tailwind CSS v4

- Theme via `@theme inline` in [app/globals.css](app/globals.css) (no `tailwind.config.js`).
- Tokens are **OKLCH**, not HSL. Canvas consumers (WaveSurfer) can't use `var(--token)` — resolve to a concrete value with `getComputedStyle` (see `AudioWaveform`).
- Dark mode via `@custom-variant dark`.

### shadcn/ui

- Config: [components.json](components.json) (style "new-york", icons lucide-react). Add components with `npx shadcn@latest add <name>`.

### ESLint

- Flat config ([eslint.config.mjs](eslint.config.mjs)), Next core-web-vitals + TypeScript.
- The `react-hooks` rule set flags **synchronous `setState` inside an effect** as an error. Prefer `useSyncExternalStore` / lazy init for capability detection (see `useWebGPUSupported` in page.tsx).

## Data models

```typescript
type DocumentType = "editor" | "segments"

interface TranscriptDocument {
  id: string
  name: string
  type: DocumentType // fixed at creation
  createdAt: number
  updatedAt: number
  editorContent: JSONContent | null // used when type === "editor" (TipTap JSON)
  segments: TranscriptSegment[] // used when type === "segments"
  speakers: Speaker[] // per-document; used when type === "segments"
}

interface TranscriptSegment {
  id: string
  text: string
  speaker: number | null // speaker id, null = unassigned
  timestamp: number // absolute (Date.now())
  isEdited: boolean

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WillemSleegers/live-transcription-tool](https://github.com/WillemSleegers/live-transcription-tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
