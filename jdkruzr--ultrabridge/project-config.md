---
trigger: always_on
description: Last verified: 2026-07-26 (Transient-failure retry: `FailJob` is no longer the only terminal path. `Processor.handleJobError` classifies via `processor.IsTransient` — OCR failures that never got a verdict from the model (dial/timeout, 5xx, 429, 408) are requeued with exponential backoff through the new `Store.RequeueJob` + the long-dormant `requeue_after` column, up to `maxJobAttempts`; a 4xx or a parse error still fails on the first attempt. Shutdown mid-job now leaves the row `in_progress` for
---

# Boox Pipeline

Last verified: 2026-07-26 (Transient-failure retry: `FailJob` is no longer the only terminal path. `Processor.handleJobError` classifies via `processor.IsTransient` — OCR failures that never got a verdict from the model (dial/timeout, 5xx, 429, 408) are requeued with exponential backoff through the new `Store.RequeueJob` + the long-dormant `requeue_after` column, up to `maxJobAttempts`; a 4xx or a parse error still fails on the first attempt. Shutdown mid-job now leaves the row `in_progress` for the startup reclaim instead of marking a good note failed. `Processor` also gained `Running()` and idempotent Start/Stop with a per-Start `done` channel — it previously panicked on a stop/start/stop cycle.)

## Purpose
Processing pipeline for Boox notes. Orchestrates parse → render → OCR → index → embed workflow
for .note files uploaded via WebDAV, triggered by file uploads.

## Contracts
- **Exposes**: `Store` (UpsertNote, EnqueueJob, ClaimNextJob, CompleteJob, FailJob, RequeueJob, GetNote, ReclaimStuckJobs, RetryAllFailed, DeleteNote, SkipNote, UnskipNote, GetQueueStatus, ListNotesWithPrefix, UpdateNotePath, ReclaimAllInProgress), `BooxNote` and `BooxJob` models, `Processor` interface (Start, Stop, Running, Enqueue), `WorkerConfig` with Indexer, ContentDeleter, OCR interfaces, and embedding interfaces (Embedder, EmbedStore from rag package), `Importer` (ScanAndEnqueue, MigrateImportedFiles) with `ImportConfig` and `ImportResult` types.
- **Guarantees**: Atomic job claiming via SQLite RETURNING. Watchdog reclaims stuck jobs (>10 min in_progress). Graceful shutdown waits for current job, and leaves it `in_progress` so the next Start reclaims it rather than failing it. Content deletion uses ContentDeleter interface to ensure FTS5 triggers fire. Embedding failures are best-effort (logged, do not fail the job). OCR failures DO fail the job (`worker.go` returns `ocr page N: %w`) — they are then classified transient/permanent per the Invariants below.
- **Expects**: SQLite `*sql.DB` with `boox_notes` and `boox_jobs` tables (created by notedb schema migrations). `WorkerConfig` with Indexer, ContentDeleter, and optional OCR, Embedder, EmbedStore.

## Dependencies
- **Uses**: `notedb` schema (boox_notes, boox_jobs tables), `booxnote` (ZIP parser), `booxrender` (page renderer), `processor.Indexer` interface (shared with Supernote processor), optional `processor.OCRClient` (vision API), `search.Store.Delete()` for content deletion, `rag` package (Embedder and EmbedStore interfaces for embedding text)
- **Used by**: `cmd/ultrabridge` (wiring in main), `webdav` handler (Enqueue callback on upload), `web` handler (bulk import and management routes via BooxImporter interface)
- **Boundary**: Does not own file discovery — WebDAV handler passes paths directly. Importer owns bulk discovery from a configured import path. Does not implement Embedder or EmbedStore — those come from `rag` package.

## Key Decisions
- Separate from Supernote processor: Boox notes use different parser/renderer (booxnote/booxrender vs go-sn for .note files; pdftoppm via pdfrender for .pdf files), no RECOGNTEXT injection, different storage format
- PDF support: worker dispatches by file extension — .note files go through booxnote+booxrender, .pdf files go through pdfrender (renderPDFPageScaled with DPI scaling); both paths produce JPEG pages fed into the same OCR+index flow
- Shared Indexer: uses same `processor.Indexer` interface and note_content/note_fts tables as Supernote for unified search
- Atomic job claiming: SQLite RETURNING clause (SQLite 3.35+) for single-statement claim, avoiding race conditions
- Content deletion via interface: `ContentDeleter` allows search.Store to maintain FTS5 triggers on re-process
- Cache lifecycle: old cached JPEGs removed on re-process via `os.RemoveAll` before new renders
- OCR source tracking: "api" if OCR enabled, empty string if OCR disabled (no "myScript" equivalent for Boox)
- resolveMetadata preserves importer-provided metadata (title, author, etc.) when present; only falls back to WebDAV path extraction for files whose path is under the WebDAV root and that have no importer-supplied metadata
- Embedding integration: OCR'd text is embedded via Embedder interface and stored via EmbedStore interface (both from `rag` package). Embedding failures are logged but do not fail the job; allows OCR to proceed if embedding is unavailable.

## Invariants
- Job statuses: pending -> in_progress -> done|failed|skipped (or back to pending via
  ReclaimStuckJobs, or via RequeueJob after a transient failure)
- A transient OCR failure (see `processor.TransientError`) is requeued, not failed:
  `RequeueJob` sets status=pending plus a future `requeue_after`, which ClaimNextJob
  honours. Backoff doubles from 1 min, capped at 30 min, and the job fails terminally

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jdkruzr/ultrabridge](https://github.com/jdkruzr/ultrabridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
