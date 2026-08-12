---
trigger: always_on
description: MCP server that reads/writes Final Cut Pro XML (FCPXML) files. 7 grouped tools (`inspect`, `diagnose`, `edit`, `mark`, `generate`, `transcript`, `deliver`) are advertised by default, dispatching into 62 underlying operations for timeline analysis, batch editing, QC, generation, multi-track support, media relink, NLE export, transcript-based editing (local Whisper), and LIVE FCP control (push_to_fcp / list_fcp_libraries via Apple events). Set `FCP_MCP_LEGACY_TOOLS=1` to advertise the original 62 
---

# CLAUDE.md — fcp-mcp-server

## What This Is

MCP server that reads/writes Final Cut Pro XML (FCPXML) files. 7 grouped tools (`inspect`, `diagnose`, `edit`, `mark`, `generate`, `transcript`, `deliver`) are advertised by default, dispatching into 62 underlying operations for timeline analysis, batch editing, QC, generation, multi-track support, media relink, NLE export, transcript-based editing (local Whisper), and LIVE FCP control (push_to_fcp / list_fcp_libraries via Apple events). Set `FCP_MCP_LEGACY_TOOLS=1` to advertise the original 62 flat tools alongside the groups (additive — 69 advertised tools, not 62). Reads FCPXML 1.8–1.14 (incl. `.fcpxmld` bundles with sidecar preservation), writes 1.13 by default. Dual-mode (XML + Live) direction: `docs/CAPABILITY-AUDIT-2026-06.md`.

## Architecture

```
server.py           — MCP server entry point. All 62 tool definitions, handlers, resources, prompts.
                      Dispatch dict pattern: TOOL_HANDLERS maps tool names → async handler functions.
TOOL_GROUPS         — 7 grouped verbs advertised by default. Dispatch into
                      TOOL_HANDLERS, which still holds all 62 flat handlers.
                      Hiding a tool from list_tools does NOT stop it dispatching.
fcpxml/preview.py   — standalone HTML timeline render, served as preview://<path>
skill/              — the final-cut-pro Claude Code skill wrapping this server

fcpxml/parser.py    — Reads FCPXML → Python objects (Timeline, Clip, ConnectedClip, Marker, etc.)
                      Parses spine, connected clips (lanes), secondary storylines, gap-attached clips, roles.
fcpxml/writer.py    — Writes modifications back to FCPXML. Handles markers, trimming, gaps, transitions,
                      connected clips, roles, reformatting, silence detection/removal.
fcpxml/rough_cut.py — Generates new timelines from source clips (rough cuts, montages, A/B rolls).
fcpxml/diff.py      — Timeline comparison engine. Detects added/removed/moved/trimmed clips & markers.
fcpxml/export.py    — DaVinci Resolve FCPXML v1.9 export + FCP7 XMEML v5 export for cross-NLE workflows.
fcpxml/models.py    — Data classes: TimeValue, Timecode, Clip, ConnectedClip, CompoundClip, Timeline, etc.
fcpxml/media_intel.py — Real media analysis (v0.10). Audio silence detection + beat detection (librosa, optional
                      [intelligence] extra, lazy import). Silence via bounded ffmpeg
                      subprocess (silencedetect), source→timeline mapping; removal composes
                      writer.cut_clip_ranges (element-based — immune to duplicate-name ambiguity). No new Python deps;
                      degrades gracefully (returns None) when ffmpeg is absent.
fcpxml/dtd.py       — Validates output against Apple's official DTDs (located inside the installed FCP app bundle;
                      xmllint needs the DTD path as a percent-encoded file:// URI — spaces in "Final Cut Pro.app" break it).
```

## Key Patterns

- **TimeValue**: All times are rational fractions (numerator/denominator) matching FCPXML's `"600/2400s"` format. Never use floats for time math.
- **_parse_project()**: Helper that parses FCPXML and returns `(tree, timeline, project)` tuple. Most handlers start with this.
- **generate_output_path()**: Creates `_modified`, `_chapters`, etc. suffixed output paths so originals aren't overwritten.
- **Tool handlers**: Each tool has its own `async def handle_<name>(arguments: dict)` function. All return via `_text_result(text)` which wraps strings in the MCP `TextContent` list.
- **Connected clips**: Clips with `lane` attribute hang off spine clips. Positive lane = above (video), negative = below (audio). Secondary `<storyline>` elements also contain connected clips.
- **XMEML export**: Converts spine-based model to track-based model. Primary storyline → Track 0, connected clip lanes → higher tracks.

## Running

```bash
uv run server.py                    # Start MCP server
uv run --extra dev pytest tests/ -v # Run tests
```

## Pre-Commit (MANDATORY)

Before committing ANY changes, run both:
```bash
ruff check . --exclude docs/   # Lint — must pass with zero errors
pytest tests/ -v               # Tests — all must pass
```
CI runs both on every push to main. If either fails, the commit gets an X on GitHub. Fix lint errors before committing, not after.

## Testing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DareDev256/fcp-mcp-server](https://github.com/DareDev256/fcp-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
