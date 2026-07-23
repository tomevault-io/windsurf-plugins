---
trigger: always_on
description: This module is built as a Claude-drivable workflow editor. Claude can:
---

# Claude integration guide — aso-video

This module is built as a Claude-drivable workflow editor. Claude can:
1. Create workflows by writing JSON files
2. Switch the active workflow in the user's browser
3. Edit nodes live — the canvas flashes yellow on the changed nodes
4. Trigger node runs (Flux / Kling / Captions / etc.)

Treat this file as the runbook before touching the codebase.

## Architecture in one paragraph

The editor is a Vite SPA on `:5190` backed by an Express API on `:5191`. The graph
state is server-authoritative (`server/lib/graphStore.ts`) and pushed to every
connected browser via SSE at `/api/graph/stream`. A file-system watcher monitors
the currently-loaded workflow's JSON (in `aso-video/workflows/` and
`~/.aso-studio/video/workflows/`) — when Claude edits the file, the watcher
hot-reloads it, fires an `external-reload` SSE hint, then broadcasts the fresh
graph. The browser diffs and animates the changes (yellow pulse on
added/changed nodes, ghost-fade on removed). Active workflow name persists
across server restarts in `~/.aso-studio/video/active-workflow`.

## Quick start — typical Claude flow

```bash
# 1. Make sure dev server is up
curl -s -o /dev/null -w '%{http_code}\n' http://localhost:5190
# If 000 — start it: `cd aso-video && npm run dev` (background)

# 2. Author a workflow JSON
# Write to: aso-studio/aso-video/workflows/<name>.json
# Schema: { version: 1, nodes: [...], edges: [...] }

# 3. Switch active workflow — browser updates live, no F5 needed
curl -s -X POST 'http://localhost:5191/api/graph/load-workflow?external=1' \
  -H 'Content-Type: application/json' \
  -d '{"name": "<workflow-name-without-json>"}'

# 4. Live-edit the JSON file — watcher hot-reloads, browser flashes yellow
# (edit nodes/edges in the same file you wrote in step 2)

# 5. Trigger a single node run
curl -s -X POST 'http://localhost:5191/api/graph/nodes/<node-id>/run'

# 6. Or run everything that isn't `done` yet (topological order)
curl -s -X POST 'http://localhost:5191/api/graph/run-all' \
  -H 'Content-Type: application/json' -d '{"force": false}'
```

## API surface — what Claude actually needs

All endpoints live on `http://localhost:5191`. The `?external=1` query (or
`X-Agent-Edit: 1` header) marks the call as Claude-driven; the browser then
animates the diff so the user can watch the change land.

### Workflows
| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/graph/workflows` | List available workflows |
| `POST` | `/api/graph/load-workflow` | Load a workflow by name → updates user's browser |
| `POST` | `/api/graph/save-workflow` | Persist current graph to a named workflow file |
| `DELETE` | `/api/graph/workflows/:name` | Remove a workflow |
| `GET` | `/api/graph/workflows/active` | Get the active workflow name |

### Graph mutations
| Method | Path | Purpose |
|---|---|---|
| `GET` | `/api/graph` | Read the current graph |
| `PUT` | `/api/graph` | Replace the whole graph |
| `POST` | `/api/graph/nodes` | Add a node `{type, position, data}` |
| `PATCH` | `/api/graph/nodes/:id` | Patch a node's `data` partial |
| `DELETE` | `/api/graph/nodes/:id` | Remove a node |
| `POST` | `/api/graph/edges` | Add an edge `{source, sourceHandle, target, targetHandle}` |
| `DELETE` | `/api/graph/edges/:id` | Remove an edge |

### Execution
| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/graph/nodes/:id/run` | Trigger a single node — async, status persists on the node |
| `POST` | `/api/graph/run-all` | Run topologically; body `{force: bool}` to re-run done nodes |
| `POST` | `/api/graph/auto-layout` | Re-layout the canvas |

## Node types & their data shape

The exhaustive list lives in `server/routes/graph.ts:33` (`VALID_TYPES`). Common
ones:

| Type | Inputs (handles) | `data` keys | Run output |
|---|---|---|---|
| `reference-image` | — (upload) | `url`, `label` | passive (`url` is its output) |
| `reference-video` | — (upload) | `url`, `label` | passive |
| `flux-image` | optional `prompt` | `prompt`, `aspectRatio`, `model` (`gpt-image-2` or `flux-1.1-pro`), `quality` | `outputUrl`, `cost`, `status` |
| `tts-voice` | optional `prompt` | `text`, `voice` | `outputUrl` (mp3), `status` |
| `video-gen` | `image_url`, `image_url_2`, …, optional `prompt` | `model` (`kling`/`seedance`/`happy-horse`), `mode`, `resolution`, `prompt` OR `multiShot+shots[]`, `duration`, `audio` | `outputUrl`, `cost`, `elapsed`, `status` |
| `transcribe` | `video` | — | `outputUrl` (passthrough), `words[]`, `cost` |
| `captions` | `video` | `preset`, `fontSize`, `marginV` | `outputUrl`, `cost` |
| `image-overlay` | `video` (base), `image` (overlay) | `start`, `end`, `position`, `fadeMs`, `opacity` | `outputUrl` |
| `video-overlay` | `base`, `overlay` | `start`, `duration`, `keepBaseAudio`, `position`, `fadeMs` | `outputUrl` |
| `split-screen` | `top`, `bottom` | `ratio`, `audioSource` | `outputUrl` |
| `stitch` | `video_a`, `video_b` | — | `outputUrl` (concatenated) |
| `end-card` | `video` | `duration`, `cta`, `subtitle`, `brand` | `outputUrl` (with appended card) |
| `output` | `video` | `label` | passive — terminal marker |

### Run-order rules

- `runNode` refuses to execute when an upstream node has `status !== 'done'`.
  Reference uploads count as "done" once `url` is set.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dsm5e/aso-tracker](https://github.com/dsm5e/aso-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
