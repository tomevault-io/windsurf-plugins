---
trigger: always_on
description: A production-style non-linear video editor (Premiere-style) that runs in the
---

# FableCut — browser video editor, drivable by Claude Code

A production-style non-linear video editor (Premiere-style) that runs in the
browser. An AI agent edits videos by **editing `project.json`** (or calling the
REST API / MCP tools) — the open browser UI live-reloads within ~150 ms via SSE.
No build step, no npm dependencies.

**This file is the master manual.** Any model pointed at this document (or at
the `fablecut_docs` MCP tool, which returns it) has everything needed to fully
drive the editor.

## MCP connection (preferred — works from any session, any directory)

Register the MCP server (`mcp-server.js`) once at user scope as `fablecut`:
`claude mcp add -s user fablecut -- node "<path-to>/fablecut/mcp-server.js"`.
Every Claude Code session then has these tools:

- `fablecut_status` — auto-starts the editor server, returns URL + project summary. Call first.
- `fablecut_docs` — returns this document (`section: "…"` returns only matching `## ` sections).
- `fablecut_get_project` / `fablecut_set_project` — read / replace the timeline JSON.
  `fablecut_get_project {compact:true}` returns a one-line-per-clip summary instead.
- `fablecut_patch_project` — apply targeted ops (add/update/remove clip/media,
  set project fields) without round-tripping the document. **Prefer this for edits.**
- `fablecut_import_media` — copy a local file into `./media/` and register it.
- `fablecut_analyze_reference` — turn a reference video into an edit blueprint
  (shots, beats, BPM, energy, drop) + extract its music. See "Remake a reference video".

### Token-efficient editing (important for agents)

Editing via full get→modify→set costs thousands of tokens per change. Cheaper:

1. **Plan** from `fablecut_get_project {compact:true}` (≈10× smaller than the JSON)
   and `fablecut_status` — fetch the full JSON only to inspect exact keyframes.
2. **Edit** with `fablecut_patch_project` ops — send only what changes, e.g.
   `{ops:[{op:"updateClip", id:"c_v2", set:{props:{filterPreset:"noir"}}}]}`.
   It re-reads the latest document internally, so it is merge-safe by design
   (no CONFLICT dance) and never destroys concurrent UI tweaks.
3. **Docs**: request `fablecut_docs {section:"props"}` (or "Recipes", "Remake", …)
   instead of the whole manual; skip it entirely if the schema is already in context.
4. **Media questions** (duration, fps, size): read them from the registered media
   entries — don't shell out to ffprobe; the browser probes and writes them back.
5. Batch related changes into ONE patch call (ops apply in order, one revision bump).

**`fablecut_set_project` is conflict-checked.** The MCP server remembers the
`revision` from the most recent `fablecut_get_project` call. If `project.json`
has been written by anyone else since that read (e.g. the user dragged a clip in
the UI), `fablecut_set_project` refuses with a "CONFLICT — not saved" error
instead of overwriting. Protocol:

1. `fablecut_get_project` → read the document and note its `revision`.
2. Apply your edits in memory, bump `revision`.
3. `fablecut_set_project` → if it succeeds you're done.
4. **On conflict**: call `fablecut_get_project` again to get the latest document,
   re-apply your intended changes on top of it, bump `revision`, and call
   `fablecut_set_project` again.

Pass `force: true` to `fablecut_set_project` only when the user explicitly
asks to overwrite conflicting changes. `fablecut_import_media` only appends a
new media entry and always merges safely — no conflict check needed.

For Claude Desktop, add to its MCP config:
`{"mcpServers":{"fablecut":{"command":"node","args":["<path-to>/fablecut/mcp-server.js"]}}}`
Direct file editing of `project.json` (below) works too and is equivalent.

## Run

```
node server.js        # → http://localhost:7777
```

Files: `index.html` + `style.css` + `app.js` (editor UI), `server.js` (API + hosting),
`project.json` (the timeline — THE file to edit), `media/` (project footage),
`library/` (default asset library, see below).

## How Claude Code edits a video

1. Ensure the server is running (background: `node server.js`, or `fablecut_status`).
2. Put source files in `./media/` (copy them in, or the user imports via the UI).
3. Read `project.json`, modify `media` / `clips`, **increment `revision`**, write it back.
4. The browser UI (if open) reloads instantly. The user previews/exports from the UI.

Rules:
- **Prefer `fablecut_set_project`** over direct file writes — it detects conflicts
  automatically (see the MCP section above). If you do write `project.json`
  directly, read it **immediately** before writing (never write from a stale read:
  if the user tweaked something in the UI between your read and write, that write
  destroys their changes). The UI detects external changes by revision comparison,
  so a write that does not bump `revision` is invisible to it.
- Make each edit a single atomic write (read → modify → write once), and bump
  `revision` (integer). Partial multi-step edits can be picked up half-finished.
- New media entries may omit `duration` — the browser probes it and writes it back.
  If you need the duration yourself, re-read `project.json` after a second or two,
  or probe with ffprobe.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ronak-create/FableCut](https://github.com/ronak-create/FableCut) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-09 -->
