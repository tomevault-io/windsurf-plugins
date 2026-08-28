---
trigger: always_on
description: Guidance for AI coding agents working in this repository.
---

# AGENTS.md

Guidance for AI coding agents working in this repository.

`README.md` covers what the app does and how an end user installs it — read that for the feature
tour. This file covers what you can't get by reading the code: invariants, traps, and the reasons
behind non-obvious choices.

## Architecture

### Backend (`backend/app/`)

`database.py` keeps **dual async/sync engines**: async for FastAPI and the orchestrator control
plane, sync for job bodies running in lane threads and in the ML child process.

**Orchestrator** (`orchestrator/`) — in-process engine for all background work. The parts you wouldn't
get from the filenames:
- `lanes.py` - pop order `(priority, queue_sequence)`, ties by submission order
- `wrapper.py` - `run_job_sync`: before_start → body → on_success/on_cancel/on_retry/on_failure, NOT_READY guard, never-overwrite-CANCELLED guard, downstream chaining
- `recovery.py` - startup recovery rebuilds lanes from TaskRecord truth, since Postgres is the only durable task state; plus the due-retry loop
- `subprocess_runner.py` / `child_main.py` - spawned child for transcription; faster-whisper/ctranslate2 load **only there**, so it's SIGTERM-cancellable and memory is reclaimed per job

**Tasks** (`tasks/`) — job bodies are plain functions taking `(ctx, payload)`; `registry.py` wires
bodies/lanes/hooks/policies via `register_all_jobs()`.

`__init__.py` re-exports job bodies and deliberately does **not** import `tasks.transcription` —
that keeps faster-whisper out of the main process.

**`tasks/media.py` — the deferral rules.** Unreleased videos get a visible NOT_READY placeholder
TaskRecord (upserted per URL at populate time, soft-deleted and replaced by a QUEUED chain once the
video airs). "Unreleased" = live or upcoming premiere, or a finished livestream still flagged
`post_live` **whose VOD formats aren't available yet** — yt-dlp can keep `post_live` set for hours
or days after a downloadable VOD exists, so `is_video_ready_for_download` (`ytdlp/info.py`) allows
post-live once real formats are present.

**A deferral also persists a `NOT_READY` `MediaDetails` row** (`_defer_media`) carrying the expected
`release_timestamp` and a `next_check_at`, because the TaskRecord placeholder alone is invisible to
the subscription filter — which keys on `MediaDetails`, so without the row every premiere and every
unavailable video is re-fetched on *every* tick forever. `next_check_at` is a known future premiere
time, else an age-keyed ladder; `_evaluate_not_ready_media` honours it, except for direct downloads,
which always bypass the backoff. Two ladders: a short one for unreleased videos, a much longer one
for videos yt-dlp *positively* reports as gone. `get_url_info` swallows every failure into a `None`,
so `get_url_info_with_failure` + `classify_extraction_error` decide which — and the long ladder is
opt-in by whitelist, since a 429 or bot-check is otherwise indistinguishable from a private video and
parking a rate-limited channel for a week is the worse error.

**Three places must never accept a deferred row as resolved**, or an unreleased video downloads early
using its premiere metadata: the reuse branch and the pre-fetched-metadata shortcut in
`_use_pending_or_fetch_fresh` (both bypass the yt-dlp fetch, hence the readiness check —
`_reuse_or_delete_existing_media` blocks both at once), and the upsert on the way back out. Deferred
rows are updated in place, never delete-and-recreated like SKIPPED: the delete would reset
`created_at`, which the ladder reads. Clearing needs the explicit `sync_clear_deferral` because
`_copy_upsert_fields` refuses to write `status` when it is `NONE`.

`services/embeddings.py` — `OnnxEmbedder`: all-MiniLM-L6-v2 on onnxruntime, replacing sentence-transformers so
**torch is not a dependency at all** (~1 GB off the venv). onnxruntime/tokenizers/huggingface-hub were
already present for faster-whisper's Silero VAD. Reproduces the sentence-transformers pipeline exactly
— transformer → attention-masked mean pool → L2 normalize — so vectors stay interchangeable with those
written by the old torch build (measured: max component delta 2.3e-07, cosine 0.9999999, identical
result ordering). Two things are load-bearing: `max_seq_length` comes from the repo's
`sentence_bert_config.json` (**256**, not the tokenizer's 512 — getting it wrong silently shifts
embeddings), and the ONNX feed dict is filtered by `session.get_inputs()` since exports differ on
whether `token_type_ids` is declared. `resolve_model_repo` hard-rejects any model but
all-MiniLM-L6-v2; a different model means a different vector space, so allowing one would silently
invalidate every stored embedding.

**Models** (`models.py`) — the constraints worth knowing before you write a query:
- `Tag` unique per `(user_id, name)`; `MediaTag` unique per `(user_id, media_details_id, tag_id)`
- `MediaRating` one per `(user_id, media_details_id)`, 1–5 enforced by a CHECK constraint
- `MediaAccess` carries a source type: DIRECT, PLAYLIST, SUBSCRIPTION
- `User` also holds the recovery columns: `password_reset_requested_at`, `must_change_password`, `password_changed_at`, `recovery_code_hash`, `recovery_code_expires_at`

**Repositories** (`repositories/`) — two non-obvious bits:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kkuhlmann/ytdl-hoarder](https://github.com/kkuhlmann/ytdl-hoarder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
