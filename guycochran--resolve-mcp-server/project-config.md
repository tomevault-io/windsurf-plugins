---
trigger: always_on
description: You are controlling DaVinci Resolve via the MCP tools registered by this server.
---

# DaVinci Resolve MCP Server

You are controlling DaVinci Resolve via the MCP tools registered by this server.

## Project Status

**Built and verified working.** 53 tools across 11 categories, plus Moondream AI vision.

### What's been tested end-to-end:
- Server starts in stdio mode (for Claude Desktop) and HTTP mode (for remote/mobile)
- All 53 tools register successfully
- Resolve connection works — reads product name, version, current page, project name
- Project listing and loading works (tested with "tutorial - clawdbot")
- Timeline listing works (9 timelines found with track counts)
- Frame export works — `ExportCurrentFrameAsStill()` from the Color page
- Moondream vision works — frame exported as 6MB PNG, converted to 253KB JPEG via Pillow, sent to Moondream API, got back detailed scene description
- **Clip transform** — zoom punch-in (1.2x) on interview clip, verified and reset
- **Markers** — added colored markers with AI-generated descriptions at specific timecodes
- **Clip replacement** — three-point overwrite edit: deleted b-roll clip on V2 and inserted replacement at same record position (AOA_CLIP_04 swapped with SOUTH_POLE_TAKE_OFF)
- **Clip deletion** — delete with optional ripple, returns frame range info
- **YouTube export** — quick export to Desktop using YouTube 1080p preset (3.4MB output)
- **Render with optimized media** — fallback when full-res decode fails (`UseOptimizedMedia: True`)
- **Media pool browsing** — listed 28 clips with durations and metadata
- **Track inspection** — listed all clips on all tracks with frame ranges
- **HTTP transport** — running on port 3001 via Cloudflare tunnel at `resolve.cochran.cloud`

### What still needs real-world testing:
- Color grading tools (LUT application, color versions)
- Title tools (Fusion Text+ insertion)
- Fusion tools (comp access)
- Media import tools
- Speed change tool
- Compound clip creation

## Architecture

```
resolve-mcp-server/
├── src/
│   ├── server.py                    # FastMCP server, stdio + HTTP transport
│   ├── services/
│   │   ├── resolve_connection.py    # Resolve API connection, helpers
│   │   └── moondream.py             # Moondream vision API client
│   └── tools/                       # 11 tool modules, each exports register(mcp)
│       ├── connection.py            # 4 tools
│       ├── project.py               # 6 tools
│       ├── timeline.py              # 8 tools
│       ├── media.py                 # 5 tools
│       ├── editing.py               # 7 tools
│       ├── color.py                 # 6 tools
│       ├── markers.py               # 3 tools
│       ├── titles.py                # 2 tools
│       ├── render.py                # 6 tools
│       ├── fusion.py                # 3 tools
│       └── vision.py                # 3 tools
├── .env                             # MOONDREAM_API_KEY (do NOT commit)
├── .env.example
├── .venv/                           # Python 3.14 virtualenv (Homebrew)
├── requirements.txt                 # mcp[cli], httpx, Pillow
├── start.sh                         # Launcher for Claude Desktop
├── CLAUDE.md
└── README.md
```

## Quick Reference

- **Always check status first** with `resolve_get_status` to see what project/timeline is active
- **Switch pages** before page-specific operations: `resolve_open_page("color")` before applying LUTs or exporting frames
- **Refresh LUTs** before applying: the `resolve_apply_lut` tool handles this automatically
- **mediaType=1 for b-roll**: When appending clips as b-roll, use `media_type=1` to add video only (preserves interview audio)
- **Timeline start frame is typically 86400** (01:00:00:00 at 24fps)
- **Frame export requires Color or Edit page** — switch with `resolve_open_page("color")` first

## Tool Categories

| Category | Tools | What they do |
|----------|-------|-------------|
| Connection | 4 | Status, page navigation, reconnect |
| Project | 6 | List/load/save/create projects, settings |
| Timeline | 8 | List/switch timelines, playhead, tracks, create |
| Media | 5 | List clips, import, create bins, append to timeline |
| Editing | 7 | Transform, speed, enable/disable, compound clips, delete, replace |
| Color | 6 | LUTs, color versions, export grades |
| Markers | 3 | Add/get/delete markers |
| Titles | 2 | Insert Fusion titles, modify text |
| Render | 6 | Quick export, render jobs, timeline export |
| Fusion | 3 | Fusion comp and tool management |
| Vision | 3 | AI frame analysis (Moondream) |

## Common Workflows

### "Apply a film look to all clips"
1. `resolve_open_page("color")`
2. For each clip, navigate playhead and `resolve_apply_lut("Film Looks/Rec709 Kodak 2383 D65.cube")`

### "Export for YouTube"
1. `resolve_quick_export("YouTube", output_dir="/path/to/output")`

### "What's in this shot?"
1. `resolve_open_page("color")` (needed for frame export)
2. `resolve_describe_frame()` — exports frame, converts to JPEG, sends to Moondream AI

### "Replace b-roll clip on V2"
1. `resolve_get_track_items("video", 2)` — find the clip to replace (note its clip_index)
2. `resolve_list_media()` — find the replacement clip name in the media pool
3. `resolve_replace_clip(track_index=2, clip_index=3, new_clip_name="NEW_CLIP.mov")` — replaces in place

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [guycochran/resolve-mcp-server](https://github.com/guycochran/resolve-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
