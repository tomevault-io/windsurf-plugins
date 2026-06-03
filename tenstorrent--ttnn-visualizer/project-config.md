---
trigger: always_on
description: This file orients coding agents and contributors who work on this repository.
---

# Agent guidance for TT-NN Visualizer

This file orients coding agents and contributors who work on this repository.

## Purpose

TT-NN Visualizer lets users inspect **memory profiler** and **performance profiler** reports produced when using the Tenstorrent **TTNN** library (with TT-Metal). The UI is built around loading, browsing, and analyzing those reports—not general-purpose ML training or arbitrary workloads.

## Where reports come from

Reports can reach the app in several ways:

- **Upload** through the application.
- **Sync from a remote machine** over SSH.
- **Local TT-Metal tree** when the app runs on the same machine as TTNN/TT-Metal: default/conventional locations include paths under the TT-Metal checkout such as `tt-metal/generated/ttnn/reports/` and `tt-metal/generated/profiler/reports/` (memory vs profiler outputs).

When changing ingestion, sync, or path logic, keep these flows and paths in mind.

## Deployment and security posture

The app ships in two first-class shapes, and changes should work in both unless a feature is explicitly local-only:

- **Local install** on the engineer's own machine — full feature set, including local filesystem access, uploads, and remote SSH sync.
- **Hosted** at **ttnn-visualizer.tenstorrent.com** — runs with `SERVER_MODE` enabled, so `@local_only` endpoints return 403 and the frontend hides the matching UI via `getServerConfig()`.

Treat the hosted deployment as **multi-user and untrusted-input**: requests can come from anyone, instances are not mutually trusted, and uploaded payloads must be validated rather than blindly parsed. Authentication is not part of the app's model, so the `@local_only` boundary *is* the security boundary — when adding endpoints, sockets, or data flows, decide consciously whether they're safe under `SERVER_MODE`, and gate genuinely local-only features on both backend and frontend.

## Python environment

- Use a **Python virtual environment** when running or developing the backend.
- Supported versions are Python 3.10–3.14.

Backend package layout lives under `backend/` (e.g. `python -m ttnn_visualizer.app` with `PYTHONPATH=backend`).

## Architecture (high level)

- **Flask** exposes APIs, file/sync plumbing, and gateway-style behavior.
- **React** (Vite-built SPA under `src/`) holds **most product and visualization logic**. Backend changes are often thin routes, proxies, or services—not a second copy of report semantics.

If you mainly work in Python, you still benefit from knowing that many behaviors live in the frontend; for UI-only issues, prefer `src/` and API contracts over growing Flask-only business rules.

## HTTP API conventions

- **Instance scoping:** Report-backed routes (operations, tensors, buffers, metadata, stack-trace, etc.) expect **`instanceId` as a query parameter**. The React app’s `axiosInstance` injects it on every request from session storage / URL; paths do not embed instance IDs.
- **`/api/remote` subtree:** Remote SSH flows are grouped under **`/api/remote/...`**. Canonical names include `POST /api/remote/profiler-reports`, `POST /api/remote/performance-reports`, `POST /api/remote/test`, `POST /api/remote/sync`, `POST /api/remote/use`.
- **GET vs POST:** Read-only stack trace file checks use **`GET /api/remote/stack-trace/test`** and **`GET /api/remote/stack-trace/read`** with `?filePath=...`. **POST** is used where the body carries SSH connection material (folder listing, sync, use, test).

## Running the app from a development checkout

Prerequisites: **Node** + **pnpm** (see `package.json` `engines`) and an activated **Python virtual environment**.

- **`FLASK_ENV=development`** (or unset) — frontend dev. Run **`pnpm dev`** (Vite, hot reload) and **`pnpm flask:start-debug`** in parallel.
- **`FLASK_ENV=production`** — Flask serves the built SPA. Run **`pnpm build`** after frontend changes, then `pnpm flask:start-debug`. Suits Python-focused developers who rarely touch the UI.

## Code quality and linting

### Backend

All **Python** code in this project should satisfy **Black**, **isort**, and **mypy** as configured here. The exact `pnpm` wrappers live in **`package.json`** (for example `flask:lint`, `flask:format`, `flask:mypy`); use those scripts so flags and paths stay consistent with CI.

### Frontend

`tsconfig*`, **ESLint**, **Stylelint**, and **Prettier** are the source of truth for TypeScript/React and stylesheet work (`.css`, `.scss`, `.sass`). Match the style already in the file you're editing and keep typing strict. Don't add lint suppressions unless explicitly requested — assess whether the warning is right first (it usually is). Format only the code you touched; don't reformat unrelated paths. All frontend changes should pass **`pnpm lint`** (`pnpm lint:fix` for auto-fixes).

### SPDX

New source files need a valid SPDX header in the project format with the **current year**. Validate with **`pnpm lint:spdx`**. Don't bump the year when editing existing files.

## Repository and issue tracking

Development happens on GitHub under:

- **Organization:** `tenstorrent`
- **Repository:** `ttnn-visualizer`

When looking up **issues, pull requests, or releases**, use **github.com/tenstorrent/ttnn-visualizer** as the canonical source.

### Pull request base branch


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tenstorrent/ttnn-visualizer](https://github.com/tenstorrent/ttnn-visualizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-03 -->
