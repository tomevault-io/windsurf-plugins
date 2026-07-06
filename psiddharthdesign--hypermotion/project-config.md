---
trigger: always_on
description: hyper-motion ships a CLI and a Model Context Protocol (MCP) server so you
---

# Using hyper-motion from the terminal and from AI agents

hyper-motion ships a CLI and a Model Context Protocol (MCP) server so you
can drive renders from a script, a CI job, or an AI coding agent. This
document is the canonical guide.

The CLI can render the scene currently loaded in the desktop app, create
`.hype` scene files from JSON, inspect saved scenes, and render saved
scene files.

---

## Install

### 1. Install the desktop app

Download the latest `.dmg` from
[github.com/psiddharthdesign/hypermotion/releases](https://github.com/psiddharthdesign/hypermotion/releases)
and install it. v0.1.x is unsigned — after dragging to `/Applications`,
run:

```sh
xattr -cr /Applications/hyper-motion.app
codesign --force --deep --sign - /Applications/hyper-motion.app
```

Then double-click. Full reasoning + alternatives in the
[main README's install section](https://github.com/psiddharthdesign/hypermotion#install-macos).

Windows build is not shipping yet — track upcoming platform support in releases.

The CLI uses the installed desktop app to actually run renders — it
doesn't ship its own render engine. If the app isn't installed, the CLI
exits with a clean message pointing you back here.

### 2. Install the CLI

```sh
pnpm add -g @psiddharthdesign/hypermotion
# or
npm install -g @psiddharthdesign/hypermotion
```

This puts two binaries on your `$PATH`:

- `hypermotion` — interactive CLI for humans.
- `hypermotion-mcp` — MCP server for AI agents.

Requires Node 20+.

### 3. Verify

```sh
hypermotion --version
# prints the installed CLI version

hypermotion render --help
```

---

## CLI — direct human usage

### Render the current scene

```sh
# Open the desktop app, design a scene, let it auto-save (IndexedDB).
# Then from terminal:
hypermotion render -o demo.mp4
```

The CLI spawns the desktop app off-screen (or in the background if your
editor is already open — see [How it works](#how-it-works)), drives the
existing export pipeline, and writes the file.

### Format and quality

```sh
# Format inferred from the extension
hypermotion render -o demo.webm
hypermotion render -o demo.gif

# Explicit format
hypermotion render -o out.mp4 -f mp4

# Quality presets
hypermotion render -o out.mp4 -q comp     # match scene canvas (fastest, default)
hypermotion render -o out.mp4 -q 720p     # 1280 × 720
hypermotion render -o out.mp4 -q 2k       # 2560 × 1440
hypermotion render -o out.mp4 -q 4k       # 3840 × 2160

# Frame rate
hypermotion render -o out.mp4 --fps 60
```

### Environment variables

`HYPERMOTION_APP_PATH` — override the desktop app's location if it's not
in the standard places. Useful when testing a dev build from your repo.

```sh
export HYPERMOTION_APP_PATH=/Applications/hyper-motion.app/Contents/MacOS/hyper-motion
hypermotion render -o test.mp4
```

`HYPERMOTION_VERBOSE` — set to `1` to stream the desktop app's stderr
through the CLI. Essential for debugging renders that hang or fail.

```sh
HYPERMOTION_VERBOSE=1 hypermotion render -o test.mp4
```

---

## MCP integration — AI coding agents

The CLI ships an MCP server (`hypermotion-mcp`) that speaks the
[Model Context Protocol](https://modelcontextprotocol.io) over stdio.
Any MCP-compatible agent can register it and call its tools.

### Tools registered

| Tool           | Status   | What it does                                                                  |
|----------------|----------|-------------------------------------------------------------------------------|
| `doctor`       | ✅ v0.1.2 | Check desktop app and CLI environment health.                                |
| `get_capabilities` | ✅ v0.1.2 | List supported formats, quality presets, patch ops, and scene features.   |
| `create_scene` | ✅ v0.1.2 | Build a `.hype` scene file from a JSON description. Authoring entrypoint.    |
| `render_scene` | ✅ v0.1.2 | Render the current desktop scene, or a provided `.hype` scene path, to MP4 / WebM / GIF. |
| `info_scene`   | ✅ v0.1.2 | Read a `.hype` file and return canvas, duration, layer/track/section counts. |
| `inspect_scene` | ✅ v0.1.2 | Inspect a saved scene's structural contents.                                |
| `patch_scene`  | ✅ v0.1.2 | Create a patched copy of an existing `.hype` scene.                          |
| `validate_scene` | ✅ v0.1.2 | Validate a `.hype` scene for structural consistency after agent edits.       |
| `list_layers`  | ✅ v0.1.2 | List layer nodes in a saved `.hype` scene.                                   |
| `get_layer`    | ✅ v0.1.2 | Return details for one layer in a saved `.hype` scene.                       |
| `list_tracks`  | ✅ v0.1.2 | List animation tracks in a saved `.hype` scene.                              |
| `list_cameras` | ✅ v0.1.2 | List cameras in a saved `.hype` scene.                                       |
| `open_scene`   | ✅ v0.1.2 | Open a saved `.hype` scene in the desktop app.                               |
| `list_keyframeable_properties` | ✅ v0.1.2 | List property ids that can be animated with tracks/keyframes. |

### Claude Code

Register the server at user scope (available from any project):

```sh
claude mcp add -s user hypermotion -- hypermotion-mcp
```

Add environment variables if you've put the desktop app somewhere
non-standard, or want verbose logs:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [psiddharthdesign/hypermotion](https://github.com/psiddharthdesign/hypermotion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-06 -->
