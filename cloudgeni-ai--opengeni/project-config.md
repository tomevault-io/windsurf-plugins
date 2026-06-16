---
trigger: always_on
description: This repository is a clean TypeScript/Bun stack. The public API is session-based.
---

# Agent / automation notes (OpenGeni)

This repository is a clean TypeScript/Bun stack. The public API is session-based.

When the user says **"start the dev server"**, **"spin it up"**, or **"run the full stack"**, they mean the steps under **Full local stack**.

## Full Local Stack

The stack means everything needed to run the Hono API, React web app, Temporal worker, Postgres event store, Core NATS realtime bus, Temporal service, and configured OpenAI Agents SDK sandbox backend.

1. Start the full local stack:

   ```bash
   bun run dev
   ```

   This installs dependencies, starts the Docker infrastructure, runs migrations, builds the local sandbox image, and starts API, worker, and web processes.

Manual equivalent:

1. Install dependencies:

   ```bash
   bun install
   ```

2. Start infrastructure:

   ```bash
   docker compose up -d postgres nats temporal minio minio-init
   bun run db:migrate
   ```

3. Build the local sandbox image when using Docker sandbox:

   ```bash
   docker build -f docker/sandbox.Dockerfile -t opengeni-sandbox:local .
   ```

4. Copy `.env.example` to `.env` and configure at least:

   - `OPENGENI_DATABASE_URL`
   - `OPENGENI_NATS_URL`
   - `OPENGENI_TEMPORAL_HOST`
   - `OPENGENI_STARTUP_DEPENDENCY_RETRY_*` when dependencies need longer startup windows
   - OpenAI or Azure OpenAI credentials
   - `OPENGENI_SANDBOX_BACKEND=docker` or `modal`
   - sandbox preparation profiles / env allowlist when needed

5. Start long-running processes in separate terminals:

   ```bash
   bun run dev:api
   bun run dev:worker
   bun run dev:web
   ```

Default URLs:

- API: `http://127.0.0.1:8000`
- API health: `http://127.0.0.1:8000/healthz`
- Web: `http://127.0.0.1:3000`
- NATS monitor: `http://127.0.0.1:8222`
- Temporal gRPC: `127.0.0.1:7233`

`bun run dev` may auto-select alternate Docker Compose host ports when defaults are already in use; it wires those selected ports into the API and worker environment for that run.

MinIO is the local S3-compatible object storage default for Docker Compose and optional self-contained Kubernetes smoke tests. Production deployments should use provider-native storage instead of deploying MinIO manually: `azure-blob` for Azure Blob, `aws-s3` for AWS S3, and `gcs` for Google Cloud Storage.

## Architecture Notes

- Public clients talk only to the API.
- Browser streaming uses `GET /v1/workspaces/:workspaceId/sessions/:id/events/stream` with SSE.
- Core NATS is the realtime bus between producers and API instances.
- Postgres is the durable event store and replay source.
- Temporal is orchestration only. Token streams do not go through workflow history.
- OpenAI Agents SDK execution happens inside non-retryable worker activities.
- Agent activities are side-effectful. Do not add automatic Temporal retries around full agent turns unless each model/tool/sandbox boundary has been made idempotent.

## Run Lifecycle (read `docs/run-lifecycle.md` before changing the session workflow, the agent turn activity, or memory)

Three principles here are load-bearing and easy to break by accident:

- **No run-length limits, by design.** OpenGeni runs agents that legitimately work for days. Run length is bounded by symptoms (no-progress detection, budget exhaustion), never by counts or clocks. Do not add or lower caps on model calls per turn, continuation count, or activity timeout as a way to "be safe" — fix the pathology instead. See `docs/run-lifecycle.md` and `docs/goals.md`.
- **Three memory stores, three jobs.** `session_history_items` is conversation truth fed to the model (default read path). `agent_run_states` is the serialized RunState blob, used *only* to resume a turn paused for a human approval — never as conversation memory. `session_events` is the redacted, lossy human/audit timeline and must never be fed back to the model. Sandbox recovery state is separate again in `sandbox_session_envelopes`.
- **Goals drive long runs.** An active goal turns "stop" into an explicit act; the continuation loop is replay-safe and lives in the session workflow. See `docs/goals.md`.
- **Worker deaths are survivable, but never via blind retries.** A graceful shutdown (SIGTERM) preempts the in-flight turn: checkpoint, requeue, `preempted` activity result, re-dispatch on a healthy worker via a `turn.preempted` resume notice. An ungraceful death (heartbeat-timeout activity failure) re-dispatches the turn from dual-written conversation truth via `requeueTurnAfterWorkerDeath`, bounded by a per-turn redispatch counter (3) before the session fails for real. Failed sessions are revivable: a new user message transitions failed → queued and restarts the workflow; only cancelled is terminal. Do not replace any of this with automatic Temporal retries of agent turns. See `docs/run-lifecycle.md`.

The agent turn activity is `runAgentTurn`, also registered under the legacy alias `runAgentSegment` for in-flight workflow histories — schedule `runAgentTurn` in new code.

## Keeping these notes current

If a change alters architecture, terminology, the run lifecycle, the memory model, or a "do not" guardrail above, update this file and the relevant `docs/*.md` in the same change. An out-of-date AGENTS.md or doc is a bug, not a nicety.

## Sandbox Notes


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Cloudgeni-ai/opengeni](https://github.com/Cloudgeni-ai/opengeni) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
