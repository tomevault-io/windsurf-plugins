---
trigger: always_on
description: This file gives future coding agents the durable context needed to work safely
---

# OpenReader Agent Guide

This file gives future coding agents the durable context needed to work safely
and effectively in this repository. Read the relevant documents in `v5/`
before changing an architectural boundary. Never place passwords, API keys,
database URLs, or other secrets in this file, commits, logs, or PR text.

## Project

OpenReader is a self-host-friendly Next.js document reader with synchronized
text-to-speech playback for EPUB, PDF, TXT, Markdown, and DOCX. It supports
progressive generation, a durable reusable document-audio timeline, word-level
alignment, PDF layout analysis, previews, and audiobook export.

The v5 architecture is a hard redesign, not a compatibility layer over every
intermediate implementation. Because v5 has not been released yet, delete
superseded v5-only paths rather than retaining duplicate logic or dead fallback
code. Preserve released v4 data and documented upgrade behavior.

The root package version intentionally remains `4.4.0` until the owner has
deployed and smoke-tested the final v5 production build and is ready to create
the `v5.0.0` release. Do not bump it early.

## Architecture Boundaries

- Next.js is the authenticated control plane. It owns users, sessions, SQL,
  authorization, provider configuration, and bounded operation creation.
- The compute worker is the heavy/long-running data plane. It owns parsing,
  layout analysis, TTS generation/alignment, derived artifacts, and cleanup.
- NATS JetStream carries durable operation/job state and events.
- Object storage is SeaweedFS locally or an S3-compatible service externally.
- The worker must not query the application database. It obtains provider
  execution configuration from the authenticated app-owned credential broker.
- Provider credentials must never appear in NATS jobs, operation state, events,
  logs, browser responses, or artifacts.
- Keep the four authentication boundaries distinct:
  1. browser to Next.js: Better Auth session/cookie;
  2. Next.js to worker: `COMPUTE_WORKER_TOKEN`;
  3. browser to worker audio: short-lived HMAC playback token;
  4. worker to app broker: `COMPUTE_CREDENTIAL_BROKER_TOKEN`.
- Existing v4 provider ciphertext remains app-owned and encrypted with
  `AUTH_SECRET`; do not invent a second at-rest key without a complete migration.

Normative architecture and history:

- `v5/PLAYBACK_ARCHITECTURE.md`
- `v5/READER_READINESS_STATE_MACHINE.md`
- `v5/AUTHENTICATION_PLAN.md`
- `v5/COMPUTE_RATE_LIMITING_PLAN.md`
- `v5/CLEANUP_PLAN.md`
- `v5/TEST_MIGRATION_PLAN.md`

## Playback Invariants

Playback behavior matters more than making a test pass or making a file short.
Preserve these contracts:

- A document has one durable canonical playback timeline. Cached/generated
  ranges remain visible and reusable across pause, seek, section changes,
  reloads, and export.
- A seek or resume must not discard the timeline or create a replacement
  playback session merely to restart audio. Refill jobs may change internally,
  but the canonical session/cache identity remains stable.
- Pause, cancellation, and superseded location changes must stop or supersede
  unnecessary forward generation. They must not leave stale generation running.
- Playback should generate far enough ahead to stay smooth under real network
  latency, while remaining responsive to a changed cursor.
- Readiness and progress should use existing SSE state rather than new polling
  loops. Avoid parallel sources of truth.
- The UI must distinguish preparing/loading/buffering from actually playing.
  Never show “playing” during a silent generation wait.
- Word highlighting must follow audio timestamps closely, especially in EPUB.
  PDF highlighting is a useful known-good comparison.
- Audio streaming must remain bounded and backpressure-aware; do not buffer an
  entire document or many whole segments in worker memory.
- Stalled-stream recovery reopens the same session only when the SSE read model
  proves cached audio is available. It must not spin or create new sessions.

The client playback controller was deliberately simplified on `main`: browser
audio lifecycle, seek policy, projection, foreground SSE/cursor ownership, and
recovery have separate owners. Do not reintroduce manual competing processing
flags, independent seek polling, duplicate cursor writers, or scattered teardown.

## Testing Workflow

Commands:

- `pnpm test` runs all Vitest projects.
- `pnpm test:e2e` runs Playwright.
- `pnpm exec tsc --noEmit` checks the application.
- `pnpm --dir packages/compute-worker exec tsc --noEmit` checks the worker.
- `pnpm build` is the production build and lint/type gate.
- `pnpm lint:route-errors` and `pnpm check:compute-boundary` enforce boundaries.

Browser tests were rebuilt from a clean slate. For new or changed user journeys:

1. Run the application and inspect the real visible behavior with computer use.
2. Reproduce the journey through user-facing controls.
3. Fix product defects in product code; do not encode a broken state into tests.
4. Add the smallest useful Playwright assertion, preferring roles and labels.
5. Use targeted runs only while diagnosing. Before completion, run the complete
   Vitest suite and complete Playwright matrix.

Current Playwright policy:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richardr1126/openreader](https://github.com/richardr1126/openreader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-13 -->
