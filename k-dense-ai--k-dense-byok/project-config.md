---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

K-Dense BYOK is a local AI research-assistant app ("Kady") that brings the user's own API keys. It runs natively on macOS, Linux, and Windows. It is one repo with **two** runtime services started together by the cross-platform launcher `start.mjs` (wrapped by `./start.sh` on macOS/Linux and `start.cmd` on Windows):

| Service | Port | Code |
|---|---|---|
| Frontend (Next.js 16 / React 19) | 3000 | `web/` |
| Backend (TypeScript + Pi coding-agent SDK) | 8000 | `server/` |

The backend embeds the **Pi SDK** (`@earendil-works/pi-coding-agent`) and runs a **single flat agent** with built-in tools (`read`/`bash`/`edit`/`write`/`grep`/`find`/`ls`), a `subagent` delegation tool (the [pi-subagents](https://github.com/nicobailon/pi-subagents) extension package), an `interview` clarifying-questions tool (a native re-implementation of [pi-interview](https://pi.dev/packages/pi-interview) — see `server/src/agent/interview.ts`; the form renders inline in the chat UI instead of the package's own browser window), the [pi-web-access](https://pi.dev/packages/pi-web-access) web tools (`web_search`/`code_search`/`fetch_content`/`get_search_content`), live PDF annotation tools (`add_pdf_annotation`/`list_pdf_annotations`/`remove_pdf_annotation`), a hybrid durable Modal tool family (`modal_run`/`modal_submit`/`modal_status`/`modal_wait`/`modal_cancel`/`modal_results`/`modal_submit_batch`; `server/src/agent/modal-tool.ts`), and per-project MCP tools (`.pi/mcp.json`). PDF annotations and Modal jobs are also available to child agents through the vendored `kady-pdf-annotations` and `kady-modal` Pi packages; annotation sidecars use a shared cross-process lock and atomic replacement. Modal jobs are server-owned and restart-recoverable. There is no orchestrator/expert split, no Gemini CLI, and no LiteLLM proxy (all removed in the Pi migration). Models go directly to **OpenRouter** (built-in Pi provider) or **Ollama** (local). Everything runs locally; user data lives in `projects/`.

## Commands

Backend (`cd server` first; Node ≥ 22.19 recommended):

```bash
npm install                 # install deps
npm run dev                 # tsx watch on port 8000
npm run start               # run backend (tsx)
npm run prep                # ensure default project + seed scientific skills
npm run typecheck           # tsc --noEmit
npm test                    # vitest
```

Frontend (`cd web` first):

```bash
npm install
npm run dev                 # Next.js dev server (port 3000)
npm run build               # production build
npm run test                # vitest
```

Full app (both services):

```bash
./start.sh                  # installs deps, seeds skills, starts backend + frontend
start.cmd                   # same, on Windows (both wrap `node start.mjs`)
```

## Architecture: how a turn flows

1. **UI → backend.** A chat tab posts to the TS backend. Each tab carries its own `sessionId` (a Pi JSONL session); requests are scoped to a project via the `X-Project-Id` header (→ `?project` → `kady-project` cookie → `default`), resolved in an `onRequest` hook using `AsyncLocalStorage` (`server/src/scope.ts`).
2. **Sessions.** `server/src/agent/session-registry.ts` holds live Pi `AgentSession` objects (one per tab, ≤10 per project) and persists each as a JSONL file under `projects/<id>/sandbox/.pi/sessions/`. `AuthStorage` + `ModelRegistry` are process singletons (shared OpenRouter key).
3. **Models.** `server/src/agent/models.ts` resolves a model ref (`openrouter/<vendor>/<model>` or `ollama/<name>`) to a Pi `Model`, synthesizing OpenRouter models from `web/src/data/models.json` pricing when not built in.
4. **Streaming.** `POST /sessions/:id/run` calls `session.prompt()` and streams an SSE schema mapped from Pi's `AgentSessionEvent` (`server/src/agent/events.ts`): `text_delta`, `thinking_delta`, `tool_start/update/end`, `turn_start/end`, `error`, a terminal `cost` frame, and `done`. The run body may carry inline **image attachments** (`images: [{data, mimeType}]`, base64; validated by `server/src/agent/prompt-images.ts` — ≤12 × 5MB, png/jpeg/webp/gif) that ride the user message as Pi image blocks so the model sees them directly; the composer collects them via paste/drop/file-picker (`web/src/lib/image-attachments.ts` downscales >3MB client-side), other file types keep the sandbox-upload path, steering stays text-only (image sends mid-run queue instead), and `GET /sessions/:id/history` replays them for reopened tabs.
5. **Cost ledger + budgets.** Pi reports `usage.cost` inline (no async backfill). `server/src/cost/ledger.ts` snapshots `getSessionStats()` before/after each run and appends a row to `projects/<id>/sandbox/.kady/runs/<sessionId>/costs.jsonl` (role `agent`|`subagent`|`compute`). Durable Modal jobs reserve their strict worst-case estimated cost before admission; project summaries expose spent/reserved/committed USD, and every terminal path reconciles the hold to estimated sandbox wall-time. A project `spendLimitUsd` blocks work when the new commitment would exceed the cap.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [K-Dense-AI/k-dense-byok](https://github.com/K-Dense-AI/k-dense-byok) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
