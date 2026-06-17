---
trigger: always_on
description: `editorctl` is the Monet CLI. Always try it directly first — Monet keeps it on `PATH` for every shell it spawns and persists the bin dir to `~/.zshrc` / `~/.bash_profile` so it survives subshells and PATH-rewriting wrappers. **If `command -v editorctl` returns nothing**, fall back in order:
---

# AI Video Editor — Agent Control Reference

## ⚠️ FINDING `editorctl` — MANDATORY (read this first)

`editorctl` is the Monet CLI. Always try it directly first — Monet keeps it on `PATH` for every shell it spawns and persists the bin dir to `~/.zshrc` / `~/.bash_profile` so it survives subshells and PATH-rewriting wrappers. **If `command -v editorctl` returns nothing**, fall back in order:

1. `"$HOME/Library/Application Support/Monet/bin/editorctl"` — macOS install (most users)
2. `node /Applications/Monet.app/Contents/Resources/app.asar.unpacked/out/cli/cli/editorctl.js …`
3. `node ./out/cli/cli/editorctl.js …` — dev tree

Never give up after the first "command not found". Prefer `editorctl` over raw `curl localhost:51847` — it always exposes the current command surface and arg shapes.

## ⚠️ OUTPUT FILE NAMING — MANDATORY (read this first)

**Never reuse a filename when editing or regenerating a video or image.** The asset cache holds the previous file by path, so writing to the same name silently shows stale content. Every edit/regenerate must produce a **new unique filename** (e.g. `clip_v1.mp4`, `clip_v2.mp4`, or a timestamp suffix). Applies to all video renders, image generations, canvas exports, and thumbnails. If a target path already exists, append `_v2`, `_v3`, … — do not overwrite.

## ⚠️ AUDIO ON MULTI-CLIP TIMELINES — MANDATORY

When adding audio to a timeline with **more than one video clip**, audio cuts at every clip boundary. Required flow:

1. Tell the user: *"To keep audio from cutting between clips, I'll merge all video clips into one combined video first, then add the audio. OK to proceed?"*
2. Wait for confirmation — do not auto-merge.
3. Concatenate all clips into a single new file (unique filename per the rule above).
4. Replace the multi-clip video track with that merged clip, then add the audio.

Single-clip timelines: skip the merge.

---

## ⚠️ CANVAS MODE — MANDATORY RULES (read this first)

When `activeView=canvas` is reported by the hook or `editorctl get-state`:

**There are EXACTLY 3 canvas options. No others exist.**
1. **Paper.js** — code drawing with vector graphics (`canvas-run-paperjs`)
2. **Matter.js** — physics and animation (`canvas-run-matterjs`)
3. **GPT image 2** — AI-generated image (`generate-image` + `canvas-add-image`)

**REMOVED — do NOT offer these under any name:**
- ~~Design mode~~ — removed
- ~~Editable layers~~ — removed
- ~~Figma-style layout~~ — removed
- ~~Node-based design~~ — removed
- ~~Direct canvas design~~ — removed

When the user asks for something visual in canvas mode, present ONLY these three options using these EXACT labels. Copy them verbatim. Do not invent a fourth option or rephrase option 1 as anything design-related.

---

**API Bridge:** `POST http://localhost:51847` — JSON body `{"command":"<name>","args":{...}}`

```python
import urllib.request, json

def call(command, **args):
    data = json.dumps({"command": command, "args": args}).encode()
    req = urllib.request.Request(
        "http://localhost:51847", data=data,
        headers={"Content-Type": "application/json"}, method="POST"
    )
    return json.loads(urllib.request.urlopen(req).read())["result"]

call("ping")   # → {"status":"ok","version":"1.0.0","port":51847}
call("help")   # → full command list
```

---

## Commands

### Project / Settings
| Command | Args | Returns |
|---------|------|---------|
| `get_project` | — | Full project JSON |
| `get_settings` | — | Model + provider config |

### Assets
| Command | Args | Returns |
|---------|------|---------|
| `list_assets` | — | `[{id, name, path, type, duration, semantic}]` |
| `get_asset` | `assetId` | Single asset with semantic + transcript |
| `import_files` | `paths: string[]` | Imported asset records |
| `transcribe_asset` | `assetId`, `language?` | `{segments}` via Whisper |
| `embed_assets` | `all?: bool` | `{embedded, total}` via text-embedding-3-small |
| `search_media` | `query, limit?` | Cosine similarity search (falls back to keyword) |
| `search_spoken` | `query, limit?` | Substring search within transcribed segments |

### Sequences & Tracks
| Command | Args | Returns |
|---------|------|---------|
| `list_sequences` | — | All sequences |
| `create_sequence` | `name` | New sequence |
| `activate_sequence` | `sequenceId` | Activated sequence |
| `get_tracks` | — | `[{id, name, kind, clipCount}]` for active sequence |
| `add_track` | `kind: video\|audio\|caption` | Updated sequence |

### Clips
| Command | Args | Returns |
|---------|------|---------|
| `list_clips` | `sequenceId?` | All clips sorted by startTime, with trackKind |
| `add_clip` | `assetId, trackId, startTime, duration?, inPoint?` | `{clipId, clip}` |
| `remove_clip` | `clipId` | `{success}` |
| `move_clip` | `clipId, startTime` | `{success}` |
| `trim_clip` | `clipId, inPoint?, duration?, startTime?` | Updated clip |
| `split_clip` | `clipId, time` | Updated sequence |
| `duplicate_clip` | `clipId, offsetSeconds?` | `{clipId, clip}` |
| `update_clip_label` | `clipId, label` | `{success}` |


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Monet-AI-Editor/Monet](https://github.com/Monet-AI-Editor/Monet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
