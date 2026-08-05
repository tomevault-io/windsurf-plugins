---
trigger: always_on
description: - You MUST be blunt, direct, and easy to understand.
---

# Repository Guidelines

## Communication Style

- You MUST be blunt, direct, and easy to understand.
- You MUST avoid filler, corporate phrasing, and vague reassurance.
- You MUST stay technically precise.

## Daydream Overview

Daydream is a local-first desktop recall assistant. Its job is to help a user
answer "what did I do today?" from private screen, audio, activity, and window
context recorded on hardware the user controls.

The intended product is an on-demand Tauri or Electron desktop app that can
record a full working day, process eligible local chunks automatically, extract
searchable timeline annotations, task/todo candidates, conversation summaries,
and a daily digest, then let the user cut or hide sensitive spans with derived
data invalidated for those spans.

## RFC 2119 Language

All instructions in this document use RFC-style terms. MUST, MUST NOT, SHOULD,
SHOULD NOT, and MAY must be interpreted accordingly.

## Current Repository State

- This repository is a Yarn 4 monorepo.
- The desktop app lives in `apps/desktop` and uses Tauri 2, React, Vite,
  Tailwind, shadcn-style components, TanStack Router, and Zustand.
- Shared TypeScript config, lint config, diagnostics types, and the development
  FFmpeg resolver live in `packages/`.
- SQLite persistence is owned by the Rust/Tauri backend and initialized on app
  startup.
- Real capture, redaction, OCR, transcription, VLM inference, vector indexing,
  and timeline playback are not implemented yet.

## Engineering Approach

- You MUST solve root causes instead of adding workaround branches.
- You MUST verify important assumptions with repo evidence, targeted commands,
  package metadata, or official docs before depending on them.
- You SHOULD prefer straightforward logic with clear invariants over
  fallback-heavy implementations.
- You MUST keep privacy, local execution, and explicit user control central to
  every architecture decision.
- You SHOULD keep processing steps deterministic and resumable where practical,
  because full-day recordings will be expensive to process.
- You MUST document non-obvious decisions, native permission requirements,
  storage formats, and model/tool tradeoffs as they are introduced.
- You MUST use Yarn via Corepack for JavaScript tooling.
- You SHOULD run `yarn lint:fix` frequently while editing. In this repo,
  ESLint is the main formatting and code-smell gate.

## Target Architecture Direction

- Desktop shell: Tauri 2 is the current scaffold choice.
- Package manager: Yarn 4 stable through Corepack.
- Capture tooling: FFmpeg SHOULD be embedded, vendored, or otherwise managed by
  the app distribution so users are not expected to install it manually. The
  current dev scaffold resolves FFmpeg through `ffmpeg-static`.
- Local metadata store: SQLite.
- Local vector/search index: LanceDB or Qdrant. Keep the index boundary isolated
  so the backend can switch between these without rewriting capture or timeline
  logic.
- AI processing: Whisper-style transcription, OCR on retained frames, VLM
  inference for visual context, and LLM chunk summarization/digest generation.
- Storage: local video/audio/object store with explicit retention and deletion
  semantics.
- UI: timeline playback, searchable annotations, source-linked digest entries,
  and manual scissors/redaction after processing.

## Privacy & Security

- You MUST treat screen recordings, audio, OCR text, transcripts, focused window
  titles, open-window lists, keyboard/mouse activity, and generated summaries as
  highly sensitive personal data.
- You MUST NOT upload captured data, derived text, embeddings, or summaries to a
  remote service unless the user has explicitly enabled that capability and the
  code path is clearly labeled.
- You MUST NOT log raw frames, raw OCR text, transcripts, window titles, audio
  content, embeddings, or model prompts by default.
- You SHOULD prefer local models and local indexing for baseline operation.
- You MUST design deletion as real deletion: if the user cuts or removes a span,
  derived frames, audio chunks, OCR, transcripts, embeddings, and summaries for
  that span must also be removed or invalidated.
- You SHOULD make privacy-sensitive operations auditable through local metadata
  without exposing private content in logs.

## Capture Pipeline Expectations

- The app SHOULD support multi-screen recording.
- The app SHOULD capture both desktop audio and microphone audio, keeping the
  streams distinguishable during processing.
- The default recording concept is roughly 1 FPS with high visual quality and
  FFmpeg tuning that balances OCR accuracy, storage size, and write rate.
- The app SHOULD capture keyboard and mouse activity metadata sufficient to
  infer whether the user was actively using the computer.
- The app SHOULD capture focused window identity, focused window title, and open
  window inventory where the OS permits it.
- The app MUST provide a scissors/redaction workflow so users can remove
  sensitive spans after capture and AI processing; OCR, transcripts, VLM output,
  embeddings, summaries, and search data for removed spans MUST be deleted or
  invalidated.
- Capture and processing SHOULD be decoupled. A full day can be recorded first

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [snackbit/daydream](https://github.com/snackbit/daydream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
