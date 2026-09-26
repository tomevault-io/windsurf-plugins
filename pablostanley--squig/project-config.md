---
trigger: always_on
description: squig is a wireframing tool: an infinite canvas of UI components that render as
---

# Driving squig as an agent

squig is a wireframing tool: an infinite canvas of UI components that render as
a hand-drawn sketch. A document is a flat map of nodes saved as
[`.squig.json`](format.md), with browser and agent edits using the same node model.

Choose the connection that matches where the drawing lives.

1. **The browser door.** For a drawing already open in Squig, use **Connect
   agent** and give its instructions to an agent with access to that same tab.
   Edits through WebMCP or `window.squig` autosave in browser storage. No
   download, installation or companion is needed.
2. **The local companion.** Human and agent edit a chosen disk file in the
   full editor, with MCP or HTTP tools and live updates.
3. **The file CLI.** You have a shell and want to create or edit a file directly.
4. **The library door.** You are writing TypeScript in this repo.

---

## The local companion

Clone this repository, use Node.js 24 and pnpm 10, then build the editor once:

```bash
git clone https://github.com/pablostanley/squig.git
cd squig
pnpm install --frozen-lockfile
pnpm build:local
pnpm squig serve /absolute/path/canvas.squig.json
```

The command prints a local editor URL and connection details. Open that URL
before drawing so the user can watch. A missing file is created; existing
files are validated before editing. The selected file is the source of truth.
Keep the process running. It binds only to `127.0.0.1` and serves the editor,
HTTP tools and MCP from that local origin. No database or account is involved.

If the drawing is already open on squig.sh, keep working in that tab using
the browser door below. Only export a `.squig.json` copy when the user wants
to move the drawing to a disk file, then start the companion for that file.
The website cannot infer its absolute path or silently link browser storage
to a downloaded copy.

### MCP clients

Use a stdio server entry with absolute paths in your client's MCP config:

```json
{
  "mcpServers": {
    "squig": {
      "command": "node",
      "args": [
        "--experimental-strip-types",
        "--disable-warning=MODULE_TYPELESS_PACKAGE_JSON",
        "--import", "/absolute/squig/scripts/register-loader.mjs",
        "/absolute/squig/scripts/squig.ts",
        "mcp", "/absolute/path/canvas.squig.json"
      ]
    }
  }
}
```

Replace both the checkout and document paths. The client starts the companion;
it also serves a local editor. Use the returned editor URL to work together.
Do not launch another `serve` or `mcp` process for the same file. To connect
another client to an already running companion, use that session's HTTP MCP
address and bearer token. Launch Node directly for stdio: package-manager
banners on stdout would corrupt the MCP protocol.

There is no published `npx squig` package. The optional Squig plugin supplies
a workflow skill; the checkout and selected file supply the runtime.

### Full agent tools

MCP prefixes tool names with `squig_`. The local HTTP equivalent is
`POST /api/v1/tools/{name}` with the same JSON input and the session's bearer
token. Use the actual loopback URL printed by the companion, never squig.sh.

- `squig_local_session` over MCP, or `GET /api/local/session` over HTTP: the editor URL, selected file path and connection addresses.
- `catalog`, `documents`, `get_document`: discover components and read the file.
- `edit_document`, `replace_document`: validated, atomic edits using the current revision.
- `history`, `restore`: bounded local snapshots and revision-checked restoration.
- `comment`, `resolve_comment`: feedback stored with the local document.
- `measure_text`, `render_document`, `export_document`: local measurement, SVG/PNG and portable JSON.

Call `squig_local_session` for the editor URL and connection details.
Start with `documents`, read `get_document`, then return the editor URL before
editing. Use explicit node IDs and small batches. Re-read after a stale revision
error; revision tokens describe content and must not be incremented by the client.
No-op saves add no snapshots. The companion keeps at most 50 history entries
and 16 MiB of history beside the file in a `.squig.json.history` directory;
older history expires. Save a separate copy or
use your own backup tools for versions you must keep.

Portable files may use up to 16 MiB, including comments. MCP responses are
capped at 8 MiB, including their protocol envelope. Larger results return an
`isError` tool result with `status: 413`, `filePath`, `editorUrl` and the
revision when available. The error says whether the operation completed or
the request failed. If it completed, the edit remains saved. Read the selected
`filePath` from disk and use `documents` for
the current revision before editing again; do not retry the mutation blindly.
For a large render, inspect the editor or export an image from the browser.

The full engine supports all six node types, layouts, grouping, connector
bindings, locks, variations and notes. Put feedback the user must see on the
canvas with `note`; structured comments are available to tools but have no
canvas comment UI. Rendering and font measurement run on the computer.
Your external agent's model calls still follow that agent's provider and billing.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pablostanley/squig](https://github.com/pablostanley/squig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
