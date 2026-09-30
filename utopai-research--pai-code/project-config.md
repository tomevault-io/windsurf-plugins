---
trigger: always_on
description: This file is dev-only. The per-project filmmaking agent's operating manual is at `agent-templates/PROJECT_AGENT.md` (copied into each project as `projects/<id>/PROJECT_AGENT.md`).
---

# pai-pro — repo maintainer guide

This file is dev-only. The per-project filmmaking agent's operating manual is at `agent-templates/PROJECT_AGENT.md` (copied into each project as `projects/<id>/PROJECT_AGENT.md`).

When you `cd pai-pro && codex` to work on this repo, this is the file Codex auto-loads. Per-project Codex sessions also load a narrower `projects/<id>/AGENTS.md` wrapper that points at `PROJECT_AGENT.md`; for filmmaking work, treat that project-local wrapper as authoritative.

## Maintaining this repo

Below is for when you're editing the pai-pro repo itself, not running a project session. Skip if the user is asking for filmmaking work.

### Editing principles

A few rules of thumb when modifying source — bias toward caution over speed.

- **Surgical edits.** Every changed line should trace to the request. Don't "improve" adjacent code, refactor things that aren't broken, or delete unrelated dead code (mention it, don't remove it).
- **Simplicity first.** No speculative features, no abstractions for single-use code, no configurability that wasn't asked for, no error handling for impossible scenarios. If 200 lines could be 50, rewrite.
- **Surface confusion early.** State assumptions before implementing. If multiple interpretations of the request exist, present them — don't pick silently. Stop and ask when something's unclear.
- **Goal-driven, verifiable.** For non-trivial changes, define what "done" looks like upfront (a test that passes, a curl that returns 200, a visible UI behavior). Loop on that signal, not on intuition.

Spirit borrowed from [Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

### Architecture

- `server/local_viewer.js` — single Node server. Project CRUD, pty spawn for each project's owning agent (cwd = `projects/<id>/`), canvas file watcher, Socket.IO push to the browser. Routes: `/projects` (list / create), `/projects/:id` (bundle), `/projects/:id/activate`, `/projects/:id/positions`, `/projects/:id/group-frames/...`, `/projects/:id/nodes/...`. Socket events: `canvas-state`, `canvas-positions`, `title`, `pending-generations`, `pty:spawned` / `pty:output` / `pty:exit` / `pty:error`.
- `server/cli/*.js` — synchronous CLI wrappers (image, video, voice, split, switch_project, reel_stitch). Each prints one `{ ok, ... }` JSON line on stdout; non-zero exit with `{ ok: false, klass, message }` on failure. Shared arg parser + emit helpers in `server/cli/_cli.js`.
- `server/pai_*.js` — PAI media API clients imported by the CLIs:
  - **Shared HTTP**: `pai_client.js` (auth, retry policy, classified errors, `callGenerate` / `callSubmit` / `pollStatus`).
  - **Image**: `pai_image_client.js`.
  - **Image Pro**: `pai_image_pro_client.js`.
  - **Video**: `pai_video_client.js` (upstream payload forwarded byte-for-byte; async submit + poll).
  - **Voice**: `pai_voice_client.js` (PAI raw `tts`, `body_base64`-decoded).
  - **Asset uploads**: `pai_assets_client.js` (`video-generation-assets` raw; chip-UX cache + event-emitter surface — exports `paiAssetEvents`, `snapshotAssetStates`, `seedAssetCache`, `uploadReferenceUrl`, `preuploadReferenceUrl`, `preuploadCanvasUrl`, `uploadReferences`).
  - `local_mirror.js` handles the project-side I/O (write bytes, build viewer URLs, resolve refs to data URIs).
- `web/src/` — React + Vite + React Flow + Socket.IO client.
- `skills/*` — local skills. `./scripts/setup --agent codex` validates the Codex CLI; Codex-owned projects get project-local symlinks under `.agents/skills/`. Skill-authoring rules live at `skills/CLAUDE.md` until that subtree gets a Codex-named guide too.
- `agent-templates/PROJECT_AGENT.md` — canonical per-project agent operating manual. `server/services/projects.js` copies it into `projects/<id>/PROJECT_AGENT.md` at project create time, alongside provider-specific wrapper files.
- `projects/<id>/` — runtime project data. Gitignored. Created via `POST /projects` or by `local_viewer.js`'s bootstrap on first run. Each contains `workflow.json`, `meta.json`, `assets/{images,videos,audios,notes,.tmp}/`, `canvas_positions.json`, `PROJECT_AGENT.md`, and provider-specific files (`CLAUDE.md` / `.claude/` for Claude, `AGENTS.md` / `.agents/` for Codex).

### When adding a new media CLI

1. Add a new `pai_<x>_client.js` wrapping `callGenerate({ model: "<pai-raw-model>", payload, ... })` (sync) or `callSubmit + pollStatus` (async). Decode the upstream model's response shape and return `{ bytes, mime, model, durationSeconds, costUsd }` so the CLI is decode-agnostic. See `pai_image_client.js` for the sync template, `pai_video_client.js` for async.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Utopai-Research/pai-code](https://github.com/Utopai-Research/pai-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
